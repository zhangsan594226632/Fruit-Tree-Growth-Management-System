<template>
  <div class="home-page">
    <!-- 顶部横幅 -->
    <div class="banner">
      <div class="banner-content">
        <h1>车辆租赁平台</h1>
        <p>品质车辆 · 价格透明 · 服务保障</p>
      </div>
    </div>

    <!-- 搜索区域 -->
    <div class="search-section">
      <div class="search-container">
        <div class="search-box">
          <!-- 城市选择 -->
          <div class="search-item">
            <div class="search-icon">
              <n-icon :component="LocationOutline" size="20" />
            </div>
            <div class="search-content">
              <label>取还车城市</label>
              <n-select
                v-model:value="search.cityCode"
                :options="cityOptions"
                placeholder="请选择城市"
                size="large"
                filterable
                @update:value="handleCityChange"
              />
            </div>
          </div>

          <!-- 取车时间 -->
          <div class="search-item">
            <div class="search-icon">
              <n-icon :component="CalendarOutline" size="20" />
            </div>
            <div class="search-content">
              <label>取车时间</label>
              <n-date-picker
                v-model:value="search.pickupTime"
                type="datetime"
                placeholder="请选择取车时间"
                size="large"
                style="width: 100%"
                :is-date-disabled="disablePreviousDate"
              />
            </div>
          </div>

          <!-- 还车时间 -->
          <div class="search-item">
            <div class="search-icon">
              <n-icon :component="ReturnDownForwardOutline" size="20" />
            </div>
            <div class="search-content">
              <label>还车时间</label>
              <n-date-picker
                v-model:value="search.returnTime"
                type="datetime"
                placeholder="请选择还车时间"
                size="large"
                style="width: 100%"
                :is-date-disabled="disablePreviousDate"
              />
            </div>
          </div>
        </div>

        <!-- 操作按钮 -->
        <div class="action-buttons">
          <n-button type="primary" size="large" class="search-btn" @click="handleSearch">
            <template #icon>
              <n-icon :component="SearchOutline" />
            </template>
            搜索车辆
          </n-button>
          <n-button size="large" class="reset-btn" @click="handleReset">
            <template #icon>
              <n-icon :component="RefreshOutline" />
            </template>
            重置
          </n-button>
        </div>
      </div>
    </div>

    <!-- 车辆列表 -->
    <div class="car-list-section">
      <div class="section-header">
        <h2>精选车辆</h2>
        <span class="subtitle">为您推荐优质车辆</span>
      </div>
      <n-spin :show="loading">
        <div class="car-grid" v-if="carList.length > 0">
          <div class="car-card" v-for="car in carList" :key="car.id" @click="goToDetail(car.id)">
            <div class="car-image">
              <img :src="car.image || 'https://via.placeholder.com/400x300?text=Car+Image'" :alt="car.model" />
              <div class="price-tag">¥{{ car.dailyPrice }}/天</div>
            </div>
            <div class="car-info">
              <h3>{{ car.brand }} {{ car.model }}</h3>
              <div class="car-tags">
                <n-tag size="small" type="info">{{ car.year }}款</n-tag>
                <n-tag size="small">{{ car.seatCount || 5 }}座</n-tag>
              </div>
              <div class="car-store">
                <n-icon :component="LocationOutline" size="14" />
                <span>{{ car.store?.storeName || '暂无门店信息' }}</span>
              </div>
              <div class="car-footer">
                <span class="deposit">押金 ¥{{ car.deposit }}</span>
                <n-button type="primary" size="small" @click.stop="goToOrder(car.id)">立即租车</n-button>
              </div>
            </div>
          </div>
        </div>
        <n-empty v-else description="暂无车辆数据" />
      </n-spin>

      <!-- 分页 -->
      <div class="pagination" v-if="total > 0">
        <n-pagination
          v-model:page="query.pageNum"
          :page-count="Math.ceil(total / query.pageSize)"
          @update:page="loadData"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import {
  NSelect,
  NDatePicker,
  NButton,
  NTag,
  NSpin,
  NEmpty,
  NPagination,
  NIcon,
  useMessage,
  useDialog
} from 'naive-ui'
import {
  LocationOutline,
  CalendarOutline,
  ReturnDownForwardOutline,
  SearchOutline,
  RefreshOutline
} from '@vicons/ionicons5'
import { getCarList, getStoreList } from '@/api'
import { useUserStore } from '@/store'

const router = useRouter()
const message = useMessage()
const dialog = useDialog()
const userStore = useUserStore()

const loading = ref(false)
const carList = ref([])
const total = ref(0)

// 搜索条件
const search = reactive({
  cityCode: null,
  pickupTime: null,
  returnTime: null
})

// 城市选项
const cityOptions = ref([])

// 查询参数
const query = reactive({
  brand: '',
  cityCode: null,
  pageNum: 1,
  pageSize: 12
})

// 加载城市列表
async function loadCities() {
  try {
    const res = await getStoreList({ pageNum: 1, pageSize: 100 })
    const stores = res.data?.records || []
    // 提取城市列表
    const cityMap = new Map()
    stores.forEach(store => {
      if (!cityMap.has(store.cityCode)) {
        cityMap.set(store.cityCode, {
          label: store.cityName,
          value: store.cityCode
        })
      }
    })
    cityOptions.value = Array.from(cityMap.values()).sort((a, b) => a.label.localeCompare(b.label))
  } catch (error) {
    console.error('加载城市失败:', error)
  }
}

// 城市变化处理
function handleCityChange(value) {
  query.cityCode = value
  query.pageNum = 1
  loadData()
}

