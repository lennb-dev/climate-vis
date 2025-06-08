<template>
  <div>
    <!-- WorldMap mit allen nötigen Props -->
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
      valueKey="ElectricityShare"
    />
    <!-- TimeSlider zum Wechseln des Jahres -->
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

// Komponenten
import WorldMap from './WorldMap.vue'
import TimeSlider from './TimeSlider.vue'

// CSV- und GeoJSON-Assets
import csvRaw from '../assets/share-electricity-renewables.csv?raw'
import worldGeoUrl from '../assets/worldmap.geojson?url'

// 1) Schwellen und Farben
const thresholds = [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
const colors = [
  '#ffffff', '#ffffe5', '#f7fcb9', '#d9f0a3', '#addd8e', '#78c679',
  '#41ab5d', '#238443', '#006837', '#004529', '#00331d'
]

// 2) GeoJSON laden (einmalig)
const geoData = ref([])
onMounted(async () => {
  try {
    const worldData = await d3.json(worldGeoUrl)
    geoData.value = worldData.features || worldData
  } catch (err) {
    console.error('Fehler beim Laden des GeoJSON:', err)
  }
})

// 3) CO2-Daten parsen
const rawRows = d3.csvParse(csvRaw)
const csvData = rawRows
  .filter(d => d.Code && !isNaN(+d.Share))
  .map(d => ({ iso: d.Code, year: +d.Year, value: +d.Share }))

// 4) Jahresbereich bestimmen
const years = Array.from(new Set(csvData.map(d => d.year))).sort((a, b) => a - b)
const minYear = years[0]
const maxYear = years[years.length - 1]
const year = ref(maxYear)

// 5) Map<ISO, Wert> für aktuelles Jahr
const yearDataMap = computed(() => {
  const m = new Map()
  const seen = new Set()

  // Sortiere Daten nach Jahr absteigend
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


// 6) Label- und Tooltip-Funktionen
const formatLabel = v => d3.format('.0f')(v) + '%'
const getISO = feature => feature.properties['ISO3166-1-Alpha-3']
const getTooltipContent = (feature, value, actualYear) => {
  const name = feature.properties.ADMIN || feature.properties.name || 'Unbekannt'
  if (value == null) {
    return `<strong>${name}</strong><br/>keine Daten`
  }
  const val = d3.format('.1f')(value)
  return `<strong>${name}</strong><br/>${val}% Anteil Erneuerbare Elektrizität<br/><em>Daten von ${actualYear}</em>`
}

</script>
