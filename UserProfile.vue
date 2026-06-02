<template>
  <div class="user-profile-page">
    <div class="container">
      <h1>个人中心</h1>

      <n-layout has-sider class="user-layout">
        <!-- 左侧菜单 -->
        <n-layout-sider bordered width="220" class="side-menu">
          <!-- 用户信息卡片 -->
          <div class="user-card" v-if="userInfo">
            <div class="avatar">
              <n-icon :component="PersonOutline" size="64" />
            </div>
            <h3>{{ userInfo.nickname || '用户' }}</h3>
            <p>{{ userInfo.phone }}</p>
          </div>

          <!-- 侧边菜单 -->
          <n-menu
            :value="activeMenuKey"
            :options="menuOptions"
            @update:value="handleMenuSelect"
          />
        </n-layout-sider>

        <!-- 右侧内容区 -->
        <n-layout-content class="content-area">
          <!-- 用户信息详情 -->
          <n-card title="基本信息" v-if="userInfo">
            <n-descriptions :column="2" label-style="width: 100px">
              <n-descriptions-item label="手机号">{{ userInfo.phone }}</n-descriptions-item>
              <n-descriptions-item label="昵称">{{ userInfo.nickname || '-' }}</n-descriptions-item>
              <n-descriptions-item label="真实姓名">{{ userInfo.realName || '-' }}</n-descriptions-item>
              <n-descriptions-item label="状态">
                <n-tag :type="userInfo.status === 1 ? 'success' : 'error'">
                  {{ userInfo.status === 1 ? '正常' : '封禁' }}
                </n-tag>
              </n-descriptions-item>
              <n-descriptions-item label="冻结押金">¥{{ userInfo.depositLocked || 0 }}</n-descriptions-item>
            </n-descriptions>
          </n-card>

          <!-- 订单统计 -->
          <n-card title="订单统计" style="margin-top: 16px">
            <n-grid :cols="4" :x-gap="12">
              <n-grid-item>
                <div class="stat-item">
                  <div class="value">{{ orderStats.all || 0 }}</div>
                  <div class="label">全部订单</div>
                </div>
              </n-grid-item>
              <n-grid-item>
                <div class="stat-item">
                  <div class="value">{{ orderStats.pending || 0 }}</div>
                  <div class="label">待支付</div>
                </div>
              </n-grid-item>
              <n-grid-item>
                <div class="stat-item">
                  <div class="value">{{ orderStats.renting || 0 }}</div>
                  <div class="label">租赁中</div>
                </div>
              </n-grid-item>
              <n-grid-item>
                <div class="stat-item">
                  <div class="value">{{ orderStats.completed || 0 }}</div>
                  <div class="label">已完成</div>
                </div>
              </n-grid-item>
            </n-grid>
          </n-card>

          <n-card title="最近订单" style="margin-top: 16px">
            <div class="recent-orders" v-if="recentOrders.length > 0">
              <div class="order-item" v-for="order in recentOrders" :key="order.id" @click="$router.push(`/order/detail/${order.orderNo}`)">
                <div class="order-info">
                  <span class="car-name">{{ order.car?.brand }} {{ order.car?.model }}</span>
                  <span class="order-time">{{ formatDate(order.createTime) }}</span>
                </div>
                <div class="order-right">
                  <span class="amount">¥{{ order.actualAmount }}</span>
                  <n-tag :type="getStatusType(order.status)" size="small">{{ getStatusText(order.status) }}</n-tag>
                </div>
              </div>
            </div>
            <n-empty v-else description="暂无订单" size="small" />
            <template #footer>
              <n-button text block @click="$router.push('/order/list')">查看全部订单 →</n-button>
            </template>
          </n-card>
        </n-layout-content>
      </n-layout>
    </div>

    <!-- 修改密码弹窗 -->
    <n-modal v-model:show="showPasswordModal" preset="dialog" title="修改密码" positive-text="确认" negative-text="取消" @positive-click="handleUpdatePassword">
      <n-form ref="pwdFormRef" :model="pwdForm" :rules="pwdRules" :label-width="80">
        <n-form-item label="原密码" path="oldPassword">
          <n-input v-model:value="pwdForm.oldPassword" type="password" show-password-on="click" placeholder="请输入原密码" />
        </n-form-item>
        <n-form-item label="新密码" path="newPassword">
          <n-input v-model:value="pwdForm.newPassword" type="password" show-password-on="click" placeholder="请输入新密码" />
        </n-form-item>
        <n-form-item label="确认密码" path="confirmPassword">
          <n-input v-model:value="pwdForm.confirmPassword" type="password" show-password-on="click" placeholder="请再次输入新密码" />
        </n-form-item>
      </n-form>
    </n-modal>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, h } from 'vue'
import { useRouter } from 'vue-router'
import { NCard, NGrid, NGridItem, NDescriptions, NDescriptionsItem, NTag, NSpace, NButton, NSpin, NEmpty, NModal, NForm, NFormItem, NInput, NLayout, NLayoutSider, NLayoutContent, NMenu, NIcon, useMessage } from 'naive-ui'
import { PersonOutline, DocumentTextOutline, KeyOutline, LogOutOutline } from '@vicons/ionicons5'
import { getUserInfo, getMyOrders } from '@/api'
import { useUserStore } from '@/store'

