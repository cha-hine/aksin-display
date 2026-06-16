<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick, watch } from 'vue'

const props = defineProps({
  data: {
    type: Array,
    default: () => [],
  },
  vitesse_helane: {
    type: Number,
    default: 1,
  },
})

const tickerStyle = computed(() => ({
  backgroundColor: '#000000',
  color: '#ffffff',
  fontSize: '3.25em',
  fontWeight: 500,
  height: '80px',
}))

// Vitesse en pixels par frame
const speed = computed(() => props.vitesse_helane)

const bandeau1 = ref(null)
const bandeau2 = ref(null)
const scrollRef = ref(null)
const x1 = ref(0)
const x2 = ref(0)
let width = 0
let animationFrame = null

// Texte du bandeau avec séparateurs
const bandeauText = computed(() => {
  const sep = '\u00A0\u00A0\u00A0\u00A0-\u00A0\u00A0\u00A0\u00A0'
  const end = '\u00A0\u00A0\u00A0\u00A0|\u00A0\u00A0\u00A0\u00A0'
  const noms = props.data.map((item) => item.helane || item.nom || item.name || '')
  if (noms.length === 0) return ''
  return noms.join(sep) + end
})

function recalcAndReposition() {
  if (bandeau1.value) {
    const newWidth = bandeau1.value.offsetWidth
    let offset = 0
    if (width > 0) {
      offset = x1.value % newWidth
      if (offset > 0) offset -= newWidth
    }
    width = newWidth
    x1.value = offset
    x2.value = x1.value + width
  }
}

function animate() {
  x1.value -= speed.value
  x2.value -= speed.value

  if (x1.value <= -width) {
    x1.value = x2.value + width
  }
  if (x2.value <= -width) {
    x2.value = x1.value + width
  }

  animationFrame = requestAnimationFrame(animate)
}

watch(
  () => props.data,
  async () => {
    await nextTick()
    recalcAndReposition()
  },
  { deep: true },
)

watch(bandeauText, async () => {
  await nextTick()
  recalcAndReposition()
})

onMounted(async () => {
  await nextTick()
  recalcAndReposition()
  animationFrame = requestAnimationFrame(animate)
  window.addEventListener('resize', recalcAndReposition)
})

onBeforeUnmount(() => {
  if (animationFrame) {
    cancelAnimationFrame(animationFrame)
  }
  window.removeEventListener('resize', recalcAndReposition)
})
</script>

<template>
  <div v-if="props.data.length" class="bandeau-container" :style="tickerStyle">
    <div class="bandeau-scroll" ref="scrollRef">
      <span class="bandeau-content" ref="bandeau1" :style="{ transform: `translateX(${x1}px)` }">
        {{ bandeauText }}
      </span>
      <span class="bandeau-content" ref="bandeau2" :style="{ transform: `translateX(${x2}px)` }">
        {{ bandeauText }}
      </span>
    </div>
  </div>
</template>

<style scoped>
.bandeau-container {
  width: 100vw;
  overflow: hidden;
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 1000;
  display: flex;
  align-items: center;
}

.bandeau-scroll {
  position: relative;
  width: 100vw;
  height: 100%;
  overflow: hidden;
  display: flex;
  align-items: center;
}

.bandeau-content {
  display: inline-block;
  white-space: nowrap;
  font-family: Arial, sans-serif;
  will-change: transform;
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  line-height: 80px;
  pointer-events: none;
  user-select: none;
}
</style>
