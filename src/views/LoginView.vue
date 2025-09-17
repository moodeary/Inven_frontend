<template>
  <div class="login-page">
    <div class="login-container">
      <div class="login-header">
        <h1>로그인</h1>
        <p>계정에 로그인하여 인벤토리를 관리하세요</p>
      </div>

      <form @submit.prevent="handleLogin" class="login-form">
        <div class="form-group">
          <label for="email">이메일</label>
          <input
            id="email"
            v-model="form.email"
            type="email"
            placeholder="이메일을 입력하세요"
            required
            :disabled="isLoading"
          />
        </div>

        <div class="form-group">
          <label for="password">비밀번호</label>
          <input
            id="password"
            v-model="form.password"
            type="password"
            placeholder="비밀번호를 입력하세요"
            required
            :disabled="isLoading"
          />
        </div>

        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>

        <button
          type="submit"
          class="login-button"
          :disabled="isLoading || !form.email || !form.password"
        >
          <span v-if="isLoading">로그인 중...</span>
          <span v-else>로그인</span>
        </button>
      </form>

      <!-- 구분선 -->
      <div class="divider">
        <span>또는</span>
      </div>

      <!-- 카카오 로그인 버튼 -->
      <div class="social-login">
        <button @click="handleKakaoLogin" class="kakao-login-button" :disabled="isLoading">
          <span class="kakao-icon">💬</span>
          <span>카카오로 로그인</span>
        </button>
      </div>

      <div class="signup-link">
        <p>계정이 없으신가요? <router-link to="/signup">회원가입</router-link></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useModal } from '@/composables/useModal'

const router = useRouter()
const authStore = useAuthStore()
const { confirm, handleApiSuccess, handleApiError } = useModal()

const form = reactive({
  email: '',
  password: ''
})

const isLoading = ref(false)
const errorMessage = ref('')

const handleLogin = async () => {
  if (!form.email || !form.password) return

  // 로그인 전 확인
  try {
    await confirm({
      title: '로그인 확인',
      message: '입력하신 정보로 로그인하시겠습니까?',
      confirmText: '로그인',
      cancelText: '취소'
    })
  } catch {
    return // 사용자가 취소한 경우
  }

  isLoading.value = true
  errorMessage.value = ''

  try {
    const response = await authStore.login({
      email: form.email,
      password: form.password
    })

    await handleApiSuccess('로그인에 성공했습니다!', () => {
      router.push('/dashboard')
    })
  } catch (error) {
    await handleApiError(error)
    errorMessage.value = error.message || '로그인에 실패했습니다'
  } finally {
    isLoading.value = false
  }
}

// 카카오 로그인 핸들러
const handleKakaoLogin = () => {
  // Spring Security OAuth2 URL 사용 (state 파라미터 자동 관리)
  window.location.href = 'http://localhost:8080/oauth2/authorization/kakao'
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 20px;
}

.login-container {
  background: white;
  padding: 40px;
  border-radius: 16px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 400px;
  border: 1px solid rgba(0, 0, 0, 0.05);
}

.login-header {
  text-align: center;
  margin-bottom: 30px;
}

.login-header h1 {
  font-size: 28px;
  font-weight: 700;
  margin: 0 0 8px 0;
  color: #1a1a1a;
  letter-spacing: -0.5px;
}

.login-header p {
  color: #666;
  margin: 0;
  font-size: 14px;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.form-group label {
  font-weight: 500;
  color: #333;
  font-size: 14px;
}

.form-group input {
  padding: 14px 16px;
  border: 1.5px solid #e1e5e9;
  border-radius: 10px;
  font-size: 16px;
  transition: all 0.3s ease;
  background-color: #fafbfc;
}

.form-group input:focus {
  outline: none;
  border-color: #000;
  background-color: #fff;
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.05);
}

.form-group input:disabled {
  background-color: #f8f9fa;
  cursor: not-allowed;
}

.error-message {
  background-color: #f8d7da;
  color: #721c24;
  padding: 12px;
  border-radius: 4px;
  font-size: 14px;
  border: 1px solid #f5c6cb;
}

.login-button {
  background: linear-gradient(135deg, #4b5563, #6b7280, #374151);
  color: white;
  border: none;
  padding: 16px 24px;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  letter-spacing: -0.2px;
  box-shadow: 0 4px 12px rgba(75, 85, 99, 0.3);
}

.login-button:hover:not(:disabled) {
  background: linear-gradient(135deg, #374151, #4b5563, #1f2937);
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(75, 85, 99, 0.4);
}

.login-button:disabled {
  background-color: #9ca3af;
  cursor: not-allowed;
  transform: none;
}

.signup-link {
  text-align: center;
  margin-top: 20px;
}

.signup-link p {
  color: #666;
  margin: 0;
  font-size: 14px;
}

.signup-link a {
  color: #374151;
  text-decoration: none;
  font-weight: 600;
}

.signup-link a:hover {
  text-decoration: underline;
}

/* 구분선 */
.divider {
  text-align: center;
  position: relative;
  margin: 24px 0;
}

.divider::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 1px;
  background-color: #e1e5e9;
}

.divider span {
  background-color: white;
  color: #666;
  padding: 0 16px;
  font-size: 14px;
  position: relative;
}

/* 소셜 로그인 */
.social-login {
  margin-bottom: 20px;
}

.kakao-login-button {
  width: 100%;
  background-color: #fee500;
  color: #000000;
  border: none;
  padding: 16px 24px;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  letter-spacing: -0.2px;
  box-shadow: 0 2px 8px rgba(254, 229, 0, 0.3);
}

.kakao-login-button:hover:not(:disabled) {
  background-color: #fada0a;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(254, 229, 0, 0.4);
}

.kakao-login-button:disabled {
  background-color: #f0f0f0;
  color: #999;
  cursor: not-allowed;
  transform: none;
}

.kakao-icon {
  font-size: 18px;
  filter: grayscale(0);
}
</style>