<template>
  <div>
    <cart-header 
      @create-cart="createCart" 
      :creating-cart="creatingCart" 
      :api-token="apiToken" 
    />
    
    <v-card-text>
      <div v-if="loading">
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
        <span class="ml-2">Loading cart data...</span>
      </div>
      
      <v-alert
        v-else-if="error"
        type="error"
        title="Error"
        :text="error.message || 'Failed to load cart data'"
      ></v-alert>
      
      <v-alert
        v-if="!activeCartId && !loading && cartItems.length > 0"
        type="warning"
        text="No active cart selected. Please select one from below."
        class="mb-4"
      ></v-alert>
      
      <!-- Active Cart Section -->
      <active-cart-display 
        v-if="activeCartId && activeCartDetails" 
        :cart-details="activeCartDetails"
        :cart-item-details="cartItemDetails"
        :read-only="activeCartDetails.readOnly" 
        @refresh="fetchActiveCartDetails"
        @checkout="checkoutCart"
        @clear-cart="clearActiveCart"
      />
      
      <!-- Available Carts -->
      <available-carts
        :cart-items="cartItems"
        :active-cart-id="activeCartId"
        :loading="loading"
        :api-token="apiToken"
        @set-active="setActiveCart"
      />
    </v-card-text>
    
    <v-card-actions>
      <v-btn
        color="primary"
        variant="elevated"
        prepend-icon="mdi-refresh"
        @click="fetchCart"
        :loading="loading"
        :disabled="!apiToken"
      >
        Refresh Carts
      </v-btn>
    </v-card-actions>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import CartHeader from './CartHeader.vue';
import ActiveCartDisplay from './ActiveCartDisplay.vue';
import AvailableCarts from './AvailableCarts.vue';

const props = defineProps({
  apiToken: {
    type: String,
    required: true
  },
  activeCartId: {
    type: String,
    default: ''
  },
  cartItems: {
    type: Array,
    default: () => []
  },
  activeCartDetails: {
    type: Object,
    default: null
  },
  cartItemDetails: {
    type: Object,
    default: () => ({})
  },
  loading: {
    type: Boolean,
    default: false
  },
  creatingCart: {
    type: Boolean,
    default: false
  },
  error: {
    type: Object,
    default: null
  }
});

const emit = defineEmits([
  'create-cart',
  'fetch-cart',
  'set-active-cart',
  'clear-active-cart',
  'fetch-active-cart-details',
  'checkout-cart'
]);

const createCart = () => {
  emit('create-cart');
};

const fetchCart = () => {
  emit('fetch-cart');
};

const setActiveCart = (cartId) => {
  emit('set-active-cart', cartId);
};

const clearActiveCart = () => {
  emit('clear-active-cart');
};

const fetchActiveCartDetails = () => {
  emit('fetch-active-cart-details');
};

const checkoutCart = () => {
  emit('checkout-cart');
};

onMounted(() => {
  if (props.apiToken) {
    fetchCart();
  }
});
</script>
