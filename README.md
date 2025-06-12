# Global Environmental Data Visualization

An interactive web application developed with Vue 3 and D3.js that visualizes global environmental data through dynamic world maps. The application provides insights into climate change, CO2 emissions, and renewable energy usage across different countries.

Webhosted: https://climate-vis.vercel.app/

## Features

- **CO2 Emissions per Capita**: Display and comparison of CO2 emissions per capita across different countries over time
- **Temperature Anomalies**: Visualization of global temperature changes and anomalies in different regions
- **Renewable Energy Share**: Representation of the percentage of renewable energy in total energy consumption by country
- **Renewable Electricity Share**: Display of electricity generation from renewable sources

## Main Components

- Interactive world map visualization using D3.js
- Time slider for temporal data exploration
- Color-coded data representation
- Detailed tooltips with country-specific information

## Technology Stack

- **Vue 3**: Frontend framework with Composition API and `<script setup>`
- **Vite**: Modern frontend development environment
- **D3.js**: Data visualization library
- **CSV and GeoJSON**: Data sources for environmental metrics and geographical information

## Getting Started

1. Clone repository
2. Install dependencies:
   ```bash
   npm install
   ```
3. Start development server:
   ```bash
   npm run dev
   ```

## Data Sources

The application uses various datasets, including:
- Annual CO2 emissions per country
- CO2 emissions per capita
- Temperature anomaly data
- Renewable energy and electricity statistics
- Geographic world data (GeoJSON)

## Project Structure

```
src/
├── assets/         # Data files and other static assets
├── components/     # Vue components
│   ├── WorldMap.vue
│   ├── TimeSlider.vue
│   ├── CO2CapitaView.vue
│   ├── TempView.vue
│   └── ...
└── ...
```
