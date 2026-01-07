<script setup lang="ts">
defineProps<{
  modelValue: boolean
  label?: string
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
}>()
</script>

<template>
  <div class="w-full max-w-2xl">
    <div class="bg-white rounded-2xl shadow-lg p-6 md:p-8 border border-slate-100 text-center">
      <label class="flex items-center gap-3 cursor-pointer select-none">
        <!-- label -->
        <span v-if="label" class="text-sm font-medium text-gray-700">
          {{ label }}
        </span>

        <!-- switch -->
        <span class="relative inline-flex items-center">
          <input type="checkbox" class="sr-only peer" :checked="modelValue" @click.prevent="$emit('toggle-request')" />

          <!-- track -->
          <span class="w-11 h-6 rounded-full
               bg-gray-300
               peer-checked:bg-green-500
               transition-all duration-300 ease-in-out
               peer-checked:shadow-[0_0_0_4px_rgba(34,197,94,0.25)]"></span>

          <!-- knob -->
          <span class="absolute left-0.5 top-0.5 w-5 h-5 bg-white rounded-full
               shadow-md
               transition-transform duration-300 ease-in-out
               peer-checked:translate-x-5"></span>
        </span>
      </label>
    </div>
  </div>
</template>