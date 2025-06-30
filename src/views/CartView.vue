<template>
  <section class="relative">
    <!-- Blocking full-page loading screen -->
    <div
      v-if="submitting"
      class="fixed inset-0 bg-white bg-opacity-90 z-50 flex flex-col items-center justify-center space-y-6"
    >
      <!-- Animated delivery truck SVG -->
      <svg class="w-20 h-20 text-orange-500 animate-bounce" fill="none" viewBox="0 0 48 48">
        <rect x="4" y="18" width="28" height="14" rx="2" fill="#FDBA74"/>
        <rect x="32" y="24" width="12" height="8" rx="2" fill="#FDBA74"/>
        <circle cx="12" cy="36" r="4" fill="#FB923C"/>
        <circle cx="36" cy="36" r="4" fill="#FB923C"/>
        <rect x="8" y="22" width="8" height="4" rx="1" fill="#fff" opacity="0.5"/>
        <rect x="36" y="28" width="4" height="2" rx="1" fill="#fff" opacity="0.5"/>
      </svg>
      <!-- Animated sending message -->
      <div class="flex flex-col items-center">
        <p class="text-lg text-gray-700 font-semibold mb-2">Sending your order<span class="loading-dots"></span></p>
        <p class="text-sm text-gray-500">Please wait while we process your request.</p>
      </div>
    </div>

    <!-- Success SVG screen -->
    <div
      v-if="orderSuccess"
      class="fixed inset-0 bg-white bg-opacity-90 z-50 flex flex-col items-center justify-center space-y-6"
    >
      <!-- Success SVG (checkmark) -->
      <svg class="w-20 h-20 text-green-500 animate-bounce" fill="none" viewBox="0 0 48 48">
        <circle cx="24" cy="24" r="22" stroke="#22C55E" stroke-width="4" fill="#DCFCE7"/>
        <path d="M16 24l6 6 10-10" stroke="#22C55E" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
      <div class="flex flex-col items-center">
        <p class="text-lg text-green-700 font-semibold mb-2">Order placed successfully!</p>
        <p class="text-sm text-gray-500">Redirecting to home...</p>
      </div>
    </div>

    <section class="max-w-screen-lg mx-auto px-4 sm:px-6 py-8 space-y-10">
      <ConfirmDialog />

      <!-- Cart List -->
      <div class="space-y-6">
        <h2 class="text-2xl font-bold text-gray-800">Your Cart</h2>

        <div
          v-if="cartStore.items.length > 0"
          v-for="item in cartStore.items"
          :key="item.productId"
          class="flex flex-col sm:flex-row sm:items-center gap-4 border-b pb-4"
        >
          <img :src="item.image" :alt="item.name" class="w-24 h-24 rounded-xl object-cover" />
          <div class="flex-1 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
            <div>
              <h3 class="text-lg font-semibold text-gray-800">
                {{ item.name }}
                <span v-if="item.size" class="ml-2 text-sm text-gray-500 font-normal">| Size: {{ item.size }}</span>
                <span v-if="item.color" class="ml-2 text-sm text-gray-500 font-normal">| Color: {{ item.color }}</span>
              </h3>
              <div class="flex items-center gap-2 mt-1">
                <label class="text-gray-700 font-medium">Qty:</label>
                <input
                  type="number"
                  :value="item.quantity"
                  disabled
                  class="w-16 px-2 py-1 border border-gray-300 rounded-md bg-gray-100 text-gray-700 cursor-not-allowed"
                />
              </div>
            </div>
            <!-- Price details horizontally aligned -->
            <div class="flex flex-row items-center gap-8 min-w-[320px] justify-end">
              <div class="flex flex-col items-center">
                <span class="text-gray-600 text-xs">Unit Price</span>
                <span class="text-orange-500 font-semibold">{{ item.price }} DA</span>
              </div>
              <div class="flex flex-col items-center">
                <span class="text-gray-600 text-xs">Subtotal</span>
                <span class="font-bold text-gray-800">{{ item.price * item.quantity }} DA</span>
              </div>
              <button
                @click="confirmRemove(item.productId)"
                class="text-red-500 hover:text-red-700 ml-4 self-start sm:self-center"
                title="Remove"
              >
                ✕
              </button>
            </div>
          </div>
        </div>

        <!-- Empty Cart Message + Button -->
        <div v-if="cartStore.items.length === 0" class="text-center py-12 space-y-4">
          <p class="text-lg text-gray-700">Your cart is empty.</p>
          <RouterLink
            to="/"
            class="inline-block bg-orange-500 hover:bg-orange-600 text-white font-semibold py-3 px-6 rounded-full transition"
          >
            Add Products
          </RouterLink>
        </div>

        <!-- Cart Summary -->
        <div v-if="cartStore.items.length > 0" class="flex flex-col items-end space-y-1 mt-6">
          <div class="bg-gray-50 rounded-xl p-4 w-full sm:w-96 shadow space-y-2">
            <div class="flex justify-between text-gray-700 text-base">
              <span>Products Total:</span>
              <span class="font-semibold text-orange-500">{{ totalPrice }} DA</span>
            </div>
            <div v-if="selectedState" class="flex justify-between text-gray-700 text-base">
              <span>Delivery ({{ deliveryType === 'home' ? 'Home' : 'Bureau' }}):</span>
              <span class="font-semibold text-orange-500">{{ deliveryFees }} DA</span>
            </div>
            <div v-if="selectedState" class="flex justify-between text-gray-800 text-lg font-bold border-t pt-2">
              <span>Grand Total:</span>
              <span class="text-orange-500">{{ totalPrice + deliveryFees }} DA</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Delivery Form -->
      <div v-if="cartStore.items.length > 0" class="space-y-6">
        <h2 class="text-2xl font-bold text-gray-800">Delivery Information</h2>

        <!-- Full Name -->
        <div>
          <label for="name" class="block text-gray-700 font-medium mb-1">Full Name</label>
          <InputText
            id="name"
            v-model="name"
            :class="{'p-invalid': errors.name}"
            type="text"
            placeholder="Your full name"
            class="w-full sm:w-96"
          />
          <Message v-if="errors.name" severity="error" class="mt-2 animate-fade-in">
            {{ errors.name }}
          </Message>
        </div>

        <!-- Phone Number -->
        <div class="mb-4">
          <label for="phone" class="block text-gray-700 font-medium mb-1">Phone Number</label>
          <InputText
            id="phone"
            v-model="phone"
            :class="{'p-invalid': errors.phone}"
            type="tel"
            placeholder="06XXXXXXXX"
            class="w-full sm:w-96"
          />
          <Message v-if="errors.phone" severity="error" class="mt-2 animate-fade-in">
            {{ errors.phone }}
          </Message>
        </div>

        <div class="flex flex-row gap-4 pt-4 pb-4">
          <div class="flex items-center gap-2">
            <RadioButton inputId="home" name="delivery" value="home" v-model="deliveryType" class="!text-orange-500" />
            <label for="home" class="text-gray-700">Home</label>
          </div>
          <div class="flex items-center gap-2">
            <RadioButton inputId="bureau" name="delivery" value="bureau" v-model="deliveryType" class="!text-orange-500" />
            <label for="bureau" class="text-gray-700">Bureau</label>
          </div>
        </div>

        <div class="flex flex-col sm:flex-row gap-4">
          <!-- Wilaya Dropdown -->
          <Dropdown
            v-model="selectedState"
            :options="wilaya"
            :class="{'p-invalid': errors.selectedState}"
            optionLabel="name"
            placeholder="Select Wilaya"
            class="w-full sm:w-64 mt-4 mb-2"
            filter
          />
          <Message v-if="errors.selectedState" severity="error" class="mt-2 animate-fade-in">
            {{ errors.selectedState }}
          </Message>
          <div v-if="selectedState" class="text-sm text-gray-500 mt-1">
            Delivery price for {{ selectedState.name }} 
            <span class="font-semibold">
              ({{ deliveryType === 'home' ? 'Home' : 'Bureau' }})
            </span>
            : <span class="text-orange-500">{{ deliveryFees }} DA</span>
          </div>
        </div>

        <div class="pt-4">
          <button
            class="w-full sm:w-auto bg-orange-500 hover:bg-orange-600 text-white font-semibold py-3 px-6 rounded-full transition"
            @click="confirmOrder"
          >
            Submit Order
          </button>
        </div>
      </div>
    </section>
  </section>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useCartStore } from '@/stores/cart'
