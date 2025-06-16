If possible check out the README.md for better readability. Or visit: https://github.com/lennb-dev/climate-vis/tree/main

Global Environmental Data Visualization
An interactive web application developed with Vue 3 and D3.js that visualizes global environmental data through dynamic world maps. The application provides insights into climate change, CO₂ emissions, and renewable energy usage across different countries.
Developed by Lennard Beer & Marc Simon Curcic

Webhosted Version: https://climate-vis.vercel.app/

Main Components
1. Data Switcher

    Data Switcher: Toggle between different environmental datasets (e.g., CO₂, temperature, renewables) to update the map display.
    CO₂ Emissions: Display and comparison of country-level CO₂ emissions over time.
    CO₂ Emissions per Capita: Display and comparison of CO₂ emissions per capita across different countries over time.
    Temperature: Visualization of global temperature changes and anomalies in different countries (compared to the mean of the temperatures from 1991-2020).
    Share of Renewable Energy: Representation of the percentage of renewable energy in total energy consumption by country.
    Share of Renewable Electricity: Display of electricity generation from renewable sources, compared to the overall electricity consumption.

2. World Map Visualization

    Color-Encoded Heatmap: Countries are shaded based on dataset values using custom thresholds.
    Custom Thresholds: Each dataset defines custom scale domains to ensure visual clarity across varying value ranges.
    Interactive Legend: Automatically updates with the selected data layer, providing context for color-value mappings.

3. Timeslider

    Timeslider: Select any year to view corresponding data.
    Playback Control: Animate the timeline from the selected year to the dataset’s end using play/pause functionality.

4. Tooltips

    Hover Details: Hover over a country to display a line chart showing its historical data for the selected dataset, using dynamic colors that match the current legend scale.
    Legend Interaction: Hovering over legend entries highlights all countries within that threshold range on the map.
    Country Dashboard: Clicking a country opens a detailed dashboard view with extended statistics and charts.

5. Country Dashboard

    Multi-Dataset Line Chart: Visualizes all available datasets for the selected country over the full time range. Users can:
        Adjust the start year with a slider below the chart.
        Toggle individual datasets on/off via buttons.
        Temperature data is plotted on a separate °C axis, while CO₂, Renewable Energy Share, and Renewable Electricity Share are shown on a 0–100% scale.
        CO₂ emissions are min-max normalized per country for better comparative scaling.
        Hover over the line chart to get exact values at the respective year.

    CO₂ Emissions by Sector (Donut Chart): Displays the distribution of CO₂ emissions across sectors (e.g., transport, industry, energy).
        Hovering over a sector highlights it in the legend and shows the percentage share inside the donut.

    Electricity Share over Time (Stacked Line Chart): Shows the evolution of electricity generation from different sources (e.g., fossil, hydro, solar, wind) over time.
        Focuses only on domestically produced electricity, excluding imports.

Technology Stack

    Vue 3: Frontend framework with Composition API and <script setup>
    Vite: Modern frontend development environment
    Tailwind CSS: Utility‑first styling framework for fast, responsive design directly in Vue 3 templates
    D3.js: Data visualization library
    ChartJS: Stacked line chart visualisation
    CSV and GeoJSON: Data sources for environmental metrics and geographical information

Getting Started
	1. Clone repository
	2. Install node.js and npm (windows: https://nodejs.org/en)
	3. Install dependencies: npm install
	4. Start development server: npm run dev

Data Sources
The application uses various datasets, including:
    Geographic world data (GeoJSON: https://github.com/nvkelso/natural-earth-vector)
    Annual CO₂ emissions per country (https://ourworldindata.org/grapher/annual-co2-emissions-per-country)
    CO₂ emissions per capita (https://ourworldindata.org/grapher/co-emissions-per-capita)
    Temperature anomaly data (https://ourworldindata.org/grapher/annual-temperature-anomalies)
    Renewable energy statistics (https://ourworldindata.org/grapher/share-of-final-energy-consumption-from-renewable-sources)
    Renewable electricity statistics (https://ourworldindata.org/grapher/share-electricity-renewables)
    CO₂ emissions by sector (https://ourworldindata.org/grapher/co-emissions-by-sector)
    Electricity share over time (https://ourworldindata.org/grapher/share-elec-by-source)

Project Structure
src/
├── assets/         # Data files and other static assets
├── components/     # Vue components
│   ├── WorldMap.vue
│   ├── TimeSlider.vue
│   ├── DataSwitcher.vue
│   ├── CO2View.vue
│   ├── CO2CapitaView.vue
│   ├── TempView.vue
│   ├── RenewableEnergyView.vue
│   ├── RenewableElectricityView.vue
│   ├── CountryDashboard.vue
│   ├── CountryLineChart.vue
│   ├── CountryPieChart.vue
│   └── CountryStackedLineChart.vue
└── app.vue