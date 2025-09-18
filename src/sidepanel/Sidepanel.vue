<script setup lang="ts">
function openOptionsPage() {
  browser.runtime.openOptionsPage()
}
</script>

<template>
  <main class="w-full px-4 py-5 text-center text-gray-700">
    <button class="btn mt-2" @click="openOptionsPage">
      打开options页面
    </button>
    <div class="mt-[10px]">
      <VideoRecorder />
    </div>
  </main>
</template>