import Dropdown from 'primevue/dropdown'
import RadioButton from 'primevue/radiobutton'
import ConfirmDialog from 'primevue/confirmdialog'
import { useConfirm } from 'primevue/useconfirm'
import { RouterLink, useRouter } from 'vue-router'
import InputText from 'primevue/inputtext'
import axios from 'axios'
import Message from 'primevue/message'
import { yalidinePrices } from '@/assets/delivery/yalidine_prices'


// ✅ Toastify
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'

const cartStore = useCartStore()
const confirm = useConfirm()
const router = useRouter()

const name = ref('')
const phone = ref('')
const deliveryType = ref('home')
const selectedState = ref(null)
const submitting = ref(false)
const orderSuccess = ref(false) // <-- Add this

const errors = ref({
  name: '',
  phone: '',
  selectedState: '',
})

const wilaya = Object.entries(yalidinePrices).map(([name, data]) => ({
  name, 
  domicile: data.domicile,
  bureau: data.bureau,
  id: data.code
}))


const totalPrice = computed(() => cartStore.totalPrice)

function confirmRemove(productId) {
  confirm.require({
    message: 'Remove this item from your cart?',
    header: 'Please Confirm',
    icon: 'pi pi-exclamation-triangle',
    accept: () => {
      cartStore.removeItem(productId)
      toast.info('Item removed from cart')
    }
  })
}

