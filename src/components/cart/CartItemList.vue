<template>
  <div>
    <h3 class="text-subtitle-1 font-weight-bold mb-2">Cart Items</h3>
    <cart-item 
      v-for="(itemId, index) in items" 
      :key="index" 
      :item-id="itemId"
      :index="index"
      :item-details="itemDetails[itemId]"
      :read-only="readOnly"
      @remove-item="$emit('remove-item', itemId)"
      @configure-datacenter="(datacenterCode) => $emit('configure-datacenter', { itemId, datacenterCode })"
      @configure-os="(osCode) => $emit('configure-os', { itemId, osCode })"
      @update-configuration="(label, value, configId) => $emit('update-configuration', { itemId, label, value, configId })"
      @delete-configuration="(configId) => $emit('delete-configuration', { itemId, configId })"
      @fetch-available-configurations="() => $emit('fetch-available-configurations', itemId)"
    />
  </div>
</template>

<script setup>
import CartItem from './CartItem.vue';

defineProps({
  items: {
    type: Array,
    required: true
  },
  itemDetails: {
    type: Object,
    default: () => ({})
  },
  readOnly: {
    type: Boolean,
    default: false
  }
});

defineEmits([
  'remove-item', 
  'configure-datacenter',
  'configure-os',
  'update-configuration',
  'delete-configuration',
  'fetch-available-configurations'
]);
</script>
