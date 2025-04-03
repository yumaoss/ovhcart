<template>
  <v-container>
    <v-card>
      <v-tabs
        v-model="activeTab"
        bg-color="primary"
      >
        <v-tab
          v-for="tab in tabs"
          :key="tab.id"
          :value="tab.id"
        >
          <v-icon :icon="tab.icon" class="mr-2"></v-icon>
          {{ tab.name }}
        </v-tab>
      </v-tabs>

      <v-card-text>
        <v-window v-model="activeTab">
          <!-- Cart Tab -->
          <v-window-item value="cart">
            <v-card-title class="text-h5 py-4 d-flex align-center">
              Shopping Cart
              <v-spacer></v-spacer>
              <v-btn
                color="success"
                variant="elevated"
                prepend-icon="mdi-cart-plus"
                @click="createCart"
                :loading="creatingCart"
                :disabled="!apiToken"
              >
                Create New Cart
              </v-btn>
            </v-card-title>
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
              <div v-if="activeCartId && activeCartDetails" class="mb-6">
                <v-card variant="outlined" class="mb-4">
                  <v-card-title>
                    <div class="d-flex align-center">
                      <span>Active Cart</span>
                      <v-chip
                        :color="activeCartDetails.readOnly ? 'error' : 'success'"
                        size="small"
                        class="ml-2"
                      >
                        {{ activeCartDetails.readOnly ? 'Read Only' : 'Active' }}
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
                          <v-list-item-subtitle>{{ activeCartDetails.cartId }}</v-list-item-subtitle>
                        </v-list-item>
                      </v-col>
                      <v-col cols="12" sm="6">
                        <v-list-item>
                          <template v-slot:prepend>
                            <v-icon icon="mdi-text-box-outline" color="primary"></v-icon>
                          </template>
                          <v-list-item-title>Description</v-list-item-title>
                          <v-list-item-subtitle>{{ activeCartDetails.description || 'No description' }}</v-list-item-subtitle>
                        </v-list-item>
                      </v-col>
                      <v-col cols="12" sm="6">
                        <v-list-item>
                          <template v-slot:prepend>
                            <v-icon icon="mdi-calendar-clock" color="primary"></v-icon>
                          </template>
                          <v-list-item-title>Expiration</v-list-item-title>
                          <v-list-item-subtitle>{{ formatDate(activeCartDetails.expire) }}</v-list-item-subtitle>
                        </v-list-item>
                      </v-col>
                      <v-col cols="12" sm="6">
                        <v-list-item>
                          <template v-slot:prepend>
                            <v-icon icon="mdi-package-variant-closed" color="primary"></v-icon>
                          </template>
                          <v-list-item-title>Items</v-list-item-title>
                          <v-list-item-subtitle>{{ activeCartDetails.items?.length || 0 }} items</v-list-item-subtitle>
                        </v-list-item>
                      </v-col>
                    </v-row>
                    
                    <!-- Cart Items Section -->
                    <v-divider class="my-4"></v-divider>
                    <div v-if="activeCartDetails.items && activeCartDetails.items.length > 0">
                      <h3 class="text-subtitle-1 font-weight-bold mb-2">Cart Items</h3>
                      <v-list>
                        <v-list-item v-for="(item, index) in activeCartDetails.items" :key="index">
                          <v-list-item-title>Item #{{ index + 1 }}</v-list-item-title>
                          <v-list-item-subtitle>{{ item }}</v-list-item-subtitle>
                        </v-list-item>
                      </v-list>
                    </div>
                    <div v-else>
                      <v-alert type="info" text="No items in this cart" class="mt-2"></v-alert>
                    </div>
                  </v-card-text>
                  <v-card-actions>
                    <v-btn
                      color="primary"
                      variant="text"
                      prepend-icon="mdi-refresh"
                      @click="fetchActiveCartDetails"
                      :loading="loadingActiveCart"
                    >
                      Refresh Cart Details
                    </v-btn>
                    <v-spacer></v-spacer>
                    <v-btn
                      color="error"
                      variant="text"
                      prepend-icon="mdi-close-circle"
                      @click="clearActiveCart"
                      :disabled="activeCartDetails.readOnly"
                    >
                      Clear Active Cart
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </div>
              
              <!-- Available Carts -->
              <h3 class="text-subtitle-1 font-weight-bold mb-2">Available Carts</h3>
              <v-alert
                v-if="cartItems.length === 0 && !loading"
                type="info"
                text="Your cart is empty"
                class="mb-4"
              ></v-alert>
              <v-list v-else lines="three">
                <v-list-item
                  v-for="(item, index) in cartItems"
                  :key="index"
                  :title="item.description || 'Cart Item'"
                  :subtitle="'ID: ' + item.cartId"
                  :active="activeCartId === item.cartId"
                  @click="setActiveCart(item.cartId)"
                >
                  <template v-slot:prepend>
                    <v-chip
                      :color="item.readOnly ? 'error' : 'success'"
                      size="small"
                      class="text-caption"
                    >
                      {{ item.readOnly ? 'Read Only' : 'Active' }}
                    </v-chip>
                    <v-icon 
                      v-if="activeCartId === item.cartId"
                      icon="mdi-check-circle"
                      color="primary"
                      class="ml-2"
                    ></v-icon>
                  </template>
                  <template v-slot:append>
                    <div class="d-flex align-center">
                      <span class="text-caption text-grey me-2">
                        Expires: {{ formatDate(item.expire) }}
                      </span>
                      <v-btn
                        icon="mdi-delete"
                        variant="text"
                        color="error"
                        density="compact"
                        @click="removeFromCart(item)"
                        :disabled="item.readOnly"
                      ></v-btn>
                    </div>
                  </template>
                </v-list-item>
              </v-list>
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
                Refresh Cart
              </v-btn>
            </v-card-actions>
          </v-window-item>

          <!-- Servers Tab -->
          <v-window-item value="servers">
            <v-card-title class="text-h5 py-4 d-flex align-center">
              Server Plans
              <v-chip 
                class="ml-2" 
                color="primary" 
                size="small"
              >
                Subsidiary: {{ selectedSite?.code || 'IE' }}
              </v-chip>
              <v-spacer></v-spacer>
              <v-btn
                color="primary"
                variant="elevated"
                prepend-icon="mdi-refresh"
                @click="fetchServers"
                :loading="loadingServers"
                :disabled="!apiToken"
              >
                Refresh Servers
              </v-btn>
            </v-card-title>
            <v-card-text>
              <div v-if="loadingServers">
                <v-progress-circular indeterminate color="primary"></v-progress-circular>
                <span class="ml-2">Loading server catalog...</span>
              </div>
              
              <v-alert
                v-else-if="serverError"
                type="error"
                title="Error"
                :text="serverError.message || 'Failed to load server catalog'"
              ></v-alert>
              
              <v-alert
                v-else-if="!serverCatalog || !serverCatalog.plans || serverCatalog.plans.length === 0"
                type="info"
                text="No server plans available"
              ></v-alert>
              
              <div v-else>
                <v-row>
                  <v-col cols="12" md="4" v-for="(plan, index) in serverCatalog.plans" :key="index">
                    <v-card class="h-100">
                      <v-card-title class="text-h6">
                        {{ plan.planCode || 'Server Plan' }}
                      </v-card-title>
                      <v-card-subtitle v-if="plan.description">
                        {{ plan.description }}
                      </v-card-subtitle>
                      <v-card-text>
                        <v-list density="compact">
                          <v-list-item v-if="plan.family">
                            <template v-slot:prepend>
                              <v-icon icon="mdi-server" color="primary"></v-icon>
                            </template>
                            <v-list-item-title>Family</v-list-item-title>
                            <v-list-item-subtitle>{{ plan.family }}</v-list-item-subtitle>
                          </v-list-item>
                          
                          <v-list-item v-if="plan.productType">
                            <template v-slot:prepend>
                              <v-icon icon="mdi-tag" color="primary"></v-icon>
                            </template>
                            <v-list-item-title>Type</v-list-item-title>
                            <v-list-item-subtitle>{{ plan.productType }}</v-list-item-subtitle>
                          </v-list-item>
                          
                          <v-list-item v-if="getPlanPrice(plan)">
                            <template v-slot:prepend>
                              <v-icon icon="mdi-currency-eur" color="primary"></v-icon>
                            </template>
                            <v-list-item-title>Price</v-list-item-title>
                            <v-list-item-subtitle>{{ getPlanPrice(plan) }}</v-list-item-subtitle>
                          </v-list-item>
                        </v-list>
                        
                        <v-chip-group v-if="plan.properties && plan.properties.length > 0" class="mt-3">
                          <v-chip
                            v-for="(prop, propIndex) in plan.properties"
                            :key="propIndex"
                            size="small"
                            variant="outlined"
                            :color="getPropertyColor(prop.name)"
                          >
                            {{ prop.name }}: {{ prop.value }}
                          </v-chip>
                        </v-chip-group>
                      </v-card-text>
                      <v-card-actions>
                        <v-btn
                          color="primary"
                          variant="text"
                          @click="addServerToCart(plan)"
                          :loading="plan.addingToCart"
                          :disabled="!activeCartId || !plan.planCode"
                        >
                          Add to Cart
                        </v-btn>
                        <v-btn
                          color="primary"
                          variant="text"
                          @click="showServerDetails(plan)"
                        >
                          View Details
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-col>
                </v-row>
              </div>
            </v-card-text>
          </v-window-item>

          <!-- Settings Tab -->
          <v-window-item value="settings">
            <v-card-title class="text-h5 py-4">Settings</v-card-title>
            <v-card-text>
              <v-form @submit.prevent="saveSettings">
                <v-text-field
                  v-model="apiToken"
                  label="OVH API Token"
                  type="password"
                  hint="Your OVH API token will be stored locally"
                  placeholder="Enter your OVH API token"
                  persistent-hint
                  :rules="[v => !!v || 'Token is required']"
                  class="mb-4"
                ></v-text-field>
                
                <v-select
                  v-model="selectedSite"
                  label="OVH Subsidiary for Cart Creation"
                  :items="availableSites"
                  hint="Select your OVH region for cart creation (ovhSubsidiary)"
                  persistent-hint
                  item-title="name"
                  item-value="code"
                  return-object
                  class="mb-4"
                >
                  <template v-slot:selection="{ item }">
                    <div class="d-flex align-center">
                      <v-avatar size="24" class="me-2">
                        <v-img :src="getCountryFlag(item?.raw?.code)" alt="Country flag"></v-img>
                      </v-avatar>
                      {{ item?.raw?.name || 'Select site' }} {{ item?.raw?.code ? `(${item.raw.code})` : '' }}
                    </div>
                  </template>
                  
                  <template v-slot:item="{ item, props }">
                    <v-list-item v-bind="props">
                      <template v-slot:prepend>
                        <v-avatar size="24">
                          <v-img :src="getCountryFlag(item?.code)" alt="Country flag"></v-img>
                        </v-avatar>
                      </template>
                      <v-list-item-title>{{ item?.name || 'Unknown site' }}</v-list-item-title>
                      <v-list-item-subtitle>{{ item?.code || '' }}</v-list-item-subtitle>
                    </v-list-item>
                  </template>
                </v-select>
                
                <v-card variant="outlined" class="mb-4 pa-3">
                  <v-card-title class="text-subtitle-1">Current Settings</v-card-title>
                  <v-card-text>
                    <p><strong>API Endpoint:</strong> https://eu.api.ovh.com/v1 (Fixed)</p>
                    <p><strong>Selected Site:</strong> {{ selectedSite?.code || 'Not selected' }} - {{ selectedSite?.name || 'Not selected' }}</p>
                    <p class="text-caption">Note: The API endpoint is fixed to eu.api.ovh.com regardless of site selection</p>
                  </v-card-text>
                </v-card>
                
                <v-btn
                  color="primary"
                  type="submit"
                  :loading="tokenSaving"
                >
                  Save Settings
                </v-btn>
                <v-snackbar
                  v-model="showSettingsSaved"
                  timeout="2000"
                  color="success"
                >
                  Settings saved successfully
                </v-snackbar>
              </v-form>
            </v-card-text>
          </v-window-item>
        </v-window>
      </v-card-text>
    </v-card>
    
    <!-- Success/Error Snackbars -->
    <v-snackbar
      v-model="showSuccessSnackbar"
      timeout="3000"
      color="success"
    >
      {{ successMessage }}
    </v-snackbar>
    
    <v-snackbar
      v-model="showErrorSnackbar"
      timeout="5000"
      color="error"
    >
      {{ errorMessage }}
    </v-snackbar>
    
    <!-- Create Cart Dialog -->
    <v-dialog v-model="showCreateCartDialog" max-width="500px">
      <v-card>
        <v-card-title class="text-h5">Create New Cart</v-card-title>
        <v-card-text>
          <v-form ref="createCartForm">
            <v-text-field
              v-model="newCart.description"
              label="Cart Description"
              :rules="[v => !!v || 'Description is required']"
              hint="Enter a description for your new cart"
              persistent-hint
            ></v-text-field>
            <v-select
              v-model="newCart.ovhSubsidiary"
              label="OVH Subsidiary"
              :items="subsidiaries"
              hint="Select OVH subsidiary"
              persistent-hint
            ></v-select>
            <v-menu
              ref="expiryMenu"
              v-model="showExpiryPicker"
              :close-on-content-click="false"
              transition="scale-transition"
              offset-y
            >
              <template v-slot:activator="{ props }">
                <v-text-field
                  v-model="formattedExpireDate"
                  label="Expiration Date"
                  readonly
                  v-bind="props"
                  prepend-icon="mdi-calendar"
                  hint="Cart will expire at this date/time"
                  persistent-hint
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="expireDatePart"
                @update:model-value="showExpiryTimePicker = true"
              ></v-date-picker>
            </v-menu>
            
            <v-dialog
              v-model="showExpiryTimePicker"
              max-width="290"
            >
              <v-card>
                <v-card-title class="text-h6">Select Time</v-card-title>
                <v-card-text>
                  <v-time-picker
                    v-model="expireTimePart"
                    format="24hr"
                    @update:model-value="setExpireDateTime"
                  ></v-time-picker>
                </v-card-text>
              </v-card>
            </v-dialog>
            
            <v-switch
              v-model="newCart.readOnly"
              label="Read Only"
              hint="Make this cart read-only"
              persistent-hint
            ></v-switch>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="error" variant="text" @click="showCreateCartDialog = false">Cancel</v-btn>
          <v-btn 
            color="success" 
            variant="elevated" 
            @click="submitCreateCart"
            :loading="creatingCart"
          >
            Create
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Server Details Dialog -->
    <v-dialog v-model="showServerDetailsDialog" max-width="800px">
      <v-card v-if="selectedServer">
        <v-card-title class="text-h5">
          {{ selectedServer.planCode || 'Server Details' }}
          <v-spacer></v-spacer>
          <v-btn
            icon="mdi-close"
            variant="text"
            @click="showServerDetailsDialog = false"
          ></v-btn>
        </v-card-title>
        
        <v-card-text>
          <v-row>
            <v-col cols="12" md="6">
              <v-list>
                <v-list-item>
                  <template v-slot:prepend>
                    <v-icon icon="mdi-server" color="primary"></v-icon>
                  </template>
                  <v-list-item-title>Plan Code</v-list-item-title>
                  <v-list-item-subtitle>{{ selectedServer.planCode }}</v-list-item-subtitle>
                </v-list-item>
                
                <v-list-item v-if="selectedServer.description">
                  <template v-slot:prepend>
                    <v-icon icon="mdi-information-outline" color="primary"></v-icon>
                  </template>
                  <v-list-item-title>Description</v-list-item-title>
                  <v-list-item-subtitle>{{ selectedServer.description }}</v-list-item-subtitle>
                </v-list-item>
                
                <v-list-item v-if="selectedServer.family">
                  <template v-slot:prepend>
                    <v-icon icon="mdi-server-network" color="primary"></v-icon>
                  </template>
                  <v-list-item-title>Family</v-list-item-title>
                  <v-list-item-subtitle>{{ selectedServer.family }}</v-list-item-subtitle>
                </v-list-item>
                
                <v-list-item v-if="selectedServer.productType">
                  <template v-slot:prepend>
                    <v-icon icon="mdi-tag" color="primary"></v-icon>
                  </template>
                  <v-list-item-title>Product Type</v-list-item-title>
                  <v-list-item-subtitle>{{ selectedServer.productType }}</v-list-item-subtitle>
                </v-list-item>
                
                <v-list-item v-if="getPlanPrice(selectedServer)">
                  <template v-slot:prepend>
                    <v-icon icon="mdi-currency-eur" color="primary"></v-icon>
                  </template>
                  <v-list-item-title>Price</v-list-item-title>
                  <v-list-item-subtitle>{{ getPlanPrice(selectedServer) }}</v-list-item-subtitle>
                </v-list-item>
              </v-list>
            </v-col>
            
            <v-col cols="12" md="6" v-if="selectedServer.properties && selectedServer.properties.length > 0">
              <v-card variant="outlined">
                <v-card-title class="text-subtitle-1">
                  Technical Specifications
                </v-card-title>
                <v-card-text>
                  <v-list density="compact">
                    <v-list-item v-for="(prop, propIndex) in selectedServer.properties" :key="propIndex">
                      <v-list-item-title>{{ prop.name }}</v-list-item-title>
                      <v-list-item-subtitle>{{ prop.value }}</v-list-item-subtitle>
                    </v-list-item>
                  </v-list>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
          <v-divider class="my-4"></v-divider>
          <v-card-text>
            <v-select
              v-model="selectedDuration"
              :items="durations"
              item-title="text"
              item-value="value"
              label="Duration"
              hint="Select contract duration"
              persistent-hint
              variant="outlined"
              density="comfortable"
              class="mb-4"
            ></v-select>
          </v-card-text>
        </v-card-text>
        
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="primary"
            variant="elevated"
            @click="addServerToCart(selectedServer)"
            :loading="selectedServer.addingToCart"
            :disabled="!activeCartId || !selectedServer.planCode"
          >
            Add to Cart
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const tabs = [
  { id: 'cart', name: 'Shopping Cart', icon: 'mdi-cart' },
  { id: 'servers', name: 'Servers', icon: 'mdi-server' },
  { id: 'settings', name: 'Settings', icon: 'mdi-cog' }
]

