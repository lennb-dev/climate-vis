<template>
  <div>
    <!-- WorldMap with required props -->
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
      valueKey="CO2"
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

// CSV and GeoJSON assets
import csvRaw from '../assets/co-emissions-per-capita.csv?raw'
import worldGeoUrl from '../assets/worldmap.geojson?url'

// CO2 per capita thresholds and color scale
const thresholds = [0, 2.5, 5, 7.5, 10, 12.5, 15]
const colors = [
  '#ffffff', '#ffeda0', '#feb24c', '#fd8d3c', '#fc4e2a', '#e31a1c',
  '#bd0026', '#800026'
]

// Load GeoJSON data once
const geoData = ref([])
onMounted(async () => {
  try {
    const worldData = await d3.json(worldGeoUrl)
    geoData.value = worldData.features || worldData
  } catch (err) {
    console.error('Error loading GeoJSON:', err)
  }
})

// Parse CO2 per capita CSV data
const rawRows = d3.csvParse(csvRaw)
const csvData = rawRows
  .filter(d => d.Code && !isNaN(+d.CO2))
  .map(d => ({ iso: d.Code, year: +d.Year, value: +d.CO2 }))

// Determine year range
const years = Array.from(new Set(csvData.map(d => d.year))).sort((a, b) => a - b)
const minYear = years[0]
const maxYear = years[years.length - 1]
const year = ref(maxYear)

// Map<ISO, value> for selected year
const yearDataMap = computed(() => {
  const m = new Map()
  const seen = new Set()
  // Sort by year descending
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

// Map<ISO, actual year> for selected year
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
const formatLabel = v => d3.format('.2s')(v) + ' t'
const getISO = feature => feature.properties['ISO3166-1-Alpha-3']
const getTooltipContent = (feature, value, actualYear) => {
  const name = feature.properties.ADMIN || feature.properties.name || 'Unknown'
  if (value == null) {
    return `<strong>${name}</strong><br/>no Data`
  }
  const val = d3.format(',.2f')(value) + ' t CO₂'
  return `<strong>${name}</strong><br/>${val} per capita <br/><em>Data from ${actualYear}</em>`
}

</script>
