<script setup lang="ts">
import { ref } from 'vue'

const currentEffort = ref<'high' | 'medium' | 'low'>('high')

const descriptions = {
  high: '<b>high</b> for complex tasks demanding multi-step reasoning, coding, and mathematical verification.',
  medium: '<b>medium</b> to balance reasoning depth and response speed.',
  low: '<b>low</b> for efficient reasoning when speed and cost matter most.'
}

function setEffort(effort: 'high' | 'medium' | 'low') {
  currentEffort.value = effort
}
</script>

<template>
  <div class="card accent3 thinking-widget">
    <div class="widget-heading">
      <b>Reasoning effort simulator</b>
      <span class="widget-hint">Select effort level to see token budget trade-offs</span>
    </div>
    <div class="thinking-controls">
      <button
        class="thinking-btn"
        :class="{ active: currentEffort === 'high' }"
        type="button"
        @click="setEffort('high')"
      >
        high
      </button>
      <button
        class="thinking-btn"
        :class="{ active: currentEffort === 'medium' }"
        type="button"
        @click="setEffort('medium')"
      >
        medium
      </button>
      <button
        class="thinking-btn"
        :class="{ active: currentEffort === 'low' }"
        type="button"
        @click="setEffort('low')"
      >
        low
      </button>
    </div>
    <div class="thinking-output" v-html="descriptions[currentEffort]"></div>
  </div>
</template>

<style scoped>
.thinking-widget {
  padding: 12px 16px;
  margin-top: 6px;
  margin-bottom: 8px;
  background: #ffffff;
  border: 1px solid var(--border);
  border-left: 4px solid var(--accent3);
}

.widget-heading {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 12px;
  color: var(--text);
  font-size: 13px;
}

.widget-hint {
  color: #64748b;
  font-family: 'Geist Mono', monospace;
  font-size: 10px;
}

.thinking-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin: 8px 0 6px;
  padding: 4px;
  background: #f1f5f9;
  border-radius: 8px;
  width: fit-content;
}

.thinking-btn {
  min-width: 68px;
  padding: 5px 12px;
  border: none;
  border-radius: 6px;
  background: transparent;
  color: #475569;
  font-family: 'Geist Mono', monospace;
  font-size: 11.5px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s ease;
}

.thinking-btn:hover {
  color: #0f172a;
}

.thinking-btn.active {
  background: #ffffff;
  color: var(--accent3);
  font-weight: 700;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.thinking-output {
  margin-top: 6px;
  color: var(--muted);
  font-size: 12.5px;
  line-height: 1.45;
}

.thinking-output :deep(b) {
  color: var(--accent3);
  font-family: 'Geist Mono', monospace;
}
</style>