// 禁用之前的日期
function disablePreviousDate(timestamp) {
  return timestamp < Date.now()
}

// 加载车辆列表
async function loadData() {
  loading.value = true
  try {
    const params = {
      pageNum: query.pageNum,
      pageSize: query.pageSize
    }
    if (query.cityCode) params.cityCode = query.cityCode
    if (query.brand) params.brand = query.brand

    const res = await getCarList(params)
    carList.value = (res.data.records || []).map(car => ({
      ...car,
      image: car.images || null
    }))
    total.value = res.data.total || 0
  } catch (error) {
    message.error('加载失败')
  } finally {
    loading.value = false
  }
}

// 搜索
function handleSearch() {
  if (!search.cityCode) {
    message.warning('请选择取还车城市')
    return
  }
  if (!search.pickupTime) {
    message.warning('请选择取车时间')
    return
  }
  if (!search.returnTime) {
    message.warning('请选择还车时间')
    return
  }

  // 检查还车时间是否晚于取车时间
  if (new Date(search.returnTime) <= new Date(search.pickupTime)) {
    message.warning('还车时间必须晚于取车时间')
    return
  }

  query.cityCode = search.cityCode
  query.pageNum = 1

  // 存储搜索条件到 sessionStorage
  sessionStorage.setItem('searchConditions', JSON.stringify({
    cityCode: search.cityCode,
    pickupTime: search.pickupTime,
    returnTime: search.returnTime
  }))

  loadData()
  message.success('搜索成功')
}

// 重置
function handleReset() {
  search.cityCode = null
  search.pickupTime = null
  search.returnTime = null
  query.cityCode = null
  query.pageNum = 1
  sessionStorage.removeItem('searchConditions')
  loadData()
  message.info('已重置搜索条件')
}

// 跳转到详情
function goToDetail(id) {
  router.push(`/car/${id}`)
}

// 跳转到订单
function goToOrder(id) {
  if (!userStore.token) {
    message.warning('请先登录')
    router.push('/login')
    return
  }

  // 检查是否有搜索条件
  if (!search.cityCode || !search.pickupTime || !search.returnTime) {
    dialog.warning({
      title: '提示',
      content: '请先选择取还车城市和时间',
      positiveText: '去选择'
    })
    return
  }

  router.push({
    path: '/order/create',
    query: {
      carId: id,
      cityCode: search.cityCode,
      pickupTime: search.pickupTime,
      returnTime: search.returnTime
    }
  })
}

onMounted(() => {
  loadCities()
  loadData()
})
</script>

<style scoped>
.home-page {
  min-height: 100vh;
}

.banner {
  background: linear-gradient(135deg, #166534 0%, #1f883d 50%, #22c55e 100%);
  color: white;
  padding: 100px 24px 80px;
  text-align: center;
}

.banner-content h1 {
  font-size: 48px;
  margin-bottom: 16px;
  font-weight: 600;
  white-space: nowrap;
}

.banner-content p {
  font-size: 20px;
  opacity: 0.95;
}

.search-section {
  margin: -60px auto 40px;
  padding: 0 24px;
  position: relative;
  z-index: 10;
}

.search-container {
  max-width: 1000px;
  margin: 0 auto;
  background: white;
  border-radius: 16px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
  padding: 32px;
}

.search-box {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
  margin-bottom: 24px;
}

.search-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.search-icon {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #166534, #1f883d);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  flex-shrink: 0;
  margin-top: 20px;
}

.search-content {
  flex: 1;
}

.search-content label {
  display: block;
  font-size: 13px;
  color: #666;
  margin-bottom: 6px;
  font-weight: 500;
}

.action-buttons {
  display: flex;
  gap: 16px;
  padding-top: 8px;
  border-top: 1px solid #f0f0f0;
}

.search-btn {
  flex: 2;
  height: 48px;
  font-size: 16px;
  border-radius: 10px;
  background: linear-gradient(135deg, #166534, #1f883d);
  border: none;
}

.search-btn:hover {
  background: linear-gradient(135deg, #14532d, #166534);
}

.reset-btn {
  flex: 1;
  height: 48px;
  font-size: 16px;
  border-radius: 10px;
}

.car-list-section {
  max-width: 1200px;
  margin: 0 auto 40px;
  padding: 0 24px;
}

.section-header {
  margin-bottom: 24px;
}

.section-header h2 {
  font-size: 28px;
  color: #1f2937;
  margin-bottom: 4px;
}

.subtitle {
  font-size: 14px;
  color: #9ca3af;
}

.car-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
  margin-bottom: 32px;
}

.car-card {
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid #f0f0f0;
}

.car-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.12);
}

.car-image {
  position: relative;
  height: 200px;
  background: #f9fafb;
}

.car-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.price-tag {
  position: absolute;
  top: 16px;
  right: 16px;
  background: linear-gradient(135deg, #166534, #1f883d);
  color: white;
  padding: 6px 14px;
  border-radius: 8px;
  font-weight: 600;
  font-size: 14px;
}

.car-info {
  padding: 18px;
}

.car-info h3 {
  font-size: 18px;
  margin-bottom: 10px;
  color: #1f2937;
}

.car-tags {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.car-store {
  display: flex;
  align-items: center;
  gap: 4px;
  color: #6b7280;
  font-size: 13px;
  margin-bottom: 14px;
}

.car-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 12px;
  border-top: 1px solid #f3f4f6;
}

.deposit {
  color: #6b7280;
  font-size: 14px;
}

.pagination {
  display: flex;
  justify-content: center;
  padding: 32px 0;
}
</style>
