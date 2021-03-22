<template>
  <div class="auth-container">
    <div class="auth-card">
      <h2>Login to Personal Finance</h2>
      <form @submit.prevent="handleLogin">
        <div class="form-group">
          <label class="form-label">Email</label>
          <input 
            v-model="credentials.email" 
            type="email" 
            class="form-input" 
            required 
          />
        </div>
        <div class="form-group">
          <label class="form-label">Password</label>
          <input 
            v-model="credentials.password" 
            type="password" 
            class="form-input" 
            required 
          />
        </div>
        <div v-if="error" class="error-message">{{ error }}</div>
        <button type="submit" class="btn btn-primary" :disabled="loading">
          {{ loading ? 'Logging in...' : 'Login' }}
        </button>
      </form>
      <p class="auth-link">
        Don't have an account? 
        <router-link to="/register">Sign up here</router-link>
      </p>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '../stores/auth'

export default {
  name: 'Login',
  setup() {
    const router = useRouter()
    const authStore = useAuthStore()
    
    const credentials = ref({
      email: '',
      password: ''
    })
    
    const error = ref('')
    const loading = ref(false)

    const handleLogin = async () => {
      loading.value = true
      error.value = ''
      
      const result = await authStore.login(credentials.value)
      
      if (result.success) {
        router.push('/dashboard')
      } else {
        error.value = result.error
      }
      
      loading.value = false
    }

    return {
      credentials,
      error,
      loading,
      handleLogin
    }
  }
}
</script>

<style scoped>
.auth-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 70vh;
}

.auth-card {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  width: 100%;
  max-width: 400px;
}

.auth-card h2 {
  text-align: center;
  margin-bottom: 2rem;
  color: #2c3e50;
}

.auth-link {
  text-align: center;
  margin-top: 1rem;
}

.auth-link a {
  color: #3498db;
  text-decoration: none;
}

.auth-link a:hover {
  text-decoration: underline;
}
</style>