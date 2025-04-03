<template>
  <div class="mb-6">
    <v-card variant="outlined" class="mb-4">
      <v-card-title>
        <div class="d-flex align-center">
          <span>Active Cart</span>
          <v-chip
            :color="cartDetails.readOnly ? 'error' : 'success'"
            size="small"
            class="ml-2"
          >
            {{ cartDetails.readOnly ? 'Read Only' : 'Active' }}
          </v-chip>
        </div>
      </v-card-title>
      <v-card-text>
        <v-row>
          <v-col cols="12" sm="6">
            <v-list-item>
              <template v-slot:prepend>
                <v-icon icon="mdi-cart-outline" color="primary"></v-icon>
              </template>
              <v-list-item-title>Cart ID</v-list-item-title>
              <v-list-item-subtitle>{{ cartDetails.cartId }}</v-list-item-subtitle>
            </v-list-item>
          </v-col>
          <v-col cols="12" sm="6">
            <v-list-item>
              <template v-slot:prepend>
                <v-icon icon="mdi-text-box-outline" color="primary"></v-icon>
              </template>
              <v-list-item-title>Description</v-list-item-title>
              <v-list-item-subtitle>{{ cartDetails.description || 'No description' }}</v-list-item-subtitle>
            </v-list-item>
          </v-col>
          <v-col cols="12" sm="6">
            <v-list-item>
              <template v-slot:prepend>
                <v-icon icon="mdi-calendar-clock" color="primary"></v-icon>
              </template>
              <v-list-item-title>Expiration</v-list-item-title>
              <v-list-item-subtitle>{{ formatDate(cartDetails.expire) }}</v-list-item-subtitle>
            </v-list-item>
          </v-col>
          <v-col cols="12" sm="6">
            <v-list-item>
              <template v-slot:prepend>
                <v-icon icon="mdi-package-variant-closed" color="primary"></v-icon>
              </template>
              <v-list-item-title>Items</v-list-item-title>
              <v-list-item-subtitle>{{ cartDetails.items?.length || 0 }} items</v-list-item-subtitle>
            </v-list-item>
          </v-col>
        </v-row>
        
        <!-- Cart Items Section -->
        <v-divider class="my-4"></v-divider>
        <cart-item-list 
          v-if="cartDetails && cartDetails.items && cartDetails.items.length > 0"
          :items="cartDetails.items"
          :item-details="cartItemDetails"
          :read-only="readOnly"
          @remove-item="$emit('remove-item', $event)"
          @configure-datacenter="$emit('configure-datacenter', $event.itemId, $event.datacenterCode)"
          @configure-os="$emit('configure-os', $event.itemId, $event.osCode)"
          @update-configuration="$emit('update-configuration', $event.itemId, $event.label, $event.value, $event.configId)"
          @delete-configuration="$emit('delete-configuration', $event.itemId, $event.configId)"
          @fetch-available-configurations="$emit('fetch-available-configurations', $event)"
        />
        <div v-else>
          <v-alert type="info" text="No items in this cart" class="mt-2"></v-alert>
        </div>
      </v-card-text>
      <v-card-actions>
        <v-btn
          color="primary"
          variant="text"
          prepend-icon="mdi-refresh"
          @click="$emit('refresh')"
          :disabled="loading"
        >
          Refresh
        </v-btn>
        <v-spacer></v-spacer>
        <v-btn
          v-if="!readOnly && cartDetails.items && cartDetails.items.length > 0"
          color="success"
          variant="elevated"
          prepend-icon="mdi-cart-check"
          @click="$emit('checkout')"
          :disabled="loading"
        >
          Checkout
        </v-btn>
        <v-btn
          color="error"
          variant="outlined"
          prepend-icon="mdi-cart-remove"
          @click="$emit('clear-cart')"
          :disabled="loading"
          class="ml-2"
        >
          Clear
        </v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import CartItemList from './CartItemList.vue';

const props = defineProps({
  cartDetails: {
    type: Object,
    required: true
  },
  cartItemDetails: {
    type: Object,
    default: () => ({})
  },
  readOnly: {
    type: Boolean,
    default: false
  },
  loading: {
    type: Boolean,
    default: false
  }
});

defineEmits([
  'refresh', 
  'checkout', 
  'clear-cart',
  'remove-item',
  'configure-datacenter',
  'configure-os',
  'update-configuration',
  'delete-configuration',
  'fetch-available-configurations'
]);

// Format date helper function
const formatDate = (dateString) => {
  if (!dateString) return 'Not set';
  const date = new Date(dateString);
  return date.toLocaleString();
};
</script>