const activeTab = ref('cart')
const cartItems = ref([])
const apiToken = ref('')
const loading = ref(false)
const error = ref(null)
const tokenSaving = ref(false)
const showTokenSaved = ref(false)

// Active cart tracking
const activeCartId = ref(null)
const activeCartDetails = ref(null)
const loadingActiveCart = ref(false)

// Create cart related refs
const showCreateCartDialog = ref(false)
const creatingCart = ref(false)
const newCart = ref({
  description: '',
  ovhSubsidiary: 'IE',
  readOnly: false,
  expire: getDefaultExpireDate()
})
const createCartForm = ref(null)

// Date picker related refs
const showExpiryPicker = ref(false)
const showExpiryTimePicker = ref(false)
const expireDatePart = ref('')
const expireTimePart = ref('')

// Subsidiary options
const subsidiaries = ['EU', 'CA', 'US', 'IE']

// Format for form display
const formattedExpireDate = computed(() => {
  if (!newCart.value.expire) return ''
  return new Date(newCart.value.expire).toLocaleString()
})

// Snackbar notifications
const showSuccessSnackbar = ref(false)
const showErrorSnackbar = ref(false)
const successMessage = ref('')
const errorMessage = ref('')

// Server catalog data
const serverCatalog = ref(null)
const loadingServers = ref(false)
const serverError = ref(null)
const selectedServer = ref(null)
const showServerDetailsDialog = ref(false)

