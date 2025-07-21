<script setup>
import { ref } from 'vue'
import ProductCard from '../components/ProductCard.vue'

const props = defineProps({
  title: { type: String, required: true },
  products: { type: Array, required: true },
  loading: { type: Boolean, default: false },
})

const scrollRef = ref(null)

const scrollLeft = () => {
  scrollRef.value?.scrollBy({ left: -300, behavior: 'smooth' })
}
const scrollRight = () => {
  scrollRef.value?.scrollBy({ left: 300, behavior: 'smooth' })
}
</script>

<template>
  <div class="relative w-full">
    <!-- Arrows -->
    <button
      @click="scrollLeft"
      class="absolute left-2 top-1/2 -translate-y-1/2 z-20 bg-white shadow-md p-2 rounded-full hover:bg-orange-100 hidden md:inline-flex"
    >
      <i class="pi pi-chevron-left text-xl text-orange-500"></i>
    </button>
    <button
      @click="scrollRight"
      class="absolute right-2 top-1/2 -translate-y-1/2 z-20 bg-white shadow-md p-2 rounded-full hover:bg-orange-100 hidden md:inline-flex"
    >
      <i class="pi pi-chevron-right text-xl text-orange-500"></i>
    </button>

    <!-- Title -->
    <div class="px-4 max-w-screen-xl mx-auto pt-8">
      <h2 class="text-2xl font-bold text-gray-800 mb-4">{{ title }}</h2>
    </div>

    <!-- If loading -->
    <div v-if="loading" class="grid grid-cols-2 md:flex gap-4 px-4 max-w-screen-xl mx-auto pb-6">
      <div
        v-for="n in 4"
        :key="n"
        class="w-full md:w-[210px] flex-shrink-0 animate-pulse space-y-2"
      >
        <div class="w-full h-40 bg-gray-300 rounded-lg"></div>
        <div class="h-3 w-3/4 bg-gray-200 rounded mx-auto"></div>
        <div class="h-3 w-1/2 bg-gray-200 rounded mx-auto"></div>
      </div>
    </div>

    <!-- If no products available -->
    <div
      v-else-if="products.length === 0"
      class="flex flex-col items-center justify-center text-center text-gray-500 py-10"
    >
      <svg
        class="w-16 h-16 text-orange-400 mb-4"
        fill="none"
        stroke="currentColor"
        stroke-width="1.5"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M12 9v3m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
        />
      </svg>
      <p class="text-lg font-medium">No products available</p>
      <p class="text-sm">Check back soon for new arrivals and deals.</p>
    </div>

    <!-- If products exist -->
    <div
      v-else
      ref="scrollRef"
      class="md:flex gap-4 scroll-smooth overflow-x-auto px-4 max-w-screen-xl mx-auto pb-6 touch-pan-x hide-scrollbar"
    >
      <RouterLink
        v-for="(product, index) in products"
        :key="index"
        :to="{
          name: 'productDetail',
          params: { id: product.id },
          state: { product }
        }"
        class="block flex-shrink-0 w-full md:w-[210px]"
      >
        <ProductCard :product="product" />
      </RouterLink>
    </div>
  </div>
</template>

<style scoped>
.animate-pulse {
  animation: pulse 1.5s infinite;
}
@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}
</style>
