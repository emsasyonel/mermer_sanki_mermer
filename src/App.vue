<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// --- VERİ ---
const marbles = ref([
  { id: 1, name: 'Emperador Dark', image: '/img/mermer1.jpg' },
  { id: 2, name: 'Carrara White', image: '/img/mermer1.jpg' },
  { id: 3, name: 'Toros Black', image: '/img/mermer1.jpg' },
  { id: 4, name: 'Rosso Levanto', image: '/img/mermer1.jpg' },
  { id: 5, name: 'Traverten', image: '/img/mermer1.jpg' },
  { id: 6, name: 'Verde Guatemala', image: '/img/mermer1.jpg' },
])

// --- AYARLAR ---
const cardWidth = 300 
const autoRotateSpeed = 0.05

// --- STATE ---
const currentAngle = ref(0)
const isDragging = ref(false)
const startX = ref(0)
const startAngle = ref(0)
let animationFrameId: number

// --- MATEMATİK ---
const radius = computed(() => {
  const count = marbles.value.length
  return Math.round((cardWidth / 2) / Math.tan(Math.PI / count)) + 40 
})

// --- ETKİLEŞİM ---
const startDrag = (event: MouseEvent | TouchEvent) => {
  isDragging.value = true
  
  // DÜZELTME BURADA:
  // event.touches[0]?.clientX diyerek "varsa al" dedik.
  // || 0 diyerek "yoksa 0 kabul et" dedik.
  if ('touches' in event) {
    startX.value = event.touches[0]?.clientX || 0
  } else {
    startX.value = event.clientX
  }
  
  startAngle.value = currentAngle.value
}

const onDrag = (event: MouseEvent | TouchEvent) => {
  if (!isDragging.value) return

  let currentX = 0

  // DÜZELTME BURADA DA AYNI:
  if ('touches' in event) {
     currentX = event.touches[0]?.clientX || 0
  } else {
     currentX = event.clientX
  }

  const deltaX = currentX - startX.value
  currentAngle.value = startAngle.value - (deltaX * 0.2)
}

const stopDrag = () => {
  isDragging.value = false
}

const animate = () => {
  if (!isDragging.value) {
    currentAngle.value += autoRotateSpeed
  }
  animationFrameId = requestAnimationFrame(animate)
}

onMounted(() => {
  animate()
  window.addEventListener('mouseup', stopDrag)
  window.addEventListener('touchend', stopDrag)
  window.addEventListener('mousemove', onDrag)
  window.addEventListener('touchmove', onDrag)
})

onUnmounted(() => {
  cancelAnimationFrame(animationFrameId)
  window.removeEventListener('mouseup', stopDrag)
  window.removeEventListener('touchend', stopDrag)
  window.removeEventListener('mousemove', onDrag)
  window.removeEventListener('touchmove', onDrag)
})
</script>
<template>
  <div 
    class="stage" 
    :class="{ 'grabbing': isDragging }"
    @mousedown="startDrag"
    @touchstart="startDrag"
  >
    <div class="atmosphere"></div>

    <div class="carousel-container">
      <div 
        class="carousel"
        :style="{ transform: `rotateY(${currentAngle}deg)` }"
      >
        <div 
          v-for="(marble, index) in marbles" 
          :key="marble.id" 
          class="marble-card"
          :style="{ 
            transform: `rotateY(${index * (360 / marbles.length)}deg) translateZ(${radius}px)`
          }"
        >
          <img :src="marble.image" :alt="marble.name" class="marble-img" />
          
          <div class="marble-info">
            <h3>{{ marble.name }}</h3>
          </div>

          <div class="shine"></div>
          <div class="border-light"></div>
        </div>
      </div>
    </div>
    
    <div class="instruction">
      KEŞFETMEK İÇİN SÜRÜKLEYİN
    </div>
  </div>
</template>

<style scoped>
/* SAHNE AYARLARI */
.stage {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: 1000px;
  cursor: grab;
  user-select: none;
}
.stage.grabbing { cursor: grabbing; }

/* ARKA PLAN */
.atmosphere {
  position: absolute;
  inset: 0;
  z-index: -1;
  background: radial-gradient(circle at center, #4a0404 0%, #1a0000 100%);
  animation: pulse 10s infinite alternate;
}
@keyframes pulse {
  0% { filter: brightness(1); }
  100% { filter: brightness(1.2); }
}

/* CAROUSEL YAPISI */
.carousel-container {
  position: relative;
  width: 300px;
  height: 500px; /* Dikdörtgen mermer plakası formu */
  transform-style: preserve-3d;
}

.carousel {
  position: absolute;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
}

/* MERMER KARTI */
.marble-card {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: 4px;
  background-color: #000;
  /* Yansımasız net görüntü için backface hidden */
  backface-visibility: hidden;
  /* Aynalama efekti (zemine yansıma) */
  -webkit-box-reflect: below 10px linear-gradient(transparent, transparent, rgba(0,0,0,0.4));
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
}

/* RESİM AYARLARI */
.marble-img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Resmi bozmadan kutuya sığdır */
  border-radius: 4px;
  display: block;
}

/* İSİM ETİKETİ */
.marble-info {
  position: absolute;
  bottom: 20px;
  left: 0;
  width: 100%;
  text-align: center;
  z-index: 2;
}
.marble-info h3 {
  color: rgba(255, 255, 255, 0.9);
  font-weight: 300;
  letter-spacing: 2px;
  font-size: 1.2rem;
  text-shadow: 0 2px 4px rgba(0,0,0,0.8);
  background: rgba(0, 0, 0, 0.3); /* Okunurluk için hafif gölge */
  padding: 5px 0;
  backdrop-filter: blur(2px);
}

/* PARLAMA EFEKTİ (Gerçekçi mermer yüzeyi) */
.shine {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, rgba(255,255,255,0.15) 0%, transparent 40%, transparent 60%, rgba(255,255,255,0.05) 100%);
  pointer-events: none;
  z-index: 3;
}

/* KENAR IŞIĞI */
.border-light {
  position: absolute;
  inset: 0;
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 4px;
  pointer-events: none;
  z-index: 4;
}

/* TALİMAT YAZISI */
.instruction {
  position: absolute;
  bottom: 50px;
  color: rgba(255, 255, 255, 0.4);
  letter-spacing: 4px;
  font-size: 0.8rem;
  pointer-events: none;
}
</style>