// Site selection
const availableSites = [
  { code: 'CZ', name: 'Czech Republic' },
  { code: 'DE', name: 'Germany' },
  { code: 'ES', name: 'Spain' },
  { code: 'EU', name: 'Europe' },
  { code: 'FI', name: 'Finland' },
  { code: 'FR', name: 'France' },
  { code: 'GB', name: 'United Kingdom' },
  { code: 'IE', name: 'Internationnal' },
  { code: 'IT', name: 'Italy' },
  { code: 'LT', name: 'Lithuania' },
  { code: 'MA', name: 'Morocco' },
  { code: 'NL', name: 'Netherlands' },
  { code: 'PL', name: 'Poland' },
  { code: 'PT', name: 'Portugal' },
  { code: 'SN', name: 'Senegal' },
  { code: 'TN', name: 'Tunisia' }
]

const selectedSite = ref(null)
const showSettingsSaved = ref(false)

// In the script section, add these variables
const selectedDuration = ref('P1M'); // Default to 1 month
const durations = [
  { value: 'P1M', text: '1 Month' },
  { value: 'P3M', text: '3 Months' },
  { value: 'P6M', text: '6 Months' },
  { value: 'P12M', text: '12 Months' }
];

// On mounted, load saved site from localStorage
onMounted(() => {
  // Set default site first to avoid undefined errors
  selectedSite.value = availableSites.find(site => site.code === 'EU')
  
  const savedToken = localStorage.getItem('ovhApiToken')
  if (savedToken) {
    apiToken.value = savedToken
    
    // Load active cart ID
    const savedCartId = localStorage.getItem('ovhActiveCartId')
    if (savedCartId) {
      activeCartId.value = savedCartId
      fetchActiveCartDetails()
    }
    
    // Load saved site
    const savedSiteCode = localStorage.getItem('ovhSiteCode')
    if (savedSiteCode) {
      const foundSite = availableSites.find(site => site.code === savedSiteCode)
      if (foundSite) {
        selectedSite.value = foundSite
      }
    }
    
    fetchCart()
  }
})

