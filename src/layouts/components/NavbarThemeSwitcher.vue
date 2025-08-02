<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Icon } from '@iconify/vue'

// Track the theme
const theme = ref<'light' | 'dark'>('light')

const themeMap: Record<'light' | 'dark', string> = {
  light: 'mdi:weather-sunny',
  dark: 'mdi:weather-night',
}

function changeMode() {
  if (theme.value === 'light') {
    document.documentElement.classList.remove('light')
    document.documentElement.classList.add('dark')
    document.documentElement.setAttribute('data-theme', 'dark')
    window.localStorage.setItem('theme', 'dark')
    theme.value = 'dark'
  } else {
    document.documentElement.classList.remove('dark')
    document.documentElement.classList.add('light')
    document.documentElement.setAttribute('data-theme', 'light')
    window.localStorage.setItem('theme', 'light')
    theme.value = 'light'
  }
}

// Load saved theme on mount
onMounted(() => {
  const saved = window.localStorage.getItem('theme') as 'light' | 'dark' | null
  if (saved === 'dark') {
    theme.value = 'dark'
    document.documentElement.classList.add('dark')
    document.documentElement.classList.remove('light')
    document.documentElement.setAttribute('data-theme', 'dark')
  } else {
    theme.value = 'light'
    document.documentElement.classList.add('light')
    document.documentElement.classList.remove('dark')
    document.documentElement.setAttribute('data-theme', 'light')
  }
})
</script>

<template>
  <div class="tooltip tooltip-bottom delay-1000" data-tip="Toggle Theme">
    <button
      class="btn btn-circle btn-sm mx-1"
      @click="changeMode"
      :style="{
        backgroundColor: 'rgba(255, 255, 255, 0.05)',
        backdropFilter: 'blur(6px)',
        border: '1px solid rgba(255, 255, 255, 0.1)',
      }"
    >
      <Icon :icon="themeMap[theme]" class="text-2xl text-white dark:text-gray-300" />
    </button>
  </div>
</template>
