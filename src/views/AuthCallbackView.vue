<template>
  <div class="auth-callback">
    <div class="loading-container">
      <div class="loading-spinner"></div>
      <h2>카카오 로그인 처리 중...</h2>
      <p>잠시만 기다려 주세요.</p>
    </div>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useModal } from '@/composables/useModal'

const router = useRouter()
const route = useRoute()
const authStore = useAuthStore()
const { handleApiSuccess, handleApiError } = useModal()

onMounted(async () => {
  try {
    // URL에서 토큰 추출
    const token = route.query.token

    if (!token) {
      throw new Error('로그인 토큰이 없습니다.')
    }

    // 토큰을 localStorage에 저장하고 사용자 정보 가져오기
    await authStore.setTokenAndFetchUser(token)

    await handleApiSuccess('카카오 로그인에 성공했습니다!', () => {
      router.push('/dashboard')
    })
  } catch (error) {
    console.error('OAuth 콜백 처리 오류:', error)
    await handleApiError(error)
    router.push('/login?error=oauth_failed')
  }
})
</script>

<style scoped>
.auth-callback {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.loading-container {
  text-align: center;
  background: white;
  padding: 40px;
  border-radius: 16px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  max-width: 400px;
}

.loading-spinner {
  width: 60px;
  height: 60px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #fee500;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 24px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-container h2 {
  margin: 0 0 8px 0;
  color: #333;
  font-size: 24px;
  font-weight: 600;
}

.loading-container p {
  margin: 0;
  color: #666;
  font-size: 16px;
}
</style>