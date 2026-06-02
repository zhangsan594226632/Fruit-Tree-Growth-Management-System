<template>
  <div class="login-container">
    <div class="login-left">
      <div class="login-left-content">
        <h1>车辆租赁平台</h1>
        <p>专业的车辆租赁服务</p>
        <div class="features">
          <div class="feature">
            <span class="icon">🚗</span>
            <span>丰富车源</span>
          </div>
          <div class="feature">
            <span class="icon">💰</span>
            <span>价格透明</span>
          </div>
          <div class="feature">
            <span class="icon">🔒</span>
            <span>安全保障</span>
          </div>
        </div>
      </div>
    </div>
    <div class="login-right">
      <div class="login-form">
        <div class="login-title">
          <h2>用户登录</h2>
          <p>欢迎回来</p>
        </div>
        <n-form ref="formRef" :model="form" :rules="rules" size="large">
          <n-form-item path="account">
            <n-input v-model:value="form.account" placeholder="请输入手机号" />
          </n-form-item>
          <n-form-item path="password">
            <n-input v-model:value="form.password" type="password" show-password-on="click" placeholder="请输入密码" />
          </n-form-item>
          <n-form-item>
            <n-button type="primary" block :loading="loading" @click="handleLogin">
              登录
            </n-button>
          </n-form-item>
        </n-form>
        <div class="login-footer">
          <p>还没有账号？ <router-link to="/register">立即注册</router-link></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { NForm, NFormItem, NInput, NButton, useMessage } from 'naive-ui'
import { login } from '@/api'
import { useUserStore } from '@/store'

const router = useRouter()
const message = useMessage()
const userStore = useUserStore()

const formRef = ref(null)
const loading = ref(false)

const form = reactive({
  account: '13800138001',
  password: '123456'  // BCrypt加密后的123456对应hash: $2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM8lE9lBOsl7iKTVKIUi
})

const rules = {
  account: { required: true, message: '请输入手机号', trigger: 'blur' },
  password: { required: true, message: '请输入密码', trigger: 'blur' }
}

async function handleLogin() {
  try {
    await formRef.value?.validate()
    loading.value = true

    const res = await login(form)
    userStore.setToken(res.data)

    // 获取用户信息
    // const userInfo = await getUserInfo()
    // userStore.setUserInfo(userInfo.data)

    message.success('登录成功')
    router.push('/')
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.login-container {
  display: flex;
  height: 100vh;
}

.login-left {
  flex: 1;
  background: linear-gradient(135deg, #1f883d 0%, #2a9f4a 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}

.login-left-content {
  text-align: center;
}

.login-left-content h1 {
  font-size: 42px;
  margin-bottom: 16px;
  font-weight: 600;
}

.login-left-content p {
  font-size: 18px;
  opacity: 0.9;
  margin-bottom: 40px;
}

.features {
  display: flex;
  gap: 40px;
  justify-content: center;
}

.feature {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.feature .icon {
  font-size: 32px;
}

.login-right {
  width: 450px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
}

.login-form {
  width: 360px;
}

.login-title {
  margin-bottom: 32px;
  text-align: center;
}

.login-title h2 {
  font-size: 28px;
  color: #333;
  margin-bottom: 8px;
}

.login-title p {
  color: #999;
  font-size: 14px;
}

.login-footer {
  margin-top: 24px;
  text-align: center;
}

.login-footer p {
  color: #999;
  font-size: 14px;
}

.login-footer a {
  color: #1f883d;
}
</style>
