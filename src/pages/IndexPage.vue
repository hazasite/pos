<template>
  <q-page class="q-pa-md bg-black text-white overflow-hidden full-height-page">
    <div class="row full-height q-col-gutter-lg">
      <!-- LEFT SECTION: Product Grid (70%) -->
      <div class="col-12 col-md-8 column">
        <!-- Search Bar -->
        <div class="q-mb-md">
          <q-input
            v-model="search"
            dark
            filled
            placeholder="Search Products..."
            class="neon-input text-h6 rounded-borders"
            input-class="text-white"
            debounce="300"
          >
            <template v-slot:prepend>
              <q-icon name="search" color="grey-5" />
            </template>
          </q-input>
        </div>

        <!-- Product Grid -->
        <div class="col scroll q-pr-sm custom-scrollbar">
          <div class="row q-col-gutter-md">
            <!-- Product Card -->
            <div
              v-for="product in filteredProducts"
              :key="product.id"
              class="col-12 col-sm-6 col-md-4 col-lg-3"
            >
              <q-card
                class="glass-card product-card cursor-pointer column no-wrap"
                @click="addToCart(product)"
              >
                <!-- Image Placeholder (or actual image if available) -->
                <div class="product-image-placeholder relative-position flex flex-center">
                  <q-icon :name="product.icon || 'inventory_2'" size="4rem" color="grey-8" />
                  <div class="absolute-top-right q-pa-sm">
                    <q-badge
                      :color="product.stock > 10 ? 'green' : 'orange'"
                      floating
                      rounded
                      transparent
                    >
                      {{ product.stock }}
                    </q-badge>
                  </div>
                </div>

                <q-card-section class="q-pt-sm">
                  <div class="text-h6 text-overflow-ellipses">{{ product.name }}</div>
                  <div class="text-subtitle2 text-neon-red">${{ product.price.toFixed(2) }}</div>
                </q-card-section>
              </q-card>
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT SECTION: Invoice Panel (30%) -->
      <div class="col-12 col-md-4 column">
        <q-card class="glass-card col column no-border-radius full-height relative-position">
          <!-- Invoice Header -->
          <q-card-section class="bg-glass-header q-py-md">
            <div class="text-h5 text-weight-bold letter-spacing-1">CURRENT INVOICE</div>
            <div class="text-caption text-grey-5">Transaction #{{ transactionId }}</div>
          </q-card-section>

          <!-- Cart Items List -->
          <q-card-section class="col scroll custom-scrollbar q-pa-none">
            <q-list dark separator class="q-pa-sm">
              <transition-group name="list">
                <q-item
                  v-for="(item, index) in cart"
                  :key="item.id"
                  class="cart-item q-my-xs rounded-borders"
                >
                  <q-item-section avatar>
                    <q-avatar size="md" color="grey-9" text-color="white" font-size="14px">
                      {{ item.qty }}x
                    </q-avatar>
                  </q-item-section>

                  <q-item-section>
                    <q-item-label class="text-weight-medium">{{ item.name }}</q-item-label>
                    <q-item-label caption class="text-grey-5"
                      >${{ item.price.toFixed(2) }} / unit</q-item-label
                    >
                  </q-item-section>

                  <q-item-section side>
                    <div class="text-neon-red text-weight-bold">
                      ${{ (item.price * item.qty).toFixed(2) }}
                    </div>
                  </q-item-section>

                  <q-item-section side>
                    <q-btn
                      flat
                      round
                      dense
                      icon="close"
                      size="sm"
                      color="grey-6"
                      class="hover-red"
                      @click.stop="removeFromCart(index)"
                    />
                  </q-item-section>
                </q-item>
              </transition-group>

              <div
                v-if="cart.length === 0"
                class="full-height flex flex-center text-grey-7 q-pa-xl column"
              >
                <q-icon name="shopping_cart_checkout" size="4rem" class="q-mb-md opacity-50" />
                <div>Cart is empty</div>
              </div>
            </q-list>
          </q-card-section>

          <!-- Invoice Footer / Checkout -->
          <q-card-section class="bg-glass-footer q-pa-md">
            <div class="row items-center justify-between q-mb-md">
              <div class="text-h6 text-grey-4">Subtotal</div>
              <div class="text-h6">${{ cartTotal.toFixed(2) }}</div>
            </div>
            <div class="row items-center justify-between q-mb-lg">
              <div class="text-h6 text-grey-4">Tax (5%)</div>
              <div class="text-h6 text-grey-5">${{ (cartTotal * 0.05).toFixed(2) }}</div>
            </div>

            <q-separator dark class="q-mb-md" />

            <div class="row items-center justify-between q-mb-lg">
              <div class="text-h4 text-weight-bold text-white">TOTAL</div>
              <div class="text-h3 text-neon-red text-weight-bold glow-text">
                ${{ (cartTotal * 1.05).toFixed(2) }}
              </div>
            </div>

            <q-btn
              class="full-width text-h6 shadow-neon-red"
              color="neon-red"
              label="CHECKOUT"
              unelevated
              size="lg"
              :disable="cart.length === 0"
              @click="checkout"
              style="border-radius: 8px; font-weight: 800; letter-spacing: 2px"
            />
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

