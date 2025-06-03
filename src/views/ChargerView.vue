<template class="mb-8">
  <div class="flex justify-between items-center">
    <h1 class="text-3xl font-semibold">Chargers</h1>

    <button
      class="py-1 px-3 bg-black text-white rounded-md cursor-pointer"
      @click="isModalOpen = !isModalOpen"
    >
      Add
    </button>
  </div>
  <section v-show="chargers.length !== 0" class="mt-8 mb-16">
    <div>
      <p class="text-gray-500">Status</p>
      <div class="space-x-4 mt-2">
        <button
          v-for="option in filterOptions"
          :key="option"
          :class="[
            'px-3 py-1 rounded-full cursor-pointer',
            selectedFilter === option ? 'bg-black text-white' : 'border',
          ]"
          @click="handleStatusFilter(option)"
        >
          {{ option }}
        </button>
      </div>
    </div>
  </section>
  <div
    class="fixed inset-0 flex items-center justify-center z-50 bg-black/75"
    v-if="isModalOpen"
    @click.self="isModalOpen = false"
  >
    <AddChargerModal @add-charger="onAddCharger" />
  </div>
  <div
    class="fixed inset-0 flex items-center justify-center z-50 bg-black/75"
    v-if="isUpdateModalOpen"
    @click.self="isUpdateModalOpen = false"
  >
    <UpdateChargerModal :charger-to-update="chargerToUpdate" />
  </div>
  <p class="mt-4 bg-red-200 border" v-if="err">{{ err }}</p>

  <div class="z-10 flex flex-row space-x-12">
    <div class="flex flex-col w-1/4">
      <li
        class="p-4 border border-gray-300 rounded-xl flex flex-row justify-between items-start mb-4"
        v-for="charger in filteredItems"
        :key="charger.id"
      >
        <div class="space-y-2">
          <p class="font-semibold text-lg">{{ charger.name }}</p>
          <p class="text-sm">Latitude: {{ charger.latitude }}</p>
          <p class="text-sm">Longitude: {{ charger.longitude }}</p>
          <p class="text-sm">Power Output: {{ charger.powerOutput }}</p>
          <p class="text-sm">Connector Type: {{ charger.connectorType }}</p>
          <p
            :class="[
              'text-sm w-fit px-2 py-1 rounded-full',
              charger.status === 'active'
                ? 'bg-green-200 text-green-700 '
                : 'bg-red-200 text-red-700',
            ]"
          >
            {{ charger.status }}
          </p>
        </div>
        <div class="space-y-4">
          <button
            class="py-1 px-2 rounded-md text-sm text-red-500 border hover:bg-red-500 hover:text-white border-red-500 cursor-pointer"
            @click="deleteCharger(charger)"
          >
            Delete
          </button>
          <button
            class="py-1 px-2 rounded-md text-sm text-black border cursor-pointer"
            @click="editCharger(charger)"
          >
            Edit
          </button>
        </div>
      </li>
    </div>

    <div
      style="height: 400px"
      id="map"
      v-show="chargers.length !== 0"
      class="rounded-xl z-10 w-3/4 sticky top-0"
    ></div>
  </div>
</template>

<script setup lang="ts">
import AddChargerModal from '@/components/AddChargerModal.vue'
import UpdateChargerModal from '@/components/UpdateChargerModal.vue'
import type { Charger } from '@/types'
import L from 'leaflet'
import { ref, onMounted, computed, nextTick } from 'vue'

const chargers = ref<Charger[]>([])
const err = ref('')
const map = ref<L.Map | null | undefined | L.LayerGroup>()
const isModalOpen = ref(false)
const isUpdateModalOpen = ref(false)
const chargerToUpdate = ref<Charger | null>(null)

const filterOptions = ['active', 'inactive'] as const
type FilterOption = (typeof filterOptions)[number]
const selectedFilter = ref<FilterOption | null>()

function handleStatusFilter(clickedOption: FilterOption) {
  //handle click, such that the same click twice will deselect
  if (clickedOption === selectedFilter.value) {
    selectedFilter.value = null
  } else {
    selectedFilter.value = clickedOption
  }
}

const filteredItems = computed(() => {
  if (!selectedFilter.value) return chargers.value
  return chargers.value.filter((charger) => charger.status === selectedFilter.value)
})

// function to edit a charger that will open the update modal and send existing charger data and pre-filled UpdateChargerModal
function editCharger(charger: Charger) {
  chargerToUpdate.value = charger
  isUpdateModalOpen.value = true
  // once is update modal is open, we can pass the charger data to the UpdateChargerModal component
}

async function deleteCharger(charger: Charger) {
  try {
    const response = await fetch(`/api/charging-stations/${charger.id}`, {
      method: 'DELETE',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
    })

    if (!response.ok) {
      throw new Error('Failed to delete charger')
    }
    // Remove the charger from the list
    chargers.value = chargers.value.filter((c) => c.id !== charger.id)
    // Remove the marker from the map
    removeSingleMarker(charger)
  } catch (error: any) {
    err.value = error.message
  }
}

async function fetchChargers() {
  try {
    chargers.value = [] // Clear previous chargers
    err.value = '' // Clear previous error

    const response = await fetch('/api/charging-stations', {
      method: 'GET',
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`,
      },
    })

    const chargerResponse = await response.json()
    if (!response.ok) {
      throw new Error(`${response.status}` || 'Failed to fetch chargers')
    }

    chargers.value = chargerResponse
  } catch (error: any) {
    console.error('Error fetching chargers:', error)
    err.value = error.message
  }
}

function initMap() {
  map.value = L.map('map').setView([12.9629, 77.5775], 13)

  // Add OpenStreetMap tiles
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors',
  }).addTo(map.value)
}

function addSingleMarker(charger: Charger) {
  if (!map.value) {
    console.log('Map is not initialized')
    return
  }
  L.marker([charger.latitude, charger.longitude]).addTo(map.value!)
    .bindPopup(`<b>${charger.name}</b>
      <br>Latitude: ${charger.latitude}
      <br>Longitude: ${charger.longitude}
      <br>Status: ${charger.status}`)
}

function removeSingleMarker(charger: Charger) {
  if (!map.value) {
    console.log('Map is not initialized')
    return
  }
  const marker = L.marker([charger.latitude, charger.longitude])
  map.value.removeLayer(marker)
}

function onAddCharger(newCharger: Charger) {
  chargers.value.push(newCharger)
  addSingleMarker(newCharger)
}

onMounted(async () => {
  initMap()

  await fetchChargers()
  if (chargers.value.length > 0) {
    chargers.value.forEach((charger) => {
      addSingleMarker(charger)
    })
  } else {
    err.value = 'No chargers available to display on the map'
  }
})
</script>
