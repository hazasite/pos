<template>
  <q-layout view="lHh Lpr lFf" class="bg-black text-white font-antigravity">
    <q-header
      class="bg-black text-white q-py-xs"
      style="border-bottom: 1px solid rgba(255, 49, 49, 0.2)"
    >
      <q-toolbar>
        <q-toolbar-title class="text-weight-bold" style="letter-spacing: 1px">
          <span class="text-neon-red">HAZA</span> POS
        </q-toolbar-title>

        <div class="row items-center q-gutter-md">
          <div class="row items-center q-gutter-sm">
            <q-icon name="wifi" :color="isOnline ? 'positive' : 'grey-7'" />
            <span class="text-caption text-grey-5">{{ isOnline ? 'ONLINE' : 'OFFLINE' }}</span>
          </div>

          <div class="row items-center q-gutter-sm">
            <q-icon name="sync" class="cursor-pointer hover-rotate" color="neon-red" />
            <span class="text-caption text-grey-5">SYNC</span>
          </div>

          <div class="text-caption text-weight-medium q-pl-sm border-left">
            {{ formattedTime }}
          </div>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { date } from 'quasar'

const isOnline = ref(true) // Mock status
const currentTime = ref(Date.now())
const timer = ref(null)

const formattedTime = computed(() => {
  return date.formatDate(currentTime.value, 'HH:mm')
})

const updateTime = () => {
  currentTime.value = Date.now()
}

onMounted(() => {
  timer.value = setInterval(updateTime, 1000)
  // Mock random offline/online switching for demo
  window.addEventListener('online', () => {
    isOnline.value = true
  })
  window.addEventListener('offline', () => {
    isOnline.value = false
  })
})

onUnmounted(() => {
  if (timer.value) clearInterval(timer.value)
})
</script>

<style lang="scss">
// Custom Global Styles for the Layout
.bg-black {
  background-color: #000000 !important;
}

.text-neon-red {
  color: #ff3131 !important;
}

.text-positive {
  color: #21ba45 !important; // Quasar default positive
}

.border-left {
  border-left: 1px solid rgba(255, 255, 255, 0.1);
}

.hover-rotate {
  transition: transform 0.5s ease;
  &:hover {
    transform: rotate(180deg);
  }
}

// Global Font Fix (optional, using system fonts for now)
.font-antigravity {
  font-family: 'Inter', 'Roboto', 'Helvetica Neue', Helvetica, Arial, sans-serif;
}
</style>
