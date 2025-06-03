<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <main class="min-h-screen w-3/4 mx-auto">
    <header class="flex justify-between items-center py-8">
      <div>
        <nav class="space-x-8">
          <RouterLink to="/">Chargers</RouterLink>
          <RouterLink to="/login">Login</RouterLink>
          <RouterLink to="/sign-up">Sign Up</RouterLink>
        </nav>
      </div>
    </header>

    <RouterView />
  </main>
</template>
