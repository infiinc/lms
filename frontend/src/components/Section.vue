<template>
    <div class="rounded-lg p-4 mb-6 shadow-md">
      <h2 class="text-xl font-semibold border-b border-gray-700 pb-2 mb-4">
        {{ title }}
      </h2>
      <slot></slot>
    </div>
  </template>
  
  <script setup>
  import { defineProps } from 'vue'
  
  const props = defineProps({
    title: {
      type: String,
      required: true
    }
  })
  </script>
  
  <style scoped>
  </style>