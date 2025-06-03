<template>
  <div class="w-1/2 bg-white p-6 rounded-lg shadow-lg border">
    <h3 class="text-2xl mb-4">Update charger</h3>
    <form @submit.prevent="submit">
      <div>
        <label class="text-sm">Name</label>
        <input
          required
          type="text"
          v-model="updateCharger.name"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Charger Name"
        />
      </div>

      <div>
        <label class="text-sm">Latitude</label>
        <input
          required
          type="number"
          v-model="updateCharger.latitude"
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
          v-model="updateCharger.longitude"
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
          v-model="updateCharger.powerOutput"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Power Output (e.g., 50kW)"
        />
      </div>
      <div>
        <label class="text-sm">Connector Type</label>
        <input
          required
          type="text"
          v-model="updateCharger.connectorType"
          class="border w-full p-2 rounded-md mb-4"
          placeholder="Connector Type (e.g., CCS, CHAdeMO)"
        />
      </div>

      <div>
        <label class="text-sm">Status</label>
        <select v-model="updateCharger.status" class="border w-full p-2 rounded-md mb-4">
          <option value="active">Active</option>
          <option value="inactive">Inactive</option>
        </select>
      </div>

      <div class="flex justify-end mt-4">
        <button type="submit" class="py-1 px-3 bg-black text-white rounded-md cursor-pointer">
          Update
        </button>
      </div>
    </form>
    <p v-if="message">{{ message }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import type { Charger } from '@/types'

// --- Props ---
const props = defineProps<{
  chargerToUpdate: Charger | null
}>()

// --- Local Form State ---
const updateCharger = ref<Charger>({
  id: '',
  name: '',
  latitude: 0,
  longitude: 0,
  status: 'active',
  powerOutput: 0,
  connectorType: '',
})

const message = ref('')

// --- Sync with Prop on Change ---
watch(
  () => props.chargerToUpdate,
  (newCharger) => {
    if (newCharger?.id) {
      updateCharger.value = { ...newCharger }
    }
  },
  { immediate: true }, // Run the first time too
)

// --- Reset form (optional after submit) ---
function resetForm() {
  updateCharger.value = {
    id: '',
    name: '',
    latitude: 0,
    longitude: 0,
    status: 'active',
    powerOutput: 0,
    connectorType: '',
  }
}

// --- Submit Handler ---
async function submit() {
  try {
    const response = await fetch(`/api/charging-stations/${updateCharger.value.id}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
        Connection: 'keep-alive',
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
      body: JSON.stringify(updateCharger.value),
    })

    if (!response.ok) {
      throw new Error('Failed to update charger')
    }

    const data = await response.json()
    message.value = 'Charger updated successfully!'
    window.location.reload()
    resetForm()
  } catch (error: any) {
    message.value = 'Error updating charger: ' + (error as Error).message
    console.error('Error updating charger:', error)
  }
}
</script>
