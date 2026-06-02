<template>
  <div class="create-order-page">
    <div class="container">
      <n-breadcrumb style="margin-bottom: 16px">
        <n-breadcrumb-item>首页</n-breadcrumb-item>
        <n-breadcrumb-item>创建订单</n-breadcrumb-item>
      </n-breadcrumb>

      <n-grid :cols="3" :x-gap="24" v-if="carData">
        <!-- 左侧车辆信息 -->
        <n-grid-item :span="2">
          <n-card title="订单信息">
            <n-form ref="formRef" :model="form" :rules="rules" :label-width="100">
              <n-form-item label="车辆">
                <div class="car-info">
                  <strong>{{ carData.brand }} {{ carData.model }}</strong>
                  <span style="color: #1f883d; font-weight: 600">¥{{ carData.dailyPrice }}/天</span>
                </div>
              </n-form-item>

              <n-form-item label="取车时间" path="startTime">
                <n-date-picker
                  v-model:value="form.startTime"
                  type="datetime"
                  placeholder="请选择取车时间"
                  style="width: 100%"
                  :is-date-disabled="disablePreviousDate"
                  @update:value="handleTimeChange"
                />
              </n-form-item>

              <n-form-item label="还车时间" path="endTime">
                <n-date-picker
                  v-model:value="form.endTime"
                  type="datetime"
                  placeholder="请选择还车时间"
                  style="width: 100%"
                  :is-date-disabled="disablePreviousDate"
                  @update:value="handleTimeChange"
                />
              </n-form-item>

              <n-form-item label="取车门店" path="storeId">
                <n-select
                  v-model:value="form.storeId"
                  :options="storeOptions"
                  placeholder="请选择取车门店"
                />
              </n-form-item>

              <n-divider />

              <div class="price-summary">
                <div class="price-item">
                  <span>租赁天数</span>
                  <strong>{{ rentDays }}天</strong>
                </div>
                <div class="price-item">
                  <span>日租金</span>
                  <span>¥{{ carData.dailyPrice }}</span>
                </div>
                <div class="price-item">
                  <span>租金总额</span>
                  <strong>¥{{ totalAmount }}</strong>
                </div>
                <div class="price-item">
                  <span>押金</span>
                  <span>¥{{ carData.deposit }}</span>
                </div>
                <n-divider style="margin: 12px 0" />
                <div class="price-item total">
                  <span>应付总额</span>
                  <strong style="color: #1f883d; font-size: 20px">¥{{ finalAmount }}</strong>
                </div>
              </div>

              <n-space vertical size="large" style="margin-top: 24px; width: 100%">
                <n-button type="primary" size="large" block :loading="loading" @click="handleSubmit">
                  提交订单
                </n-button>
              </n-space>
            </n-form>
          </n-card>
        </n-grid-item>

        <!-- 右侧说明 -->
        <n-grid-item>
          <n-card title="租车须知">
            <div class="notice-list">
              <p>1. 请确保您的驾照在有效期内</p>
              <p>2. 取车时需要出示身份证和驾照原件</p>
              <p>3. 押金将在还车检查后退还</p>
              <p>4. 如有车损，需承担相应赔偿责任</p>
              <p>5. 请按约定时间还车，逾期将收取费用</p>
            </div>
          </n-card>
        </n-grid-item>
      </n-grid>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { NCard, NGrid, NGridItem, NBreadcrumb, NBreadcrumbItem, NForm, NFormItem, NDatePicker, NSelect, NDivider, NSpace, NButton, useMessage } from 'naive-ui'
import { getCarDetail, createOrder, calculatePrice, getStoreList } from '@/api'
import { useUserStore } from '@/store'

const route = useRoute()
const router = useRouter()
const message = useMessage()
const userStore = useUserStore()

const formRef = ref(null)
const loading = ref(false)
const carData = ref(null)
const storeOptions = ref([])

const form = reactive({
  carId: Number(route.query.carId),
  storeId: null,
  startTime: null,
  endTime: null,
  returnStoreId: null
})

const rules = {
  startTime: { required: true, type: 'number', message: '请选择取车时间', trigger: 'change' },
  endTime: { required: true, type: 'number', message: '请选择还车时间', trigger: 'change' },
  storeId: { required: true, type: 'number', message: '请选择取车门店', trigger: 'change' }
}

// 计算租赁天数
const rentDays = computed(() => {
  if (!form.startTime || !form.endTime) return 0
  const start = new Date(form.startTime)
  const end = new Date(form.endTime)
  const days = Math.ceil((end - start) / (1000 * 60 * 60 * 24))
  return Math.max(0, days)
})

// 计算租金总额
const totalAmount = computed(() => {
  if (!carData.value || rentDays.value <= 0) return '0.00'
  return (carData.value.dailyPrice * rentDays.value).toFixed(2)
})

// 应付总额
const finalAmount = computed(() => {
  if (!carData.value) return '0.00'
  return (parseFloat(totalAmount.value) + carData.value.deposit).toFixed(2)
})

async function loadCarData() {
  try {
    const res = await getCarDetail(form.carId)
    carData.value = res.data
  } catch (error) {
    message.error('加载车辆信息失败')
  }
}

async function loadStoreOptions() {
  try {
    const res = await getStoreList({ pageNum: 1, pageSize: 100 })
    const records = res.data?.records || []
    if (records.length === 0) {
      message.warning('暂无可用门店，请先在管理后台添加门店')
      return
    }
    storeOptions.value = records.map(item => ({
      label: `${item.cityName}-${item.storeName}`,
      value: item.id
    }))
  } catch (error) {
    console.error('加载门店列表失败:', error)
    message.error('加载门店列表失败，请稍后重试')
  }
}

function handleTimeChange() {
  // 价格会在 computed 中自动更新
}

function disablePreviousDate(timestamp) {
  return timestamp < Date.now() - 86400000
}

async function handleSubmit() {
  try {
    await formRef.value?.validate()

    if (rentDays.value <= 0) {
      message.warning('还车时间必须晚于取车时间')
      return
    }

    loading.value = true

    const orderData = {
      carId: form.carId,
      storeId: form.storeId,
      startTime: new Date(form.startTime).toISOString(),
      endTime: new Date(form.endTime).toISOString()
    }

    const res = await createOrder(orderData)

    message.success('订单创建成功')
    router.push(`/order/detail/${res.data.orderNo}`)
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  if (!form.carId) {
    message.error('缺少车辆信息')
    router.back()
    return
  }
  loadCarData()
  loadStoreOptions()
})
</script>

<style scoped>
.create-order-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
  min-height: calc(100vh - 150px);
}

.car-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 12px;
  background: #f5f5f5;
  border-radius: 4px;
}

.price-summary {
  background: #f9f9f9;
  padding: 16px;
  border-radius: 4px;
}

.price-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
}

.price-item.total {
  font-size: 16px;
}

.notice-list p {
  margin-bottom: 12px;
  color: #666;
  line-height: 1.6;
}
</style>