// Function to format date
const formatDate = (dateString) => {
  if (!dateString) return 'N/A'
  const date = new Date(dateString)
  return date.toLocaleString()
}

// Function to show success notification
const showSuccess = (message) => {
  successMessage.value = message
  showSuccessSnackbar.value = true
}

// Function to show error notification
const showError = (message) => {
  errorMessage.value = message
  showErrorSnackbar.value = true
}

// Function to get country flag for site code
const getCountryFlag = (siteCode) => {
  // Check if siteCode is defined
  if (!siteCode) return 'https://flagcdn.com/48x36/xx.png'; // Return a placeholder or default flag
  
  // Convert site code to country code for flags
  const countryCode = siteCode === 'EU' ? 'eu' : siteCode.toLowerCase();
  return `https://flagcdn.com/48x36/${countryCode}.png`;
}

// Function to get API endpoint based on selected site
const getApiEndpoint = () => {
  // Always use eu.api.ovh.com regardless of site selection
  return 'https://eu.api.ovh.com/v1';
}

// Function to save settings
const saveSettings = async () => {
  try {
    tokenSaving.value = true
    
    // Save token
    localStorage.setItem('ovhApiToken', apiToken.value)
    
    // Save selected site
    if (selectedSite.value) {
      localStorage.setItem('ovhSiteCode', selectedSite.value.code)
    }
    
    showSettingsSaved.value = true
    
    // Try to fetch cart data with the new settings
    await fetchCart()
  } catch (err) {
    console.error('Error saving settings:', err)
    showError(`Error saving settings: ${err.message}`)
  } finally {
    tokenSaving.value = false
  }
}

