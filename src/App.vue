<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// --- TİP TANIMLAMA (TypeScript) ---
interface Marble {
  id: number
  name: string
  image: string
  desc: string
  price: string
}

// --- VERİ ---
const marbles = ref<Marble[]>([
  { 
    id: 1, 
    name: 'Emperador Dark', 
    image: '/img/mermer1.jpg', // Kendi resimlerini ekledikçe burayı güncelle
    desc: 'İspanya kökenli, kahverengi tonlarında ve altın rengi damarlara sahip lüks bir mermer türüdür.',
    price: 'Premium Koleksiyon'
  },
  { 
    id: 2, 
    name: 'Carrara White', 
    image: '/img/mermer1.jpg',
    desc: 'İtalya\'nın Carrara bölgesinden çıkarılan, antik çağlardan beri heykeltıraşların gözdesi olan beyaz mermer.',
    price: 'Klasik Seri'
  },
  { 
    id: 3, 
    name: 'Toros Black', 
    image: '/img/mermer1.jpg',
    desc: 'Türkiye\'nin en prestijli siyah mermerlerinden biri. Beyaz damarlarıyla güçlü bir kontrast oluşturur.',
    price: 'Özel Üretim'
  },
  { 
    id: 4, 
    name: 'Rosso Levanto', 
    image: '/img/mermer1.jpg',
    desc: 'Koyu vişne rengi ve beyaz damar yapısıyla mekanlara sıcaklık ve asalet katan nadide bir taş.',
    price: 'Elit Seri'
  },
  { 
    id: 5, 
    name: 'Traverten', 
    image: '/img/mermer1.jpg',
    desc: 'Doğal delikli yapısı ve sıcak bej tonlarıyla hem iç hem dış mekanlarda huzurlu bir atmosfer yaratır.',
    price: 'Doğal Seri'
  },
  { 
    id: 6, 
    name: 'Verde Guatemala', 
    image: '/img/mermer1.jpg',
    desc: 'Yağmur ormanlarının derin yeşilini mekanlarınıza taşıyan, egzotik ve büyüleyici bir mermer.',
    price: 'Premium Koleksiyon'
  },
])

// --- AYARLAR ---
const cardWidth = 300 
const autoRotateSpeed = 0.05

// --- STATE ---
const currentAngle = ref(0)
const isDragging = ref(false)
const startX = ref(0)
const startAngle = ref(0)
const selectedMarble = ref<Marble | null>(null)
let animationFrameId: number

// --- MATEMATİK ---
const radius = computed(() => {
  const count = marbles.value.length
  return Math.round((cardWidth / 2) / Math.tan(Math.PI / count)) + 40 
})

// --- ETKİLEŞİM: SÜRÜKLEME ---
const startDrag = (event: MouseEvent | TouchEvent) => {
  if (selectedMarble.value) return 
  isDragging.value = true
  
  if ('touches' in event) {
    startX.value = event.touches[0]?.clientX || 0
  } else {
    startX.value = (event as MouseEvent).clientX
  }
  
  startAngle.value = currentAngle.value
}

const onDrag = (event: MouseEvent | TouchEvent) => {
  if (!isDragging.value) return
  
  let currentX = 0
  if ('touches' in event) {
     currentX = event.touches[0]?.clientX || 0
  } else {
     currentX = (event as MouseEvent).clientX
  }

  const deltaX = currentX - startX.value
  currentAngle.value = startAngle.value - (deltaX * 0.2)
}

const stopDrag = () => {
  isDragging.value = false
}

// --- ETKİLEŞİM: IŞIK EFEKTİ ---
const handleCardMouseMove = (e: MouseEvent) => {
  const card = e.currentTarget as HTMLElement
  const rect = card.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  
  card.style.setProperty('--mouse-x', `${x}px`)
  card.style.setProperty('--mouse-y', `${y}px`)
}

// --- ODAKLANMA MODU ---
const openDetail = (marble: Marble) => {
  if (!isDragging.value) {
    selectedMarble.value = marble
  }
}

const closeDetail = () => {
  selectedMarble.value = null
}

