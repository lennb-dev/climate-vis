<template>
  <!-- Map Bereich -->
  <div v-show="!selectedCountryISO" class="relative flex flex-col items-center max-w-[960px] mx-auto">
    <svg ref="svgRef" :width="width" :height="height" class="block"></svg>
    <svg ref="legendRef" :width="width" height="60" class="mt-2.5 block"></svg>
  </div>

  <!-- Dashboard Bereich -->
  <div v-if="selectedCountryISO" class="p-4 bg-gray-100 absolute top-0 left-0 w-full h-full z-50 overflow-y-auto">
    <Dashboard
      :iso="selectedCountryISO"
      :countryName="selectedCountryName"
      @close="() => { selectedCountryISO = null; selectedCountryName = '' }"
    />
  </div>

  <!-- Tooltip -->
  <div
    v-if="tooltip.visible"
    :style="{ top: tooltip.y + 'px', left: tooltip.x + 'px' }"
    class="absolute bg-white border border-gray-700 p-1.5 text-xs shadow-md rounded pointer-events-none text-gray-800 z-50"
  >
    <div v-html="tooltip.content"></div>
    <svg
      ref="tooltipChart"
      :width="tooltipChartWidth"
      :height="tooltipChartHeight"
      class="block overflow-visible"
    ></svg>
  </div>
</template>

<script setup>
import Dashboard from './CountryDashboard.vue'
import { ref, reactive, onMounted, watch, nextTick, defineProps } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  geoData: Array,
  year: Number,
  dataMap: Object,
  actualYearMap: Object,
  thresholds: Array,
  colors: Array,
  labelFormat: { type: Function, default: v => v },
  getISO: Function,
  getTooltipContent: Function,
  data: Array,
  valueKey: String
})

const width = 960
const height = 700
const svgRef = ref(null)
const legendRef = ref(null)
const tooltipChart = ref(null)
const tooltipChartWidth = 250
const tooltipChartHeight = 100
const selectedCountryISO = ref(null)
const selectedCountryName = ref('')

const tooltip = reactive({ visible: false, x: 0, y: 0, content: '' })

let colorScale, projection, pathGenerator, mapGroup

function initScales() {
  colorScale = d3.scaleThreshold()
    .domain(props.thresholds)
    .range(props.colors)
}

function initMap() {
  projection = d3.geoMercator().scale(140).translate([width / 2, height / 1.7])
  pathGenerator = d3.geoPath().projection(projection)

  const svg = d3.select(svgRef.value)
  mapGroup = svg.append('g')
}

function drawInitialMap() {
  mapGroup
    .selectAll('path.country')
    .data(props.geoData, d => props.getISO(d))
    .join(
      enter => enter
        .append('path')
          .attr('class', 'country')
          .attr('d', pathGenerator)
          .attr('stroke', '#000')
          .attr('stroke-width', 0.5)
          .on('mouseover', onMouseOver)
          .on('mousemove', onMouseMove)
          .on('click', onMouseClick)
          .on('mouseout', onMouseOut),
      update => update,
      exit => exit.remove()
    )
  updateFills()
  drawLegend()
}

function updateFills() {
  mapGroup
    .selectAll('path.country')
    .attr('fill', d => {
      const iso = props.getISO(d)
      const val = props.dataMap.get(iso)
      return val != null ? colorScale(val) : 'url(#legendHatch)'
    })
    .attr('fill-opacity', d => {
      const iso = props.getISO(d)
      const actualYear = props.actualYearMap.get(iso)
      // hier kannst du noch Logik für fill-opacity ergänzen, falls gewünscht
      return 1
    })
}

function drawLegend() {
  const svgL = d3.select(legendRef.value)
  svgL.selectAll('*').remove()

  svgL.append('defs')
    .append('pattern')
      .attr('id', 'legendHatch')
      .attr('patternUnits', 'userSpaceOnUse')
      .attr('width', 8)
      .attr('height', 8)
    .append('path')
      .attr('d', 'M0,0 l8,8')
      .attr('stroke', '#999')
      .attr('stroke-width', 1)

  const legendMargin = { left: 30, right: 30 }
  const legendWidth = width - legendMargin.left - legendMargin.right
  const segW = legendWidth / props.colors.length
  props.colors.forEach((color, i) => {
    const fill = i === 0 ? 'url(#legendHatch)' : color
    svgL.append('rect')
      .attr('x', legendMargin.left + (i * segW))
      .attr('y', 0)
      .attr('width', segW)
      .attr('height', 10)
      .attr('fill', fill)
      .attr('class', 'legend-block')
      .attr('data-color', fill)
      .on('mouseover', () => highlightCountriesByColor(fill))
      .on('mouseout', resetHighlight)
  })

  const xScale = d3.scaleLinear()
    .domain([0, props.colors.length])
    .range([legendMargin.left, width - legendMargin.right])

  const axis = d3.axisBottom(xScale)
    .tickValues(d3.range(props.colors.length))
    .tickFormat(i => {
      if (i === 0) return 'No Data'
      const v = props.thresholds[i - 1]
      switch(props.valueKey) {
        case 'CO2':
          return v >= 1e9 ? `${v/1e9}B t` : v >= 1e6 ? `${v/1e6}M t` : `${v} t`
        case 'Temperature':
          return `${d3.format('.1f')(v)}°C`
        case 'ElectricityShare':
        case 'EnergyShare':
          return `${d3.format('.0f')(v)}%`
        default:
          return d3.format('.1f')(v)
      }
    })

  svgL.append('g')
    .attr('transform', 'translate(0,12)')
    .call(axis)
    .selectAll('text')
      .style('font-size', '10px')
      .style('fill', '#000')
      .attr('dy', '0.6em')

  svgL.selectAll('path,line').style('stroke', '#000')

  svgL.append('rect')
  .attr('x', legendMargin.left)
  .attr('y', 0)
  .attr('width', width - legendMargin.left - legendMargin.right)
  .attr('height', 10)
  .attr('fill', 'none')
  .attr('stroke', '#000')
}

