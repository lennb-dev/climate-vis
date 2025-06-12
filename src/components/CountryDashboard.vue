<template>
  <div class="p-8 bg-gray-50 min-h-screen">
    <!-- Header with modern styling -->
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold text-gray-800">{{ countryName }}</h1>
      <button @click="$emit('close')" 
        class="px-4 py-2 rounded-lg bg-white border border-gray-200 text-gray-600 hover:bg-gray-50 hover:border-gray-300 transition-colors duration-200 shadow-sm">
        Close
      </button>
    </div>

    <!-- Main content grid -->
    <div class="grid gap-6">
      <!-- Line Chart - Full width -->
      <div class="w-full bg-white rounded-xl shadow-sm p-6 border border-gray-100">
        <CountryLineChart
          :iso="iso"
          :countryName="countryName"
          @processed-data="updateProcessedData"
        />
      </div>

      <!-- Two column layout for smaller charts -->
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
import CountryLineChart from './CountryLineChart.vue'
import CountryPieChart from './CountryPieChart.vue'
import StackedLineChart from './CountryStackedLineChart.vue'

const props = defineProps({
  iso: String,
  countryName: String
})

const emit = defineEmits(['close'])

const processedData = ref([])

function updateProcessedData(data) {
  processedData.value = data
}
</script>