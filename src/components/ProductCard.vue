<template>
  <div
    class="bg-white rounded-2xl shadow-md hover:shadow-lg transition-shadow overflow-hidden w-full max-w-xs border border-gray-100 hover:border-orange-400 flex flex-col h-76 relative"
  >
    <!-- Product Image -->
    <div
      class="relative w-full flex-1 min-h-[200px] sm:min-h-[220px] md:min-h-[240px] overflow-hidden group flex items-center justify-center"
    >
      <img
        :src="getOptimizedImage(product.image)"
        :alt="product.name"
        loading="lazy"
        class="w-full h-[80%] object-contain group-hover:scale-105 transition-transform duration-300 max-w-full max-h-full"
        :class="{ 'opacity-60': product.stock === 0 }"
      />

      <!-- "Out of Stock" Badge -->
      <span
        v-if="product.stock === 0"
        class="absolute top-2 right-2 bg-red-600 text-white text-xs font-semibold px-2 py-1 rounded-full shadow-sm"
      >
        Out of Stock
      </span>

      <!-- "New" Badge -->
      <span
        v-else-if="product.isNew"
        class="absolute top-2 left-2 bg-lime-500 border-2 border-white text-white text-xs font-bold px-3 py-1 rounded-full shadow z-20"
      >
        New
      </span>

      <!-- "Sale" Badge -->
      <span
        v-else-if="product.discount_price"
        class="absolute top-2 right-2 bg-orange-500 text-white text-xs font-semibold px-2 py-1 rounded-full shadow-sm z-10"
      >
        Sale
      </span>
    </div>

    <!-- Product Details -->
    <div class="p-4 flex-1 flex flex-col">
      <h3 class="text-sm font-semibold text-gray-800 line-clamp-2">
        {{ product.name }}
      </h3>

      <!-- Price -->
      <div class="mt-auto pt-2 space-x-2">
        <span class="text-base font-bold text-orange-500">
          {{ displayPrice(product).toLocaleString() }} DA
        </span>
        <span
          v-if="product.discount_price"
          class="text-sm text-gray-400 line-through"
        >
          {{ product.price.toLocaleString() }} DA
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  product: {
    type: Object,
    required: true,
    default: () => ({
      image: 'https://via.placeholder.com/300x300',
      name: 'Product Name',
      price: 999,
      discount_price: null,
      isNew: false,
      stock: 0,
    }),
  },
})

function getOptimizedImage(url) {
  if (!url.includes('res.cloudinary.com')) return url

  const parts = url.split('/upload/')
  return `${parts[0]}/upload/f_auto,q_auto,w_600,h_400,c_fit/${parts[1]}`
}


function displayPrice(product) {
  return product.discount_price ?? product.price
}
</script>
