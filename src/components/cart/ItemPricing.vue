<template>
  <v-col cols="12">
    <v-card variant="outlined" class="mt-2">
      <v-card-title class="text-subtitle-1">
        <v-icon icon="mdi-currency-eur" color="primary" class="mr-2"></v-icon>
        Pricing Summary
      </v-card-title>
      <v-card-text>
        <v-table density="compact">
          <thead>
            <tr>
              <th>Description</th>
              <th>Duration</th>
              <th class="text-right">Price</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(price, i) in prices" :key="i" :class="price.label === 'TOTAL' ? 'font-weight-bold' : ''">
              <td>{{ price.description }}</td>
              <td>{{ formatDuration(price.duration) }}</td>
              <td class="text-right">{{ price.price.text }} {{ price.price.currencySymbol }}</td>
            </tr>
          </tbody>
        </v-table>
      </v-card-text>
    </v-card>
  </v-col>
</template>

<script setup>
defineProps({
  prices: {
    type: Array,
    required: true
  }
});

// Function to format duration string
const formatDuration = (duration) => {
  if (!duration) return 'N/A';
  
  const match = duration.match(/P(?:(\d+)Y)?(?:(\d+)M)?(?:(\d+)D)?(?:T(?:(\d+)H)?(?:(\d+)M)?(?:(\d+)S)?)?/);
  if (!match) return duration;
  
  const [_, years, months, days, hours, minutes, seconds] = match;
  let result = [];
  
  if (years) result.push(`${years} ${parseInt(years) === 1 ? 'year' : 'years'}`);
  if (months) result.push(`${months} ${parseInt(months) === 1 ? 'month' : 'months'}`);
  if (days) result.push(`${days} ${parseInt(days) === 1 ? 'day' : 'days'}`);
  if (hours) result.push(`${hours} ${parseInt(hours) === 1 ? 'hour' : 'hours'}`);
  if (minutes) result.push(`${minutes} ${parseInt(minutes) === 1 ? 'minute' : 'minutes'}`);
  if (seconds) result.push(`${seconds} ${parseInt(seconds) === 1 ? 'second' : 'seconds'}`);
  
  return result.length > 0 ? result.join(', ') : 'One-time';
};
</script>
