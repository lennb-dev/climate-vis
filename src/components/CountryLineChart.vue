<template>
  <div class="bg-white rounded-2xl p-6 max-w-6xl mx-auto w-full relative">
    <h2 class="text-2xl font-semibold text-center mb-4">{{ countryName }}</h2>

    <!-- Metric filter checkboxes -->
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

    <!-- Chart and Legend -->
    <div class="flex">
      <!-- Main Chart -->
      <div class="flex-grow overflow-x-hidden">
        <svg
          ref="chartRef"
          class="w-full h-[500px]"
          viewBox="0 0 1000 500"
          preserveAspectRatio="xMinYMin meet"
        ></svg>
      </div>

      <!-- Legend -->
      <div class="w-40 ml-1 flex flex-col gap-4 text-sm text-gray-800 mt-4">
        <div
          v-for="metric in selectedMetrics"
          :key="metric"
          class="flex gap-2 items-start"
        >
          <span
            class="w-4 h-4 rounded-full flex-shrink-0 mt-1"
            :style="{ backgroundColor: color(metric), display: 'inline-block' }"
          ></span>
          <span class="break-words">{{ metric }}</span>
        </div>
      </div>
    </div>

    <!-- Year slider -->
    <div
      v-if="minYear !== null && maxYear !== null"
      class="flex flex-col items-center gap-2"
    >
      <label for="startYearRange" class="font-medium text-sm text-gray-800">
        Starting year: <span class="font-bold text-gray-800">{{ startYear }}</span>
      </label>
      <input
        id="startYearRange"
        type="range"
        :min="minYear"
        :max="2020"
        v-model.number="startYear"
        class="w-full max-w-xl h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-thumb"
      />
    </div>

    <!-- Tooltip Box -->
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
// Imports
import { ref, onMounted, watch, nextTick } from 'vue'
import * as d3 from 'd3'

// Props & Emits
const props = defineProps({ iso: String, countryName: String })
const emit = defineEmits(['processed-data'])

// References
const chartRef = ref(null)
const tooltip = ref(null)

// Dimensions
const width = ref(10000)
const height = ref(500)

// UI State
const allMetrics = ref([])
const selectedMetrics = ref([])

// Year range
const startYear = ref(null)
const minYear = ref(1940)
const maxYear = ref(null)

// Global extent for temperature
const globalTempExtent = ref([0, 1])

// Color palette & scale
const customPalette = ['#1f77b4', '#d62728', '#ff7f0e', '#2ca02c']
const colorScale = d3.scaleOrdinal().range(customPalette)
function color(metric) {
  return colorScale(metric)
}

// Import raw CSVs
import CO2CapitaRaw from '../assets/co-emissions-per-capita.csv?raw'
import CO2Raw from '../assets/annual-co2-emissions-per-country.csv?raw'
import EnergyShareRaw from '../assets/energyshare-from-renewables.csv?raw'
import ElectricityShareRaw from '../assets/share-electricity-renewables.csv?raw'
import TempRaw from '../assets/annual-temperature-anomalies.csv?raw'

// CSV parsing helper
function parseCSV(raw, valueKey, metricName) {
  const data = d3.csvParse(raw)
  return data.map(row => ({
    iso: row.Code,
    year: +row.Year,
    value: +row[valueKey],
    metric: metricName
  })).filter(d => d.iso && !isNaN(d.value))
}

// Final unified dataset
const processedData = [
  ...parseCSV(CO2Raw, 'CO2', 'CO₂'),
  // ...parseCSV(CO2CapitaRaw, 'CO2', 'CO₂ per capita'),
  ...parseCSV(TempRaw, 'Temperature anomaly', 'Temperature'),
  ...parseCSV(EnergyShareRaw, 'Share', 'Share of renewable energy'),
  ...parseCSV(ElectricityShareRaw, 'Share', 'Share of renewable electricity')
]

// Watch for changes and re-render chart
watch([() => props.iso, selectedMetrics, startYear], drawChart)

// Lifecycle: on mount, initialize layout, state, and draw chart
onMounted(async () => {
  await nextTick()
  const container = chartRef.value?.parentElement
  if (container) {
    width.value = container.getBoundingClientRect().width
    height.value = 500
  }

  const allTempData = processedData.filter(d => d.metric === 'Temperature' && d.iso === props.iso)
  globalTempExtent.value = d3.extent(allTempData, d => d.value)

  const metrics = [...new Set(processedData.map(d => d.metric))]
  allMetrics.value = metrics
  selectedMetrics.value = metrics

  const allYearsSorted = [...new Set(processedData.map(d => d.year))].sort((a, b) => a - b)
  minYear.value = Math.max(1940, allYearsSorted[0])
  maxYear.value = allYearsSorted.at(-1)
  startYear.value = minYear.value

  drawChart()
  emit('processed-data', processedData)
})

