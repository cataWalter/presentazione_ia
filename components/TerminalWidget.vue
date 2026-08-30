<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const props = withDefaults(
  defineProps<{
    prompt?: string
    text?: string
  }>(),
  {
    prompt: 'POST model=Qwen/Qwen3.8-27B ...',
    text: '[local endpoint] model=Qwen/Qwen3.8-27B\n[request] reasoning_effort=high preserve_thinking=true\n[input] text + 1024 visual tokens ingested\n[prefill] 180ms TTFT | [decode] 68 tokens/s streamed privately'
  }
)

const displayedOutput = ref('')
const outputColor = ref('#fdcb6e')

let timeoutId: number | null = null
let intervalId: number | null = null

function runTrace() {
  if (timeoutId) clearTimeout(timeoutId)
  if (intervalId) clearInterval(intervalId)

  displayedOutput.value = props.prompt
  outputColor.value = '#fdcb6e'

  timeoutId = window.setTimeout(() => {
    displayedOutput.value = ''
    outputColor.value = '#10b981'
    let index = 0
    const fullText = props.text

    intervalId = window.setInterval(() => {
      displayedOutput.value += fullText[index]
      index += 1
      if (index >= fullText.length) {
        if (intervalId) clearInterval(intervalId)
        intervalId = null
      }
    }, 18)
  }, 400)
}

onMounted(() => {
  runTrace()
})

onUnmounted(() => {
  if (timeoutId) clearTimeout(timeoutId)
  if (intervalId) clearInterval(intervalId)
})
</script>

<template>
  <div class="terminal-card">
    <div class="terminal-header">
      <div class="window-controls">
        <span class="dot red"></span>
        <span class="dot yellow"></span>
        <span class="dot green"></span>
      </div>
      <span class="window-title">bash — localhost:8000</span>
      <button type="button" class="terminal-button" @click="runTrace">
        Re-run trace
      </button>
    </div>
    <div class="terminal-body">
      <div class="command-line">
        <span class="prompt">$</span>
        <span class="command">curl http://localhost:8000/v1/chat/completions</span>
      </div>
      <div class="prompt-line">
        <span class="prompt-arrow">-&gt;</span>
        <span class="prompt-text">{{ props.prompt }}</span>
      </div>
      <pre class="output-text" :style="{ color: outputColor }">{{ displayedOutput }}</pre>
    </div>
  </div>
</template>

<style scoped>
.terminal-card {
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 10px;
  overflow: hidden;
  margin-top: 10px;
  box-shadow: 0 4px 20px -2px rgba(15, 23, 42, 0.15);
  text-align: left;
  font-family: 'Geist Mono', ui-monospace, monospace;
}

.terminal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #1e293b;
  padding: 8px 14px;
  border-bottom: 1px solid #334155;
}

.window-controls {
  display: flex;
  gap: 6px;
  align-items: center;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.dot.red { background: #ef4444; }
.dot.yellow { background: #f59e0b; }
.dot.green { background: #10b981; }

.window-title {
  color: #94a3b8;
  font-size: 11px;
  font-weight: 500;
}

.terminal-button {
  padding: 3px 10px;
  border: 1px solid #3b82f6;
  border-radius: 5px;
  background: #2563eb;
  color: #ffffff;
  font-family: 'Geist Mono', monospace;
  font-size: 10.5px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.15s ease;
}

.terminal-button:hover {
  background: #1d4ed8;
}

.terminal-body {
  padding: 12px 16px;
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 11.5px;
  line-height: 1.5;
}

.command-line {
  display: flex;
  gap: 8px;
  color: #94a3b8;
}

.command {
  color: #f1f5f9;
}

.prompt-line {
  display: flex;
  gap: 8px;
  align-items: center;
}

.prompt-arrow {
  color: #f59e0b;
  font-weight: bold;
}

.prompt-text {
  color: #94a3b8;
}

.output-text {
  margin: 4px 0 0 0;
  white-space: pre-wrap;
  font-family: inherit;
  font-size: 11.5px;
  min-height: 52px;
}
</style>