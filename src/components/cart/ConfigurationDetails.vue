<template>
  <v-col cols="12">
    <v-card variant="outlined" class="mt-2">
      <v-card-title class="text-subtitle-1 d-flex align-center">
        <v-icon icon="mdi-cog" color="primary" class="mr-2"></v-icon>
        Configuration Details
        <v-spacer></v-spacer>
        <v-btn
          v-if="!readOnly"
          color="primary"
          variant="text"
          size="small"
          prepend-icon="mdi-refresh"
          @click="$emit('fetch-available-configurations')"
        >
          Refresh Configs
        </v-btn>
      </v-card-title>
      
      <v-card-text>
        <div v-if="!itemDetails?.configurationDetails || itemDetails?.configurationDetails.length === 0">
          <v-alert
            type="info"
            density="compact"
            text="No additional configuration details available"
            class="mb-2"
          ></v-alert>
        </div>
        
        <div v-else>
          <v-list density="compact">
            <v-list-item v-for="(config, i) in itemDetails.configurations" :key="i">
              <div class="d-flex align-center w-100">
                <div>
                  <v-list-item-title>{{ config.label }}</v-list-item-title>
                  <v-list-item-subtitle>{{ config.value }}</v-list-item-subtitle>
                </div>
                <v-spacer></v-spacer>
                <v-btn
                  v-if="!readOnly"
                  icon="mdi-pencil"
                  size="small"
                  color="primary"
                  variant="text"
                  @click="openEditDialog(config)"
                ></v-btn>
                <v-btn
                  v-if="!readOnly && !isRequiredConfig(config.label)"
                  icon="mdi-delete"
                  size="small"
                  color="error"
                  variant="text"
                  @click="$emit('delete-configuration', config.id)"
                ></v-btn>
              </div>
            </v-list-item>
          </v-list>
        </div>
        
        <!-- Add new configuration -->
        <div class="mt-4" v-if="!readOnly">
          <v-divider class="mb-2"></v-divider>
          <v-row>
            <v-col cols="12">
              <v-select
                v-model="newConfigType"
                :items="configTypes"
                label="Add Configuration"
                item-title="label"
                item-value="type"
                return-object
                hide-details
                class="mb-2"
              >
                <template v-slot:item="{ item, props }">
                  <v-list-item v-bind="props">
                    <template v-slot:prepend>
                      <v-icon :icon="getConfigIcon(item.raw.type)" color="primary"></v-icon>
                    </template>
                  </v-list-item>
                </template>
              </v-select>
            </v-col>
            <v-col cols="12" sm="8">
              <v-text-field
                v-model="newConfigValue"
                :label="newConfigType ? `Enter ${newConfigType.label} value` : 'Value'"
                :disabled="!newConfigType"
                hide-details
              ></v-text-field>
            </v-col>
            <v-col cols="12" sm="4" class="d-flex align-center">
              <v-btn
                color="primary"
                variant="text"
                block
                @click="addNewConfig"
                :disabled="!newConfigType || !newConfigValue"
              >
                Add Configuration
              </v-btn>
            </v-col>
          </v-row>
        </div>
      </v-card-text>
    </v-card>

    <!-- Edit Configuration Dialog -->
    <v-dialog v-model="showEditDialog" max-width="500">
      <v-card>
        <v-card-title>
          Edit Configuration
        </v-card-title>
        <v-card-text>
          <v-text-field
            v-model="editConfig.label"
            label="Configuration Key"
            disabled
          ></v-text-field>
          <v-text-field
            v-model="editConfig.value"
            label="Configuration Value"
            autofocus
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="text" @click="showEditDialog = false">
            Cancel
          </v-btn>
          <v-btn color="primary" @click="saveConfigEdit">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-col>
</template>

<script setup>
import { ref, computed } from 'vue';

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

const emit = defineEmits([
  'update-configuration',
  'delete-configuration',
  'fetch-available-configurations'
]);

// State
const showEditDialog = ref(false);
const editConfig = ref({ label: '', value: '', id: null });
const newConfigType = ref(null);
const newConfigValue = ref('');

// Common configuration types
const configTypes = ref([
  { type: 'hostname', label: 'Hostname' },
  { type: 'domain', label: 'Domain' },
  { type: 'ssh_key', label: 'SSH Key' },
  { type: 'language', label: 'Language' },
  { type: 'custom', label: 'Custom' }
]);

// Helper to check if a config is required and can't be deleted
const isRequiredConfig = (label) => {
  const requiredConfigs = ['os', 'datacenter', 'hostname', 'domain'];
  return requiredConfigs.includes(label.toLowerCase());
};

// Helper to get icon for config types
const getConfigIcon = (type) => {
  const iconMap = {
    'hostname': 'mdi-server',
    'domain': 'mdi-web',
    'ssh_key': 'mdi-key',
    'language': 'mdi-translate',
    'custom': 'mdi-cog'
  };
  return iconMap[type] || 'mdi-cog';
};

// Open edit dialog
const openEditDialog = (config) => {
  editConfig.value = { ...config };
  showEditDialog.value = true;
};

// Save config edit
const saveConfigEdit = () => {
  if (editConfig.value.label && editConfig.value.value) {
    emit('update-configuration', editConfig.value.label, editConfig.value.value, editConfig.value.id);
    showEditDialog.value = false;
  }
};

// Add new config
const addNewConfig = () => {
  if (newConfigType.value && newConfigValue.value) {
    const label = newConfigType.value.type === 'custom' 
      ? prompt('Enter custom configuration name:', '') 
      : newConfigType.value.type;
      
    if (label) {
      emit('update-configuration', label, newConfigValue.value, null);
      newConfigType.value = null;
      newConfigValue.value = '';
    }
  }
};
</script>
