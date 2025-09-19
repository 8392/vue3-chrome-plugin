<script setup lang="ts">
const inputVal = ref('')

function openOptionsPage() {
  browser.runtime.openOptionsPage()
}
</script>

<template>
  <main class="w-full px-4 py-5 text-center text-gray-700">
    <div>
      <input v-model="inputVal" class="px-[5px] h-[36px] border-1px border-[#ccc]">
      <div>
        inputVasss测试l：{{ inputVal }}
      </div>
    </div>

    <button class="btn mt-2" @click="openOptionsPage">
      打开options页面
    </button>
    <div class="mt-[10px]">
      <VideoRecorder />
    </div>
  </main>
</template>