// --- State ---
const search = ref('')
const transactionId = ref(Math.floor(100000 + Math.random() * 900000))

// Mock Products
const allProducts = [
  { id: 1, name: 'Quantum Processor', price: 299.99, stock: 45, icon: 'memory' },
  { id: 2, name: 'Neural Link Interface', price: 1250.0, stock: 5, icon: 'developer_board' },
  { id: 3, name: 'Holographic Display', price: 899.5, stock: 12, icon: 'desktop_windows' },
  { id: 4, name: 'Plasma Battery Cell', price: 49.99, stock: 156, icon: 'battery_charging_full' },
  { id: 5, name: 'Stealth Drone Chassis', price: 450.0, stock: 8, icon: 'flight' },
  { id: 6, name: 'Cybernetic Arm v4', price: 3200.0, stock: 2, icon: 'precision_manufacturing' },
  { id: 7, name: 'Nano-Fiber Cable', price: 15.0, stock: 500, icon: 'cable' },
  { id: 8, name: 'Optical Sensor', price: 85.0, stock: 34, icon: 'visibility' },
  { id: 9, name: 'Dark Matter Containment', price: 9999.99, stock: 1, icon: 'science' },
  { id: 10, name: 'Fusion Core', price: 560.0, stock: 10, icon: 'blur_on' },
  { id: 11, name: 'Cryo-Coolant', price: 25.5, stock: 88, icon: 'ac_unit' },
  { id: 12, name: 'Gravity Stabilizer', price: 340.0, stock: 15, icon: 'anchor' },
]

const cart = ref([])

// --- Computed ---
const filteredProducts = computed(() => {
  if (!search.value) return allProducts
  const term = search.value.toLowerCase()
  return allProducts.filter((p) => p.name.toLowerCase().includes(term))
})

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + item.price * item.qty, 0)
})

// --- Methods ---
const addToCart = (product) => {
  const existingItem = cart.value.find((item) => item.id === product.id)
  if (existingItem) {
    existingItem.qty++
  } else {
    cart.value.push({ ...product, qty: 1 })
  }
}

const removeFromCart = (index) => {
  cart.value.splice(index, 1)
}

const checkout = () => {
  $q.notify({
    type: 'positive',
    message: 'Transaction Completed Successfully!',
    position: 'top',
    timeout: 2000,
  })
  cart.value = []
  transactionId.value = Math.floor(100000 + Math.random() * 900000)
}
</script>

<style lang="scss" scoped>
// Variables
$neon-red: #ff3131;
$glass-bg: rgba(255, 255, 255, 0.03);
$glass-border: 1px solid rgba(255, 255, 255, 0.05);

.full-height-page {
  height: calc(100vh - 50px); // Adjust based on header height
}

.full-height {
  height: 100%;
}

// Glassmorphism
.glass-card {
  background: $glass-bg;
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: $glass-border;
  transition: all 0.3s ease;
}

// Product Card Specifics
.product-card {
  border-radius: 12px;
  overflow: hidden;

  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 0 20px rgba($neon-red, 0.3);
    border-color: rgba($neon-red, 0.5);

    .text-neon-red {
      text-shadow: 0 0 8px $neon-red;
    }
  }
}

.product-image-placeholder {
  height: 140px;
  background: rgba(255, 255, 255, 0.02);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

// Neon Search Input
.neon-input {
  :deep(.q-field__control) {
    background: rgba(255, 255, 255, 0.05) !important;
    border: 1px solid transparent;
    transition: all 0.3s ease;
    border-radius: 8px;
  }

  :deep(.q-field__control:hover),
  :deep(.q-field__control.q-field--focused) {
    background: rgba(0, 0, 0, 0.8) !important;
    border-color: $neon-red;
    box-shadow: 0 0 15px rgba($neon-red, 0.2);
  }
}

// Invoice Panel
.bg-glass-header {
  background: rgba(255, 255, 255, 0.02);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.bg-glass-footer {
  background: rgba(20, 20, 20, 0.6); // Slightly darker for contrast
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.cart-item {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid transparent;
  transition: all 0.2s;

  &:hover {
    background: rgba(255, 255, 255, 0.05);
    border-color: rgba(255, 255, 255, 0.1);
  }
}

// Utilities
.text-neon-red {
  color: $neon-red !important;
}

.bg-neon-red {
  background-color: $neon-red !important;
}

.shadow-neon-red {
  box-shadow: 0 0 20px rgba($neon-red, 0.4);
  transition: box-shadow 0.3s;

  &:hover {
    box-shadow: 0 0 30px rgba($neon-red, 0.6);
  }
}

.glow-text {
  text-shadow: 0 0 10px rgba($neon-red, 0.5);
}

.hover-red:hover {
  color: $neon-red !important;
}

.opacity-50 {
  opacity: 0.5;
}

.text-overflow-ellipses {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

// Scrollbar
.custom-scrollbar {
  &::-webkit-scrollbar {
    width: 6px;
  }
  &::-webkit-scrollbar-track {
    background: rgba(0, 0, 0, 0.1);
  }
  &::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 4px;
    &:hover {
      background: rgba(255, 255, 255, 0.2);
    }
  }
}

// List Transitions
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(20px);
}
</style>
