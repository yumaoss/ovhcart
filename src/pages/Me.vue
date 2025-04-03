<template>
  <v-container>
    <v-card>
      <v-card-title class="text-h5 py-4 d-flex align-center">
        User Profile
        <v-spacer />
        <v-btn
          color="primary"
          variant="text"
          prepend-icon="mdi-refresh"
          :loading="loading"
          :disabled="!apiToken"
          @click="fetchUserProfile"
        >
          Refresh
        </v-btn>
      </v-card-title>

      <v-card-text>
        <div v-if="loading">
          <v-progress-circular 
            indeterminate 
            color="primary" 
          />
          <span class="ml-2">Loading user profile...</span>
        </div>
        
        <v-alert
          v-else-if="error"
          type="error"
          title="Error"
          :text="error.message || 'Failed to load user profile'"
          class="mb-4"
        />

        <div v-else-if="!apiToken">
          <v-alert
            type="warning"
            text="Please set your OVH API token in Settings tab to access your profile information."
            class="mb-4"
          />
        </div>

        <div v-else-if="userData">
          <v-row>
            <v-col 
              cols="12" 
              md="6"
            >
              <v-card 
                variant="outlined" 
                class="mb-4"
              >
                <v-card-title class="text-subtitle-1">
                  <v-icon 
                    icon="mdi-account" 
                    color="primary" 
                    class="mr-2"
                  />
                  Personal Information
                </v-card-title>
                <v-card-text>
                  <v-table density="compact">
                    <tbody>
                      <tr>
                        <th>Name</th>
                        <td>{{ userData.firstname }} {{ userData.name }}</td>
                      </tr>
                      <tr>
                        <th>Email</th>
                        <td>{{ userData.email }}</td>
                      </tr>
                      <tr>
                        <th>Nickname</th>
                        <td>{{ userData.nichandle }}</td>
                      </tr>
                      <tr>
                        <th>Customer ID</th>
                        <td>{{ userData.customerCode }}</td>
                      </tr>
                      <tr>
                        <th>Area</th>
                        <td>{{ userData.area }}</td>
                      </tr>
                      <tr>
                        <th>State</th>
                        <td>
                          <v-chip
                            :color="getUserStateColor(userData.state)"
                            size="small"
                          >
                            {{ userData.state }}
                          </v-chip>
                        </td>
                      </tr>
                    </tbody>
                  </v-table>
                </v-card-text>
              </v-card>
            </v-col>

            <v-col 
              cols="12" 
              md="6"
            >
              <v-card 
                variant="outlined" 
                class="mb-4"
              >
                <v-card-title class="text-subtitle-1">
                  <v-icon 
                    icon="mdi-map-marker" 
                    color="primary" 
                    class="mr-2"
                  />
                  Address Information
                </v-card-title>
                <v-card-text>
                  <v-table density="compact">
                    <tbody>
                      <tr>
                        <th>Country</th>
                        <td>{{ userData.country }}</td>
                      </tr>
                      <tr>
                        <th>Currency</th>
                        <td>{{ userData.currency?.code || 'N/A' }}</td>
                      </tr>
                      <tr>
                        <th>Language</th>
                        <td>{{ userData.language }}</td>
                      </tr>
                      <tr>
                        <th>OVH Subsidiary</th>
                        <td>{{ userData.ovhSubsidiary }}</td>
                      </tr>
                    </tbody>
                  </v-table>
                </v-card-text>
              </v-card>

              <v-card variant="outlined">
                <v-card-title class="text-subtitle-1">
                  <v-icon 
                    icon="mdi-cog" 
                    color="primary" 
                    class="mr-2"
                  />
                  Account Settings
                </v-card-title>
                <v-card-text>
                  <v-table density="compact">
                    <tbody>
                      <tr>
                        <th>OVHCompany</th>
                        <td>{{ userData.ovhCompany }}</td>
                      </tr>
                      <tr>
                        <th>Creation</th>
                        <td>{{ formatDate(userData.creation) }}</td>
                      </tr>
                      <tr>
                        <th>Legal Form</th>
                        <td>{{ userData.legalform }}</td>
                      </tr>
                      <tr>
                        <th>Two-Factor Auth</th>
                        <td>
                          <v-chip
                            :color="userData.hasTwoFA ? 'success' : 'error'"
                            size="small"
                          >
                            {{ userData.hasTwoFA ? 'Enabled' : 'Disabled' }}
                          </v-chip>
                        </td>
                      </tr>
                    </tbody>
                  </v-table>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </div>
        <div v-else>
          <v-alert
            type="info"
            text="No user data available. Please refresh to load your profile."
            class="mb-4"
          />
        </div>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// Props - we need the API token from the parent component
const props = defineProps({
  apiToken: {
    type: String,
    default: ''
  },
  selectedSite: {
    type: Object,
    default: () => ({ code: 'EU' })
  }
});

// State
const userData = ref(null);
const loading = ref(false);
const error = ref(null);

// Function to get API endpoint based on selected site
const getApiEndpoint = () => {
  const siteCode = props.selectedSite?.code || 'EU';
  switch (siteCode) {
    default: return 'https://eu.api.ovh.com/v1';
  }
};

// Function to fetch user profile from OVH API
const fetchUserProfile = async () => {
  if (!props.apiToken) {
    error.value = { message: 'API token is required' };
    return;
  }

  loading.value = true;
  error.value = null;

  try {
    const apiEndpoint = getApiEndpoint();
    const response = await fetch(`${apiEndpoint}/me`, {
      method: 'GET',
      headers: {
        // 'X-Ovh-Application': 'OVHCart',
        //'X-Ovh-Consumer': 'OVHCart',
        'X-Ovh-Timestamp': Math.floor(Date.now() / 1000),
        // 'X-Ovh-Signature': props.apiToken,
        'Authorization': `Bearer ${props.apiToken}`,
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new Error(`Failed to fetch user profile: ${response.status} - ${errorData.message || JSON.stringify(errorData)}`);
    }

    userData.value = await response.json();
  } catch (err) {
    console.error('Error fetching user profile:', err);
    error.value = err;
  } finally {
    loading.value = false;
  }
};

// Format date helper function
const formatDate = (dateString) => {
  if (!dateString) return 'Not set';
  const date = new Date(dateString);
  return date.toLocaleString();
};

// Get color for user state
const getUserStateColor = (state) => {
  if (!state) return 'grey';
  switch (state.toLowerCase()) {
    case 'active': return 'success';
    case 'enabled': return 'success';
    case 'disabled': return 'error';
    case 'suspended': return 'warning';
    case 'pending': return 'info';
    default: return 'grey';
  }
};

// Fetch user profile on component mount if we have a token
onMounted(() => {
  if (props.apiToken) {
    fetchUserProfile();
  }
});
</script>
