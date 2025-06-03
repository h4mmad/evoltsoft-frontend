<template>
  <h1 class="text-2xl">Login</h1>
  <div class="login">
    <form @submit.prevent="submitLogin">
      <div>
        <label>Email:</label>
        <input v-model="email" type="email" required class="border" />
      </div>
      <div>
        <label>Password:</label>
        <input v-model="password" type="password" required class="border" />
      </div>
      <button type="submit" class="border">Login</button>
    </form>
    <p v-if="error">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const email = ref('')
const password = ref('')
const error = ref('')
const router = useRouter()

const submitLogin = async () => {
  error.value = '' // Clear previous error
  try {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
        Connection: 'keep-alive',
        'Cache-Control': 'no-cache',
      },
      body: JSON.stringify({
        email: email.value,
        password: password.value,
      }),
    })

    if (!response.ok) {
      throw new Error('Invalid credentials')
    }

    const data = await response.json()
    localStorage.setItem('token', data.token) // Save token if needed

    router.push('/') // Navigate after login
  } catch (err) {
    error.value = err.message
  }
}
</script>
