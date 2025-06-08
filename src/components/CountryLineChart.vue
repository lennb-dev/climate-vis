<template>
  <div class="bg-white rounded-2xl  p-6 max-w-6xl mx-auto w-full relative">
    <h2 class="text-2xl font-semibold text-center mb-4">{{ countryName }}</h2>

    <!-- Checkbox Filter -->
    <div class="flex flex-wrap justify-center gap-4 mb-6">
      <label
        v-for="metric in allMetrics"
        :key="metric"
        class="flex items-center gap-2 text-sm text-gray-700"
      >
        <input
          type="checkbox"
          :value="metric"
          v-model="selectedMetrics"
          class="accent-blue-600 w-4 h-4"
        />
        {{ metric }}
      </label>
    </div>

    <!-- Chart + Legend Section -->
    <div class="flex">
      <!-- Chart Section -->
      <div class="flex-grow overflow-x-hidden">
        <svg
          ref="chartRef"
          class="w-full h-[500px]"
          viewBox="0 0 1000 500"
          preserveAspectRatio="xMinYMin meet"
        ></svg>
      </div>      <!-- Legend (right side) -->
      <div class="w-40 ml-1 flex flex-col gap-4 text-sm text-gray-800 mt-4">
        <div
          v-for="metric in selectedMetrics"
          :key="metric"
          class="flex items-center gap-2"
        >
          <span
            class="inline-block w-4 h-4 rounded-full"
            :style="{ backgroundColor: color(metric) }"
          ></span>
          {{ metric }}
        </div>
      </div>
    </div>

    <!-- Year Slider -->
    <div
      v-if="minYear !== null && maxYear !== null"
      class=" flex flex-col items-center gap-2"
    >
      <label for="startYearRange" class="font-medium text-sm text-gray-800">
        Startjahr: <span class="font-bold text-gray-800">{{ startYear }}</span>
      </label>
      <input
        id="startYearRange"
        type="range"
        :min="minYear"
        :max=2020
        v-model.number="startYear"
        class="w-full max-w-xl h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-thumb"
      />
    </div>

    <!-- Tooltip -->
    <div
      ref="tooltip"
      class="tooltip absolute bg-white border border-gray-300 p-2 text-xs shadow-lg pointer-events-none z-10"
      v-show="tooltipVisible"
      :style="tooltipStyle"
    >
      {{ tooltipText }}
    </div>
  </div>
</template>



<script setup>
import { ref, onMounted, watch, nextTick, computed } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  iso: String,
  countryName: String
})

const emit = defineEmits(['processed-data'])

const chartRef = ref(null)
const width = ref(10000)
const height = ref(500)

const allMetrics = ref([])
const selectedMetrics = ref([])

// Für Timeslider
const startYear = ref(null)
const minYear = ref(1940)
const maxYear = ref(null)

const tooltip = ref(null)

const globalTempExtent = ref([0, 1]) 

const customPalette = [
    '#1f77b4',
    '#d62728',
    '#ff7f0e',
    '#2ca02c',
  ]

  const colorScale = d3.scaleOrdinal()
    .domain(allMetrics.value) // oder setze es dynamisch später
    .range(customPalette)

  function color(metric) {
    return colorScale(metric)
  }


// CSV-Rohdaten importieren
import CO2CapitaRaw from '../assets/co-emissions-per-capita.csv?raw'
import CO2Raw from '../assets/annual-co2-emissions-per-country.csv?raw'
import EnergyShareRaw from '../assets/energyshare-from-renewables.csv?raw'
import ElectricityShareRaw from '../assets/share-electricity-renewables.csv?raw'
import TempRaw from '../assets/annual-temperature-anomalies.csv?raw'

// CSVs parsen
function parseCSV(raw, valueKey, metricName) {
  const data = d3.csvParse(raw)
  return data.map(row => ({
    iso: row.Code,
    year: +row.Year,
    value: +row[valueKey],
    metric: metricName
  })).filter(d => d.iso && !isNaN(d.value))
}

const processedData = [
  ...parseCSV(CO2Raw, 'CO2', 'CO₂'),
  //...parseCSV(CO2CapitaRaw, 'CO2', 'CO₂ pro Kopf'),
  ...parseCSV(TempRaw, 'Temperature anomaly', 'Temperatur'),
  ...parseCSV(EnergyShareRaw, 'Share', 'Anteil erneuerbarer Energie'),
  ...parseCSV(ElectricityShareRaw, 'Share', 'Anteil erneuerbaren Stroms')
]

// Auf ISO, Metriken und Slider-Änderungen reagieren
watch(
  [() => props.iso, selectedMetrics, startYear], 
  drawChart, 
  { immediate: false }
)