// Function to fetch cart data
const fetchCart = async () => {
  if (!apiToken.value) return
  
  loading.value = true
  error.value = null
  
  try {
    const headers = {
      'Authorization': `Bearer ${apiToken.value}`,
      'Content-Type': 'application/json'
    }
    
    // First, get the cart list
    const apiEndpoint = getApiEndpoint()
    const cartListResponse = await fetch(`${apiEndpoint}/order/cart`, {
      headers
    })
    
    if (!cartListResponse.ok) {
      throw new Error(`Failed to fetch cart list: ${cartListResponse.status}`)
    }
    
    const cartIds = await cartListResponse.json()
    
    // If there are carts, fetch details for each
    if (cartIds && cartIds.length) {
      const cartDetails = []
      
      for (const cartId of cartIds) {
        const cartDetailResponse = await fetch(`${apiEndpoint}/order/cart/${cartId}`, {
          headers
        })
        
        if (cartDetailResponse.ok) {
          const detail = await cartDetailResponse.json()
          cartDetails.push(detail)
        }
      }
      
      cartItems.value = cartDetails
    } else {
      cartItems.value = []
    }
  } catch (err) {
    console.error('Error fetching cart:', err)
    error.value = err
    showError(`Error fetching cart: ${err.message}`)
  } finally {
    loading.value = false
  }
}

