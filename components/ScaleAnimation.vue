<template>
  <div class="scale-animation" @click="next">
    <div class="phase-label">
      <span v-if="phase === 0">Today: ONNX Runtime Model Builder — ~20 curated architectures</span>
      <span v-else-if="phase === 1">The full picture: 8 modalities, 130+ architectures</span>
      <span v-else>With Mobius, we can scale across all of them.</span>
    </div>

    <div class="grid-wrapper" :class="{ zoomed: phase >= 1 }">
      <!-- Category labels -->
      <div class="category-row" v-for="cat in categories" :key="cat.name">
        <div class="category-label" :style="{ color: cat.color }">
          {{ cat.icon }} {{ cat.name }}
        </div>
        <div class="category-grid">
          <div
            v-for="(cell, i) in cat.cells"
            :key="i"
            class="cell"
            :class="{
              'builder': cell.builder,
              'mobius': cell.mobius && phase >= 2,
              'future': !cell.mobius,
              'lit': phase >= 2 && cell.mobius
            }"
            :style="{
              backgroundColor: phase >= 2 && cell.mobius ? cat.color : undefined,
              borderColor: phase >= 2 && cell.mobius ? cat.darkColor : undefined,
              animationDelay: cell.mobius ? `${(cell.animIdx % 30) * 80}ms` : '0ms'
            }"
          >
            <span v-if="phase === 0 && cell.builder && cell.name" class="label">{{ cell.name }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="bottom-bar">
      <div class="legend" v-if="phase >= 1">
        <span class="legend-item">
          <span class="dot" style="background: #3b82f6"></span> Model Builder
        </span>
        <span class="legend-item">
          <span class="dot" style="background: #10b981"></span> Mobius today
        </span>
        <span class="legend-item">
          <span class="dot" style="background: #e2e8f0"></span> Community (future)
        </span>
      </div>
      <button class="btn" @click.stop="next">
        {{ phase === 0 ? '→ Zoom out' : phase === 1 ? '→ Light them up' : '↻ Replay' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const phase = ref(0)

function next() {
  phase.value = phase.value < 2 ? phase.value + 1 : 0
}

// Model Builder supported architectures (verified)
const builderModels = [
  'AMD OLMo', 'ChatGLM', 'DeepSeek', 'ERNIE 4.5', 'Gemma', 'gpt-oss', 'Granite',
  'HunYuan', 'InternLM2', 'Llama', 'Mistral', 'Nemotron', 'Phi', 'Qwen', 'SmolLM3', 'Whisper',
]

const categories = [
  {
    name: 'Text Gen',
    icon: '💬',
    color: '#3b82f6',
    darkColor: '#1d4ed8',
    total: 18,
    mobiusCount: 14,
    builderNames: ['AMD OLMo', 'ChatGLM', 'ERNIE 4.5', 'Gemma', 'Granite', 'HunYuan', 'InternLM2', 'Llama', 'Mistral', 'Nemotron', 'Phi', 'Qwen', 'SmolLM3'],
  },
  {
    name: 'MoE',
    icon: '🔀',
    color: '#8b5cf6',
    darkColor: '#6d28d9',
    total: 10,
    mobiusCount: 6,
    builderNames: ['DeepSeek', 'gpt-oss'],
  },
  {
    name: 'Multimodal',
    icon: '👁️',
    color: '#ec4899',
    darkColor: '#be185d',
    total: 10,
    mobiusCount: 6,
    builderNames: [],
  },
  {
    name: 'Vision',
    icon: '🖼️',
    color: '#f59e0b',
    darkColor: '#d97706',
    total: 8,
    mobiusCount: 5,
    builderNames: [],
  },
  {
    name: 'Encoder',
    icon: '📝',
    color: '#06b6d4',
    darkColor: '#0891b2',
    total: 8,
    mobiusCount: 5,
    builderNames: [],
  },
  {
    name: 'Enc-Dec',
    icon: '🔄',
    color: '#14b8a6',
    darkColor: '#0d9488',
    total: 8,
    mobiusCount: 5,
    builderNames: ['Whisper'],
  },
  {
    name: 'Audio',
    icon: '🎧',
    color: '#10b981',
    darkColor: '#059669',
    total: 8,
    mobiusCount: 5,
    builderNames: [],
  },
  {
    name: 'Diffusion',
    icon: '🎨',
    color: '#f43f5e',
    darkColor: '#e11d48',
    total: 10,
    mobiusCount: 6,
    builderNames: [],
  },
].map((cat) => {
  let animIdx = 0
  const cells = []
  for (let i = 0; i < cat.total; i++) {
    const isBuilder = i < cat.builderNames.length
    const isMobius = i < cat.mobiusCount
    cells.push({
      builder: isBuilder,
      mobius: isMobius,
      name: isBuilder ? cat.builderNames[i] : '',
      animIdx: animIdx++,
    })
  }
  return { ...cat, cells }
})
</script>

<style scoped>
.scale-animation {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.8rem;
  padding: 0.5rem;
  cursor: pointer;
  user-select: none;
}

.phase-label {
  font-size: 1.1rem;
  font-weight: 600;
  color: #334155;
  text-align: center;
  min-height: 1.6rem;
}

.grid-wrapper {
  display: flex;
  flex-direction: column;
  gap: 6px;
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.category-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.category-label {
  width: 90px;
  font-size: 0.75rem;
  font-weight: 700;
  text-align: right;
  white-space: nowrap;
  transition: all 0.4s ease;
}

.zoomed .category-label {
  width: 80px;
  font-size: 0.65rem;
}

.category-grid {
  display: flex;
  gap: 4px;
  transition: all 0.6s ease;
}

.zoomed .category-grid {
  gap: 3px;
}

.cell {
  width: 36px;
  height: 28px;
  border-radius: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.45rem;
  font-weight: 600;
  color: white;
  transition: all 0.5s ease;
  background: #e2e8f0;
  border: 1px solid #cbd5e1;
}

.zoomed .cell {
  width: 28px;
  height: 22px;
}

.cell.builder {
  background: #3b82f6;
  border-color: #2563eb;
}

.zoomed .cell .label {
  display: none;
}

.cell.lit {
  animation: lightUp 0.5s ease forwards;
  box-shadow: 0 0 6px rgba(0,0,0,0.15);
}

.cell.future {
  background: #f1f5f9;
  border-color: #e2e8f0;
}

@keyframes lightUp {
  0% { transform: scale(1); filter: brightness(1); }
  40% { transform: scale(1.2); filter: brightness(1.4); }
  100% { transform: scale(1); filter: brightness(1); }
}

.bottom-bar {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  margin-top: 0.5rem;
}

.legend {
  display: flex;
  gap: 1rem;
  font-size: 0.8rem;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 2px;
  display: inline-block;
}

.btn {
  padding: 0.35rem 1rem;
  border-radius: 6px;
  background: #3b82f6;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 500;
}

.btn:hover {
  background: #2563eb;
}
</style>
