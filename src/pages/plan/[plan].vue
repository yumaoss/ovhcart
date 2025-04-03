<template>
  <div class="plan-container">
    <div v-if="loading" class="loading">
      <p>Loading server availability...</p>
    </div>
    <div v-else-if="error" class="error">
      <p>{{ error }}</p>
    </div>
    <div v-else class="server-info">
      <h1>Server Plan: {{ planData.planCode }}</h1>
      
      <div class="server-specs">
        <div class="spec-item">
          <h3>Server</h3>
          <p>{{ planData.server }}</p>
        </div>
        <div class="spec-item">
          <h3>Memory</h3>
          <p>{{ planData.memory }}</p>
        </div>
        <div class="spec-item">
          <h3>Storage</h3>
          <p>{{ planData.storage }}</p>
        </div>
      </div>

      <h2>Datacenter Availability</h2>
      <div class="datacenter-grid">
        <div 
          v-for="dc in planData.datacenters" 
          :key="dc.datacenter"
          class="datacenter-card"
          :class="getAvailabilityClass(dc.availability)"
        >
          <h3>{{ dc.datacenter.toUpperCase() }}</h3>
          <p class="availability">{{ formatAvailability(dc.availability) }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import axios from 'axios'

export default {
  name: 'PlanPage',
  setup() {
    const route = useRoute()
    const planCode = ref('')
    const planData = ref(null)
    const loading = ref(true)
    const error = ref(null)

    const fetchPlanData = async () => {
      loading.value = true
      error.value = null
      
      try {
        planCode.value = route.params.plan
        const response = await axios.get(
          `https://ca.ovh.com/engine/apiv6/dedicated/server/datacenter/availabilities/?excludeDatacenters=false&planCode=${planCode.value}`
        )
        
        if (response.data && response.data.length > 0) {
          planData.value = response.data[0]
        } else {
          error.value = 'No data available for this plan'
        }
      } catch (err) {
        console.error('Error fetching plan data:', err)
        error.value = 'Failed to load server data. Please try again later.'
      } finally {
        loading.value = false
      }
    }

    const getAvailabilityClass = (availability) => {
      if (availability.includes('high')) return 'high-availability'
      if (availability.includes('low')) return 'low-availability'
      if (availability === 'unavailable') return 'unavailable'
      return 'medium-availability'
    }

    const formatAvailability = (availability) => {
      if (availability === 'unavailable') return 'Unavailable'
      return availability
    }

    onMounted(() => {
      fetchPlanData()
    })

    return {
      planCode,
      planData,
      loading,
      error,
      getAvailabilityClass,
      formatAvailability
    }
  }
}
</script>

<style scoped>
:root {
  --v-theme-on-surface-rgb: 0, 0, 0;
}

.v-theme--dark {
  --v-theme-on-surface-rgb: 255, 255, 255;
  --v-theme-success-container: rgba(0, 200, 83, 0.12);
  --v-theme-success-border: rgba(0, 200, 83, 0.2);
  --v-theme-on-success-container: rgba(0, 200, 83, 0.87);
  --v-theme-warning-container: rgba(255, 171, 0, 0.12);
  --v-theme-warning-border: rgba(255, 171, 0, 0.2);
  --v-theme-on-warning-container: rgba(255, 171, 0, 0.87);
  --v-theme-error-container: rgba(244, 67, 54, 0.12);
  --v-theme-error-border: rgba(244, 67, 54, 0.2);
  --v-theme-on-error-container: rgba(244, 67, 54, 0.87);
}
.plan-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  color: var(--v-theme-on-background, inherit);
}

.loading, .error {
  text-align: center;
  padding: 2rem;
  font-size: 1.2rem;
}

.error {
  color: #e74c3c;
}

.server-specs {
  display: flex;
  gap: 2rem;
  margin: 2rem 0;
  background-color: var(--v-theme-surface-variant, #f8f9fa);
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(var(--v-theme-on-surface-rgb, 0, 0, 0), 0.05);
}

.spec-item {
  flex: 1;
}

.spec-item h3 {
  margin-bottom: 0.5rem;
  color: var(--v-theme-primary, #555);
}

.datacenter-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.datacenter-card {
  padding: 1.5rem;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(var(--v-theme-on-surface-rgb, 0, 0, 0), 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.datacenter-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 12px rgba(var(--v-theme-on-surface-rgb, 0, 0, 0), 0.15);
}

.datacenter-card h3 {
  margin-bottom: 0.5rem;
}

.high-availability {
  background-color: var(--v-theme-success-container, #d4edda);
  border: 1px solid var(--v-theme-success-border, #c3e6cb);
  color: var(--v-theme-on-success-container, #155724);
}

.medium-availability {
  background-color: var(--v-theme-warning-container, #fff3cd);
  border: 1px solid var(--v-theme-warning-border, #ffeeba);
  color: var(--v-theme-on-warning-container, #856404);
}

.low-availability {
  background-color: var(--v-theme-error-container, #f8d7da);
  border: 1px solid var(--v-theme-error-border, #f5c6cb);
  color: var(--v-theme-on-error-container, #721c24);
}

.unavailable {
  background-color: var(--v-theme-surface, #e9ecef);
  border: 1px solid var(--v-theme-outline, #dee2e6);
  color: var(--v-theme-on-surface-variant, #6c757d);
}

.availability {
  font-weight: bold;
  font-size: 1.1rem;
}
</style>