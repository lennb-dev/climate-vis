<template>
  <div class="stacked-linechart-container">
    <h2 class="text-center text-xl font-semibold">Strommix im zeitlichen Verlauf</h2>
    <Line ref="chartRef" :data="chartData" :options="chartOptions" />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
  Filler
} from 'chart.js'
import * as d3 from 'd3'
import ElectricitySourceRaw from '../assets/share-electricity-source.csv?raw'

// Chart.js Komponenten registrieren
ChartJS.register(
  Title,
  Tooltip,
  Legend,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
  Filler
)

const props = defineProps({
  iso: String
})

const chartRef = ref(null)

// Farbskala & Quellen
const energySources = ['Coal', 'Gas', 'Nuclear', 'Hydro', 'Solar', 'Wind']
const customPalette = {
  Coal: '#444444',
  Gas: '#fc8d62',
  Nuclear: '#66c2a5',
  Hydro: '#4a6484',
  Solar: '#ffd92f',
  Wind: '#a3c0e8'
}

// Rohdaten als reactive Variable
const rawData = ref([])

// Chart Options (stapelbarer Bereichs-Chart)
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  interaction: {
    mode: 'index',
    intersect: false
  },
  plugins: {
    title: { display: false },
    tooltip: { mode: 'index', intersect: false },
    legend: { position: 'bottom' }
  },
  scales: {
    x: {
      title: { display: true, text: 'Jahr' }
    },
    y: {
      stacked: true,
      title: { display: true, text: 'Anteil (%)' },
      min: 0,
      max: 100
    }
  }
}

// CSV Parsing & Typisierung
function parseCSV(rawCSV) {
  const parsed = d3.csvParse(rawCSV)
  return parsed.map(row => ({
    Entity: row.Entity,
    Code: row.Code,
    Year: +row.Year,
    Coal: +row.Coal,
    Gas: +row.Gas,
    Nuclear: +row.Nuclear,
    Hydro: +row.Hydro,
    Solar: +row.Solar,
    Wind: +row.Wind
  }))
}

rawData.value = parseCSV(ElectricitySourceRaw)

// Computed Labels (Jahre)
const chartLabels = computed(() => {
  if (!props.iso || rawData.value.length === 0) return []

  const filtered = rawData.value.filter(d => d.Code === props.iso)
  filtered.sort((a, b) => d3.ascending(a.Year, b.Year))
  return filtered.map(d => d.Year)
})

// Computed Datasets (Energieträger-Daten)
const chartDatasets = computed(() => {
  if (!props.iso || rawData.value.length === 0) return []

  const filtered = rawData.value.filter(d => d.Code === props.iso)
  filtered.sort((a, b) => d3.ascending(a.Year, b.Year))

  return energySources.map(source => ({
    label: source,
    data: filtered.map(d => d[source] ?? 0),
    borderColor: customPalette[source],
    backgroundColor: customPalette[source],
    pointRadius: 0,
    pointHoverRadius: 6,
    fill: true,
    tension: 0.4
  }))
})

// Zusammengesetzte Chart-Daten
const chartData = computed(() => ({
  labels: chartLabels.value,
  datasets: chartDatasets.value
}))
</script>

<style scoped>
.stacked-linechart-container {
  position: relative;
  height: 500px; /* fixe Höhe um Overflow zu verhindern */
  background: white;
  padding: 12px;
  border-radius: 8px;
  color: black;
}
</style>
