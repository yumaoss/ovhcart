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
            <v-card-title class="text-h5 py-4">Server Plans</v-card-title>
            <v-card-text>
              <v-alert
                type="info"
                text="No server plans available"
              ></v-alert>
            </v-card-text>
          </v-window-item>

          <!-- Settings Tab -->
          <v-window-item value="settings">
            <v-card-title class="text-h5 py-4">Settings</v-card-title>
            <v-card-text>
              <v-form @submit.prevent="saveToken">
                <v-text-field
                  v-model="apiToken"
                  label="OVH API Token"
                  type="password"
                  hint="Your OVH API token will be stored locally"
                  placeholder="Enter your OVH API token"
                  persistent-hint
                  :rules="[v => !!v || 'Token is required']"
                ></v-text-field>
                <v-btn
                  color="primary"
                  type="submit"
                  class="mt-4"
                  :loading="tokenSaving"
                >
                  Save Token
                </v-btn>
                <v-snackbar
                  v-model="showTokenSaved"
                  timeout="2000"
                  color="success"
                >
                  Token saved successfully
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
  </v-container>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

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

// Load saved token and cart ID from localStorage
onMounted(() => {
  const savedToken = localStorage.getItem('ovhApiToken')
  if (savedToken) {
    apiToken.value = savedToken
    
    // Load active cart ID
    const savedCartId = localStorage.getItem('ovhActiveCartId')
    if (savedCartId) {
      activeCartId.value = savedCartId
      fetchActiveCartDetails()
    }
    
    fetchCart()
  }
})

const saveToken = async () => {
  try {
    tokenSaving.value = true
    localStorage.setItem('ovhApiToken', apiToken.value)
    showTokenSaved.value = true
    // Try to fetch cart data with the new token
    await fetchCart()
  } catch (err) {
    console.error('Error saving token:', err)
  } finally {
    tokenSaving.value = false
  }
}

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
    const cartListResponse = await fetch('https://eu.api.ovh.com/v1/order/cart', {
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
        const cartDetailResponse = await fetch(`https://eu.api.ovh.com/v1/order/cart/${cartId}`, {
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
    ovhSubsidiary: 'IE',
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
    
    const response = await fetch('https://eu.api.ovh.com/v1/order/cart', {
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
    
    const response = await fetch(`https://eu.api.ovh.com/v1/order/cart/${activeCartId.value}`, {
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
</script>
