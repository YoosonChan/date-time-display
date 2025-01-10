<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import LocaleSelector from './components/LocaleSelector.vue'

const currentDateTimeText = ref('')
const currentLocale = ref(navigator.language)

// date
const formatDateTime = (dateTimeStr?: Date) => {
  const date = new Date(dateTimeStr ?? Date.now())
  const options: Intl.DateTimeFormatOptions = { hour12: false }
  const dateOptions = { year: 'numeric', month: '2-digit', day: '2-digit' }
  const timeOptions = { hour: '2-digit', minute: '2-digit', second: '2-digit' }
  if (showDate.value) Object.assign(options, dateOptions)
  if (showTime.value) Object.assign(options, timeOptions)
  return new Intl.DateTimeFormat(currentLocale.value, options).format(date)
}

const updateDateTime = () => {
  if (showDate.value || showTime.value)
    currentDateTimeText.value = formatDateTime(
      isPaused.value ? (new Date(pauseDateTime.value ?? Date.now())) : undefined)
}

// toggle
const showDate = ref(true)
const toggleDate = () => {
  endLoop()
  if (showTime.value) showDate.value = !showDate.value
  startLoop()
}
const showTime = ref(true)
const toggleTime = () => {
  endLoop()
  if (showDate.value) showTime.value = !showTime.value
  if (showTime.value) {
    startLoop()
  } else {
    updateDateTime()
  }
}

// pause
const isPaused = ref(false)
const pauseDateTime = ref<number>()
const togglePause = () => {
  endLoop()
  isPaused.value = !isPaused.value
  if (isPaused.value) {
    pauseDateTime.value = Date.now()
  } else {
    startLoop()
    pauseDateTime.value = undefined
  }
}

// loop
let timer: number | undefined
const startLoop = () => {
  updateDateTime()
  // 设置定时器，每秒更新一次
  timer = setInterval(updateDateTime, 1000)
}
const endLoop = () => {
  if (timer) {
    clearInterval(timer)
    timer = undefined
  }
}

// copy
const isCopied = ref(false)
const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(currentDateTimeText.value)
    isCopied.value = true
    setTimeout(() => {
      isCopied.value = false
    }, 1000)
  } catch (err) {
    console.error('copy failed:', err)
  }
}

// hook
onMounted(() => {
  startLoop()
})
onUnmounted(() => {
  endLoop()
})
</script>

<template>
  <div @contextmenu.prevent="copyToClipboard" style="font-family: 'Source Code Pro', serif;" class="
  min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100
  flex items-center justify-center gap-8 relative
  text-5xl text-indigo-600 font-semibold
  cursor-pointer">
    <LocaleSelector @update:locale="locale => currentLocale = locale" />
    <span @click="toggleDate" class="hover:bg-indigo-600 hover:text-white transition-colors px-1">[</span>
    <div @click="togglePause" :class="{ 'opacity-60': isPaused }"
      class="text-center hover:text-indigo-800 transition-colors">
      {{ currentDateTimeText.split('\/').join('-') }}
      <span v-if="isCopied" class="absolute top-8 left-1/2 transform -translate-x-1/2 
        text-sm bg-indigo-600 text-white px-2 py-1 rounded 
        opacity-90">
        copied
      </span>
    </div>
    <span @click="toggleTime" class="hover:bg-indigo-600 hover:text-white transition-colors px-1">]</span>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Source+Code+Pro&display=swap');
</style>
