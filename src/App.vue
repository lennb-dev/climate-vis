<template>
  <div class="h-screen w-screen">
  <div class="pt-10">

    <DataSwitcher
      v-model="currentView"
      :options="viewOptions"
    />
  </div>
    <KeepAlive>
      <component :is="viewComponent" />
    </KeepAlive>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'


// Import components
import DataSwitcher from './components/DataSwitcher.vue'
import CO2View from './components/CO2View.vue'
import CO2CapitaView from './components/CO2CapitaView.vue'
import TempView from './components/TempView.vue'
import RenewableEnergyView from './components/RenewableEnergyView.vue'
import RenweableElectricityView from './components/RenweableElectricityView.vue'

// Current view state
const currentView = ref('co2')

// View options for the DataSwitcher
const viewOptions = [
  { label: 'CO₂ emissions', value: 'co2' },
  { label: 'CO₂ per capita', value: 'co2pc' },
  { label: 'Temperature',       value: 'temp' },
  { label: 'Share of renewable energy', value: 'eEnergy' },
  { label: 'Share of renewable electricity', value: 'eElectricity' }
]

// Computed: dynamically select the view component based on currentView
const viewComponent = computed(() => {
  switch (currentView.value) {
    case 'temp':
      return TempView
    case 'co2pc':
      return CO2CapitaView
    case 'eEnergy':
      return RenewableEnergyView
    case 'eElectricity':
      return RenweableElectricityView
    case 'co2':
    default:
      return CO2View
  }
})

</script>
