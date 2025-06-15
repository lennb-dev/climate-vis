<template>
  <div class="p-8 bg-gray-50 min-h-screen">
    <!-- Header with title and close button -->
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold text-gray-800">{{ countryName }}</h1>
      <button 
        @click="$emit('close')" 
        class="px-4 py-2 rounded-lg bg-white border border-gray-200 text-gray-600 hover:bg-gray-50 hover:border-gray-300 transition-colors duration-200 shadow-sm"
      >
        Close
      </button>
    </div>

    <!-- Main content layout -->
    <div class="grid gap-6">

      <!-- Full-width Line Chart for time series visualization -->
      <div class="w-full bg-white rounded-xl shadow-sm p-6 border border-gray-100">
        <CountryLineChart
          :iso="iso"
          :countryName="countryName"
          @processed-data="updateProcessedData"
        />
      </div>

      <!-- Two-column layout -->
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        
        <!-- Pie Chart -->
        <div class="bg-white rounded-xl shadow-sm p-6 border border-gray-100 w-full">
          <CountryPieChart
            v-if="processedData.length"
            :iso="iso"
            :processedData="processedData"
          />
        </div>

        <!-- Stacked Line Chart -->
        <div class="bg-white rounded-xl shadow-sm p-6 border border-gray-100">
          <StackedLineChart :iso="iso" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Child chart components
import CountryLineChart from './CountryLineChart.vue'
import CountryPieChart from './CountryPieChart.vue'
import StackedLineChart from './CountryStackedLineChart.vue'

// Props for country ISO code and display name
const props = defineProps({
  iso: String,
  countryName: String
})

// Emit close event to parent when user clicks "Close"
const emit = defineEmits(['close'])

// Local state for storing processed chart data
const processedData = ref([])

// Update processed data from child component
function updateProcessedData(data) {
  processedData.value = data
}
</script>
