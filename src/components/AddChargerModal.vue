<template>
  <div class="w-1/2 bg-white p-6 rounded-lg shadow-lg border">
    <h3 class="text-2xl mb-4">Add charger</h3>
    <form @submit.prevent="submit">
      <div>
        <label class="text-sm">Name</label>
        <input
          required
          type="text"
          v-model="newCharger.name"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Charger Name"
        />
      </div>

      <div>
        <label class="text-sm">Latitude</label>
        <input
          required
          type="number"
          v-model="newCharger.latitude"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Latitude"
          step=".000001"
        />
      </div>
      <div>
        <label class="text-sm">Longitude</label>
        <input
          required
          type="number"
          v-model="newCharger.longitude"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Longitude"
          step=".000001"
        />
      </div>
      <div>
        <label class="text-sm">Power Output</label>
        <input
          required
          type="number"
          v-model="newCharger.powerOutput"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Power Output (e.g., 50kW)"
        />
      </div>
      <div>
        <label class="text-sm">Connector Type</label>
        <input
          required
          type="text"
          v-model="newCharger.connectorType"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Connector Type (e.g., CCS, CHAdeMO)"
        />
      </div>

      <div>
        <label class="text-sm">Status</label>
        <select v-model="newCharger.status" class="border w-full p-2 rounded-md mb-4">
          <option value="active">Active</option>
          <option value="inactive">Inactive</option>
        </select>
      </div>

      <div class="flex justify-end mt-4">
        <button type="submit" class="py-1 px-3 bg-black text-white rounded-md cursor-pointer">
          Submit
        </button>
      </div>
    </form>
    <p v-if="message">{{ message }}</p>
  </div>
</template>

<script setup lang="ts">
import type { Charger } from '@/types'
import { ref } from 'vue'

const newCharger = ref<Charger>({
  id: '',
  name: '',
  latitude: 0,
  longitude: 0,
  status: 'active',
  powerOutput: 0,
  connectorType: '',
})
const message = ref('')
const emit = defineEmits(['add-charger'])

function resetForm() {
  newCharger.value = {
    id: '',
    name: '',
    latitude: 0,
    longitude: 0,
    status: 'active',
    powerOutput: 0,
    connectorType: '',
  }
}

async function submit() {
  try {
    const response = await fetch('/api/charging-stations', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
        Connection: 'keep-alive',
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
      body: JSON.stringify(newCharger.value),
    })

    if (!response.ok) {
      throw new Error('Failed to add charger')
    }

    const data = await response.json()
    emit('add-charger', data.data)
    console.log('Charger added:', data)
    message.value = 'Charger added successfully!'
    resetForm()
  } catch (error: any) {
    message.value = 'Error adding charger: ' + (error as Error).message
    console.error('Error adding charger:', error)
  }
}
</script>