const removeFromCart = (item) => {
  // Implement cart item removal functionality
  console.log('Remove item from cart:', item)
}

// Function to get default expire date (current time + 7 days)
function getDefaultExpireDate() {
  const date = new Date()
  date.setDate(date.getDate() + 7)
  return date.toISOString()
}

// Function to set expire date from date and time pickers
function setExpireDateTime() {
  if (!expireDatePart.value || !expireTimePart.value) return
  
  const [hour, minute] = expireTimePart.value.split(':')
  const date = new Date(expireDatePart.value)
  
  date.setHours(parseInt(hour), parseInt(minute), 0)
  newCart.value.expire = date.toISOString()
  
  showExpiryTimePicker.value = false
  showExpiryPicker.value = false
}

// Function to show create cart dialog
const createCart = () => {
  newCart.value = {
    description: '',
    ovhSubsidiary: selectedSite.value ? selectedSite.value.code : 'IE',
    readOnly: false,
    expire: getDefaultExpireDate()
  }
  
  // Set date picker initial values based on default expire date
  const defaultDate = new Date(newCart.value.expire)
  expireDatePart.value = defaultDate.toISOString().split('T')[0]
  
  const hours = defaultDate.getHours().toString().padStart(2, '0')
  const minutes = defaultDate.getMinutes().toString().padStart(2, '0')
  expireTimePart.value = `${hours}:${minutes}`
  
  showCreateCartDialog.value = true
}