// Main chart logic
function drawChart() {
  if (!props.iso) return

  const svg = d3.select(chartRef.value)
  svg.selectAll('*').remove()

  const margin = { top: 20, right: 60, bottom: 60, left: 60 }
  const innerWidth = width.value - margin.left - margin.right
  const innerHeight = height.value - margin.top - margin.bottom

  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`)

  // Filter data for selected country and year range
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

  // Temperature scale
  const yTemp = d3.scaleLinear()
    .domain(globalTempExtent.value)
    .range([innerHeight, 0])
    .nice()

  // Normalized scale (for other metrics)
  const yNorm = d3.scaleLinear().domain([0, 1]).range([innerHeight, 0])

  // Metric-specific normalization ranges
  const groupedFull = d3.group(
    processedData.filter(d => d.iso === props.iso && d.value != null),
    d => d.metric
  )

  const metricsToNormalize = [...grouped.keys()].filter(
    m => m !== 'Temperature' && !m.includes('renewable')
  )

  const metricRanges = {}
  for (const metric of metricsToNormalize) {
    const values = groupedFull.get(metric).map(d => d.value)
    metricRanges[metric] = { min: Math.min(...values), max: Math.max(...values) }
  }

  const x = d3.scaleLinear()
    .domain(d3.extent(allYears))
    .range([0, innerWidth])

  // Draw lines for each selected metric
  for (const [metric, values] of grouped) {
    const sorted = values.sort((a, b) => a.year - b.year)

    const line = d3.line()
      .x(d => x(d.year))
      .y(d => {
        if (metric === 'Temperature') return yTemp(d.value)
        if (metric.includes('renewable')) return yNorm(d.value / 100)
        const { min, max } = metricRanges[metric]
        return yNorm((d.value - min) / (max - min))
      })
      .curve(d3.curveMonotoneX)

    g.append('path')
      .datum(sorted)
      .attr('fill', 'none')
      .attr('stroke', color(metric))
      .attr('stroke-width', 2)
      .attr('d', line)
  }

  // Add axes
  g.append('g').call(d3.axisLeft(yTemp)).selectAll('text').style('font-size', '10px')
  g.append('text')
    .attr('transform', 'rotate(-90)')
    .attr('y', -50)
    .attr('x', -innerHeight / 2)
    .attr('dy', '1em')
    .style('text-anchor', 'middle')
    .style('font-size', '12px')
    .text('Temperature (°C)')

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

  // Tooltip logic
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

    tooltipGroup.selectAll('.tooltip-entry, text:not(#year-label), circle').remove()
    g.selectAll('.focus-point').remove()

    const visibleEntries = []

    selectedMetrics.value.forEach(metric => {
      const data = grouped.get(metric)
      const entry = data?.find(d => d.year === xYear)
      if (!entry) return

      let yVal
      if (metric === 'Temperature') yVal = yTemp(entry.value)
      else if (metric.includes('renewable')) yVal = yNorm(entry.value / 100)
      else {
        const { min, max } = metricRanges[metric]
        yVal = yNorm((entry.value - min) / (max - min))
      }

      const unit = metric === 'Temperature' ? '°C' : metric.includes('renewable') ? '%' : 't'

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
        .text(() =>
          entry.metric === 'CO₂' ? `${entry.metric}: ${formatCO2(entry.value)}` : `${entry.metric}: ${entry.value.toFixed(2)} ${entry.unit}`
        )

      g.append('circle')
        .attr('class', 'focus-point')
        .attr('r', 4)
        .attr('cx', x(xYear))
        .attr('cy', entry.y)
        .attr('fill', color(entry.metric))
        .style('opacity', 1)
    })

    tooltipBg.attr('height', visibleEntries.length * 20 + 30)

    let tooltipX = x(xYear) + 15
    if (tooltipX + 230 > innerWidth) tooltipX = x(xYear) - 230 - 15
    tooltipGroup.attr('transform', `translate(${tooltipX}, 20)`).style('opacity', 1)
  })

  overlay.on('mouseleave', () => {
    focusLine.style('opacity', 0)
    tooltipGroup.style('opacity', 0)
    g.selectAll('.focus-point').remove()
  })

  emit('processed-data', processedData)
}

// Format CO₂ values with SI suffix
function formatCO2(value) {
  if (value >= 1e9) return (value / 1e9).toFixed(2) + ' B t'
  if (value >= 1e6) return (value / 1e6).toFixed(2) + ' M t'
  if (value >= 1e3) return (value / 1e3).toFixed(2) + ' K t'
  return value.toFixed(2) + ' t'
}
</script>
