<script setup lang="ts">
import { ref, onMounted } from 'vue'

const STORAGE_KEY = 'preferred-locale'
const emit = defineEmits<{
  (e: 'update:locale', value: string): void
}>()
const locales = [
  { code: 'en', name: 'English' },
  { code: 'zh', name: '中文' },
  { code: 'ja', name: '日本語' },
  { code: 'ko', name: '한국어' },
]
const selectedLocale = ref(localStorage.getItem(STORAGE_KEY) || 'en')
const updateLocale = (locale: string) => {
  selectedLocale.value = locale
  localStorage.setItem(STORAGE_KEY, locale)
  emit('update:locale', locale)
}

onMounted(() => {
  emit('update:locale', selectedLocale.value)
})
</script>

<template>
  <div class="absolute top-4 right-4 flex items-center gap-2 text-sm">
    <select v-model="selectedLocale" @change="updateLocale(selectedLocale)" class="bg-white/80 backdrop-blur text-indigo-600 px-2 py-1 rounded border border-indigo-200 
      focus:outline-none focus:ring-2 focus:ring-indigo-400">
      <option v-for="locale in locales" :key="locale.code" :value="locale.code">
        {{ locale.name }}
      </option>
    </select>
  </div>
</template>
