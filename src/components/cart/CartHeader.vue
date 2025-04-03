<template>
  <v-card-title class="text-h5 py-4 d-flex align-center">
    Shopping Cart
    <v-spacer></v-spacer>
    <v-btn
      color="success"
      variant="elevated"
      prepend-icon="mdi-cart-plus"
      @click="$emit('create-cart')"
      :loading="creatingCart"
      :disabled="!apiToken"
    >
      Create New Cart
    </v-btn>
  </v-card-title>
</template>

<script setup>
defineProps({
  creatingCart: {
    type: Boolean,
    default: false
  },
  apiToken: {
    type: String,
    default: ''
  }
});

defineEmits(['create-cart']);
</script>
