<template>
  <div class="piechart-container">
    <h3 class="text-xl font-semibold">CO₂ emissions by sector ({{ latestYear }})</h3>
    <div class="chart-wrapper">
      <template v-if="hasData">
        <!-- SVG pie chart -->
        <svg ref="pieRef" :width="width" :height="height"></svg>

        <!-- Legend for sector colors -->
        <div class="legend">
          <div
            v-for="sector in sectors"
            :key="sector.name"
            class="legend-item"
            :class="{ highlight: hoveredSector === sector.name }"
          >
            <span
              class="legend-color"
              :style="{ backgroundColor: color(sector.name) }"
            ></span>
            <span class="legend-label">{{ sector.name }}</span>
          </div>
        </div>
      </template>

      <!-- No data fallback -->
      <template v-else>
        <div class="no-data-message">
          No data
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import * as d3 from 'd3'
import { ref, watch, computed, onMounted } from 'vue'
import CO2SectorRaw from '../assets/co-emissions-by-sector.csv?raw' // CSV as raw text

// Props
const props = defineProps({
  iso: String
})

// Chart config
const pieRef = ref(null)
const width = 400
const height = 400
const radius = Math.min(width, height) / 2
const latestYear = 2021
const hoveredSector = ref(null)

// Processed sector data
const processedData = ref([])

// Parse CSV data into numeric format
function parseCSV(rawCSV) {
  const parsed = d3.csvParse(rawCSV)
  return parsed.map(row => {
    row.year = +row.Year
    row['buildings'] = +row['buildings']
    row['industry'] = +row['industry']
    row['land use change and forestry'] = +row['land use change and forestry']
    row['other fuel combustion'] = +row['other fuel combustion']
    row['transport'] = +row['transport']
    row['manufacturing and construction'] = +row['manufacturing and construction']
    row['fugitive emissions from energy production'] = +row['fugitive emissions from energy production']
    row['electricity and heat'] = +row['electricity and heat']
    row['bunker fuels'] = +row['bunker fuels']
    return row
  })
}

processedData.value = parseCSV(CO2SectorRaw)

// Color scale by sector
const fixedColors = {
  'Buildings': d3.schemeSet3[0],
  'Industry': d3.schemeSet3[9],
  'Land Use and Forestry': d3.schemeSet3[2],
  'Other Fuel Combustion': d3.schemeSet3[3],
  'Transport': d3.schemeSet3[4],
  'Manufacturing & Construction': d3.schemeSet3[5],
  'Fugitive Emissions': d3.schemeSet3[6],
  'Electricity & Heat': d3.schemeSet3[11],
  'Bunker Fuels': d3.schemeSet3[8]
}

// Get color for a given sector
function color(sectorName) {
  return fixedColors[sectorName]
}

// Computed: Filtered and structured sector data for the selected country
const sectors = computed(() => {
  if (!props.iso || !processedData.value.length) return []

  const countryData = processedData.value.filter(d => d.Code === props.iso)
  const latest = countryData.find(d => d.year === latestYear)
  if (!latest) return []

  return [
    { name: 'Buildings', value: latest['buildings'] },
    { name: 'Industry', value: latest['industry'] },
    { name: 'Land Use and Forestry', value: latest['land use change and forestry'] },
    { name: 'Other Fuel Combustion', value: latest['other fuel combustion'] },
    { name: 'Transport', value: latest['transport'] },
    { name: 'Manufacturing & Construction', value: latest['manufacturing and construction'] },
    { name: 'Fugitive Emissions', value: latest['fugitive emissions from energy production'] },
    { name: 'Electricity & Heat', value: latest['electricity and heat'] },
    { name: 'Bunker Fuels', value: latest['bunker fuels'] }
  ].filter(s => s.value != null && !isNaN(s.value) && s.value > 0)
})

// Computed: Whether we have data to show
const hasData = computed(() => sectors.value.length > 0)

// D3 pie chart rendering
function drawPie() {
  if (!pieRef.value || sectors.value.length === 0) return

  const svg = d3.select(pieRef.value)
  svg.selectAll('*').remove()

  const g = svg.append('g')
    .attr('transform', `translate(${width / 2},${height / 2})`)

  const pie = d3.pie().value(d => d.value).sort(null)
  const arc = d3.arc()
    .innerRadius(radius - 140)
    .outerRadius(radius - 30)
  const arcHover = d3.arc()
    .innerRadius(radius - 140)
    .outerRadius(radius - 20)

  const centerText = g.append('text')
    .attr('text-anchor', 'middle')
    .attr('dy', '0.35em')
    .style('font-size', '20px')
    .style('font-weight', 'bold')
    .text('')

  const total = d3.sum(sectors.value, d => d.value)

  const arcs = g.selectAll('g.arc')
    .data(pie(sectors.value))
    .enter().append('g')
    .attr('class', 'arc')

  arcs.append('path')
    .attr('d', arc)
    .attr('fill', d => color(d.data.name))
    .style('filter', 'drop-shadow(2px 2px 2px rgba(0,0,0,0.2))')
    .on('mouseover', function (event, d) {
      d3.select(this)
        .transition()
        .duration(300)
        .attr('d', arcHover)

      hoveredSector.value = d.data.name
      const percent = ((d.data.value / total) * 100).toFixed(1) + '%'
      centerText.text(percent)
    })
    .on('mouseout', function () {
      d3.select(this)
        .transition()
        .duration(300)
        .attr('d', arc)

      hoveredSector.value = null
      centerText.text('')
    })
}

// Redraw chart on data/prop change
watch([() => props.iso, sectors], drawPie, { immediate: true })

// Initial render
onMounted(() => {
  drawPie()
})
</script>

<style scoped>
.piechart-container {
  background: white;
  padding: 12px;
  border-radius: 8px;
  color: black;
}

.piechart-container h3 {
  text-align: center;
  margin-bottom: 10px;
}

.chart-wrapper {
  display: flex;
  align-items: center;
  gap: 30px;
}

.legend {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-width: 200px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 14px;
}

.legend-color {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  flex-shrink: 0;
}

.legend-item.highlight {
  font-weight: bold;
  background-color: #f0f0f0;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.no-data-message {
  width: 100%;
  text-align: center;
  font-size: 18px;
  color: #666;
  padding: 40px;
}
</style>
