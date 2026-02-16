---
title: "Carte GIS "
last_modified_at: 2016-03-09T16:20:02-05:00
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---

This post has been updated and should show a modified date if used in a layout.

All children, except one, grow up. They soon know that they will grow up, and the way Wendy knew was this. One day when she was two years old she was playing in a garden, and she plucked another flower and ran with it to her mother. I suppose she must have looked rather delightful, for Mrs. Darling put her hand to her heart and cried, "Oh, why can't you remain like this for ever!" This was all that passed between them on the subject, but henceforth Wendy knew that she must grow up. You always know after you are two. Two is the beginning of the end.


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Boat Movements, Zones & Fish Density (OSM)</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet.heat/0.2.0/leaflet-heat.js"></script>
    
    <script src="https://d3js.org/d3.v7.min.js"></script>
    
    <style>
        body { margin: 0; padding: 0; font-family: "Helvetica Neue", Helvetica, Arial, sans-serif; }
        #map { width: 100vw; height: 100vh; }

        /* Tooltip Styling */
        .custom-tooltip {
            background: rgba(255, 255, 255, 0.95);
            border: 1px solid #ccc;
            border-radius: 4px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
            padding: 8px 12px;
            color: #333;
            font-size: 13px;
            line-height: 1.5;
        }
        .custom-tooltip b { color: #555; }
        .tooltip-title { font-weight: bold; font-size: 14px; margin-bottom: 5px; border-bottom: 2px solid; padding-bottom: 2px;}

        /* Controls Panel Styling */
        #controls-panel { 
            position: absolute; top: 15px; left: 15px; background: rgba(255, 255, 255, 0.95); 
            padding: 15px; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.2); font-size: 13px;
            z-index: 1000; width: 280px; max-height: 90vh; overflow-y: auto;
        }
        .legend-title { font-weight: bold; font-size: 14px; margin-bottom: 10px; border-bottom: 1px solid #ddd; padding-bottom: 5px;}
        .legend-item { display: flex; align-items: center; margin-bottom: 8px; }
        .legend-color { width: 16px; height: 16px; margin-right: 10px; border-radius: 3px; border: 1.5px solid #999; }
        .legend-line { width: 16px; height: 3px; margin-right: 10px; }
        
        /* Filter Sections */
        .filter-section { margin-top: 15px; border-top: 1px solid #ddd; padding-top: 10px; }
        .input-group { display: flex; flex-direction: column; margin-top: 8px; gap: 5px; }
        .input-group input { padding: 4px; border: 1px solid #ccc; border-radius: 4px; }
        .action-btn { margin-top: 8px; width: 100%; padding: 6px; background: #2c3e50; color: white; border: none; border-radius: 4px; cursor: pointer; transition: 0.2s; }
        .action-btn:hover { background: #34495e; }
        .count-text { margin-top: 8px; font-size: 0.9em; color: #666; font-style: italic;}
    </style>
</head>
<body>

    <div id="map"></div>

    <div id="controls-panel">
        <div class="legend-title">Map Layers & Legend</div>
        
        <div class="legend-item">
            <div class="legend-color" style="background: rgba(141, 211, 199, 0.5); border-color: #6da39a;"></div>
            <span>General Protection Zones</span>
        </div>
        <div class="legend-item">
            <div class="legend-color" style="background: rgba(251, 128, 114, 0.5); border-color: #d96f63; border-style: dashed;"></div>
            <span>Seasonal Protection Zones</span>
        </div>
        <div class="legend-item">
            <div class="legend-line" style="background: linear-gradient(to right, #e41a1c, #377eb8, #4daf4a);"></div>
            <span>Boat Movements (Tracks)</span>
        </div>
        <div class="legend-item">
            <div class="legend-color" style="background: linear-gradient(to right, #2b83ba, #abdda4, #fdae61, #d7191c); border: 1px solid #999;"></div>
            <span>Krill Density (Heatmap)</span>
        </div>

        <div class="filter-section">
            <strong>Filter Tracks by Date:</strong>
            <div class="input-group">
                <label>From: <input type="date" id="start-date" value="2023-01-01"></label>
                <label>To: <input type="date" id="end-date"></label>
                <button id="apply-track-filter" class="action-btn">Apply Track Filter</button>
            </div>
            <div id="track-count" class="count-text"></div>
        </div>

        <div class="filter-section">
            <strong>Filter Krill Density by Year:</strong>
            <div class="input-group">
                <label>From Year: <input type="number" id="krill-start-year"></label>
                <label>To Year: <input type="number" id="krill-end-year"></label>
                <button id="apply-krill-filter" class="action-btn">Apply Krill Filter</button>
            </div>
            <div id="krill-count" class="count-text"></div>
        </div>
    </div>

    <script>
        const map = L.map('map', { zoomControl: false }).setView([-60, -55], 5);
        L.control.zoom({ position: 'bottomright' }).addTo(map);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap',
            maxZoom: 18
        }).addTo(map);

        const generalZonesLayer = L.layerGroup().addTo(map);
        const seasonalZonesLayer = L.layerGroup().addTo(map);
        const tracksLayer = L.layerGroup().addTo(map);
        const krillHeatmapLayer = L.layerGroup().addTo(map);

        const overlayMaps = {
            "🟨 General Zones": generalZonesLayer,
            "🟥 Seasonal Zones": seasonalZonesLayer,
            "🚢 Boat Tracks": tracksLayer,
            "🐟 Krill Heatmap": krillHeatmapLayer
        };
        L.control.layers(null, overlayMaps, { collapsed: false, position: 'topright' }).addTo(map);

        let allTracksData = []; 
        let allKrillData = [];

        const colorGenFill = "#8dd3c7";
        const colorGenBorder = "#6da39a";
        const colorSeasFill = "#fb8072";
        const colorSeasBorder = "#d96f63";

        const tracksCsvUrl = "../_data/data/filtered_tracks.csv";
        const zonesCsvUrl = "../_data/data/domaine1-Zone_de_protection_general.csv";
        const zonesSaisonCsvUrl = "../_data/data/domaine1-Zone_de_protection_saison.csv";
        const krillCsvUrl = "../_data/data/processed_krill.csv";


        Promise.all([
            d3.csv(tracksCsvUrl),
            d3.dsv(";", zonesCsvUrl),
            d3.dsv(";", zonesSaisonCsvUrl),
            d3.csv(krillCsvUrl)
        ]).then(([tracksData, zonesData, zonesSaisonData, krillData]) => {

            d3.group(zonesData, d => d.zone_id).forEach((points, zoneId) => {
                points.sort((a, b) => +a.seq - +b.seq);
                const latlngs = points.map(p => [+p.lat, +p.lon]);

                L.polygon(latlngs, {
                    color: colorGenBorder, fillColor: colorGenFill,
                    weight: 2, dashArray: '4, 4', fillOpacity: 0.5
                }).bindTooltip(`
                    <div class="custom-tooltip">
                        <div class="tooltip-title" style="border-color: ${colorGenBorder}">General Protection Zone</div>
                        <b>ID:</b> ${zoneId}
                    </div>
                `, { sticky: true }).addTo(generalZonesLayer);
            });

            d3.group(zonesSaisonData, d => d.zone_id).forEach((points, zoneId) => {
                points.sort((a, b) => +a.seq - +b.seq);
                const latlngs = points.map(p => [+p.lat, +p.lon]);

                L.polygon(latlngs, {
                    color: colorSeasBorder, fillColor: colorSeasFill,
                    weight: 2.5, dashArray: '8, 5', fillOpacity: 0.5
                }).bindTooltip(`
                    <div class="custom-tooltip">
                        <div class="tooltip-title" style="border-color: ${colorSeasBorder}">Seasonal Protection Zone</div>
                        <b>ID:</b> ${zoneId}
                    </div>
                `, { sticky: true }).addTo(seasonalZonesLayer);
            });

            let maxDate = new Date("2000-01-01");
            tracksData.forEach(d => {
                d.dateObj = new Date(d.timestamp);
                if (d.dateObj > maxDate) maxDate = d.dateObj;
            });
            allTracksData = tracksData;

            document.getElementById('end-date').value = maxDate.toISOString().split('T')[0];
            renderTracks(); 

            let krillMinYear = 3000, krillMaxYear = 0;
            krillData.forEach(d => {
                d.yearNum = parseInt(d.year);
                d.lat = parseFloat(d.decimalLatitude);
                d.lon = parseFloat(d.decimalLongitude);
                d.quantity = parseFloat(d.organismQuantity);
                
                if (!isNaN(d.yearNum)) {
                    if (d.yearNum < krillMinYear) krillMinYear = d.yearNum;
                    if (d.yearNum > krillMaxYear) krillMaxYear = d.yearNum;
                }
            });
            
            allKrillData = krillData.filter(d => d.quantity > 0 && !isNaN(d.lat) && !isNaN(d.lon));

            document.getElementById('krill-start-year').value = krillMinYear;
            document.getElementById('krill-end-year').value = krillMaxYear;
            document.getElementById('krill-start-year').min = krillMinYear;
            document.getElementById('krill-end-year').max = krillMaxYear;

            renderKrillHeatmap(); 

        }).catch(error => {
            console.error("Error loading data:", error);
            alert("Error loading data. Make sure you run this on a Local Web Server.");
        });

        function renderTracks() {
            tracksLayer.clearLayers();
            const start = new Date(document.getElementById('start-date').value);
            const end = new Date(document.getElementById('end-date').value);
            end.setHours(23, 59, 59); 

            const filteredData = allTracksData.filter(d => d.dateObj >= start && d.dateObj <= end);
            document.getElementById('track-count').innerText = `Displaying ${filteredData.length} track points`;

            const tracksGrouped = d3.group(filteredData, d => d.seg_id);
            const trackColorScale = d3.scaleOrdinal(d3.schemeSet1);
            let trackIndex = 0;

            tracksGrouped.forEach((points, segId) => {
                const color = trackColorScale(trackIndex++);
                points.sort((a, b) => a.dateObj - b.dateObj); 
                const latlngs = points.map(p => [+p.lat, +p.lon]);

                if (latlngs.length > 1) {
                    L.polyline(latlngs, { color: color, weight: 1.5, opacity: 0.8 }).addTo(tracksLayer);
                }

                points.forEach(d => {
                    L.circleMarker([+d.lat, +d.lon], {
                        radius: 2, fillColor: color, color: "#fff", weight: 0.5, fillOpacity: 1
                    }).bindTooltip(`
                        <div class="custom-tooltip">
                            <div class="tooltip-title" style="border-color: ${color}">Boat Position</div>
                            <b>Time:</b> ${d.dateObj.toLocaleString()}<br>
                            <b>Speed:</b> ${(+d.speed).toFixed(2)} knots<br>
                            <b>Lat/Lon:</b> ${(+d.lat).toFixed(4)}, ${(+d.lon).toFixed(4)}
                        </div>
                    `, { sticky: true }).addTo(tracksLayer);
                });
            });
        }

        function renderKrillHeatmap() {
            krillHeatmapLayer.clearLayers();
            
            const startYear = parseInt(document.getElementById('krill-start-year').value);
            const endYear = parseInt(document.getElementById('krill-end-year').value);
            
            const filteredKrill = allKrillData.filter(d => d.yearNum >= startYear && d.yearNum <= endYear);
            document.getElementById('krill-count').innerText = `Displaying ${filteredKrill.length} krill data points`;
            
            if (filteredKrill.length === 0) return;

            const quantities = filteredKrill.map(d => d.quantity).sort((a, b) => a - b);
            const maxVal = quantities[Math.floor(quantities.length * 0.98)] || 1; 

            const heatPoints = filteredKrill.map(d => [d.lat, d.lon, d.quantity]);
            
            // ==========================================
            // THE NEW SOFTER SCIENTIFIC GRADIENT
            // ==========================================
            L.heatLayer(heatPoints, {
                radius: 18,
                blur: 15,
                maxZoom: 6,
                max: maxVal,
                gradient: {
                    0.3: '#2b83ba', // Muted Blue (Low density)
                    0.5: '#abdda4', // Soft Green
                    0.8: '#fdae61', // Soft Orange
                    1.0: '#d7191c'  // Muted Brick Red (High density)
                }
            }).addTo(krillHeatmapLayer);
        }

        document.getElementById('apply-track-filter').addEventListener('click', renderTracks);
        document.getElementById('apply-krill-filter').addEventListener('click', renderKrillHeatmap);
    </script>
</body>
</html>