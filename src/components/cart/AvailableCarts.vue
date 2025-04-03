<template>
  <div>
    <h3 class="text-subtitle-1 font-weight-bold mb-2">Available Carts</h3>
    <v-alert
      v-if="!cartItems || cartItems.length === 0"
      type="info"
      text="You don't have any carts yet. Create a new cart to start shopping."
      class="mb-2"
    ></v-alert>
    
    <v-table v-else density="compact">
      <thead>
        <tr>
          <th>Cart ID</th>
          <th>Description</th>
          <th>Expiration</th>
          <th>Items</th>
          <th class="text-center">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr 
          v-for="cart in cartItems" 
          :key="cart.cartId"
          :class="cart.cartId === activeCartId ? 'bg-primary bg-opacity-10' : ''"
        >
          <td>
            <div class="d-flex align-center">
              <v-icon
                v-if="cart.cartId === activeCartId"
                icon="mdi-check-circle"
                color="success"
                size="small"
                class="mr-1"
              ></v-icon>
              <span>{{ cart.cartId }}</span>
            </div>
          </td>
          <td>{{ cart.description || 'No description' }}</td>
          <td>{{ formatDate(cart.expire) }}</td>
          <td>{{ cart.itemsCount || 0 }}</td>
          <td class="text-center">
            <v-btn
              v-if="cart.cartId !== activeCartId"
              color="primary"
              variant="text"
              size="small"
              @click="$emit('set-active', cart.cartId)"
              :disabled="!apiToken"
            >
              Set Active
            </v-btn>
            <v-chip
              v-else
              color="success"
              size="small"
            >
              Active
            </v-chip>
          </td>
        </tr>
      </tbody>
    </v-table>
  </div>
</template>

<script setup>
defineProps({
  cartItems: {
    type: Array,
    default: () => []
  },
  activeCartId: {
    type: String,
    default: ''
  },
  loading: {
    type: Boolean,
    default: false
  },
  apiToken: {
    type: String,
    default: ''
  }
});

defineEmits(['set-active']);

// Format date helper function
const formatDate = (dateString) => {
  if (!dateString) return 'Not set';
  const date = new Date(dateString);
  return date.toLocaleString();
};
</script>
