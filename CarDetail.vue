<template>
  <div class="car-detail-page">
    <div class="container" v-if="carData">
      <!-- 面包屑 -->
      <n-breadcrumb style="margin-bottom: 16px">
        <n-breadcrumb-item>首页</n-breadcrumb-item>
        <n-breadcrumb-item>车辆详情</n-breadcrumb-item>
      </n-breadcrumb>

      <n-grid :cols="2" :x-gap="24">
        <!-- 左侧图片 -->
        <n-grid-item>
          <div class="car-gallery">
            <img :src="carData.image || 'https://via.placeholder.com/600x400?text=' + carData.model" :alt="carData.model" />
          </div>
        </n-grid-item>

        <!-- 右侧信息 -->
        <n-grid-item>
          <n-card>
            <div class="car-header">
              <h1>{{ carData.brand }} {{ carData.model }}</h1>
              <div class="price">
                <span class="daily">¥{{ carData.dailyPrice }}</span>
                <span class="unit">/天</span>
              </div>
            </div>

            <n-divider />

            <n-descriptions :column="2" label-style="color: #999; width: 80px">
              <n-descriptions-item label="年款">{{ carData.year }}款</n-descriptions-item>
              <n-descriptions-item label="颜色">{{ carData.color }}</n-descriptions-item>
              <n-descriptions-item label="变速箱">{{ carData.transmission }}</n-descriptions-item>
              <n-descriptions-item label="座位数">{{ carData.seatCount }}座</n-descriptions-item>
              <n-descriptions-item label="排量">{{ carData.displacement }}</n-descriptions-item>
              <n-descriptions-item label="燃料类型">{{ carData.fuelType }}</n-descriptions-item>
              <n-descriptions-item label="里程">{{ carData.mileage }}公里</n-descriptions-item>
              <n-descriptions-item label="车牌号">{{ carData.licensePlate }}</n-descriptions-item>
            </n-descriptions>

            <n-divider />

            <div class="deposit-info">
              <h3>押金说明</h3>
              <p>押金 ¥{{ carData.deposit }}（还车后无问题退还）</p>
            </div>

            <n-divider />

            <n-space vertical size="large">
              <n-button type="primary" size="large" block @click="goToOrder">
                立即租车
              </n-button>
            </n-space>
          </n-card>
        </n-grid-item>
      </n-grid>

      <!-- 车辆描述 -->
      <n-card title="车辆描述" style="margin-top: 24px">
        <p>{{ carData.description || '暂无描述' }}</p>
      </n-card>
    </div>

    <n-spin v-else :show="loading" style="min-height: 400px; display: flex; align-items: center; justify-content: center" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { NCard, NGrid, NGridItem, NBreadcrumb, NBreadcrumbItem, NDescriptions, NDescriptionsItem, NDivider, NSpace, NButton, NSpin, useMessage } from 'naive-ui'
import { getCarDetail } from '@/api'
import { useUserStore } from '@/store'

const route = useRoute()
const router = useRouter()
const message = useMessage()
const userStore = useUserStore()

const loading = ref(false)
const carData = ref(null)

async function loadData() {
  loading.value = true
  try {
    const res = await getCarDetail(route.params.id)
    carData.value = {
      ...res.data,
      image: res.data.images || null
    }
  } catch (error) {
    message.error('加载失败')
  } finally {
    loading.value = false
  }
}

function goToOrder() {
  if (!userStore.token) {
    message.warning('请先登录')
    router.push('/login')
    return
  }
  router.push({ path: '/order/create', query: { carId: route.params.id } })
}

onMounted(() => {
  loadData()
})
</script>

<style scoped>
.car-detail-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
}

.car-gallery img {
  width: 100%;
  border-radius: 8px;
}

.car-header h1 {
  font-size: 28px;
  margin-bottom: 16px;
  color: #333;
}

.price {
  display: flex;
  align-items: baseline;
  gap: 4px;
}

.price .daily {
  font-size: 32px;
  font-weight: 600;
  color: #1f883d;
}

.price .unit {
  color: #999;
}

.deposit-info h3 {
  font-size: 16px;
  margin-bottom: 8px;
}

.deposit-info p {
  color: #999;
}
</style>
