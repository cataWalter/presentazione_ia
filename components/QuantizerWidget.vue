<script setup lang="ts">
import { ref } from 'vue'

const currentBits = ref<16 | 8 | 4>(16)

const states = {
  16: {
    matrix: "Stored values: <span class='mono'>[0.8741, -0.4219, 0.1093, 0.9513]</span>",
    status: "Precision: <b>16-bit</b> | Idealized weights: <b>54 GB</b> | Relative error: <b>baseline</b>"
  },
  8: {
    matrix: "Quantized values: <span class='mono'>[112, -54, 14, 122]</span>",
    status: "Precision: <b>8-bit</b> | Idealized weights: <b>27 GB</b> | Relative error: <b>low, method dependent</b>"
  },
  4: {
    matrix: "Quantized values: <span class='mono'>[7, -3, 1, 8]</span>",
    status: "Precision: <b>4-bit</b> | Idealized weights: <b>13.5 GB</b> | Relative error: <b>higher, method dependent</b>"
  }
}

function selectBits(bits: 16 | 8 | 4) {
  currentBits.value = bits
}
</script>

<template>
  <div class="card gold quantizer-widget">
    <div class="widget-heading">
      <b>Precision trade-off simulator</b>
      <span class="widget-hint">Select a storage format to inspect compression &amp; memory</span>
    </div>
    <div class="quant-controls">
      <button
        class="quant-bit-btn"
        :class="{ active: currentBits === 16 }"
        type="button"
        @click="selectBits(16)"
      >
        FP16 / BF16 (16-bit)
      </button>
      <button
        class="quant-bit-btn"
        :class="{ active: currentBits === 8 }"
        type="button"
        @click="selectBits(8)"
      >
        INT8 / FP8 (8-bit)
      </button>
      <button
        class="quant-bit-btn"
        :class="{ active: currentBits === 4 }"
        type="button"
        @click="selectBits(4)"
      >
        AWQ / GGUF (4-bit)
      </button>
    </div>
    <div class="quant-matrix" v-html="states[currentBits].matrix"></div>
    <div class="quant-status" v-html="states[currentBits].status"></div>
  </div>
</template>

<style scoped>
.quantizer-widget {
  padding: 12px 16px;
  background: #ffffff;
  border: 1px solid var(--border);
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

.quant-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin: 8px 0 6px;
  padding: 4px;
  background: #f1f5f9;
  border-radius: 8px;
  width: fit-content;
}

.quant-bit-btn {
  background: transparent;
  border: none;
  color: #475569;
  padding: 5px 12px;
  border-radius: 6px;
  font-size: 11.5px;
  font-family: 'Geist Mono', monospace;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s ease;
}

.quant-bit-btn:hover {
  color: #0f172a;
}

.quant-bit-btn.active {
  background: #ffffff;
  color: var(--accent);
  font-weight: 700;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.quant-matrix,
.quant-status {
  margin-top: 4px;
  color: var(--muted);
  font-family: 'Geist Mono', monospace;
  font-size: 11.5px;
  line-height: 1.45;
}
</style>