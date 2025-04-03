<template>
  <v-card class="mb-4" variant="outlined">
    <v-card-item>
      <v-card-title>
        <div class="d-flex align-center">
          <span>Item #{{ index + 1 }}</span>
          <v-chip
            v-if="itemDetails"
            :color="getItemStatusColor(itemDetails)"
            size="small"
            class="ml-2"
          >
            {{ itemDetails?.status || 'Loading...' }}
          </v-chip>
        </div>
      </v-card-title>
      
      <v-card-text>
        <div v-if="!itemDetails" class="d-flex align-center">
          <v-progress-circular indeterminate size="20" width="2" color="primary" class="mr-2"></v-progress-circular>
          <span>Loading item details...</span>
        </div>
        
        <v-row v-else>
          <v-col cols="12" sm="6">
            <v-card-title>{{ itemDetails?.description || itemDetails?.planCode }}</v-card-title>
            <v-card-subtitle v-if="itemDetails?.description && itemDetails?.planCode">{{ itemDetails?.planCode }}</v-card-subtitle>
            
            <v-chip
              v-if="itemDetails?.settings?.quantity"
              color="primary"
              size="small"
              class="mt-2"
              prepend-icon="mdi-numeric"
            >
              Quantity: {{ itemDetails?.settings?.quantity }}
            </v-chip>
            
            <v-list density="compact" class="mt-2">
              <v-list-item>
                <template v-slot:prepend>
                  <v-icon icon="mdi-card-bulleted-outline" color="primary"></v-icon>
                </template>
                <v-list-item-title>Product</v-list-item-title>
                <v-list-item-subtitle>{{ getProductName(itemDetails?.productId) }}</v-list-item-subtitle>
              </v-list-item>
            </v-list>
          </v-col>
          
          <v-col cols="12" sm="6">
            <v-list density="compact">
              <v-list-item v-if="itemDetails?.prices">
                <template v-slot:prepend>
                  <v-icon icon="mdi-currency-eur" color="primary"></v-icon>
                </template>
                <v-list-item-title>Price</v-list-item-title>
                <v-list-item-subtitle>{{ getItemPrice(itemDetails) }}</v-list-item-subtitle>
              </v-list-item>
              
              <v-list-item>
                <template v-slot:prepend>
                  <v-icon icon="mdi-cog-outline" color="primary"></v-icon>
                </template>
                <v-list-item-title>Settings</v-list-item-title>
                <v-list-item-subtitle>{{ itemDetails?.configurations?.length || 0 }} configurations</v-list-item-subtitle>
              </v-list-item>
            </v-list>
          </v-col>
        </v-row>

        <!-- Item sections -->
        <v-row v-if="itemDetails">
          <!-- Pricing summary section if there are prices -->
          <item-pricing 
            v-if="itemDetails?.prices && itemDetails?.prices.length > 0" 
            :prices="itemDetails.prices" 
          />
          
          <!-- OS configuration section -->
          <os-configuration 
            :item-id="itemId"
            :item-details="itemDetails"
            :read-only="readOnly"
            @configure-os="$emit('configure-os', $event)"
          />
          
          <!-- Configuration details section -->
          <configuration-details
            :item-id="itemId"
            :item-details="itemDetails"
            :read-only="readOnly"
            @update-configuration="$emit('update-configuration', $event.label, $event.value, $event.configId)"
            @delete-configuration="$emit('delete-configuration', $event)"
            @fetch-available-configurations="$emit('fetch-available-configurations')"
          />
        </v-row>
        
        <!-- Action buttons -->
        <v-card-actions v-if="itemDetails && !readOnly">
          <v-btn
            color="error"
            variant="text"
            prepend-icon="mdi-delete"
            @click="$emit('remove-item')"
          >
            Remove from Cart
          </v-btn>
        </v-card-actions>
      </v-card-text>
    </v-card-item>
  </v-card>
</template>

<script setup>
import { ref } from 'vue';
import ItemPricing from './ItemPricing.vue';
import OSConfiguration from './OSConfiguration.vue';
import ConfigurationDetails from './ConfigurationDetails.vue';

const props = defineProps({
  itemId: {
    type: String,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  itemDetails: {
    type: Object,
    default: null
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

// Helper functions
const getItemStatusColor = (item) => {
  if (!item || !item.status) return 'grey';
  switch (item.status.toLowerCase()) {
    case 'delivered': return 'success';
    case 'delivering': return 'info';
    case 'todeliver': return 'info';
    case 'waiting_start': return 'warning';
    case 'error': return 'error';
    default: return 'grey';
  }
};

const getItemPrice = (item) => {
  if (!item || !item.prices || item.prices.length === 0) return 'N/A';
  
  // Find the total price
  const totalPrice = item.prices.find(p => p.label === 'TOTAL');
  if (totalPrice) {
    return `${totalPrice.price.text} ${totalPrice.price.currencySymbol}`;
  }
  
  // If no total, just return the first price
  return `${item.prices[0].price.text} ${item.prices[0].price.currencySymbol}`;
};

const getProductName = (productId) => {
  if (!productId) return 'Unknown';
  
  // Map product IDs to readable names
  const productMap = {
    'dedicated': 'Dedicated Server',
    'cloud': 'Cloud Instance',
    'vps': 'Virtual Private Server',
    'baremetal': 'Baremetal Server',
    'hosting': 'Web Hosting'
  };
  
  // Try to extract a product category from the ID
  for (const [key, value] of Object.entries(productMap)) {
    if (productId.toLowerCase().includes(key)) {
      return value;
    }
  }
  
  // If no match, return a cleaned-up version of the ID
  return productId.replace(/^.*\./, '').replace(/\./g, ' ');
};
</script>
