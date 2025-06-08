# Globale Umweltdaten-Visualisierung

Eine interaktive Webanwendung, entwickelt mit Vue 3 und D3.js, die globale Umweltdaten durch dynamische Weltkarten visualisiert. Die Anwendung bietet Einblicke in Klimawandel, CO2-Emissionen und die Nutzung erneuerbarer Energien in verschiedenen Ländern.
Webhosted: https://climate-vis.vercel.app/

## Funktionen

- **CO2-Emissionen pro Kopf**: Anzeige und Vergleich von CO2-Emissionen pro Kopf in verschiedenen Ländern im Zeitverlauf
- **Temperaturanomalien**: Visualisierung globaler Temperaturveränderungen und -anomalien in verschiedenen Regionen
- **Anteil erneuerbarer Energien**: Darstellung des prozentualen Anteils erneuerbarer Energien am Gesamtenergieverbrauch nach Ländern
- **Anteil erneuerbarer Elektrizität**: Anzeige des Anteils von Strom aus erneuerbaren Quellen

## Hauptkomponenten

- Interaktive Weltkarten-Visualisierung mit D3.js
- Zeitschieberegler für zeitliche Datenexploration
- Farbcodierte Datenrepräsentation
- Detaillierte Tooltips mit länderspezifischen Informationen

## Technologie-Stack

- **Vue 3**: Frontend-Framework mit Composition API und `<script setup>`
- **Vite**: Moderne Frontend-Entwicklungsumgebung
- **D3.js**: Bibliothek für Datenvisualisierung
- **CSV und GeoJSON**: Datenquellen für Umweltmetriken und geografische Informationen

## Erste Schritte

1. Repository klonen
2. Abhängigkeiten installieren:
   ```bash
   npm install
   ```
3. Entwicklungsserver starten:
   ```bash
   npm run dev
   ```

## Datenquellen

Die Anwendung verwendet verschiedene Datensätze, darunter:
- Jährliche CO2-Emissionen pro Land
- CO2-Emissionen pro Kopf
- Temperaturanomalien-Daten
- Statistiken zu erneuerbaren Energien und Elektrizität
- Geografische Weltdaten (GeoJSON)

## Projektstruktur

```
src/
├── assets/         # Datendateien und andere statische Assets
├── components/     # Vue-Komponenten
│   ├── WorldMap.vue
│   ├── TimeSlider.vue
│   ├── CO2CapitaView.vue
│   ├── TempView.vue
│   └── ...
└── ...
```