onMounted(async () => {
  await nextTick()
  const container = chartRef.value?.parentElement
  if (container) {
    const bounds = container.getBoundingClientRect()
    width.value = bounds.width
    height.value = 500
  }

  const allTempData = processedData.filter(d => d.metric === 'Temperatur' && d.iso === props.iso)
  globalTempExtent.value = d3.extent(allTempData, d => d.value)
  const metrics = [...new Set(processedData.map(d => d.metric))]
  allMetrics.value = metrics
  selectedMetrics.value = metrics

  const allYearsSorted = [...new Set(processedData.map(d => d.year))].sort((a,b) => a - b)
  minYear.value = allYearsSorted[0] < 1940 ? 1940 : allYearsSorted[0]
  maxYear.value = allYearsSorted[allYearsSorted.length - 1]
  startYear.value = minYear.value

  drawChart()
  emit('processed-data', processedData)
})

function drawChart() {
  if (!props.iso) return

  const svg = d3.select(chartRef.value)
  svg.selectAll('*').remove()
  const margin = { top: 20, right: 60, bottom: 60, left: 60 }
  const innerWidth = width.value - margin.left - margin.right
  const innerHeight = height.value - margin.top - margin.bottom

  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`)

  const filtered = processedData.filter(d =>
    d.iso === props.iso &&
    d.value != null &&
    d.year >= startYear.value &&
    d.year <= maxYear.value
  )

  const grouped = d3.group(
    filtered.filter(d => selectedMetrics.value.includes(d.metric)),
    d => d.metric
  )
  const allYears = [...new Set(filtered.map(d => d.year))].sort()

  const tempData = grouped.get('Temperatur') || []
  const tempExtent = d3.extent(tempData, d => d.value)

  const groupedFull = d3.group(
    processedData.filter(d => d.iso === props.iso && d.value != null),
    d => d.metric
  )

  const metricsToNormalize = [...grouped.keys()].filter(
    m => m !== 'Temperatur' && !['Anteil erneuerbarer Energie', 'Anteil erneuerbaren Stroms'].includes(m)
  )
  const metricRanges = {}
  for (const metric of metricsToNormalize) {
    const vals = groupedFull.get(metric).map(d => d.value)
    metricRanges[metric] = {
      min: Math.min(...vals),
      max: Math.max(...vals)
    }
  }

  const x = d3.scaleLinear().domain(d3.extent(allYears)).range([0, innerWidth])

  const yTemp = d3.scaleLinear()
    .domain(globalTempExtent.value)
    .range([innerHeight, 0])
    .nice()


  const yNorm = d3.scaleLinear().domain([0, 1]).range([innerHeight, 0])



  for (const [metric, values] of grouped) {
    const sorted = values.sort((a, b) => a.year - b.year)
    let lineGenerator

    if (metric === 'Temperatur') {
      lineGenerator = d3.line()
        .x(d => x(d.year))
        .y(d => yTemp(d.value))
        .curve(d3.curveMonotoneX)
    } else if (['Anteil erneuerbarer Energie', 'Anteil erneuerbaren Stroms'].includes(metric)) {
      lineGenerator = d3.line()
        .x(d => x(d.year))
        .y(d => yNorm(d.value / 100))
        .curve(d3.curveMonotoneX)
    } else {
      const { min, max } = metricRanges[metric]
      lineGenerator = d3.line()
        .x(d => x(d.year))
        .y(d => yNorm((d.value - min) / (max - min)))
        .curve(d3.curveMonotoneX)
    }

    g.append('path')
      .datum(sorted)
      .attr('fill', 'none')
      .attr('stroke', color(metric))
      .attr('stroke-width', 2)
      .attr('d', lineGenerator)
  }

  g.append('g').call(d3.axisLeft(yTemp)).selectAll('text').style('font-size', '10px')
  g.append('text')
  .attr('transform', 'rotate(-90)')
  .attr('y', -50)           // Position links von Achse anpassen
  .attr('x', -innerHeight / 2)
  .attr('dy', '1em')
  .style('text-anchor', 'middle')
  .style('font-size', '12px')
  .text('Temperatur (°C)')


  g.append('g')
    .attr('transform', `translate(${innerWidth},0)`)
    .call(d3.axisRight(yNorm).ticks(5).tickFormat(d3.format('.0%')))
    .selectAll('text')
    .style('font-size', '10px')

  g.append('g')
    .attr('transform', `translate(0,${innerHeight})`)
    .call(d3.axisBottom(x).ticks(6).tickFormat(d3.format('d')))
    .selectAll('text')
    .style('font-size', '10px')

  // Tooltip + Hover
  const focusLine = g.append('line')
    .attr('class', 'hover-line')
    .attr('y1', 0)
    .attr('y2', innerHeight)
    .attr('stroke', '#888')
    .attr('stroke-width', 1)
    .attr('stroke-dasharray', '4 2')
    .style('opacity', 0)

  const tooltipGroup = g.append('g')
    .attr('class', 'tooltip-group')
    .style('opacity', 0)

  const tooltipBg = tooltipGroup.append('rect')
    .attr('fill', '#f9f9f9')
    .attr('stroke', '#ccc')
    .attr('rx', 6)
    .attr('ry', 6)
    .attr('width', 230)
    .attr('height', 80)
    .attr('x', 0)
    .attr('y', 0)
    .attr('filter', 'drop-shadow(0 1px 4px rgba(0,0,0,0.2))')

  const yearLabel = tooltipGroup.append('text')
    .attr('id', 'year-label')
    .attr('x', 8)
    .attr('y', 14)
    .attr('font-weight', 'bold')
    .attr('font-size', '13px')
    .attr('fill', '#333')

  const overlay = g.append('rect')
    .attr('width', innerWidth)
    .attr('height', innerHeight)
    .attr('fill', 'none')
    .attr('pointer-events', 'all')

  overlay.on('mousemove', function (event) {
    const [mouseX] = d3.pointer(event)
    const xYear = Math.round(x.invert(mouseX))
    if (!allYears.includes(xYear)) return

    focusLine.attr('x1', x(xYear)).attr('x2', x(xYear)).style('opacity', 1)
    yearLabel.text(`${xYear}`)

    tooltipGroup.selectAll('.tooltip-entry').remove()
    tooltipGroup.selectAll('text:not(#year-label)').remove()
    tooltipGroup.selectAll('circle').remove()
    g.selectAll('.focus-point').remove()

    let visibleEntries = []

    selectedMetrics.value.forEach(metric => {
      const data = grouped.get(metric)
      const entry = data?.find(d => d.year === xYear)
      if (!entry) return

      let yVal
      if (metric === 'Temperatur') yVal = yTemp(entry.value)
      else if (['Anteil erneuerbarer Energie', 'Anteil erneuerbaren Stroms'].includes(metric)) {
        yVal = yNorm(entry.value / 100)
      } else {
        const { min, max } = metricRanges[metric]
        yVal = yNorm((entry.value - min) / (max - min))
      }

      let unit = ''
      if (metric === 'Temperatur') unit = '°C'
      else if (['Anteil erneuerbarer Energie', 'Anteil erneuerbaren Stroms'].includes(metric)) unit = '%'
      else unit = 't'

      visibleEntries.push({ metric, value: entry.value, unit, y: yVal })
    })

    visibleEntries.forEach((entry, i) => {
      tooltipGroup.append('g')
        .attr('class', 'tooltip-entry')
        .append('circle')
        .attr('cx', 12)
        .attr('cy', 30 + i * 20)
        .attr('r', 5)
        .attr('fill', color(entry.metric))

      tooltipGroup.append('text')
        .attr('x', 24)
        .attr('y', 34 + i * 20)
        .attr('font-size', '12px')
        .attr('fill', '#333')
        .text(() => {
          if (entry.metric === 'CO₂' || entry.metric === 'CO₂ pro Kopf') {
            return `${entry.metric}: ${formatCO2(entry.value)}`
          } else {
            return `${entry.metric}: ${entry.value.toFixed(2)} ${entry.unit}`
          }
        })

      g.append('circle')
        .attr('class', 'focus-point')
        .attr('r', 4)
        .attr('cx', x(xYear))
        .attr('cy', entry.y)
        .attr('fill', color(entry.metric))
        .style('opacity', 1)
    })

    tooltipBg.attr('height', visibleEntries.length * 20 + 30)

    // Intelligente Positionierung
    const tooltipWidth = 230
    let tooltipX = x(xYear) + 15
    if (tooltipX + tooltipWidth > innerWidth) {
      tooltipX = x(xYear) - tooltipWidth - 15
    }

    tooltipGroup
      .attr('transform', `translate(${tooltipX}, 20)`)
      .style('opacity', 1)
  })

  overlay.on('mouseleave', () => {
    focusLine.style('opacity', 0)
    tooltipGroup.style('opacity', 0)
    g.selectAll('.focus-point').remove()
  })
  emit('processed-data', processedData)
}

function formatCO2(value) {
  if (value >= 1e9) return (value / 1e9).toFixed(2) + ' Mrd t'
  if (value >= 1e6) return (value / 1e6).toFixed(2) + ' Mio t'
  if (value >= 1e3) return (value / 1e3).toFixed(2) + ' Tsd t'
  return value.toFixed(2) + ' t'
}


</script>