// Function to submit new cart creation
const submitCreateCart = async () => {
  if (!apiToken.value) return
  
  // Validate form
  const isValid = await createCartForm.value?.validate()
  if (!isValid?.valid) return
  
  creatingCart.value = true
  
  try {
    const headers = {
      'Authorization': `Bearer ${apiToken.value}`,
      'Content-Type': 'application/json'
    }
    
    const apiEndpoint = getApiEndpoint()
    const response = await fetch(`${apiEndpoint}/order/cart`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        ovhSubsidiary: newCart.value.ovhSubsidiary,
        description: newCart.value.description,
        readOnly: newCart.value.readOnly,
        expire: newCart.value.expire
      })
    })
    
    if (!response.ok) {
      throw new Error(`Failed to create cart: ${response.status}`)
    }
    
    const result = await response.json()
    
    // Store the cart ID in localStorage and set as active
    localStorage.setItem('ovhActiveCartId', result.cartId)
    activeCartId.value = result.cartId
    activeCartDetails.value = result
    
    showSuccess(`Cart created successfully! Cart ID: ${result.cartId}`)
    showCreateCartDialog.value = false
    
    // Refresh cart data to include the new cart
    await fetchCart()
  } catch (err) {
    console.error('Error creating cart:', err)
    showError(`Error creating cart: ${err.message}`)
  } finally {
    creatingCart.value = false
  }
}

// Function to set active cart
const setActiveCart = (cartId) => {
  activeCartId.value = cartId
  localStorage.setItem('ovhActiveCartId', cartId)
  fetchActiveCartDetails()
  showSuccess(`Cart ${cartId} set as active`)
}

// Function to clear active cart
const clearActiveCart = () => {
  activeCartId.value = null
  activeCartDetails.value = null
  localStorage.removeItem('ovhActiveCartId')
  showSuccess('Active cart cleared')
}

// Function to fetch active cart details
const fetchActiveCartDetails = async () => {
  if (!apiToken.value || !activeCartId.value) return
  
  loadingActiveCart.value = true
  
  try {
    const headers = {
      'Authorization': `Bearer ${apiToken.value}`,
      'Content-Type': 'application/json'
    }
    
    const apiEndpoint = getApiEndpoint()
    const response = await fetch(`${apiEndpoint}/order/cart/${activeCartId.value}`, {
      headers
    })
    
    if (!response.ok) {
      throw new Error(`Failed to fetch cart details: ${response.status}`)
    }
    
    const cartDetails = await response.json()
    activeCartDetails.value = cartDetails
    console.log('Active cart details:', cartDetails)
  } catch (err) {
    console.error('Error fetching active cart details:', err)
    showError(`Error fetching active cart details: ${err.message}`)
    activeCartDetails.value = null
  } finally {
    loadingActiveCart.value = false
  }
}

