<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// --- TİP TANIMLAMA ---
interface Product {
  id: number
  name: string
  image: string
  desc: string
  price: string
}

// --- PROPS (Dışarıdan gelen veriler) ---
const props = defineProps<{
  title: string
  items: Product[]
  index: number
}>()

// --- AYARLAR ---
const cardWidth = 300 
const autoRotateSpeed = 0.05

// --- STATE ---
const currentAngle = ref(0)
const isDragging = ref(false)
const startX = ref(0)
const startAngle = ref(0)
const selectedProduct = ref<Product | null>(null)
let animationFrameId: number

// --- MATEMATİK ---
const radius = computed(() => {
  const count = props.items.length
  return Math.round((cardWidth / 2) / Math.tan(Math.PI / count)) + 40 
})

// --- ETKİLEŞİM ---
const startDrag = (event: MouseEvent | TouchEvent) => {
  if (selectedProduct.value) return 
  isDragging.value = true
  
  if ('touches' in event) {
    startX.value = event.touches[0]?.clientX || 0
  } else {
    startX.value = (event as MouseEvent).clientX
  }
  
  startAngle.value = currentAngle.value
  
  window.addEventListener('mousemove', onDrag)
  window.addEventListener('touchmove', onDrag)
  window.addEventListener('mouseup', stopDrag)
  window.addEventListener('touchend', stopDrag)
}

const onDrag = (event: Event) => {
  const evt = event as MouseEvent | TouchEvent
  if (!isDragging.value) return
  
  let currentX = 0
  if ('touches' in evt) {
     currentX = evt.touches[0]?.clientX || 0
  } else {
     currentX = (evt as MouseEvent).clientX
  }

  const deltaX = currentX - startX.value
  currentAngle.value = startAngle.value - (deltaX * 0.2)
}

const stopDrag = () => {
  isDragging.value = false
  window.removeEventListener('mousemove', onDrag)
  window.removeEventListener('touchmove', onDrag)
  window.removeEventListener('mouseup', stopDrag)
  window.removeEventListener('touchend', stopDrag)
}

const handleCardMouseMove = (e: MouseEvent) => {
  const card = e.currentTarget as HTMLElement
  const rect = card.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  card.style.setProperty('--mouse-x', `${x}px`)
  card.style.setProperty('--mouse-y', `${y}px`)
}

const openDetail = (product: Product) => {
  if (!isDragging.value) selectedProduct.value = product
}
const closeDetail = () => { selectedProduct.value = null }

const animate = () => {
  if (!isDragging.value && !selectedProduct.value) {
    currentAngle.value += autoRotateSpeed
  }
  animationFrameId = requestAnimationFrame(animate)
}

onMounted(() => {
  animate()
})
onUnmounted(() => {
  cancelAnimationFrame(animationFrameId)
})
</script>

<template>
  <section class="product-section">
    
    <div class="header-container">
      <h2 class="section-title">{{ title }}</h2>
      <div class="title-underline"></div>
    </div>

    <div 
      class="stage" 
      :class="{ 'grabbing': isDragging }"
      @mousedown="startDrag"
      @touchstart="startDrag"
    >
      <div 
        class="carousel-container" 
        :class="{ 'blurred': selectedProduct }"
      >
        <div 
          class="carousel"
          :style="{ transform: `rotateY(${currentAngle}deg)` }"
        >
          <div 
            v-for="(item, i) in items" 
            :key="item.id" 
            class="marble-card"
            :style="{ 
              transform: `rotateY(${i * (360 / items.length)}deg) translateZ(${radius}px)`
            }"
            @click="openDetail(item)"
            @mousemove="handleCardMouseMove"
            @mouseleave="(e) => (e.currentTarget as HTMLElement).style.removeProperty('--mouse-x')"
          >
            <img :src="item.image" :alt="item.name" class="marble-img" />
            <div class="marble-info"><h3>{{ item.name }}</h3></div>
            <div class="shine"></div>
            <div class="border-light"></div>
          </div>
        </div>
      </div>
      
      <Transition name="fade">
        <div v-if="!selectedProduct" class="instruction">
          SÜRÜKLE & KEŞFET
        </div>
      </Transition>

      <Transition name="zoom">
        <div v-if="selectedProduct" class="detail-overlay" @click.self="closeDetail">
          <div class="detail-card">
            <div class="detail-image-box">
              <img :src="selectedProduct.image" :alt="selectedProduct.name" />
              <div class="detail-shine"></div>
            </div>
            <div class="detail-content">
              <button class="close-btn" @click="closeDetail">✕</button>
              <span class="detail-badge">{{ selectedProduct.price }}</span>
              <h1>{{ selectedProduct.name }}</h1>
              <p>{{ selectedProduct.desc }}</p>
              <div class="action-buttons">
                <button class="btn-primary">Teklif Al</button>
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </div>
  </section>
