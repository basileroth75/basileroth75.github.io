---
permalink: /about/
title: "About"
---

Tempor velit sint sunt ipsum tempor enim ad qui ullamco. Est dolore anim ad velit duis dolore minim sunt aliquip amet commodo labore. Ut eu pariatur aute ea aute excepteur laborum. 
<script src="https://d3js.org/d3.v4.js"></script>
<script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>
<script src="https://d3js.org/d3-geo-projection.v2.min.js"></script>

<div id="sru-map-widget" style="position: relative; width: 100%; height: 650px; background: #f9f9f9; overflow: hidden; border: 1px solid #ddd; border-radius: 8px; font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;">
    
    <div class="map-ui-layer search-container" style="position: absolute; top: 20px; left: 20px; z-index: 20; width: 300px;">
        <input type="text" id="sruSearchInput" style="width: 100%; padding: 8px 12px; border: 1px solid #e0e0e0; border-radius: 24px; font-size: 14px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); outline: none; box-sizing: border-box; transition: all 0.2s;" placeholder="Rechercher une commune...">
        <div id="sruSuggestions" style="background: white; border: 1px solid #eee; border-top: none; border-radius: 0 0 12px 12px; max-height: 200px; overflow-y: auto; position: absolute; width: 100%; box-shadow: 0 4px 12px rgba(0,0,0,0.05); display: none; box-sizing: border-box; margin-top: 1px;"></div>
    </div>

    <div id="sruTooltip" style="position: absolute; background: white; pointer-events: none; opacity: 0; border-radius: 8px; box-shadow: 0 8px 24px rgba(0,0,0,0.15); font-size: 13px; color: #333; width: 280px; z-index: 10; transition: opacity 0.2s, top 0.2s, left 0.2s; overflow: hidden;"></div>

    <div class="map-ui-layer zoom-controls" style="position: absolute; bottom: 30px; right: 30px; display: flex; flex-direction: column; box-shadow: 0 2px 8px rgba(0,0,0,0.1); border-radius: 8px; overflow: hidden; z-index: 100;">
        <button id="sruZoomIn" style="background: white; border: none; width: 40px; height: 40px; font-size: 20px; color: #555; cursor: pointer; border-bottom: 1px solid #f0f0f0; outline: none; margin: 0; padding: 0;">+</button>
        <button id="sruZoomOut" style="background: white; border: none; width: 40px; height: 40px; font-size: 20px; color: #555; cursor: pointer; outline: none; margin: 0; padding: 0;">-</button>
    </div>

    <div class="map-ui-layer legend" style="position: absolute; bottom: 30px; left: 30px; background: white; padding: 10px; border-radius: 8px; box-shadow: 0 2px 12px rgba(0,0,0,0.08); z-index: 5;">
        <div style="font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px; font-weight: 700; margin-bottom: 6px; color: #888;">Statut SRU</div>
        
        <div style="display: flex; align-items: center; margin-bottom: 2px; font-size: 12px; color: #555; line-height: 1.2;">
            <div style="width: 10px; height: 10px; margin-right: 6px; border-radius: 50%; background: #d73027;"></div> 
            <span>Commune carencée</span>
        </div>
        <div style="display: flex; align-items: center; margin-bottom: 2px; font-size: 12px; color: #555; line-height: 1.2;">
            <div style="width: 10px; height: 10px; margin-right: 6px; border-radius: 50%; background: #fc8d59;"></div> 
            <span>Commune déficitaire</span>
        </div>
        <div style="display: flex; align-items: center; margin-bottom: 2px; font-size: 12px; color: #555; line-height: 1.2;">
            <div style="width: 10px; height: 10px; margin-right: 6px; border-radius: 50%; background: #1a9850;"></div> 
            <span>Commune en règle</span>
        </div>
        <div style="display: flex; align-items: center; margin-bottom: 0; font-size: 12px; color: #555; line-height: 1.2;">
            <div style="width: 10px; height: 10px; margin-right: 6px; border-radius: 50%; background: #ccc;"></div> 
            <span>Pas de données</span>
        </div>
    </div>
</div>

