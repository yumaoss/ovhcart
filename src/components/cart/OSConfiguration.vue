<template>
  <v-col cols="12" class="mt-2">
    <v-card variant="outlined">
      <v-card-title class="text-subtitle-1">
        <v-icon icon="mdi-server-network" color="primary" class="mr-2"></v-icon>
        Operating System Configuration
      </v-card-title>
      <v-card-text>
        <div v-if="loading" class="d-flex align-center">
          <v-progress-circular indeterminate size="16" width="2" color="primary" class="mr-2"></v-progress-circular>
          <span>Loading available operating systems...</span>
        </div>
        
        <div v-else-if="osError" class="text-error">
          <v-alert type="error" text="Failed to load OS configuration options" class="mb-2"></v-alert>
        </div>
        
        <div v-else>
          <div class="d-flex align-center mb-3">
            <div>
              <div class="text-subtitle-2 mb-1">Current OS</div>
              <div class="d-flex align-center">
                <v-icon :icon="getOSIcon(currentOS?.family || '')" class="mr-2"></v-icon>
                <span>{{ currentOS?.displayName || 'Not configured' }}</span>
              </div>
            </div>
            
            <v-spacer></v-spacer>
            
            <v-select
              v-if="!readOnly && availableOS.length > 0"
              v-model="selectedOS"
              :items="availableOS"
              item-title="displayName"
              item-value="code"
              label="Change OS"
              class="max-width-200"
              return-object
              :disabled="loading"
              hide-details
            >
              <template v-slot:item="{ item, props }">
                <v-list-item v-bind="props" :subtitle="item.raw.family">
                  <template v-slot:prepend>
                    <v-icon :icon="getOSIcon(item.raw.family)" class="mr-2"></v-icon>
                  </template>
                </v-list-item>
              </template>
            </v-select>
            
            <v-btn
              v-if="!readOnly && selectedOS && selectedOS.code !== currentOS?.code"
              color="primary"
              variant="text"
              size="small"
              :loading="applying"
              @click="applyOSChange"
              class="ml-2"
            >
              Apply
            </v-btn>
          </div>
        </div>
      </v-card-text>
    </v-card>
  </v-col>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
  itemId: {
    type: String,
    required: true
  },
  itemDetails: {
    type: Object,
    required: true
  },
  readOnly: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['configure-os']);

// State
const loading = ref(false);
const applying = ref(false);
const osError = ref(null);
const selectedOS = ref(null);
const availableOS = ref([]);

// Computed
const currentOS = computed(() => {
  if (!props.itemDetails || !props.itemDetails.configurations) return null;
  
  // Find OS configuration
  const osConfig = props.itemDetails.configurations.find(
    config => config.label === 'os' || config.label === 'image' || config.label.includes('OS')
  );
  
  if (!osConfig) return null;
  
  // Try to find the OS in the available list
  if (availableOS.value.length > 0) {
    const os = availableOS.value.find(os => os.code === osConfig.value);
    if (os) return os;
  }
  
  // If not found in the available list, return a basic representation
  return {
    code: osConfig.value,
    displayName: osConfig.value,
    family: 'unknown'
  };
});

// Watch for current OS changes to update the selected OS
watch(currentOS, (newOS) => {
  if (newOS && !selectedOS.value) {
    selectedOS.value = newOS;
  }
});

// OS icon helper
const getOSIcon = (family) => {
  if (!family) return 'mdi-server';
  
  const lowercaseFamily = family.toLowerCase();
  if (lowercaseFamily.includes('windows')) return 'mdi-microsoft-windows';
  if (lowercaseFamily.includes('linux')) return 'mdi-linux';
  if (lowercaseFamily.includes('ubuntu')) return 'mdi-ubuntu';
  if (lowercaseFamily.includes('debian')) return 'mdi-debian';
  if (lowercaseFamily.includes('centos')) return 'mdi-centos';
  if (lowercaseFamily.includes('fedora')) return 'mdi-fedora';
  if (lowercaseFamily.includes('freebsd')) return 'mdi-freebsd';
  
  return 'mdi-server';
};

// Apply OS change
const applyOSChange = async () => {
  if (!selectedOS.value || selectedOS.value.code === currentOS.value?.code) return;
  
  applying.value = true;
  try {
    // Emit event to parent component to handle the change
    emit('configure-os', selectedOS.value.code);
  } catch (error) {
    console.error('Failed to apply OS change:', error);
    osError.value = error;
  } finally {
    applying.value = false;
  }
};
</script>

<style scoped>
.max-width-200 {
  max-width: 200px;
}
</style>
