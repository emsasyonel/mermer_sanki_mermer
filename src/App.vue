<script setup lang="ts">
import { ref } from 'vue'
// DİKKAT: Import yoluna @ işareti koydum ki src klasörünü direkt bulsun
import ProductShowcase from '@/components/ProductShowcase.vue'

// --- VERİ SETLERİ ---

// 1. Kategori: Ham Mermerler
const marbles = [
  { id: 1, name: 'Emperador Dark', image: '/img/mermer1.jpg', desc: 'İspanya kökenli lüks mermer.', price: 'Plaka Serisi' },
  { id: 2, name: 'Carrara White', image: '/img/mermer1.jpg', desc: 'İtalya\'nın beyaz incisi.', price: 'Plaka Serisi' },
  { id: 3, name: 'Toros Black', image: '/img/mermer1.jpg', desc: 'Asil siyah Türk mermeri.', price: 'Plaka Serisi' },
  { id: 4, name: 'Rosso Levanto', image: '/img/mermer1.jpg', desc: 'Vişne çürüğü renginde asalet.', price: 'Plaka Serisi' },
  { id: 5, name: 'Traverten', image: '/img/mermer1.jpg', desc: 'Doğal ve sıcak doku.', price: 'Plaka Serisi' },
]

// 2. Kategori: Tasarım Masalar 
const tables = [
  { id: 10, name: 'Carrara Yemek Masası', image: '/img/mermer1.jpg', desc: '6 kişilik özel tasarım yemek masası.', price: 'Mobilya Koleksiyonu' },
  { id: 11, name: 'Toros Orta Sehpa', image: '/img/mermer1.jpg', desc: 'Salonunuz için modern bir dokunuş.', price: 'Mobilya Koleksiyonu' },
  { id: 12, name: 'Verde Yan Sehpa', image: '/img/mermer1.jpg', desc: 'Yeşil mermerin pirinç ayaklarla uyumu.', price: 'Mobilya Koleksiyonu' },
  { id: 13, name: 'Traverten Konsol', image: '/img/mermer1.jpg', desc: 'Minimalist ve doğal tasarım.', price: 'Mobilya Koleksiyonu' },
]

// 3. Kategori: Mutfak & Banyo
const countertops = [
  { id: 20, name: 'Mutfak Adası', image: '/img/mermer1.jpg', desc: 'Leke tutmayan özel işlemeli tezgah.', price: 'Mimari Çözümler' },
  { id: 21, name: 'Banyo Lavabosu', image: '/img/mermer1.jpg', desc: 'Tek parça blok mermerden oyma.', price: 'Mimari Çözümler' },
  { id: 22, name: 'Duvar Paneli', image: '/img/mermer1.jpg', desc: 'Bookmatch uygulamalı duvar kaplaması.', price: 'Mimari Çözümler' },
]

// Menü Durumu
const isMenuOpen = ref(false)
</script>

<template>
  <main class="app-container">
    
    <div class="atmosphere"></div>
    
    <nav class="ui-layer">
      <div class="brand-logo">
        MARBLE<span class="light">ART</span>
        <div class="brand-sub">NATURAL STONE GALLERY</div>
      </div>
      <div class="menu-container">
        <button class="menu-btn" :class="{ 'open': isMenuOpen }" @click="isMenuOpen = !isMenuOpen">
          <span class="line"></span>
          <span class="line short"></span>
        </button>
        <Transition name="dropdown">
          <div v-if="isMenuOpen" class="menu-dropdown">
            <ul>
              <li><a href="#section-marbles">ANASAYFA</a></li>
              <li><a href="#section-tables">HAKKIMIZDA</a></li>
              <li><a href="#section-countertops">İLETİŞİM</a></li>
            </ul>
          </div>
        </Transition>
      </div>
    </nav>

    <div class="scroll-snap-wrapper">
      
      <div id="section-marbles">
        <ProductShowcase title="HAM MERMERLER" :items="marbles" :index="0" />
      </div>

      <div id="section-tables">
        <ProductShowcase title="TASARIM MASALAR" :items="tables" :index="1" />
      </div>

      <div id="section-countertops">
        <ProductShowcase title="MİMARİ ÇÖZÜMLER" :items="countertops" :index="2" />
      </div>

    </div>

  </main>
</template>