<style>
    /* Widget Internal Styles */
    #sru-map-widget input:focus { border-color: #aaa; box-shadow: 0 4px 12px rgba(0,0,0,0.12); }
    #sru-map-widget .background-layer { fill: #f0f2f5; stroke: #fff; stroke-width: 0.5px; pointer-events: all; }
    #sru-map-widget .city-layer { stroke: #fff; stroke-width: 0.2px; vector-effect: non-scaling-stroke; transition: opacity 0.2s; }
    #sru-map-widget .city-layer:hover, #sru-map-widget .city-layer.active { stroke: #333; stroke-width: 1.5px; cursor: pointer; opacity: 1; z-index: 100; }
    
    /* Search Suggestions */
    #sruSuggestions .suggestion-item { 
            padding: 6px 12px !important; 
            cursor: pointer; 
            border-bottom: 1px solid #f9f9f9; 
            font-size: 13px; 
            color: #333; 
            line-height: 1.2 !important; 
        }
        #sruSuggestions .suggestion-item:hover { 
            background: #f7f9fc; 
        }
        #sruSuggestions .suggestion-item strong { 
            display: block; 
            color: #222; 
            margin-bottom: 0px !important; 
        }
        #sruSuggestions .suggestion-item span { 
            color: #888; 
            font-size: 0.85em; 
        }

    /* Tooltip Internal CSS */
    .sru-tooltip-header { padding: 12px 16px; color: white; font-weight: 600; font-size: 15px; display: flex; justify-content: space-between; align-items: center; }
    .sru-tooltip-body { padding: 16px; }
    .sru-tooltip-row { display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 13px; line-height: 1.4; }
    .sru-tooltip-row:last-child { margin-bottom: 0; }
    .sru-label { color: #777; font-weight: 400; }
    .sru-value { color: #222; font-weight: 600; text-align: right; }
    .sru-badge { display: inline-block; padding: 3px 8px; border-radius: 12px; font-size: 11px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.5px; }
    
    /* Button Hover */
    #sruZoomIn:hover, #sruZoomOut:hover { background: #f9f9f9 !important; color: #000 !important; }
</style>

<script>
(function() {
    // 1. SETUP
    const containerID = "#sru-map-widget";
    const container = d3.select(containerID);
    const width = document.getElementById("sru-map-widget").clientWidth;
    const height = 650; 

    function hideTooltip() {
        d3.select("#sruTooltip").style("opacity", 0);
        d3.selectAll(".city-layer").classed("active", false);
    }

    const svg = container.append("svg")
        .attr("width", width)
        .attr("height", height);
        // Note: Click handler for background is added in ready() now
    
    const mapGroup = svg.append("g");

    const zoom = d3.zoom()
        .scaleExtent([1, 10])
        .on("zoom", function() {
            mapGroup.attr("transform", d3.event.transform);
            mapGroup.selectAll("path").attr("stroke-width", 0.5 / d3.event.transform.k);
        });

    svg.call(zoom);

    d3.select("#sruZoomIn").on("click", function() { zoom.scaleBy(svg.transition().duration(200), 1.5); });
    d3.select("#sruZoomOut").on("click", function() { zoom.scaleBy(svg.transition().duration(200), 1 / 1.5); });

    // 2. COLORS & STATUS LOGIC
    const COLOR_CARENCEE = "#d73027";
    const COLOR_DEFICITAIRE = "#fc8d59";
    const COLOR_OK = "#1a9850";
    const COLOR_NULL = "#ccc";

    function getStatusColor(city) {
        if (!city || !city.sru_status) return COLOR_NULL;
        const status = city.sru_status.trim();

        if (status === "Carencée") return COLOR_CARENCEE;
        if (status === "Déficitaire") return COLOR_DEFICITAIRE;
        if (status === "En règle") return COLOR_OK;
        
        // Fallback
        const lower = status.toLowerCase();
        if (lower.includes("caren")) return COLOR_CARENCEE;
        if (lower.includes("défic") || lower.includes("defic")) return COLOR_DEFICITAIRE;
        if (lower.includes("règle") || lower.includes("regle") || lower.includes("ok")) return COLOR_OK;
        
        return COLOR_NULL; 
    }

    var searchList = []; 
    var pathGenerator = null; 

    function cleanCurrencyString(value) {
        if (!value) return 0;
    let cleanedValue = value
        .replace(/\s/g, '')        // Remove spaces
        .replace(/[^0-9,-]/g, '')  // Strip the broken euro symbol
        .replace(',', '.');        // Swap French comma for decimal dot
    return parseFloat(cleanedValue);
    }

    // 2. Create a row converter that tells D3 how to read the CSV
    function parseCsvRow(row) {
    // We keep everything in the row the same, but we overwrite the penalty column
    // Note: Change "row.penalty" to whatever the exact column header is in your CSV!
    row.penalty = cleanCurrencyString(row.penalty); 
    
    return row; // Return the perfectly clean row
    }


    d3.queue()
        .defer(d3.json, "https://raw.githubusercontent.com/gregoiredavid/france-geojson/master/regions-version-simplifiee.geojson")
        .defer(d3.json, "https://raw.githubusercontent.com/basileroth75/data_viz/main/sru_2025/cities_filtered.geojson") 
        .defer(d3.csv, "https://raw.githubusercontent.com/basileroth75/data_viz/main/sru_2025/maire_sru_cleaned.csv", parseCsvRow)        .await(ready);

    function ready(error, backgroundTopo, citiesTopo, data_csv) {
        if (error) { console.error("Error loading data:", error); return; }

        // --- NEW: Track Active City for Mobile ---
        let activeCity = null;
        
        // --- NEW: Background Click resets selection ---
        svg.on("click", function() {
            activeCity = null;
            hideTooltip();
        });

        var dataMap = d3.map();
        data_csv.forEach(function(d) {
            d.taux_sru = +d.taux_sru; 
            d.taux_cible = +d.taux_cible; 
            dataMap.set(d.code_insee, d);
        });

        citiesTopo.features.forEach(function(feature) {
            let cityData = dataMap.get(feature.properties.code);
            if(cityData) {
                searchList.push({ name: feature.properties.nom, code: feature.properties.code, dept: cityData.departement });
            }
        });

        const projection = d3.geoMercator().fitSize([width, height], backgroundTopo);
        pathGenerator = d3.geoPath().projection(projection);

        // Background
        mapGroup.append("g")
            .selectAll("path")
            .data(backgroundTopo.features)
            .enter().append("path")
            .attr("class", "background-layer")
            .attr("d", pathGenerator);
            // No mouseover here to prevent conflict

        // Cities
        mapGroup.append("g")
            .selectAll("path")
            .data(citiesTopo.features)
            .enter().append("path")
            .attr("class", "city-layer")
            .attr("id", d => "city-" + d.properties.code) 
            .attr("d", pathGenerator)
            .attr("fill", d => getStatusColor(dataMap.get(d.properties.code)))
            .on("mouseover", function(d) {
                // If we are touching the screen, this fires first. 
                // We show the tooltip, but we don't 'lock' it yet.
                if (d3.event) d3.event.stopPropagation();
                showTooltip(d, this);
            })
            .on("click", function(d) {
                // LOCK THE TOOLTIP
                if (d3.event) d3.event.stopPropagation();
                activeCity = d; // Set this city as actively locked
                showTooltip(d, this); // Force show
            })
            .on("mouseout", function(d) {
                // IMPORTANT: If this city is locked, DO NOT hide.
                if (activeCity === d) return;
                hideTooltip();
            });
            
        // 3. IMPROVED TOOLTIP DESIGN
        function showTooltip(d, element, customX, customY) {
            let city = dataMap.get(d.properties.code);
            if (!city) return;

            d3.selectAll(".city-layer").classed("active", false); 
            d3.select(element).classed("active", true).raise();   

            let statusColor = getStatusColor(city);
            let displayStatus = city.sru_status || "Données non disponibles";

            const euroFormatter = new Intl.NumberFormat('fr-FR', {
            style: 'currency',
            currency: 'EUR',
            minimumFractionDigits: 2, // Ensures you keep the ,38
            maximumFractionDigits: 2  // Prevents it from going too long, like ,3845
            });


            const pop = city.population ? parseInt(city.population).toLocaleString('fr-FR') : "N/A";
            //const penalty = city.penalty ? city.penalty : "0 €";
            const penalty = euroFormatter.format(city.penalty || 0);
            let content = `
                <div class="sru-tooltip-header" style="background-color: ${statusColor};">
                    <span>${d.properties.nom}</span>
                    <span style="font-size:0.8em; opacity:0.9;">${city.departement}</span>
                </div>
                <div class="sru-tooltip-body">
                    <div class="sru-tooltip-row">
                        <span class="sru-label">Statut</span>
                        <span class="sru-badge" style="background:${statusColor}20; color:${statusColor}">${displayStatus}</span>
                    </div>
                    <div style="margin: 10px 0; border-top: 1px dashed #eee;"></div>
                    <div class="sru-tooltip-row">
                        <span class="sru-label">Maire</span>
                        <span class="sru-value">${city.maire}</span>
                    </div>
                    <div class="sru-tooltip-row">
                        <span class="sru-label">Nuance Politique</span>
                        <span class="sru-value">${city.politique}</span>
                    </div>
                                    <div class="sru-tooltip-row">
                        <span class="sru-label">Population</span>
                        <span class="sru-value">${pop}</span>
                    </div>
                    <div style="margin: 10px 0; border-top: 1px dashed #eee;"></div>
                    <div class="sru-tooltip-row">
                        <span class="sru-label">Taux SRU 2024</span>
                        <span class="sru-value" style="color:${statusColor}">${city.taux_sru}%</span>
                    </div>
                    <div class="sru-tooltip-row">
                        <span class="sru-label">Objectif</span>
                        <span class="sru-value">${city.taux_cible}%</span>
                    </div>
                    <div class="sru-tooltip-row">
                        <span class="sru-label">Commune Exemptée</span>
                        <span class="sru-value">${city.commune_exemptée}</span>
                    </div>
                    <div class="sru-tooltip-row" style="margin-top:8px;">
                        <span class="sru-label">Prélèvement net 2025</span>
                        <span class="sru-value" style="color:#d73027">${penalty}</span>
                    </div>
                </div>
            `;

            let x, y;
            if (customX !== undefined && customY !== undefined) {
                x = customX;
                y = customY;
            } else {
                let mousePos = d3.mouse(container.node());
                x = mousePos[0];
                y = mousePos[1];
            }

            const tooltipWidth = 280;
            const tooltipHeight = 320; 

            if(x > width - tooltipWidth) x = width - tooltipWidth - 20;
            if(y > height - tooltipHeight) y = height - tooltipHeight - 20; 

            d3.select("#sruTooltip")
                .style("opacity", 1)
                .html(content)
                .style("left", (x + 20) + "px")
                .style("top", (y - 20) + "px");
        }

        // 4. SEARCH LOGIC
        const input = document.getElementById('sruSearchInput');
        const suggestionsBox = document.getElementById('sruSuggestions');

        input.addEventListener('input', function() {
            const val = this.value.toLowerCase();
            suggestionsBox.innerHTML = '';
            if (!val) { suggestionsBox.style.display = 'none'; return; }
            const matches = searchList.filter(c => c.name.toLowerCase().includes(val)).slice(0, 10);
            if (matches.length > 0) {
                suggestionsBox.style.display = 'block';
                matches.forEach(city => {
                    const div = document.createElement('div');
                    div.className = 'suggestion-item';
                    div.innerHTML = `<strong>${city.name}</strong> <span>${city.dept}</span>`;
                    div.onclick = () => selectCityFromSearch(city.code);
                    suggestionsBox.appendChild(div);
                });
            } else { suggestionsBox.style.display = 'none'; }
        });

        function selectCityFromSearch(code) {
            suggestionsBox.style.display = 'none';
            input.value = ""; 
            
            const selection = d3.select("#city-" + code);
            const node = selection.node();
            const data = selection.datum();

            if (node && data) {
                // --- NEW: Lock the city on search too ---
                activeCity = data; 
                
                const centroid = pathGenerator.centroid(data);
                const x = centroid[0];
                const y = centroid[1];
                const targetScale = 4;

                const transform = d3.zoomIdentity
                    .translate(width / 2, height / 2)
                    .scale(targetScale)
                    .translate(-x, -y);

                svg.transition()
                    .duration(1000)
                    .call(zoom.transform, transform)
                    .on("end", function() {
                        const currentTransform = d3.zoomTransform(svg.node());
                        const screenX = currentTransform.applyX(x);
                        const screenY = currentTransform.applyY(y);
                        showTooltip(data, node, screenX, screenY);
                    });
            }
        }
    }
})();
</script>