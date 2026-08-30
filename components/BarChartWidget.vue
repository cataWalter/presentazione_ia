<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Tier {
  id: string
  label: string
  sublabel: string
  targetValue: number
  targetWidth: string
  suffix?: string
}

const tiers: Tier[] = [
  { id: 'tier24', label: '24 GB class', sublabel: 'RTX 3090/4090 · 14B-8bit / 27B-4bit', targetValue: 24, targetWidth: '15%' },
  { id: 'tier48', label: '48 GB class', sublabel: 'Dual GPU / RTX 6000 · 27B-8bit / 70B-4bit', targetValue: 48, targetWidth: '30%' },
  { id: 'tier80', label: '80 GB+ class', sublabel: 'A100 / H100 / Mac Studio · 70B-8bit', targetValue: 80, targetWidth: '50%' },
  { id: 'tierMulti', label: 'Multi-GPU TP', sublabel: 'Tensor Parallelism · Frontier scale', targetValue: 160, targetWidth: '100%', suffix: '+' }
]

const animatedValues = ref<Record<string, number>>({
  tier24: 0,
  tier48: 0,
  tier80: 0,
  tierMulti: 0
})

const isAnimated = ref(false)

function runAnimation() {
  isAnimated.value = false
  animatedValues.value = { tier24: 0, tier48: 0, tier80: 0, tierMulti: 0 }

  requestAnimationFrame(() => {
    isAnimated.value = true
    const duration = 1200
    let startTime: number | null = null

    const step = (timestamp: number) => {
      if (!startTime) startTime = timestamp
      const progress = Math.min((timestamp - startTime) / duration, 1)
      const eased = 1 - Math.pow(1 - progress, 3)

      tiers.forEach((tier) => {
        animatedValues.value[tier.id] = Math.floor(tier.targetValue * eased)
      })

      if (progress < 1) {
        requestAnimationFrame(step)
      } else {
        tiers.forEach((tier) => {
          animatedValues.value[tier.id] = tier.targetValue
        })
      }
    }

    requestAnimationFrame(step)
  })
}

onMounted(() => {
  runAnimation()
})
</script>

<template>
  <div class="hardware-chart">
    <div v-for="tier in tiers" :key="tier.id" class="chart-row">
      <div class="chart-label">
        <b>{{ tier.label }}</b>
        <span>{{ tier.sublabel }}</span>
      </div>
      <div class="bar-track">
        <div
          class="bar-fill hardware"
          :class="{ animate: isAnimated }"
          :style="{ width: isAnimated ? tier.targetWidth : '0%' }"
        >
          <span class="bar-value">
            {{ animatedValues[tier.id] }} GB{{ tier.suffix || '' }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.hardware-chart {
  margin-top: 6px;
  width: 100%;
}

.chart-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 9px;
}

.chart-label {
  width: 220px;
  flex-shrink: 0;
  font-size: 11.5px;
  color: #64748b;
  text-align: right;
  line-height: 1.3;
}

.chart-label b {
  color: #0f172a;
  display: block;
  font-size: 13px;
  font-family: 'Plus Jakarta Sans', sans-serif;
  font-weight: 700;
}

.bar-track {
  flex: 1;
  height: 28px;
  background: #f1f5f9;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  overflow: hidden;
  position: relative;
}

.bar-fill {
  height: 100%;
  width: 0%;
  border-radius: 5px;
  transition: width 1.2s cubic-bezier(0.16, 1, 0.3, 1);
  display: flex;
  align-items: center;
  padding-left: 10px;
}

.bar-fill.hardware {
  background: linear-gradient(90deg, #2563eb, #7c3aed);
}

.bar-value {
  font-size: 12px;
  font-weight: 700;
  color: #ffffff;
  white-space: nowrap;
  font-family: 'Geist Mono', monospace;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
}
</style>