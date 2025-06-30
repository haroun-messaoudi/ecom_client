<script setup>
import { ref, onMounted, computed, nextTick, watch } from 'vue'
import { useRouter } from 'vue-router'
import { useMotion } from '@vueuse/motion'
import ProductList from '../components/ProductList.vue'
import { useSearchStore } from '@/stores/search'
import { useProductStore } from '@/stores/products'

const router = useRouter()
const searchStore = useSearchStore()
const productStore = useProductStore()

const categories = ref([])
const activeCategoryId = ref(null)
const isMobile = ref(window.innerWidth < 768)

const recommended = computed(() => productStore.topOrdered.slice(0, 4))
const discounted = computed(() => productStore.discounted.slice(0, 4))
const newProducts = computed(() => productStore.newProducts.slice(0, 4))

const loadingRecommended = computed(() => productStore.loadingTopOrdered)
const loadingDiscounted = computed(() => productStore.loadingDiscounted)
const loadingNewProducts = computed(() => productStore.loadingNewProducts)

function goToCategory(cat) {
  activeCategoryId.value = cat.id
  searchStore.setCategory(cat.id === 0 ? null : cat.id)
  router.push({ name: 'products', query: { category: cat.id === 0 ? '' : cat.id } })
}

function setupScrollAnimations() {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('fade-in-up')
      }
    })
  }, { threshold: 0.2 })

  nextTick(() => {
    document.querySelectorAll('.scroll-animate').forEach(el => observer.observe(el))
  })
}


function getOptimizedImage(url) {
  if (!url.includes('res.cloudinary.com')) return url

  const parts = url.split('/upload/')
  return `${parts[0]}/upload/f_auto,q_auto,w_300,h_200,c_fill/${parts[1]}`
}

onMounted(async () => {
  await searchStore.fetchCategories()
  categories.value = [{ id: 0, name: 'All', image: null }, ...searchStore.categories]

  productStore.fetchTopOrdered()
  productStore.fetchDiscounted()
  productStore.fetchNewProducts()

  setupScrollAnimations()
})

watch(
  () => [
    loadingRecommended.value,
    loadingDiscounted.value,
    loadingNewProducts.value
  ],
  ([l1, l2, l3]) => {
    if (!l1 && !l2 && !l3) {
      setupScrollAnimations()
    }
  }
)
</script>