<style scoped>
/* --- ANA KAPLAYICI --- */
.app-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden; /* Dışarı taşmayı engelle */
  background-color: #1a0000;
}

/* --- KAYDIRMA YAPISI (SCROLL SNAP) --- */
.scroll-snap-wrapper {
  height: 100vh;
  overflow-y: scroll; /* Dikey kaydırma */
  scroll-snap-type: y mandatory; /* Kaydırma bitince en yakın bölüme yapış */
  scroll-behavior: smooth; /* Yumuşak geçiş */
}

.scroll-snap-wrapper > div {
  height: 100vh; /* Her bölüm tam ekran */
  scroll-snap-align: start; /* Bölümün başlangıcına yapış */
  width: 100%;
}

/* --- ATMOSFER (Global Arka Plan) --- */
.atmosphere {
  position: fixed; /* Sabit kalsın, içerik üstünden kaysın */
  inset: 0;
  z-index: 0;
  background: radial-gradient(circle at center, #4a0404 0%, #1a0000 100%);
  animation: pulse 10s infinite alternate;
}
@keyframes pulse { 0% { filter: brightness(1); } 100% { filter: brightness(1.2); } }

/* --- GLOBAL UI LAYER --- */
.ui-layer {
  position: fixed; /* Sabit kalsın */
  top: 0; left: 0; width: 100%; padding: 40px 60px;
  display: flex; justify-content: space-between; align-items: flex-start;
  z-index: 500; pointer-events: none;
}
.brand-logo {
  font-family: 'Times New Roman', serif; font-size: 2.5rem; font-weight: 700; letter-spacing: 2px; line-height: 1; cursor: pointer; pointer-events: auto;
  background: linear-gradient(to right, #bf953f, #fcf6ba, #b38728, #fbf5b7, #aa771c);
  -webkit-background-clip: text; background-clip: text; color: transparent;
  filter: drop-shadow(0 2px 5px rgba(0,0,0,0.5));
}
.brand-logo .light { font-weight: 300; font-style: italic; margin-left: 5px; }
.brand-sub { font-family: sans-serif; font-size: 0.65rem; letter-spacing: 5px; margin-top: 10px; color: rgba(255,255,255,0.6); background: none; -webkit-text-fill-color: rgba(255,255,255,0.6); font-weight: 400; }

/* Menü stilleri */
.menu-container { position: relative; pointer-events: auto; display: flex; flex-direction: column; align-items: flex-end; }
.menu-btn { background: none; border: none; cursor: pointer; padding: 10px; display: flex; flex-direction: column; align-items: flex-end; gap: 8px; z-index: 60; }
.menu-btn .line { display: block; width: 40px; height: 2px; background-color: rgba(255,255,255,0.9); transition: all 0.3s ease; box-shadow: 0 0 10px rgba(0,0,0,0.5); }
.menu-btn .line.short { width: 25px; }
.menu-btn:hover .line.short { width: 40px; }
.menu-btn.open .line:first-child { transform: rotate(45deg) translate(7px, 7px); }
.menu-btn.open .line.short { width: 40px; transform: rotate(-45deg); }

.menu-dropdown {
  position: absolute; top: 60px; right: 0; width: 220px;
  background: rgba(20, 5, 5, 0.9); backdrop-filter: blur(15px);
  border: 1px solid rgba(191, 149, 63, 0.3); border-radius: 12px; padding: 20px 0; box-shadow: 0 15px 40px rgba(0, 0, 0, 0.6); overflow: hidden;
}
.menu-dropdown a { display: block; padding: 15px 30px; color: rgba(255,255,255,0.7); text-decoration: none; font-family: 'Times New Roman', serif; font-size: 0.9rem; letter-spacing: 2px; transition: all 0.3s ease; border-left: 2px solid transparent; }
.menu-dropdown a:hover { background: linear-gradient(90deg, rgba(191,149,63,0.1), transparent); color: #fcf6ba; border-left: 2px solid #bf953f; padding-left: 40px; }

/* Vue Transitions */
.dropdown-enter-active, .dropdown-leave-active { transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1); }
.dropdown-enter-from, .dropdown-leave-to { opacity: 0; transform: translateY(-20px); }

@media (max-width: 768px) {
  .ui-layer { padding: 30px; }
  .brand-logo { font-size: 1.8rem; }
}
</style>