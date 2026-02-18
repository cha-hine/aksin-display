<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue'

const props = defineProps({
  slides: {
    type: Array,
    required: true,
    default: () => [],
    // Chaque slide: { description: string, image: string, duration: number (ms), etat: boolean }
  },
})

const uploadUUrl = import.meta.env.VITE_STRAPI_UPLOAD_URL

const currentIndex = ref(0)
let timer = null

const activeSlides = computed(() => {
  return props.slides.filter((slide) => slide.etat)
})

const currentSlide = computed(() => {
  if (activeSlides.value.length === 0) return null
  return activeSlides.value[currentIndex.value]
})

const startSlideshow = () => {
  stopSlideshow()
  if (activeSlides.value.length === 0) return
  let duration = currentSlide.value?.duration * 1000 || 5000

  timer = setTimeout(() => {
    nextSlide()
  }, duration)
}

const stopSlideshow = () => {
  if (timer) {
    clearTimeout(timer)
    timer = null
  }
}

const nextSlide = () => {
  if (activeSlides.value.length === 0) return
  currentIndex.value = (currentIndex.value + 1) % activeSlides.value.length
  startSlideshow()
}

watch(
  activeSlides,
  (newSlides) => {
    if (currentIndex.value >= newSlides.length) {
      currentIndex.value = 0
    }
    startSlideshow()
  },
  { deep: true },
)

const CONTAINER_W = 1345
const CONTAINER_H = 1000

const onImageLoad = (e) => {
  const img = e.target
  const natW = img.naturalWidth
  const natH = img.naturalHeight
  const scale = Math.min(CONTAINER_W / natW, CONTAINER_H / natH)
  img.style.width = Math.round(natW * scale) + 'px'
  img.style.height = Math.round(natH * scale) + 'px'
}

onMounted(() => {
  startSlideshow()
})

onUnmounted(() => {
  stopSlideshow()
})
</script>

<template>
  <div class="slide-container">
    <TransitionGroup name="crossfade">
      <div v-if="currentSlide" :key="currentIndex" class="slide">
        <div v-if="currentSlide.image.url" class="slide-image-wrapper">
          <img
            :src="uploadUUrl + currentSlide.image.url"
            :alt="currentSlide.description"
            class="slide-image"
            @load="onImageLoad"
          />
        </div>
      </div>
      <div v-else class="no-slide">Aucun slide actif</div>
    </TransitionGroup>
  </div>
</template>

<style scoped>
.slide-container {
  width: 1345px;
  height: 1000px;
  margin: 0;
  padding: 0;
  position: relative;
  overflow: hidden;
  background-color: #e9dead;
}

.slide {
  width: 1345px;
  height: 1000px;
  position: absolute;
  top: 0;
  left: 0;
}

.slide-image-wrapper {
  padding: 10px 0;
  width: 1350px;
  height: 1000px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.slide-image-wrapper img {

  max-width: 100%;
  max-height: 100%;
  border: 2px solid rgba(0, 0, 0, 0.2);
  border-radius: 28px;
  box-sizing: border-box;
}

.slide-description {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 20px;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.8));
  color: #fff;
  font-size: 24px;
  text-align: center;
}

.no-slide {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #666;
  font-size: 24px;
}

.slide-indicators {
  position: absolute;
  bottom: 80px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
}

.indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: background-color 0.3s;
}

.indicator.active {
  background-color: #fff;
}

.indicator:hover {
  background-color: rgba(255, 255, 255, 0.8);
}

/* Transitions - fondu croisé sans noir */
.crossfade-enter-active,
.crossfade-leave-active {
  transition: opacity 1s ease;
}

.crossfade-enter-from {
  opacity: 0;
}

.crossfade-leave-to {
  opacity: 0;
}

.crossfade-leave-active {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
