<script setup>
import { ref, onMounted, computed, nextTick, watch } from 'vue'
import { useRouter } from 'vue-router'
import { useMotion } from '@vueuse/motion'
import ProductList from '../components/ProductList.vue'
import { useSearchStore } from '@/stores/search'
import { useProductStore } from '@/stores/products'
import Button from 'primevue/button'


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

let observerInitialized = false
function setupScrollAnimations() {
  if (observerInitialized) return
  observerInitialized = true

  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add('fade-in-up')
        observer.unobserve(entry.target)
      }
    })
  }, { threshold: 0.2 })

  nextTick(() => {
    document.querySelectorAll('.scroll-animate').forEach(el => observer.observe(el))
  })
}

function getOptimizedImage(url) {
  const bunnyBase = 'https://mybunnyI.b-cdn.net'; // Replace with your real BunnyCDN hostname

  // ✅ Already BunnyCDN? Return as is
  if (url.includes('b-cdn.net') || url.startsWith(bunnyBase)) {
    return url;
  }

  // ✅ Cloudinary → BunnyCDN without resizing
  if (url.includes('res.cloudinary.com')) {
    const parts = url.split('/upload/');
    if (parts.length === 2) {
      return `${bunnyBase}/image/upload/${parts[1]}`;
    }
    return url; // fallback if unexpected
  }

  // ✅ Proxy backend/static images via BunnyCDN
  if (url.startsWith('http')) {
    return `${bunnyBase}/uploads/${encodeURIComponent(url)}`;
  }

  // ✅ Fallback: relative URLs
  return `${bunnyBase}${url.startsWith('/') ? '' : '/'}${url}`;
}


function showAllRecommended() {
  router.push({ name: 'products' })
}
function showAllOnSale() {
  router.push({ name: 'onSales' })
}
function showAllNewArrivals() {
  router.push({ name: 'newArrivals' })
}

onMounted(async () => {
  await searchStore.fetchCategories()
  categories.value = [{ id: 0, name: 'All', image: null }, ...searchStore.categories]

  setupScrollAnimations()

  // Use the home-specific fetch methods for top 4 products
  productStore.fetchTopOrderedHome()
  productStore.fetchDiscountedHome()
  await productStore.fetchNewProductsHome()
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
        <h1 class="text-4xl md:text-5xl font-extrabold">Step Up Your Style.</h1>
        <p class="text-lg max-w-xl mx-auto drop-shadow-md">
          Find the perfect shoes for every step — from everyday comfort to standout style.
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
            :class="[ 'min-w-[110px] md:min-w-[140px] snap-start flex-shrink-0 text-center scroll-animate opacity-0 translate-y-4', cat.id === 0 ? 'ml-4' : '' ]"
          >
            <div
              class="relative w-full h-28 rounded-lg overflow-hidden cursor-pointer group transition-shadow border-2"
              :class="{
                'border-orange-500': activeCategoryId === cat.id,
                'border-transparent': activeCategoryId !== cat.id
              }"
              @click="goToCategory(cat)"
            >
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
              <img
                v-else-if="cat.image"
                :src="getOptimizedImage(cat.image)"
                :alt="cat.name"
                loading="lazy"
                width="300"
                height="200"
                class="w-full h-full object-cover transition-transform group-hover:scale-105"
              />
              <div v-else class="w-full h-full bg-orange-100 flex items-center justify-center text-orange-700">
                {{ cat.name }}
              </div>
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
        <!-- Product Sections -->
    <section id="productSections" class="space-y-24">
      <!-- Best Sellers Section -->
      <div class="scroll-animate opacity-0 translate-y-8 rounded-2xl bg-white shadow-md px-6 py-10 md:px-10">
        <div class="flex items-center justify-between mb-10">
          <h2 class="text-3xl font-semibold text-gray-800 tracking-tight">Our Best Sellers</h2>
          <Button class="p-button-sm p-button-text" label="Show All" @click="showAllRecommended" />
        </div>
        <ProductList
          :products="recommended"
          title="Our Best Sellers"
          :loading="loadingRecommended"
        />
      </div>

      <!-- Hot Deals Section -->
      <div class="scroll-animate opacity-0 translate-y-8 rounded-2xl bg-gradient-to-tr from-orange-50 to-white shadow-md px-6 py-10 md:px-10">
        <div class="flex items-center justify-between mb-10">
          <h2 class="text-3xl font-semibold text-gray-800 tracking-tight">Hot Deals</h2>
          <Button class="p-button-sm p-button-text" label="Show All" @click="showAllOnSale" />
        </div>
        <ProductList
          :products="discounted"
          title="Hot Deals"
          :loading="loadingDiscounted"
        />
      </div>

      <!-- New Arrivals Section -->
      <div v-if="newProducts" class="scroll-animate opacity-0 translate-y-8 rounded-2xl bg-white shadow-md px-6 py-10 md:px-10">
        <div class="flex items-center justify-between mb-10">
          <h2 class="text-3xl font-semibold text-gray-800 tracking-tight">New Arrivals</h2>
          <Button class="p-button-sm p-button-text" label="Show All" @click="showAllNewArrivals" />
        </div>
        <ProductList
          :products="newProducts"
          title="New Arrivals"
          :loading="loadingNewProducts"
        />
      </div>
    </section>


    <!-- Footer -->
    <footer class="mt-24 bg-gradient-to-br from-gray-900 via-gray-800 to-gray-900 text-white shadow-inner border-t border-gray-700">
      <div class="max-w-screen-xl mx-auto px-6 py-16 grid grid-cols-1 md:grid-cols-3 gap-8">
        <!-- Contact Us Section -->
        <div>
          <h3 class="text-xl font-semibold mb-4">Contact Us</h3>
          <p class="mb-2">Phone: <a href="tel:+2160675986988" class="underline hover:text-orange-500 transition">0675 986 988</a></p>
          <div class="flex gap-2">
          <p class="mb-4">Follow us on Instagram:</p>
          <a href="https://instagram.com/zi__punisher/" target="_blank" rel="noopener" aria-label="Instagram" class="inline-flex items-center hover:text-orange-500 transition">
            <svg class="w-6 h-6 mr-2" fill="currentColor" viewBox="0 0 24 24">
              <rect x="2" y="2" width="20" height="20" rx="6" stroke="currentColor" stroke-width="2" fill="none"/>
              <circle cx="12" cy="12" r="5" stroke="currentColor" stroke-width="2" fill="none"/>
              <circle cx="17" cy="7" r="1.2" fill="currentColor"/>
            </svg>
          </a>
          </div>
        </div>

        <!-- Quick Links Section -->
        <div>
          <h3 class="text-xl font-semibold mb-4">Quick Links</h3>
          <ul class="space-y-2">
            <li><router-link to="/" class="hover:text-orange-500 transition">Home</router-link></li>
            <li><router-link to="/products" class="hover:text-orange-500 transition">Shop</router-link></li>
          </ul>
        </div>
      </div>
      <div class="border-t border-gray-700 mt-8 pt-4 text-center text-gray-400 text-xs">
        &copy; {{ new Date().getFullYear() }} Ecom Shoes. All rights reserved.
      </div>
    </footer>
  </div>
</template>

<style scoped>
.no-scrollbar::-webkit-scrollbar { display: none; }
.no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
.fade-in-up { opacity: 1 !important; transform: translateY(0) !important; transition: all 0.6s ease-out; }
.scroll-animate { opacity: 0; transform: translateY(30px); }
</style>