</template>

<style scoped>
/* GÜNCELLENMİŞ STİLLER (Ortalama ve Başlık Ayarlı) */

.product-section {
  height: 100vh;
  width: 100%;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  scroll-snap-align: start;
  overflow: hidden;
}

.header-container {
  padding-top: 120px; 
  text-align: center;
  z-index: 10;
  position: relative;
}

.section-title {
  font-family: 'Times New Roman', serif;
  font-size: 3.5rem;
  font-weight: 300;
  color: #f0f0f0;
  text-shadow: 0 5px 15px rgba(0, 0, 0, 0.8);
  letter-spacing: 8px;
  margin: 0;
}

.title-underline {
  width: 60px;
  height: 2px;
  background: linear-gradient(90deg, transparent, #bf953f, transparent);
  margin: 20px auto 0 auto;
}

.stage {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: 1000px;
  cursor: grab;
  margin-top: -40px;
}

.stage.grabbing { cursor: grabbing; }

/* Carousel ve Kartlar (Aynı) */
.carousel-container {
  position: relative;
  width: 300px;
  height: 500px;
  transform-style: preserve-3d;
  transition: filter 0.5s;
}
.carousel-container.blurred { filter: blur(10px) brightness(0.5); pointer-events: none; }
.carousel { width: 100%; height: 100%; position: absolute; transform-style: preserve-3d; }

.marble-card {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  background: #000; backface-visibility: hidden;
  -webkit-box-reflect: below 10px linear-gradient(transparent, transparent, rgba(0,0,0,0.4));
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
  cursor: pointer; transition: transform 0.3s;
}
.marble-card:hover { transform: scale(1.05); }
.marble-img { width: 100%; height: 100%; object-fit: cover; display: block; }

.marble-info {
  position: absolute; bottom: 20px; width: 100%; text-align: center; pointer-events: none;
}
.marble-info h3 {
  color: rgba(255,255,255,0.9); font-weight: 300; letter-spacing: 2px;
  background: rgba(0,0,0,0.6); padding: 8px 0; backdrop-filter: blur(4px);
}

/* Efektler */
.shine {
  position: absolute; inset: 0; z-index: 3; pointer-events: none; mix-blend-mode: overlay;
  background: radial-gradient(circle at var(--mouse-x, 50%) var(--mouse-y, 50%), rgba(255,255,255,0.4), transparent 60%);
  opacity: 0; transition: opacity 0.3s;
}
.marble-card:hover .shine { opacity: 1; }
.border-light { position: absolute; inset: 0; border: 1px solid rgba(255,255,255,0.15); pointer-events: none; }

.instruction {
  position: absolute; bottom: 30px; width: 100%; text-align: center;
  color: rgba(255,255,255,0.4); letter-spacing: 3px; font-size: 0.8rem; pointer-events: none;
}

/* Detay Modalı */
.detail-overlay { position: fixed; inset: 0; z-index: 100; display: flex; align-items: center; justify-content: center; background: rgba(0,0,0,0.6); }
.detail-card { display: flex; width: 800px; height: 500px; background: rgba(30,30,30,0.95); backdrop-filter: blur(20px); border-radius: 16px; border: 1px solid rgba(255,255,255,0.1); color: white; overflow: hidden; box-shadow: 0 25px 50px rgba(0,0,0,0.7); }
.detail-image-box { flex: 1; position: relative; } .detail-image-box img { width: 100%; height: 100%; object-fit: cover; }
.detail-content { flex: 1; padding: 40px; display: flex; flex-direction: column; justify-content: center; position: relative; }
.close-btn { position: absolute; top: 20px; right: 20px; background: none; border: none; color: white; font-size: 1.5rem; cursor: pointer; }
.detail-badge { background: rgba(184,50,50,0.3); color: #ffadad; padding: 6px 14px; border-radius: 20px; align-self: flex-start; margin-bottom: 15px; font-size: 0.75rem; letter-spacing: 1px; text-transform: uppercase; }
.detail-content h1 { font-family: 'Times New Roman'; font-size: 2.2rem; margin-bottom: 15px; line-height: 1.1; }
.detail-content p { color: rgba(255,255,255,0.7); line-height: 1.6; font-size: 0.95rem; }
.btn-primary { background: #8a1c1c; color: white; border: none; padding: 12px 24px; border-radius: 6px; cursor: pointer; margin-top: 25px; font-weight: 500; transition: background 0.3s; }
.btn-primary:hover { background: #a92323; }

/* Responsive */
@media (max-width: 768px) {
  .detail-card { flex-direction: column; width: 90%; height: 85vh; overflow-y: auto; }
  .detail-image-box { height: 250px; flex: none; }
  .section-title { font-size: 2rem; letter-spacing: 4px; }
  .carousel-container { width: 240px; height: 400px; }
  .header-container { padding-top: 100px; }
}
</style>