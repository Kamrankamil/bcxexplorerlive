<script setup lang="ts">
import { Icon } from '@iconify/vue'

interface Props {
  title: string
  icon: string
  stats: string
  subtitle?: string
  change?: number
  color?: string
  bordered?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  color: 'primary',
  bordered: true,
})
</script>

<template>
  <div
    class="flex items-center gap-3 px-4 py-3 bg-base-100 rounded shadow-sm"
    :class="{
      'border-r last:border-r-0 border-gray-200 dark:border-gray-700': props.bordered
    }"
  >
    <!-- Icon Circle -->
    <div class="w-10 h-10 rounded-full bg-gray-100 dark:bg-gray-800 flex items-center justify-center">
      <Icon :icon="icon" class="text-xl" :class="`text-${props.color}`" />
    </div>

    <!-- Text Content -->
    <div>
      <div class="text-sm text-gray-500 dark:text-gray-400 font-medium">
        {{ title }}
      </div>

      <div class="text-base font-semibold text-gray-900 dark:text-white">
        {{ stats }}
        <span
          v-if="change !== undefined"
          :class="change >= 0 ? 'text-success' : 'text-error'"
          class="ml-1 text-xs font-bold"
        >
          ({{ change >= 0 ? '+' : '' }}{{ change }}%)
        </span>
      </div>

      <div v-if="subtitle" class="text-xs text-gray-400">
        {{ subtitle }}
      </div>
    </div>
  </div>
</template>
