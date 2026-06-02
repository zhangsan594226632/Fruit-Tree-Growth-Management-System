<template>
  <div class="order-list-page">
    <div class="container">
      <h1>我的订单</h1>

      <n-tabs v-model:value="activeTab" type="segment" @update:value="handleTabChange">
        <n-tab-pane name="all" tab="全部" />
        <n-tab-pane name="1" tab="待支付" />
        <n-tab-pane name="2" tab="已支付" />
        <n-tab-pane name="3" tab="租赁中" />
        <n-tab-pane name="4" tab="已完成" />
      </n-tabs>

      <div class="order-list" v-if="orderList.length > 0">
        <div class="order-card" v-for="order in orderList" :key="order.id">
          <div class="order-header">
            <span class="order-no">订单号：{{ order.orderNo }}</span>
            <n-tag :type="getStatusType(order.status)">{{ getStatusText(order.status) }}</n-tag>
          </div>

          <div class="order-content" @click="goToDetail(order.orderNo)">
            <div class="car-info">
              <div class="car-image">
                <img :src="order.car?.image || 'https://via.placeholder.com/120x80?text=Car'" :alt="order.car?.model" />
              </div>
              <div class="car-detail">
                <h3>{{ order.car?.brand }} {{ order.car?.model }}</h3>
                <p class="time">{{ formatTime(order.startTime) }} - {{ formatTime(order.endTime) }}</p>
                <p class="days">共{{ order.rentDays }}天</p>
              </div>
            </div>

            <div class="order-info">
              <div class="amount">
                <span class="label">订单金额</span>
                <span class="value">¥{{ order.actualAmount }}</span>
              </div>
              <div class="amount">
                <span class="label">押金</span>
                <span class="value">¥{{ order.deposit }}</span>
              </div>
            </div>
          </div>

          <div class="order-footer">
            <n-space>
              <n-button size="small" @click="goToDetail(order.orderNo)">查看详情</n-button>
              <n-button v-if="order.status === 1" type="primary" size="small" @click="handlePay(order)">
                去支付
              </n-button>
              <n-button v-if="order.status === 1" size="small" @click="handleCancel(order)">
                取消订单
              </n-button>
            </n-space>
          </div>
        </div>
      </div>

      <n-empty v-else description="暂无订单数据" style="margin-top: 60px" />

      <!-- 分页 -->
      <div class="pagination" v-if="total > 0">
        <n-pagination
          v-model:page="pageNum"
          :page-count="Math.ceil(total / pageSize)"
          @update:page="loadData"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { NTabs, NTabPane, NTag, NButton, NSpace, NEmpty, NPagination, useMessage, useDialog } from 'naive-ui'
import { getMyOrders, cancelOrder, createPayment, mockPaymentCallback } from '@/api'

const router = useRouter()
const message = useMessage()
const dialog = useDialog()

const activeTab = ref('all')
const orderList = ref([])
const total = ref(0)
const pageNum = ref(1)
const pageSize = ref(10)

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
  return time ? new Date(time).toLocaleDateString() : '-'
}

async function loadData() {
  try {
    const params = {
      pageNum: pageNum.value,
      pageSize: pageSize.value
    }
    if (activeTab.value !== 'all') {
      params.status = Number(activeTab.value)
    }

    const res = await getMyOrders(params)
    // 解析车辆图片
    orderList.value = (res.data.records || []).map(order => ({
      ...order,
      car: order.car ? {
        ...order.car,
        image: order.car.images || null
      } : null
    }))
    total.value = res.data.total || 0
  } catch (error) {
    message.error('加载失败')
  }
}

function handleTabChange() {
  pageNum.value = 1
  loadData()
}

function goToDetail(orderNo) {
  router.push(`/order/detail/${orderNo}`)
}

function handlePay(order) {
  // 模拟支付流程
  dialog.info({
    title: '确认支付',
    content: `订单金额：¥${order.actualAmount + order.deposit}`,
    positiveText: '确认支付',
    negativeText: '取消',
    onPositiveClick: async () => {
      try {
        // 先创建支付
        const res = await createPayment(order.orderNo)
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

function handleCancel(order) {
  dialog.warning({
    title: '取消订单',
    content: '确定要取消该订单吗？',
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: async () => {
      try {
        await cancelOrder(order.orderNo, '用户主动取消')
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
.order-list-page {
  max-width: 900px;
  margin: 0 auto;
  padding: 24px;
  min-height: calc(100vh - 150px);
}

.container h1 {
  font-size: 24px;
  margin-bottom: 24px;
}

.order-list {
  margin-top: 24px;
}

.order-card {
  background: #fff;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 12px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 12px;
}

.order-no {
  color: #999;
  font-size: 14px;
}

.order-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
}

.car-info {
  display: flex;
  gap: 12px;
  align-items: center;
}

.car-image img {
  width: 100px;
  height: 70px;
  object-fit: cover;
  border-radius: 4px;
}

.car-detail h3 {
  font-size: 16px;
  margin-bottom: 4px;
}

.car-detail .time {
  color: #999;
  font-size: 14px;
}

.car-detail .days {
  color: #1f883d;
  font-size: 14px;
}

.order-info {
  text-align: right;
}

.order-info .amount {
  margin-bottom: 8px;
}

.order-info .label {
  color: #999;
  margin-right: 8px;
}

.order-info .value {
  font-weight: 600;
  font-size: 16px;
}

.order-footer {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid #f0f0f0;
}

.pagination {
  display: flex;
  justify-content: center;
  padding: 24px 0;
}
</style>