function confirmOrder() {
  if (!validateForm()) {
    return
  }
  confirm.require({
    message: 'Are you sure you want to submit your order?',
    header: 'Confirm Order',
    icon: 'pi pi-check',
    accept: submitOrder
  })
}

function validateForm() {
  let valid = true
  errors.value = { name: '', phone: '', selectedState: '', selectedCommune: '' }

  if (!name.value.trim()) {
    errors.value.name = 'Full name is required'
    valid = false
  }
  if (!phone.value.trim()) {
    errors.value.phone = 'Phone number is required'
    valid = false
  }
  if (!selectedState.value) {
    errors.value.selectedState = 'Wilaya is required'
    valid = false
  }
  return valid
}

const deliveryFees = computed(() => {
        if (!selectedState.value) return 0
        return deliveryType.value === 'home'
          ? selectedState.value.domicile
          : selectedState.value.bureau
      })

async function submitOrder() {
  submitting.value = true
  try {
    const payload = {
      costumer_name: name.value,
      costumer_phone: phone.value,
      delivery_type: deliveryType.value === 'home' ? 'A Domicile' : 'Bureau',
      delivery_fees: deliveryFees.value,
      wilaya: selectedState.value?.name,
      items: cartStore.items.map(i => ({
        product: i.productId, // No variant, just productId
        quantity: i.quantity
      }))
    }

    const response = await axios.post('https://ecom-1qve.onrender.com/api/orders/create', payload)
    toast.success('Your order was placed successfully!')
    submitting.value = false
    orderSuccess.value = true // <-- Show success SVG

    // Wait for the SVG, then redirect
    setTimeout(() => {
      router.push('/')
      setTimeout(() => {
        cartStore.clearCart()
        orderSuccess.value = false // Hide SVG after redirect
      }, 500)
    }, 1500) // Show SVG for 1.5s
  } catch (err) {
    console.error(err);

    // If backend returns a string message, show it
    if (err.response?.data?.message) {
      toast.error(err.response.data.message);
      return;
    }

    // If backend returns an object with field errors
    if (err.response?.status === 400 && err.response.data) {
      const errors = err.response.data;
      for (const key in errors) {
        // Custom handling for items field
        if (key === 'items') {
          toast.error('One or more products are out of stock or unavailable.');
          continue;
        }
        const messages = errors[key];
        if (Array.isArray(messages)) {
          messages.forEach(msg => {
            toast.error(`${key.charAt(0).toUpperCase() + key.slice(1)}: ${msg}`);
          });
        } else if (typeof messages === 'string') {
          toast.error(`${key.charAt(0).toUpperCase() + key.slice(1)}: ${messages}`);
        } else {
          toast.error(`${key}: ${JSON.stringify(messages)}`);
        }
      }
    } else {
      toast.error('Something went wrong while submitting your order.');
    }

  } finally {
    submitting.value = false
  }
}
</script>

<style scoped>
:deep(.p-focus) {
  border-color: #f97316 !important;
  box-shadow: 0 0 0 0.2rem rgba(251, 146, 60, 0.3) !important;
}
:deep(.p-radiobutton-box.p-highlight) {
  border-color: #f97316 !important;
  background-color: #f97316 !important;
}
:deep(.p-radiobutton-icon) {
  background-color: white !important;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px);}
  to { opacity: 1; transform: none;}
}
.animate-fade-in {
  animation: fade-in 0.3s;
}

@keyframes dots {
  0%, 20% { content: ""; }
  40% { content: "."; }
  60% { content: ".."; }
  80%, 100% { content: "..."; }
}
.loading-dots::after {
  display: inline-block;
  animation: dots 1.2s steps(4, end) infinite;
  content: "";
  font-weight: bold;
}
</style>
