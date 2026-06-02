<template>
  <div class="register-container">
    <div class="register-left">
      <div class="register-left-content">
        <h1>车辆租赁平台</h1>
        <p>加入我们，开启自驾之旅</p>
      </div>
    </div>
    <div class="register-right">
      <div class="register-form">
        <div class="register-title">
          <h2>用户注册</h2>
          <p>创建您的账号</p>
        </div>
        <n-form ref="formRef" :model="form" :rules="rules" size="large">
          <n-form-item path="phone">
            <n-input v-model:value="form.phone" placeholder="请输入手机号" />
          </n-form-item>
          <n-form-item path="code">
            <n-input v-model:value="form.code" placeholder="请输入验证码" />
            <n-button type="primary" ghost @click="handleSendCode" :disabled="codeCountdown > 0" style="margin-left: 8px">
              {{ codeCountdown > 0 ? `${codeCountdown}秒后重试` : '获取验证码' }}
            </n-button>
          </n-form-item>
          <n-form-item path="password">
            <n-input v-model:value="form.password" type="password" show-password-on="click" placeholder="请设置密码（6-20位）" />
          </n-form-item>
          <n-form-item path="confirmPassword">
            <n-input v-model:value="form.confirmPassword" type="password" show-password-on="click" placeholder="请再次输入密码" />
          </n-form-item>
          <n-form-item path="nickname">
            <n-input v-model:value="form.nickname" placeholder="请输入昵称（选填）" />
          </n-form-item>
          <n-form-item>
            <n-button type="primary" block :loading="loading" @click="handleRegister">
              注册
            </n-button>
          </n-form-item>
        </n-form>
        <div class="register-footer">
          <p>已有账号？ <router-link to="/login">立即登录</router-link></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { NForm, NFormItem, NInput, NButton, useMessage } from 'naive-ui'
import { register } from '@/api'

const router = useRouter()
const message = useMessage()

const formRef = ref(null)
const loading = ref(false)
const codeCountdown = ref(0)

const form = reactive({
  phone: '',
  code: '',
  password: '',
  confirmPassword: '',
  nickname: ''
})

const validatePassword = (rule, value) => {
  if (value === '') {
    return new Error('请再次输入密码')
  } else if (value !== form.password) {
    return new Error('两次输入的密码不一致')
  }
  return true
}

const rules = {
  phone: { required: true, pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号', trigger: 'blur' },
  code: { required: true, message: '请输入验证码', trigger: 'blur' },
  password: { required: true, min: 6, max: 20, message: '密码长度为6-20位', trigger: 'blur' },
  confirmPassword: { required: true, validator: validatePassword, trigger: 'blur' }
}

function handleSendCode() {
  if (!form.phone) {
    message.warning('请先输入手机号')
    return
  }
  if (!/^1[3-9]\d{9}$/.test(form.phone)) {
    message.warning('手机号格式不正确')
    return
  }

  // 模拟发送验证码
  message.success('验证码已发送（模拟：123456）')
  codeCountdown.value = 60
  const timer = setInterval(() => {
    codeCountdown.value--
    if (codeCountdown.value <= 0) {
      clearInterval(timer)
    }
  }, 1000)
}

async function handleRegister() {
  try {
    await formRef.value?.validate()
    loading.value = true

    await register({
      phone: form.phone,
      code: form.code,
      password: form.password,
      nickname: form.nickname
    })

    message.success('注册成功，请登录')
    router.push('/login')
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.register-container {
  display: flex;
  height: 100vh;
}

.register-left {
  flex: 1;
  background: linear-gradient(135deg, #1f883d 0%, #2a9f4a 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}

.register-left-content {
  text-align: center;
}

.register-left-content h1 {
  font-size: 42px;
  margin-bottom: 16px;
  font-weight: 600;
}

.register-left-content p {
  font-size: 18px;
  opacity: 0.9;
}

.register-right {
  width: 450px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
}

.register-form {
  width: 360px;
}

.register-title {
  margin-bottom: 32px;
  text-align: center;
}

.register-title h2 {
  font-size: 28px;
  color: #333;
  margin-bottom: 8px;
}

.register-title p {
  color: #999;
  font-size: 14px;
}

.register-footer {
  margin-top: 24px;
  text-align: center;
}

.register-footer p {
  color: #999;
  font-size: 14px;
}

.register-footer a {
  color: #1f883d;
}
</style>