const router = useRouter()
const message = useMessage()
const userStore = useUserStore()

// 当前激活的菜单
const activeMenuKey = ref('overview')

// 菜单选项
const menuOptions = [
  {
    label: '概览',
    key: 'overview',
    icon: () => h(NIcon, null, { default: () => h(PersonOutline) })
  },
  {
    label: '我的订单',
    key: 'orders',
    icon: () => h(NIcon, null, { default: () => h(DocumentTextOutline) })
  },
  {
    label: '修改密码',
    key: 'password',
    icon: () => h(NIcon, null, { default: () => h(KeyOutline) })
  },
  {
    label: '退出登录',
    key: 'logout',
    icon: () => h(NIcon, null, { default: () => h(LogOutOutline) })
  }
]

// 菜单选择处理
function handleMenuSelect(key) {
  activeMenuKey.value = key
  if (key === 'orders') {
    router.push('/order/list')
  } else if (key === 'password') {
    showPasswordModal.value = true
    activeMenuKey.value = 'overview'
  } else if (key === 'logout') {
    handleLogout()
  }
}

const loading = ref(false)
const userInfo = ref(null)
const recentOrders = ref([])
const orderStats = ref({ all: 0, pending: 0, renting: 0, completed: 0 })

const showPasswordModal = ref(false)
const pwdFormRef = ref(null)
const pwdForm = reactive({
  oldPassword: '',
  newPassword: '',
  confirmPassword: ''
})

const validatePassword = (rule, value) => {
  if (value === '') {
    return new Error('请再次输入新密码')
  } else if (value !== pwdForm.newPassword) {
    return new Error('两次输入的密码不一致')
  }
  return true
}

const pwdRules = {
  oldPassword: { required: true, message: '请输入原密码', trigger: 'blur' },
  newPassword: { required: true, min: 6, max: 20, message: '密码长度为6-20位', trigger: 'blur' },
  confirmPassword: { required: true, validator: validatePassword, trigger: 'blur' }
}

const statusMap = {
  1: { type: 'warning', text: '待支付' },
  2: { type: 'info', text: '已支付' },
  3: { type: 'success', text: '租赁中' },
  4: { type: 'default', text: '已完成' },
  5: { type: 'default', text: '已取消' }
}

function getStatusType(status) {
  return statusMap[status]?.type || 'default'
}

function getStatusText(status) {
  return statusMap[status]?.text || '未知'
}

function formatDate(time) {
  return time ? new Date(time).toLocaleDateString() : '-'
}

async function loadUserInfo() {
  loading.value = true
  try {
    const res = await getUserInfo()
    userInfo.value = res.data
    userStore.setUserInfo(res.data)
  } catch (error) {
    message.error('加载用户信息失败')
  } finally {
    loading.value = false
  }
}

async function loadRecentOrders() {
  try {
    const res = await getMyOrders({ pageNum: 1, pageSize: 5 })
    recentOrders.value = res.data.records || []

    // 统计各状态订单数量
    const allRes = await getMyOrders({ pageNum: 1, pageSize: 100 })
    const orders = allRes.data.records || []
    orderStats.value = {
      all: orders.length,
      pending: orders.filter(o => o.status === 1).length,
      renting: orders.filter(o => o.status === 3).length,
      completed: orders.filter(o => o.status === 4).length
    }
  } catch (error) {
    console.error('加载订单失败')
  }
}

function handleUpdatePassword() {
  message.success('密码修改成功（模拟）')
  showPasswordModal.value = false
}

function handleLogout() {
  userStore.logout()
  router.push('/login')
}

onMounted(() => {
  loadUserInfo()
  loadRecentOrders()
})
</script>

<style scoped>
.user-profile-page {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
  min-height: calc(100vh - 150px);
}

.container h1 {
  font-size: 24px;
  margin-bottom: 24px;
}

.user-layout {
  background: #fff;
  border-radius: 8px;
  overflow: hidden;
}

.side-menu {
  background: #fff;
}

.user-card {
  padding: 24px 16px;
  text-align: center;
  border-bottom: 1px solid #f0f0f0;
}

.user-card .avatar {
  margin-bottom: 12px;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #1f883d;
}

.user-card h3 {
  font-size: 16px;
  margin-bottom: 4px;
}

.user-card p {
  font-size: 13px;
  color: #999;
}

.content-area {
  padding: 24px;
  background: #f9f9f9;
}

.stat-item {
  text-align: center;
  padding: 16px;
  background: #f9f9f9;
  border-radius: 8px;
}

.stat-item .value {
  font-size: 24px;
  font-weight: 600;
  color: #1f883d;
}

.stat-item .label {
  font-size: 14px;
  color: #999;
  margin-top: 4px;
}

.recent-orders {
  margin-top: 8px;
}

.order-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
}

.order-item:last-child {
  border-bottom: none;
}

.order-item:hover {
  background: #f9f9f9;
}

.order-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.car-name {
  font-weight: 600;
}

.order-time {
  font-size: 12px;
  color: #999;
}

.order-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.order-right .amount {
  font-weight: 600;
}
</style>