// Function to fetch server catalog
const fetchServers = async () => {
  if (!apiToken.value) return
  
  loadingServers.value = true
  serverError.value = null
  
  try {
    const headers = {
      'Authorization': `Bearer ${apiToken.value}`,
      'Content-Type': 'application/json'
    }
    
    // Get the current subsidiary code from the selected site
    const subsidiaryCode = selectedSite.value ? selectedSite.value.code : 'IE'
    
    const apiEndpoint = getApiEndpoint()
    const response = await fetch(`${apiEndpoint}/order/catalog/public/eco?ovhSubsidiary=${subsidiaryCode}`, {
      headers
    })
    
    if (!response.ok) {
      throw new Error(`Failed to fetch server catalog: ${response.status}`)
    }
    
    const data = await response.json()
    
    // Initialize addingToCart property for each plan
    if (data.plans && data.plans.length > 0) {
      data.plans.forEach(plan => {
        plan.addingToCart = false;
      });
    }
    
    serverCatalog.value = data
    console.log('Server catalog for subsidiary', subsidiaryCode, ':', data)
  } catch (err) {
    console.error('Error fetching server catalog:', err)
    serverError.value = err
    showError(`Error fetching server catalog: ${err.message}`)
  } finally {
    loadingServers.value = false
  }
}

// Watch for site changes to refresh servers if on servers tab
watch(selectedSite, (newSite, oldSite) => {
  if (newSite?.code !== oldSite?.code && activeTab.value === 'servers') {
    console.log('Site changed, refreshing server catalog...');
    fetchServers();
  }
});

// Function to get plan price
const getPlanPrice = (plan) => {
  if (!plan.pricings || plan.pricings.length === 0) {
    return 'Price not available'
  }
  
  const pricing = plan.pricings[0]
  if (pricing.price) {
    return `${pricing.price.text || `${pricing.price.value} ${pricing.price.currencyCode}`}`
  }
  
  return 'Price not available'
}

// Function to get property color
const getPropertyColor = (propName) => {
  const colorMap = {
    'cpu': 'blue',
    'ram': 'green',
    'disk': 'amber',
    'storage': 'amber',
    'bandwidth': 'purple',
    'network': 'purple',
    'location': 'red',
    'region': 'red',
    'datacenter': 'red'
  }
  
  const key = Object.keys(colorMap).find(key => propName.toLowerCase().includes(key))
  return key ? colorMap[key] : 'grey'
}

// Function to add server to cart
const addServerToCart = async (plan) => {
  if (!activeCartId.value || !plan.planCode) {
    showError('Cannot add server to cart. Please select an active cart first.')
    return
  }
  
  // Set loading state for this specific plan
  if (plan) {
    plan.addingToCart = true;
  }
  
  try {
    const headers = {
      'Authorization': `Bearer ${apiToken.value}`,
      'Content-Type': 'application/json'
    }
    
    const apiEndpoint = getApiEndpoint();
    const url = `${apiEndpoint}/order/cart/${activeCartId.value}/eco`;
    
    console.log('Adding server to cart:', plan.planCode, 'URL:', url);
    
    const payload = {
      duration: selectedDuration.value, // Use selected duration
      planCode: plan.planCode,
      pricingMode: "default",
      quantity: 1
    };
    
    const response = await fetch(url, {
      method: 'POST',
      headers,
      body: JSON.stringify(payload)
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(`Failed to add server to cart: ${response.status} - ${errorData.message || JSON.stringify(errorData)}`);
    }
    
    const result = await response.json();
    showSuccess(`Added ${plan.planCode} to cart successfully!`);
    console.log('Server added to cart result:', result);
    
    // Refresh cart details
    await fetchActiveCartDetails();
    
    // Close the details dialog if it's open
    if (showServerDetailsDialog.value && selectedServer.value && selectedServer.value.planCode === plan.planCode) {
      showServerDetailsDialog.value = false;
    }
    
    return result;
  } catch (err) {
    console.error('Error adding server to cart:', err);
    showError(`Error adding server to cart: ${err.message}`);
    return null;
  } finally {
    // Reset loading state
    if (plan) {
      plan.addingToCart = false;
    }
  }
}

// Function to show server details
const showServerDetails = (plan) => {
  selectedServer.value = plan;
  selectedDuration.value = 'P1M'; // Reset to default 1 month
  showServerDetailsDialog.value = true;
}

// Fetch servers when token is available and tab is changed to servers
watch(activeTab, (newTab) => {
  if (newTab === 'servers' && apiToken.value && (!serverCatalog.value || serverCatalog.value.plans?.length === 0)) {
    fetchServers()
  }
});
</script>
