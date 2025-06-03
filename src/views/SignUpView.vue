<template>
  <main>
    <h1 class="text-2xl">Sign Up</h1>
    <form @submit.prevent="submitSignUp" class="space-y-4">
      <div>
        <label for="name">Name:</label>
        <input v-model="name" type="text" id="name" name="name" required class="border" />
      </div>
      <div>
        <label for="email">Email:</label>
        <input v-model="email" type="email" id="email" name="email" required class="border" />
      </div>
      <div>
        <label for="password">Password:</label>
        <input
          v-model="password"
          type="password"
          id="password"
          name="password"
          required
          class="border"
        />
      </div>

      <button type="submit" class="cursor-pointer border">Sign up</button>
    </form>
    <p v-if="error">{{ error }}</p>
    <p>Already have an account? <RouterLink to="/login" class="text-blue-500">Login</RouterLink></p>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
const name = ref('')
const email = ref('')
const password = ref('')
const error = ref('')
const router = useRouter()

const submitSignUp = async () => {
  error.value = '' // Clear previous error
  try {
    const response = await fetch('/api/auth/sign-up', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
        Connection: 'keep-alive',
        'Cache-Control': 'no-cache',
      },
      body: JSON.stringify({
        name: name.value,
        email: email.value,
        password: password.value,
      }),
    })

    if (!response.ok) {
      throw new Error('Sign up failed')
    }

    const data = await response.json()

    console.log('Sign up successful:', data)
    router.push('/login') // Navigate to login after successful sign up
  } catch (err: any) {
    error.value = err.message
  }
}
</script>
