<template>
  <div class="order-detail-page">
    <div class="container" v-if="orderData">
      <n-breadcrumb style="margin-bottom: 16px">
        <n-breadcrumb-item @click="router.push('/order/list')">我的订单</n-breadcrumb-item>
        <n-breadcrumb-item>订单详情</n-breadcrumb-item>
      </n-breadcrumb>

      <n-card>
        <template #header>
          <div class="detail-header">
            <h2>订单详情</h2>
            <n-tag :type="getStatusType(orderData.status)">{{ getStatusText(orderData.status) }}</n-tag>
          </div>
        </template>

        <n-descriptions :column="2" bordered label-style="width: 120px; background: #fafafa">
          <n-descriptions-item label="订单号">{{ orderData.orderNo }}</n-descriptions-item>
          <n-descriptions-item label="订单状态">
            <n-tag :type="getStatusType(orderData.status)">{{ getStatusText(orderData.status) }}</n-tag>
          </n-descriptions-item>
          <n-descriptions-item label="车辆">{{ orderData.car?.brand }} {{ orderData.car?.model }}</n-descriptions-item>
          <n-descriptions-item label="车牌号">{{ orderData.car?.licensePlate }}</n-descriptions-item>
          <n-descriptions-item label="取车门店">{{ orderData.store?.storeName }}</n-descriptions-item>
          <n-descriptions-item label="还车门店">{{ orderData.returnStore?.storeName || orderData.store?.storeName }}</n-descriptions-item>
          <n-descriptions-item label="租赁天数">{{ orderData.rentDays }}天</n-descriptions-item>
          <n-descriptions-item label="日租金">¥{{ orderData.dailyPrice }}</n-descriptions-item>
          <n-descriptions-item label="租金总额">¥{{ orderData.totalAmount }}</n-descriptions-item>
          <n-descriptions-item label="押金">¥{{ orderData.deposit }}</n-descriptions-item>
          <n-descriptions-item label="实付金额">¥{{ orderData.actualAmount }}</n-descriptions-item>
          <n-descriptions-item label="应付总额">¥{{ (orderData.actualAmount + orderData.deposit).toFixed(2) }}</n-descriptions-item>
          <n-descriptions-item label="开始时间">{{ formatTime(orderData.startTime) }}</n-descriptions-item>
          <n-descriptions-item label="结束时间">{{ formatTime(orderData.endTime) }}</n-descriptions-item>
          <n-descriptions-item label="创建时间">{{ formatTime(orderData.createTime) }}</n-descriptions-item>
          <n-descriptions-item label="支付时间">{{ orderData.paymentTime ? formatTime(orderData.paymentTime) : '-' }}</n-descriptions-item>
        </n-descriptions>

        <n-divider />

        <n-space>
          <n-button @click="router.back()">返回</n-button>
          <n-button v-if="orderData.status === 1" type="primary" @click="handlePay">去支付</n-button>
          <n-button v-if="orderData.status === 1" @click="handleCancel">取消订单</n-button>
        </n-space>
      </n-card>
    </div>

    <n-spin v-else :show="loading" style="min-height: 400px; display: flex; align-items: center; justify-content: center" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { NCard, NBreadcrumb, NBreadcrumbItem, NDescriptions, NDescriptionsItem, NTag, NDivider, NSpace, NButton, NSpin, useMessage, useDialog } from 'naive-ui'
import { getOrderDetail, cancelOrder, createPayment, mockPaymentCallback } from '@/api'

const router = useRouter()
const route = useRoute()
const message = useMessage()
const dialog = useDialog()

const loading = ref(false)
const orderData = ref(null)

const statusMap = {
  1: { type: 'warning', text: '待支付' },
  2: { type: 'info', text: '已支付' },
  3: { type: 'success', text: '租赁中' },
  4: { type: 'default', text: '已完成' },
  5: { type: 'default', text: '已取消' },
  6: { type: 'warning', text: '退款中' },
  7: { type: 'error', text: '已退款' }
}

function getStatusType(status) {
  return statusMap[status]?.type || 'default'
}

function getStatusText(status) {
  return statusMap[status]?.text || '未知'
}

function formatTime(time) {
  return time ? new Date(time).toLocaleString() : '-'
}

async function loadData() {
  loading.value = true
  try {
    const res = await getOrderDetail(route.params.orderNo)
    orderData.value = res.data
  } catch (error) {
    message.error('加载失败')
  } finally {
    loading.value = false
  }
}

function handlePay() {
  dialog.info({
    title: '确认支付',
    content: `订单金额：¥${orderData.value.actualAmount + orderData.value.deposit}`,
    positiveText: '确认支付',
    negativeText: '取消',
    onPositiveClick: async () => {
      try {
        // 先创建支付
        const res = await createPayment(orderData.value.orderNo)
        // 模拟支付回调，从 res.data 中获取 paymentNo
        await mockPaymentCallback(res.data)
        message.success('支付成功')
        loadData()
      } catch (error) {
        message.error('支付失败')
      }
    }
  })
}

function handleCancel() {
  dialog.warning({
    title: '取消订单',
    content: '确定要取消该订单吗？',
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: async () => {
      try {
        await cancelOrder(orderData.value.orderNo, '用户主动取消')
        message.success('订单已取消')
        loadData()
      } catch (error) {
        message.error('取消失败')
      }
    }
  })
}

onMounted(() => {
  loadData()
})
</script>

<style scoped>
.order-detail-page {
  max-width: 900px;
  margin: 0 auto;
  padding: 24px;
  min-height: calc(100vh - 150px);
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.detail-header h2 {
  margin: 0;
}
</style>