// --- ANİMASYON DÖNGÜSÜ ---
const animate = () => {
  if (!isDragging.value && !selectedMarble.value) {
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

    <div 
      class="carousel-container" 
      :class="{ 'blurred': selectedMarble }"
    >
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
          @click="openDetail(marble)"
          @mousemove="handleCardMouseMove"
          @mouseleave="(e) => (e.currentTarget as HTMLElement).style.removeProperty('--mouse-x')"
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
    
    <Transition name="fade">
      <div v-if="!selectedMarble" class="instruction">
        İNCELEMEK İÇİN TIKLAYIN VEYA SÜRÜKLEYİN
      </div>
    </Transition>

    <Transition name="zoom">
      <div v-if="selectedMarble" class="detail-overlay" @click.self="closeDetail">
        <div class="detail-card">
          
          <div class="detail-image-box">
            <img :src="selectedMarble.image" :alt="selectedMarble.name" />
            <div class="detail-shine"></div>
          </div>

          <div class="detail-content">
            <button class="close-btn" @click="closeDetail">✕</button>
            <span class="detail-badge">{{ selectedMarble.price }}</span>
            <h1>{{ selectedMarble.name }}</h1>
            <p>{{ selectedMarble.desc }}</p>
            
            <div class="action-buttons">
              <button class="btn-primary">Teklif Al</button>
              <button class="btn-secondary">Kataloğa Ekle</button>
            </div>
          </div>

        </div>
      </div>
    </Transition>

  </div>
</template>

<style scoped>
/* --- GENEL AYARLAR --- */
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

/* --- CAROUSEL --- */
.carousel-container {
  position: relative;
  width: 300px;
  height: 500px;
  transform-style: preserve-3d;
  transition: filter 0.5s ease, transform 0.5s ease;
}

.carousel-container.blurred {
  filter: blur(10px) brightness(0.5);
  transform: scale(0.9);
  pointer-events: none;
}

.carousel {
  position: absolute;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
}

/* --- MERMER KARTI --- */
.marble-card {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: 4px;
  background-color: #000;
  backface-visibility: hidden;
  -webkit-box-reflect: below 10px linear-gradient(transparent, transparent, rgba(0,0,0,0.4));
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
  cursor: pointer;
  transition: transform 0.3s;
  overflow: hidden; /* Işık taşmasın */
}

.marble-card:hover {
  transform: scale(1.05);
  box-shadow: 0 0 30px rgba(255, 255, 255, 0.2);
}

.marble-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.marble-info {
  position: absolute;
  bottom: 20px;
  left: 0;
  width: 100%;
  text-align: center;
  z-index: 2;
  pointer-events: none;
}
.marble-info h3 {
  color: rgba(255, 255, 255, 0.9);
  font-weight: 300;
  letter-spacing: 2px;
  font-size: 1.2rem;
  text-shadow: 0 2px 4px rgba(0,0,0,0.8);
  background: rgba(0, 0, 0, 0.3);
  padding: 5px 0;
  backdrop-filter: blur(2px);
}

/* --- IŞIK EFEKTLERİ --- */
.shine {
  position: absolute;
  inset: 0;
  z-index: 3;
  background: radial-gradient(
    circle at var(--mouse-x, 50%) var(--mouse-y, 50%), 
    rgba(255, 255, 255, 0.4) 0%, 
    rgba(255, 255, 255, 0.1) 30%, 
    transparent 60%
  );
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
  mix-blend-mode: overlay;
}

.marble-card:hover .shine {
  opacity: 1;
}

.border-light {
  position: absolute;
  inset: 0;
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 4px;
  pointer-events: none;
  z-index: 4;
}

.instruction {
  position: absolute;
  bottom: 50px;
  color: rgba(255, 255, 255, 0.4);
  letter-spacing: 4px;
  font-size: 0.8rem;
  pointer-events: none;
}

/* --- DETAY MODALI --- */
.detail-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.2); 
}

.detail-card {
  display: flex;
  width: 800px;
  height: 500px;
  background: rgba(30, 30, 30, 0.85);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.7);
  overflow: hidden;
  color: white;
}

.detail-image-box {
  flex: 1;
  position: relative;
  overflow: hidden;
}
.detail-image-box img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.detail-shine {
  position: absolute;
  inset: 0;
  background: linear-gradient(45deg, transparent, rgba(255,255,255,0.1), transparent);
}

.detail-content {
  flex: 1;
  padding: 40px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  background: none;
  border: none;
  color: rgba(255,255,255,0.5);
  font-size: 1.5rem;
  cursor: pointer;
  transition: color 0.3s;
}
.close-btn:hover { color: white; }

.detail-badge {
  display: inline-block;
  background: rgba(184, 50, 50, 0.2);
  color: #ff8a8a;
  padding: 5px 12px;
  border-radius: 50px;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  align-self: flex-start;
  margin-bottom: 15px;
}

.detail-content h1 {
  font-size: 2.5rem;
  font-weight: 300;
  margin-bottom: 15px;
  font-family: 'Times New Roman', serif;
}

.detail-content p {
  color: rgba(255, 255, 255, 0.7);
  line-height: 1.6;
  margin-bottom: 30px;
}

.action-buttons {
  display: flex;
  gap: 15px;
}

.btn-primary {
  background: #8a1c1c;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.3s;
}
.btn-primary:hover { background: #a32424; }

.btn-secondary {
  background: transparent;
  color: white;
  border: 1px solid rgba(255,255,255,0.3);
  padding: 12px 24px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s;
}
.btn-secondary:hover { border-color: white; background: rgba(255,255,255,0.1); }

/* --- VUE GEÇİŞLERİ --- */
.fade-enter-active, .fade-leave-active { transition: opacity 0.5s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.zoom-enter-active, .zoom-leave-active { transition: all 0.4s cubic-bezier(0.25, 0.8, 0.25, 1); }
.zoom-enter-from, .zoom-leave-to { 
  opacity: 0; 
  transform: scale(0.8) translateY(20px); 
}

/* RESPONSIVE */
@media (max-width: 850px) {
  .detail-card {
    flex-direction: column;
    width: 90%;
    height: 80vh;
    overflow-y: auto;
  }
  .detail-image-box { flex: none; height: 40%; }
  .detail-content { flex: none; padding: 20px; }
}
</style>