function highlightCountriesByColor(targetColor) {
  d3.select(svgRef.value).selectAll('path.country').each(function(feature) {
    const iso = props.getISO(feature)
    const val = props.dataMap.get(iso)
    const fill = val != null ? colorScale(val) : 'url(#legendHatch)'
    const isNoData = fill === 'url(#legendHatch)'
    const match = (targetColor === 'url(#legendHatch)' && isNoData) || (!isNoData && fill === targetColor)
    d3.select(this)
      .attr('stroke-width', match ? 2 : 0.5)
      .style('fill-opacity', match ? 1 : 0.2)
      .style('stroke-opacity', match ? 1 : 0.2)
  })
  d3.select(legendRef.value).selectAll('.legend-block')
    .attr('stroke', function() {
      return d3.select(this).attr('data-color') === targetColor ? '#000' : 'none'
    })
}

function resetHighlight() {
  d3.select(svgRef.value).selectAll('path.country')
    .attr('stroke-width', 0.5).style('fill-opacity', 1).style('stroke-opacity', 1)
  d3.select(legendRef.value).selectAll('.legend-block')
    .attr('stroke', 'none')
}

function onMouseOver(event, feature) {
  const iso = props.getISO(feature)
  const val = props.dataMap.get(iso)
  const actualYear = props.actualYearMap.get(iso)
  tooltip.content = props.getTooltipContent(feature, val, actualYear)
  tooltip.visible = true
  nextTick(() => drawTooltipChart(iso))
  d3.select(event.currentTarget).raise().attr('stroke-width', 2)
}

function onMouseMove(event) {
  tooltip.x = event.pageX - 300
  tooltip.y = event.pageY - 80
}

function onMouseOut(event) {
  tooltip.visible = false
  d3.select(event.currentTarget).attr('stroke-width', 0.5)
}

function onMouseClick(event, feature) {
  selectedCountryISO.value = props.getISO(feature)
  selectedCountryName.value = feature.properties.ADMIN || feature.properties.name
}

onMounted(() => {
  initScales()
  initMap()
  drawInitialMap()
})

watch(() => props.geoData, (newGeo) => {
  if (newGeo && newGeo.length) drawInitialMap()
}, { immediate: true })

watch([
  () => props.year,
  () => props.dataMap
], () => updateFills())

function drawTooltipChart(iso) {
  const svg = d3.select(tooltipChart.value)
  svg.selectAll('*').remove()

  const series = props.data
    .filter(d => d.iso === iso && d.value != null)
    .sort((a, b) => a.year - b.year)
  if (!series.length) return

  const margin = { top: 5, right: 5, bottom: 20, left: 50 }
  const w = tooltipChartWidth - margin.left - margin.right
  const h = tooltipChartHeight - margin.top - margin.bottom

  const x = d3.scaleLinear()
    .domain(d3.extent(series, d => d.year))
    .range([0, w])

  const yMin = d3.min(series, d => d.value)
  const yMax = d3.max(series, d => d.value)
  const y = d3.scaleLinear()
    .domain([yMin, yMax])
    .nice()
    .range([h, 0])

  const line = d3.line()
    .x(d => x(d.year))
    .y(d => y(d.value))
    .curve(d3.curveMonotoneX)

  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`)

  // Zeichne jedes Liniensegment mit der Farbe des zweiten Punktes
  for (let i = 1; i < series.length; i++) {
    const segment = [series[i - 1], series[i]]
    g.append('path')
    .datum(segment)
    .attr('fill', 'none')
    .attr('stroke', '#999')
    .attr('stroke-width', 3.2)
    .attr('stroke-linejoin', 'round')
    .attr('stroke-linecap', 'round')
    .attr('d', line)

    g.append('path')
      .datum(segment)
      .attr('fill', 'none')
      .attr('stroke', colorScale(series[i].value))
      .attr('stroke-width', 3)
      .attr('stroke-linejoin', 'round')
      .attr('stroke-linecap', 'round')
      .attr('d', line)
  }

  g.append('g')
    .attr('transform', `translate(0,${h})`)
    .call(d3.axisBottom(x).ticks(5).tickFormat(d3.format('d')))
    .selectAll('text')
    .style('font-size', '10px')

  function formatYAxisTick(value) {
    const absVal = Math.abs(value)
    if (absVal >= 1e9) return (value / 1e9).toFixed(1) + ' B'
    if (absVal >= 1e6) return (value / 1e6).toFixed(1) + ' M'
    if (absVal >= 1e3) return (value / 1e3).toFixed(1) + ' K'
    return value.toFixed(0)
  }

  g.append('g')
    .call(d3.axisLeft(y).ticks(5).tickFormat(formatYAxisTick))
    .selectAll('text')
    .style('font-size', '10px')
}

</script>