<template>
  <div>
    <!-- Hero Section -->
    <section class="relative h-[500px] overflow-hidden flex items-center justify-center text-center mb-10">
      <div class="absolute inset-0 bg-cover bg-center scale-110 transition-all duration-1000"
           :style="{ backgroundImage: 'url(/hero.jpeg)' }"></div>
      <div class="absolute inset-0 bg-black/50"></div>
      <div
        class="relative z-10 text-white space-y-4 px-4 max-w-2xl mx-auto text-center"
        v-motion
        :initial="{ y: 30, opacity: 0 }"
        :enter="{ y: 0, opacity: 1, transition: { duration: 800 } }"
      >
        <h1 class="text-4xl md:text-5xl font-extrabold">Fuel Your Gains</h1>
        <p class="text-lg max-w-xl mx-auto drop-shadow-md">
          Discover protein powders, pre‑workouts, and recovery shakes.
        </p>
        <button
          @click="$el.querySelector('#productSections')?.scrollIntoView({ behavior: 'smooth' })"
          class="px-8 py-3 bg-[#FF6F00] text-white rounded-full font-medium hover:scale-105 transition"
        >
          Shop Now
        </button>
      </div>
    </section>

    <!-- Category Carousel -->
    <section class="w-full px-0 mb-12">
      <h2 class="text-xl font-semibold mb-4 text-center">Shop by Category</h2>
      <div class="max-w-screen-xl mx-auto">
        <div class="flex gap-4 overflow-x-auto no-scrollbar pb-2 snap-x snap-mandatory justify-start pl-0 -mr-4">
          <div
            v-for="cat in categories"
            :key="cat.id"
            :class="[
              'min-w-[110px] md:min-w-[140px] snap-start flex-shrink-0 text-center scroll-animate opacity-0 translate-y-4',
              cat.id === 0 ? 'ml-4' : ''
            ]"
          >
            <div
              class="relative w-full h-28 rounded-lg overflow-hidden cursor-pointer group transition-shadow border-2"
              :class="{
                'border-orange-500': activeCategoryId === cat.id,
                'border-transparent': activeCategoryId !== cat.id
              }"
              @click="goToCategory(cat)"
            >
              <!-- 'All' Icon Card -->
              <div
                v-if="cat.id === 0"
                class="w-full h-full relative bg-gray-400 text-orange-700 font-semibold text-sm flex items-center justify-center"
              >
                <svg class="w-9 h-9 text-white opacity-90" fill="none" viewBox="0 0 28 28" stroke="currentColor" stroke-width="1.5">
                  <rect x="4" y="4" width="6" height="6" rx="2" fill="currentColor" />
                  <rect x="4" y="14" width="6" height="6" rx="2" fill="currentColor" />
                  <rect x="14" y="4" width="6" height="6" rx="2" fill="currentColor" />
                  <rect x="14" y="14" width="6" height="6" rx="2" fill="currentColor" />
                </svg>
                <div class="absolute inset-0 flex items-end justify-center pb-2">
                  <span class="text-white font-semibold text-base drop-shadow">All</span>
                </div>
              </div>

              <!-- Category image -->
              <img
                v-else-if="cat.image"
                :src="getOptimizedImage(cat.image)"
                :alt="cat.name"
                loading="lazy"
                width="300"
                height="200"
                class="w-full h-full object-cover transition-transform group-hover:scale-105"
              />

              <!-- Fallback background for categories without image -->
              <div v-else class="w-full h-full bg-orange-100 flex items-center justify-center text-orange-700">
                {{ cat.name }}
              </div>

              <!-- Name overlay -->
              <div
                v-if="cat.id !== 0"
                class="absolute inset-0 bg-black/40 flex items-center justify-center text-white font-semibold text-sm"
              >
                {{ cat.name }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>


    <!-- Product Sections -->
    <section id="productSections" class="space-y-24">
      <!-- Recommended Section -->
      <div class="scroll-animate opacity-0 translate-y-8 rounded-2xl bg-white shadow-md px-6 py-10 md:px-10">
        <h2 class="text-center text-3xl font-semibold text-gray-800 mb-10 tracking-tight">Recommended for You</h2>
        <ProductList :products="recommended" :loading="loadingRecommended" />
      </div>

      <!-- Discounted Section -->
      <div class="scroll-animate opacity-0 translate-y-8 rounded-2xl bg-gradient-to-tr from-orange-50 to-white shadow-md px-6 py-10 md:px-10">
        <h2 class="text-center text-3xl font-semibold text-gray-800 mb-10 tracking-tight">Products on Sale</h2>
        <ProductList :products="discounted" :loading="loadingDiscounted" />
      </div>

      <!-- New Arrivals Section -->
      <div class="scroll-animate opacity-0 translate-y-8 rounded-2xl bg-white shadow-md px-6 py-10 md:px-10">
        <h2 class="text-center text-3xl font-semibold text-gray-800 mb-10 tracking-tight">New Arrivals</h2>
        <ProductList :products="newProducts" :loading="loadingNewProducts" />
      </div>
    </section>
        <footer class="mt-24 bg-gradient-to-br from-gray-900 via-gray-800 to-gray-900 text-white shadow-inner border-t border-gray-700">
      <div class="max-w-screen-xl mx-auto px-6 py-10 flex flex-col md:flex-row items-center justify-between gap-6">
        
        <!-- Social Links -->
        <div class="flex items-center gap-6">
          <a href="https://instagram.com/yourstore" target="_blank" rel="noopener" aria-label="Instagram" class="hover:text-orange-500 transition">
            <!-- Instagram SVG -->
            <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <rect x="2" y="2" width="20" height="20" rx="6" stroke="currentColor" stroke-width="2" fill="none"/>
              <circle cx="12" cy="12" r="5" stroke="currentColor" stroke-width="2" fill="none"/>
              <circle cx="17" cy="7" r="1.2" fill="currentColor"/>
            </svg>
          </a>
          <a href="https://tiktok.com/@yourstore" target="_blank" rel="noopener" aria-label="TikTok" class="hover:text-orange-500 transition">
            <!-- TikTok SVG -->
            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
              <path d="M17.5 2v2.5a3.5 3.5 0 0 0 3.5 3.5h1V10h-1a6 6 0 0 1-6-6V2h2.5zM9 8a6 6 0 1 0 6 6h-2a4 4 0 1 1-4-4V8z"/>
            </svg>
          </a>
        </div>

        <!-- Contact & Copyright -->
        <div class="text-center md:text-right text-sm space-y-1">
          <p>Call us: <a href="tel:+1234567890" class="underline hover:text-orange-500 transition">+1 234 567 890</a></p>
          <p class="text-gray-400 text-xs">&copy; {{ new Date().getFullYear() }} Ecom Shoes. All rights reserved.</p>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.no-scrollbar::-webkit-scrollbar {
  display: none;
}
.no-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
.fade-in-up {
  opacity: 1 !important;
  transform: translateY(0) !important;
  transition: all 0.6s ease-out;
}
.scroll-animate {
  opacity: 0;
  transform: translateY(30px);
}
</style>
