<script setup lang="ts">
import { ref } from 'vue'

const videoRef = ref<HTMLVideoElement | null>(null)
const includeMic = ref(false)
const status = ref('空闲')

let screenStream: MediaStream | null = null
let micStream: MediaStream | null = null
let combinedStream: MediaStream | null = null
let mediaRecorder: MediaRecorder | null = null
let recordedChunks: BlobPart[] = []
const isRecording = ref(false)
const isPaused = ref(false)

async function startRecording() {
  try {
    status.value = '请求屏幕权限...'
    screenStream = await navigator.mediaDevices.getDisplayMedia({
      // video: true,
      video: {
        width: { ideal: 7680, max: 7680 },
        height: { ideal: 4320, max: 4320 },
      },
      audio: true, // 尝试获取系统声音
    })

    if (includeMic.value) {
      try {
        micStream = await navigator.mediaDevices.getUserMedia({ audio: true })
      }
      catch (err) {
        console.warn('麦克风获取失败:', err)
      }
    }

    const tracks = [
      ...screenStream.getVideoTracks(),
      ...screenStream.getAudioTracks(),
      ...(micStream ? micStream.getAudioTracks() : []),
    ]
    combinedStream = new MediaStream(tracks)

    if (videoRef.value) {
      videoRef.value.srcObject = combinedStream
      videoRef.value.muted = false
      await videoRef.value.play()
    }

    recordedChunks = []
    mediaRecorder = new MediaRecorder(combinedStream, { mimeType: 'video/webm; codecs=vp8,opus' })
    mediaRecorder.ondataavailable = (e) => {
      if (e.data.size > 0)
        recordedChunks.push(e.data)
    }
    mediaRecorder.onstop = handleStop
    mediaRecorder.start(1000)

    status.value = '录制中...'
    isRecording.value = true
    isPaused.value = false
  }
  catch (err) {
    console.error(err)
    status.value = `录制失败: ${(err as Error).message}`
  }
}

function pauseRecording() {
  if (mediaRecorder && mediaRecorder.state === 'recording') {
    mediaRecorder.pause()
    status.value = '已暂停'
    isPaused.value = true
  }
}

function resumeRecording() {
  if (mediaRecorder && mediaRecorder.state === 'paused') {
    mediaRecorder.resume()
    status.value = '录制中...'
    isPaused.value = false
  }
}

function stopRecording() {
  if (mediaRecorder && (mediaRecorder.state === 'recording' || mediaRecorder.state === 'paused')) {
    mediaRecorder.stop()
  }
  [screenStream, micStream, combinedStream].forEach(s => s?.getTracks().forEach(t => t.stop()))
  isRecording.value = false
  isPaused.value = false
  status.value = '已停止'
}

function handleStop() {
  const blob = new Blob(recordedChunks, { type: 'video/webm' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `screen-record-${Date.now()}.webm`
  a.click()
  URL.revokeObjectURL(url)
}
</script>

<template>
  <div class="screen-recorder">
    <h3>屏幕录制</h3>

    <div class="controls">
      <label>
        <input v-model="includeMic" type="checkbox"> 包含麦克风
      </label>

      <button :disabled="isRecording" @click="startRecording">
        开始录制
      </button>
      <button :disabled="!isRecording || isPaused" @click="pauseRecording">
        暂停
      </button>
      <button :disabled="!isPaused" @click="resumeRecording">
        继续
      </button>
      <button :disabled="!isRecording" @click="stopRecording">
        停止并下载
      </button>
    </div>

    <div class="preview">
      <video ref="videoRef" autoplay playsinline muted />
    </div>

    <p>状态: {{ status }}</p>
  </div>
</template>

<style scoped>
.screen-recorder {
  font-family: Arial, sans-serif;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
}
.controls {
  margin-bottom: 12px;
}
.controls button {
  margin-right: 8px;
}
.preview {
  margin-top: 12px;
}
video {
  width: 100%;
  max-height: 360px;
  border: 1px solid #999;
}
</style>
