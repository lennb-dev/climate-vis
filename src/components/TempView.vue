<template>
  <div>
    <!-- WorldMap with all required props -->
    <WorldMap
      :geoData="geoData"
      :year="year"
      :dataMap="yearDataMap"
      :actualYearMap="actualYearMap"
      :thresholds="thresholds"
      :colors="colors"
      :labelFormat="formatLabel"
      :getISO="getISO"
      :getTooltipContent="getTooltipContent"
      :data="csvData"
      valueKey="Temperature"
    />
    <!-- TimeSlider for year selection -->
    <TimeSlider
      v-model="year"
      :minYear="minYear"
      :maxYear="maxYear"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import * as d3 from 'd3'

// Components
import WorldMap from './WorldMap.vue'
import TimeSlider from './TimeSlider.vue'

// Import CSV and GeoJSON
import csvRaw from '../assets/annual-temperature-anomalies.csv?raw'
import worldGeoUrl from '../assets/worldmap.geojson?url'

// Thresholds and colors for temperature anomalies
const thresholds = [-2, -1.5, -1, -0.5, 0, 0.5, 1, 1.5]
const colors = [
  '#6baed6', '#9ecae1', '#c6dbef', '#e0ecf4',
  '#ffffff', '#ffeda0', '#feb24c', '#fd8d3c', '#e31a1c'
]

// Load GeoJSON data
const geoData = ref([])
onMounted(async () => {
  try {
    const worldData = await d3.json(worldGeoUrl)
    geoData.value = worldData.features || worldData
  } catch (err) {
    console.error('Error loading GeoJSON:', err)
  }
})

// Parse temperature data
const rawRows = d3.csvParse(csvRaw)
const csvData = rawRows
  .filter(d => d.Code && !isNaN(+d['Temperature anomaly']))
  .map(d => ({ iso: d.Code, year: +d.Year, value: +d['Temperature anomaly'] }))

// Calculate year range
const years = Array.from(new Set(csvData.map(d => d.year))).sort((a, b) => a - b)
const minYear = years[0]
const maxYear = years[years.length - 1]
const year = ref(maxYear)

// Map<ISO, value> for current year
const yearDataMap = computed(() => {
  const m = new Map()
  const seen = new Set()

  // Sort data by year descending
  const sortedData = csvData
    .filter(d => d.year <= year.value)
    .sort((a, b) => b.year - a.year)

  for (const d of sortedData) {
    if (!seen.has(d.iso)) {
      m.set(d.iso, d.value)
      seen.add(d.iso)
    }
  }

  return m
})

// Map<ISO, year> for current year
const actualYearMap = computed(() => {
  const m = new Map()
  const seen = new Set()
  const sortedData = csvData
    .filter(d => d.year <= year.value)
    .sort((a, b) => b.year - a.year)
  for (const d of sortedData) {
    if (!seen.has(d.iso)) {
      m.set(d.iso, d.year)
      seen.add(d.iso)
    }
  }
  return m
})

// Label and tooltip formatting
const formatLabel = v => d3.format('.1f')(v) + ' °C'
const getISO = feature => feature.properties['ISO3166-1-Alpha-3']
const getTooltipContent = (feature, value, actualYear) => {
  const name = feature.properties.ADMIN || feature.properties.name || 'Unknown'
  if (value == null) {
    return `<strong>${name}</strong><br/>no Data`
  }
  const val = d3.format('.1f')(value)
  return `<strong>${name}</strong><br/>${val} °C temperature anomaly<br/><em>Data from ${actualYear}</em>`
}
</script>
