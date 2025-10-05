<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Urban-Planet - Bolivia (NASA Worldview / GIBS Integration)</title>

    <!-- Leaflet -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
      /* Minimal styles trimmed for readability; 
         keep your more complex styles if you prefer */
      body { margin:0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #071322; color: #fff; }
      header { padding: 12px 18px; background: linear-gradient(90deg,#06314a,#091b36); display:flex; align-items:center; gap:16px;}
      header h1 { font-size:1.15rem; margin:0; }
      #main { display:flex; gap:18px; padding:18px; max-width:1400px; margin:0 auto;}
      .sidebar { width:320px; background: rgba(10,20,30,0.8); padding:16px; border-radius:8px; }
      .map-wrap { flex:1; min-height:720px; position:relative; border-radius:8px; overflow:hidden; }
      #map { width:100%; height:100%; display:block; }
      .legend { 
        position:absolute; 
        right:18px; 
        bottom:18px; 
        background: rgba(0,0,0,0.7); 
        padding:12px; 
        border-radius:8px; 
        max-width:320px; 
        font-size:0.9rem;
        z-index: 1000; /* Asegurar que esté por encima del mapa */
      }
      .overlay-controls button { margin:6px 6px 6px 0; padding:8px 12px; border-radius:6px; border:none; cursor:pointer; background:#0b4b6f; color:#fff;}
      .overlay-controls button.active { background:#27ae60; }
      .layer-control { margin-bottom:10px; }
      input[type="checkbox"] { transform:scale(1.1); margin-right:8px; }
    </style>
</head>
<body>
  <header>
    <div style="display:flex; align-items:center; gap:12px;">
      <div style="background:#27AE60;color:#fff;padding:10px;border-radius:6px;font-weight:700;">UP</div>
      <div>
        <h1>URBAN-PLANET — Bolivia · NASA Worldview Integration</h1>
        <div style="font-size:0.85rem;color:#bcd;">Satellite visualization + Thematic zones (construction, ANP, deforestation, risk)</div>
      </div>
    </div>
  </header>

  <div id="main">
    <aside class="sidebar">
      <div style="margin-bottom:12px;">
        <label for="location-search">Search location (e.g., La Paz, Santa Cruz)</label>
        <input id="location-search" placeholder="Type and press Enter" style="width:100%; padding:8px; border-radius:6px; background:#0b1722; color:#fff; border:1px solid #163a4d;">
      </div>

      <h3>Layers (show/hide)</h3>
      <div class="layer-control"><label><input id="optimal-construction" type="checkbox" checked> Optimal Construction Zones</label></div>
      <div class="layer-control"><label><input id="moderate-construction" type="checkbox" checked> Moderate Zones</label></div>
      <div class="layer-control"><label><input id="limited-construction" type="checkbox" checked> Limited Zones</label></div>
      <div class="layer-control"><label><input id="challenging-construction" type="checkbox" checked> Challenging Zones</label></div>
      <div class="layer-control"><label><input id="restricted-zones" type="checkbox" checked> Restricted / High Risk Zones</label></div>
      <div class="layer-control"><label><input id="protected-areas" type="checkbox" checked> Protected Natural Areas (ANP)</label></div>
      <div class="layer-control"><label><input id="urban-growth" type="checkbox" checked> Urban Growth Areas</label></div>
      <div class="layer-control"><label><input id="deforestation" type="checkbox" checked> Deforestation Areas</label></div>
      <div class="layer-control"><label><input id="recovery-zones" type="checkbox" checked> Ecosystem Recovery Zones</label></div>
      <div class="layer-control"><label><input id="disaster-risk" type="checkbox" checked> Disaster Risk Zones</label></div>

      <div style="margin-top:14px;">
        <button id="export-data" style="width:100%; padding:10px; border-radius:6px; border:none; background:#27AE60; color:#04202e; font-weight:700;">
          <i class="fas fa-file-export"></i> Export Analysis (simulated)
        </button>
      </div>

      <div style="margin-top:12px; font-size:0.85rem; color:#9cc;">
        <strong>Sources:</strong><br>
        NASA GIBS / Worldview · MODIS · VIIRS · Landsat · SRTM
      </div>
    </aside>

    <div class="map-wrap">
      <div id="map"></div>

      <div style="position:absolute; top:14px; left:14px; z-index:1200;">
        <div style="background:rgba(0,0,0,0.6); padding:8px; border-radius:8px; color:#fff;">
          <div style="margin-bottom:8px;"><strong>Quick Views</strong></div>
          <div class="overlay-controls">
            <button class="quick" data-view="all">All</button>
            <button class="quick" data-view="construction">Construction</button>
            <button class="quick" data-view="environment">Environment</button>
            <button class="quick" data-view="risk">Risk</button>
          </div>
        </div>
      </div>

      <!-- Leyenda posicionada sobre el mapa -->
      <div class="legend" id="legend">
        <h4 style="margin:0 0 6px 0;">Legend — Bolivia</h4>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#4CAF50;border:1px solid #ccc;"></span> Optimal Construction</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#8BC34A;border:1px solid #ccc;"></span> Moderate Construction</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#FFC107;border:1px solid #ccc;"></span> Limited Construction</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#FF9800;border:1px solid #ccc;"></span> Challenging</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#FC3D21;border:1px solid #ccc;"></span> Restricted/High Risk</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#2196F3;border:1px solid #ccc;"></span> Protected Areas</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#9C27B0;border:1px solid #ccc;"></span> Urban Growth</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#795548;border:1px solid #ccc;"></span> Deforestation</div>
        <div style="display:flex; gap:8px; align-items:center; margin-bottom:6px;"><span style="display:inline-block;width:18px;height:14px;background:#607D8B;border:1px solid #ccc;"></span> Recovery Zones</div>
        <div style="display:flex; gap:8px; align-items:center;"><span style="display:inline-block;width:18px;height:14px;background:#FF5722;border:1px solid #ccc;"></span> Disaster Risk</div>
      </div>

    </div>
  </div>

  <footer style="padding:12px 18px; text-align:center; color:#9bb;">
    Urban-Planet · NASA Worldview (GIBS) integration demo — Default date: 2025-10-04
  </footer>

  <!-- Leaflet -->
  <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
  <script>
    /*************************************************************************
     * Main configuration
     *************************************************************************/
    const GIBS_TIME = '2025-10-04'; // change if you want another date (YYYY-MM-DD)
    const GIBS_LEVEL = 'GoogleMapsCompatible_Level9'; // GIBS resolution level
    const GIBS_FORMAT = 'jpg'; // jpg for corrected reflectance; png for transparent overlays

    // Initialize map centered on Bolivia
    const map = L.map('map', {
      center: [-16.5, -64.5],
      zoom: 5,
      minZoom: 2,
      maxZoom: 12,
      zoomControl: true
    });

    /*************************************************************************
     * Base layers (GIBS + fallback)
     * URL pattern for GIBS (WMTS/XYZ compatible)
     *************************************************************************/
    const gibsBaseUrl = `https://gibs.earthdata.nasa.gov/wmts/epsg3857/best/VIIRS_SNPP_CorrectedReflectance_TrueColor/default/${GIBS_TIME}/${GIBS_LEVEL}/{z}/{y}/{x}.jpg`;

    const gibsBase = L.tileLayer(gibsBaseUrl, {
      attribution: 'NASA GIBS / Worldview',
      tileSize: 256,
      minZoom: 1,
      maxZoom: 9,
      tms: false,
      continuousWorld: true
    });

    // Reference labels overlay (transparent png)
    const gibsLabelsUrl = `https://gibs.earthdata.nasa.gov/wmts/epsg3857/best/Reference_Labels_15m/default/${GIBS_TIME}/${GIBS_LEVEL}/{z}/{y}/{x}.png`;
    const gibsLabels = L.tileLayer(gibsLabelsUrl, {
      attribution: '',
      tileSize: 256,
      minZoom: 1,
      maxZoom: 9,
      transparent: true
    });

    // Blue Marble as alternate base (optional)
    const blueMarbleUrl = `https://gibs.earthdata.nasa.gov/wmts/epsg3857/best/BlueMarble_NextGeneration/default/${GIBS_TIME}/${GIBS_LEVEL}/{z}/{y}/{x}.jpg`;
    const blueMarble = L.tileLayer(blueMarbleUrl, {
      attribution: 'NASA Blue Marble (GIBS)',
      tileSize: 256,
      minZoom: 1,
      maxZoom: 9
    });

    // Fallback (OpenStreetMap) if GIBS tiles fail
    const osm = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'OpenStreetMap',
      maxZoom: 19
    });

    // Initially add GIBS (if fails, user can manually activate OSM)
    gibsBase.addTo(map);
    gibsLabels.addTo(map);

    /*************************************************************************
     * Thematic layers (layerGroups)
     *************************************************************************/
    const optimalConstructionLayer = L.layerGroup().addTo(map); // #4CAF50
    const moderateConstructionLayer = L.layerGroup().addTo(map); // #8BC34A
    const limitedConstructionLayer = L.layerGroup().addTo(map); // #FFC107
    const challengingConstructionLayer = L.layerGroup().addTo(map); // #FF9800
    const restrictedZonesLayer = L.layerGroup().addTo(map); // #FC3D21
    const protectedAreasLayer = L.layerGroup().addTo(map); // #2196F3
    const urbanGrowthLayer = L.layerGroup().addTo(map); // #9C27B0
    const deforestationLayer = L.layerGroup().addTo(map); // #795548
    const recoveryZonesLayer = L.layerGroup().addTo(map); // #607D8B
    const disasterRiskLayer = L.layerGroup().addTo(map); // #FF5722

    /*************************************************************************
     * Helper function to create zones from a coordinate array
     * coords: [ [lat,lng], [lat,lng], ... ]
     *************************************************************************/
    function createZone(coords, color, opacity, title, description, targetLayer) {
      const poly = L.polygon(coords, {
        color: color,
        fillColor: color,
        fillOpacity: opacity,
        weight: 2
      }).bindPopup(`
        <div style="min-width:220px;">
          <h3 style="margin:0 0 6px 0; color:${color};">${title}</h3>
          <p style="margin:0 0 8px 0;">${description}</p>
          <small style="opacity:0.8;">Source: Urban-Planet · NASA GIBS (visual)</small>
        </div>
      `);

      poly.addTo(targetLayer);
      return poly;
    }

    /*************************************************************************
     * Example coverage for all Bolivia
     * (sample polygons per department / subregions)
     *
     * Note: coordinates here are approximate polygons for representation
     *     — replace with real GeoJSON for accuracy.
     *************************************************************************/
    function populateBoliviaZones() {
      // LA PAZ - Optimal area (e.g., metropolitan strip)
      createZone([[-16.5,-68.2],[-16.2,-67.9],[-16.7,-67.6],[-17.0,-68.0]], '#4CAF50', 0.5,
        'La Paz - Optimal Construction Zone', 'Favorable terrain, stable infrastructure', optimalConstructionLayer);

      // SANTA CRUZ - Urban growth (expansion)
      createZone([[-17.9,-63.5],[-17.6,-63.1],[-18.1,-62.8],[-18.3,-63.2]], '#9C27B0', 0.5,
        'Santa Cruz - Urban Expansion Area', 'Rapid urban growth and agricultural conversion', urbanGrowthLayer);

      // COCHABAMBA - moderate zone
      createZone([[-17.4,-66.3],[-17.1,-66.0],[-17.6,-65.7],[-17.8,-66.2]], '#8BC34A', 0.45,
        'Cochabamba - Moderate Conditions Zone', 'Gentle slopes and good opportunities', moderateConstructionLayer);

      // POTOSÍ - limited areas (altitude)
      createZone([[-19.6,-65.8],[-19.3,-65.5],[-19.8,-65.3],[-20.1,-65.9]], '#FFC107', 0.45,
        'Potosí - Limited Zone', 'High altitude and difficult access, construction limit', limitedConstructionLayer);

      // ORURO - challenging (terrain and mining)
      createZone([[-18.9,-67.1],[-18.6,-66.8],[-19.0,-66.5],[-19.2,-66.9]], '#FF9800', 0.45,
        'Oruro - Challenging Zone', 'Geological conditions and mining', challengingConstructionLayer);

      // TARIJA - restricted (sensitive areas)
      createZone([[-21.6,-64.9],[-21.3,-64.6],[-21.8,-64.3],[-21.9,-64.8]], '#FC3D21', 0.45,
        'Tarija - Restricted/High Risk Zone', 'Watershed protection and critical slopes', restrictedZonesLayer);

      // BENI - deforestation and flood risk
      createZone([[-14.6,-65.5],[-14.2,-65.0],[-14.9,-64.6],[-15.2,-65.1]], '#795548', 0.5,
        'Beni - Deforested Areas', 'Converted for agriculture; forest loss detected', deforestationLayer);

      // PANDO - ANP and recovery (Amazon)
      createZone([[-10.5,-66.0],[-10.2,-65.6],[-10.8,-65.2],[-11.0,-65.8]], '#2196F3', 0.45,
        'Pando - Protected Natural Area / Ecosystem', 'High biodiversity, conservation priority', protectedAreasLayer);

      // CHUQUISACA (Sucre) - ecosystem recovery
      createZone([[-19.0,-65.3],[-18.7,-64.9],[-19.3,-64.7],[-19.5,-65.1]], '#607D8B', 0.45,
        'Chuquisaca - Ecosystem Recovery Zone', 'Ongoing reforestation projects', recoveryZonesLayer);

      // SALAR / UYUNI - seasonal flood risk
      createZone([[-20.2,-67.0],[-19.9,-66.6],[-20.4,-66.3],[-20.8,-66.9]], '#FF5722', 0.4,
        'Salar / South - Disaster Risk', 'Seasonal flooding dynamics and saline soils', disasterRiskLayer);

      // Northern Amazon and eastern edge - protected zones and deforestation (multiple polygons)
      createZone([[-12.5,-66.5],[-12.0,-66.1],[-12.8,-65.7],[-13.0,-66.3]], '#2196F3', 0.4,
        'Northern Amazon - Partial ANP', 'Protected areas and biodiversity corridors', protectedAreasLayer);

      createZone([[-13.5,-64.2],[-13.1,-63.7],[-13.8,-63.3],[-14.0,-63.9]], '#795548', 0.5,
        'Amazon - Deforestation Strip', 'Agricultural expansion detected by satellite', deforestationLayer);

      // Andean strip - landslide risk (example)
      createZone([[-16.9,-69.5],[-16.6,-69.2],[-17.1,-68.9],[-17.4,-69.3]], '#FF5722', 0.45,
        'Cordillera - Landslide Risk', 'Steep slopes and intense rainfall', disasterRiskLayer);

      // More example polygons for smaller regions (add real GeoJSON)
      // ...
    }

    populateBoliviaZones();

    /*************************************************************************
     * UI controls: toggle checkboxes in sidebar
     *************************************************************************/
    function bindLayerToggle(checkboxId, layer) {
      const cb = document.getElementById(checkboxId);
      cb.addEventListener('change', (e) => {
        if (e.target.checked) map.addLayer(layer);
        else map.removeLayer(layer);
      });
    }

    bindLayerToggle('optimal-construction', optimalConstructionLayer);
    bindLayerToggle('moderate-construction', moderateConstructionLayer);
    bindLayerToggle('limited-construction', limitedConstructionLayer);
    bindLayerToggle('challenging-construction', challengingConstructionLayer);
    bindLayerToggle('restricted-zones', restrictedZonesLayer);
    bindLayerToggle('protected-areas', protectedAreasLayer);
    bindLayerToggle('urban-growth', urbanGrowthLayer);
    bindLayerToggle('deforestation', deforestationLayer);
    bindLayerToggle('recovery-zones', recoveryZonesLayer);
    bindLayerToggle('disaster-risk', disasterRiskLayer);

    /*************************************************************************
     * Quick view buttons
     *************************************************************************/
    document.querySelectorAll('.quick').forEach(btn => {
      btn.addEventListener('click', function() {
        const view = this.getAttribute('data-view');

        // First hide all
        const allLayers = [optimalConstructionLayer, moderateConstructionLayer, limitedConstructionLayer,
                           challengingConstructionLayer, restrictedZonesLayer, protectedAreasLayer,
                           urbanGrowthLayer, deforestationLayer, recoveryZonesLayer, disasterRiskLayer];
        allLayers.forEach(l => map.removeLayer(l));

        // Activate according to view
        if (view === 'all') {
          allLayers.forEach(l => map.addLayer(l));
        } else if (view === 'construction') {
          map.addLayer(optimalConstructionLayer);
          map.addLayer(moderateConstructionLayer);
          map.addLayer(limitedConstructionLayer);
          map.addLayer(challengingConstructionLayer);
        } else if (view === 'environment') {
          map.addLayer(protectedAreasLayer);
          map.addLayer(deforestationLayer);
          map.addLayer(recoveryZonesLayer);
        } else if (view === 'risk') {
          map.addLayer(restrictedZonesLayer);
          map.addLayer(disasterRiskLayer);
        }

        // Visual active button
        document.querySelectorAll('.quick').forEach(b => b.classList.remove('active'));
        this.classList.add('active');
      });
    });

    // Initially mark "All"
    document.querySelector('.quick[data-view="all"]').classList.add('active');

    /*************************************************************************
     * Simple name search (no external geocoding)
     *************************************************************************/
    document.getElementById('location-search').addEventListener('keypress', function(e){
      if (e.key === 'Enter') {
        const q = this.value.trim().toLowerCase();
        if (!q) return;
        if (q.includes('la paz') || q.includes('lapaz')) map.setView([-16.4897, -68.1193], 10);
        else if (q.includes('santa cruz')) map.setView([-17.7833, -63.1821], 10);
        else if (q.includes('cochabamba')) map.setView([-17.3895, -66.1568], 10);
        else if (q.includes('sucre')) map.setView([-19.0196, -65.2620], 10);
        else if (q.includes('potosi')) map.setView([-19.5833, -65.7531], 8);
        else alert('Location not found in simple search. Try: La Paz, Santa Cruz, Cochabamba, Sucre, Potosí');
      }
    });

    /*************************************************************************
     * Export / Simulation and fallback handling
     *************************************************************************/
    document.getElementById('export-data').addEventListener('click', () => {
      alert('Simulating export: would generate a package with visible layer GeoJSON and a summary PDF (feature to implement).');
    });

    // Detect black tiles: simple error check -> activate OSM if many errors
    let tileErrorCount = 0;
    gibsBase.on('tileerror', () => {
      tileErrorCount++;
      if (tileErrorCount > 8 && !map.hasLayer(osm)) {
        if (confirm('Some GIBS tiles failed to load. Do you want to activate the backup layer (OpenStreetMap)?')) {
          osm.addTo(map);
        }
      }
    });

    /*************************************************************************
     * Click on map: example to fill report field (if you had one)
     *************************************************************************/
    map.on('click', function(e) {
      L.popup()
        .setLatLng(e.latlng)
        .setContent(`<strong>Coordinates:</strong><br/>Lat: ${e.latlng.lat.toFixed(5)}<br/>Lng: ${e.latlng.lng.toFixed(5)}`)
        .openOn(map);
    });

    /*************************************************************************
     * End script
     *************************************************************************/
  </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Urban-Planet - Territorial analysis platform for Bolivia with NASA satellite data">
    <meta name="keywords" content="NASA, Bolivia, territorial analysis, satellite, urbanism, environment">
    <title>Urban-Planet Bolivia - Analysis with NASA Data</title>
    
    <!-- External stylesheets -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- NASA Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        /* CSS styles will go here */
    </style>
</head>
<body>
    <!-- Application content -->
    
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
    <script>
        // JavaScript code will go here
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Urban Resilience Plan - Bolivia</title>
    
    <!-- External stylesheets -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- NASA-inspired Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Exo+2:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --nasa-blue: #0B3D91;
            --nasa-red: #FC3D21;
            --nasa-green: #27AE60;
            --nasa-dark: #061A38;
            --nasa-light: #E6F2FF;
            --nasa-orange: #FF6B35;
            --nasa-purple: #6A4C93;
        }
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Exo 2', 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, var(--nasa-dark), #0A2342);
            color: white;
            line-height: 1.6;
            overflow-x: hidden;
        }
        
        .nasa-header {
            background: linear-gradient(90deg, var(--nasa-blue), #082A5E);
            padding: 1rem 2rem;
            border-bottom: 3px solid var(--nasa-green);
            box-shadow: 0 4px 20px rgba(0,0,0,0.3);
            position: relative;
            overflow: hidden;
        }
        
        .nasa-header::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 2px;
            background: linear-gradient(90deg, var(--nasa-red), var(--nasa-orange), var(--nasa-green));
        }
        
        .header-content {
            display: flex;
            justify-content: space-between;
            align-items: center;
            max-width: 1400px;
            margin: 0 auto;
        }
        
        .logo-container {
            display: flex;
            align-items: center;
            gap: 1rem;
        }
        
        .nasa-logo {
            width: 60px;
            height: 60px;
            background: var(--nasa-green);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 1.4rem;
            color: white;
            position: relative;
            box-shadow: 0 0 20px rgba(39, 174, 96, 0.4);
        }
        
        .logo-text {
            display: flex;
            flex-direction: column;
        }
        
        .main-title {
            font-family: 'Space Grotesk', sans-serif;
            font-size: 2rem;
            font-weight: 700;
            background: linear-gradient(135deg, #ffffff, #a8d8ff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            letter-spacing: 1px;
        }
        
        .tagline {
            font-size: 0.9rem;
            color: var(--nasa-green);
            font-weight: 600;
            letter-spacing: 0.5px;
        }
        
        .plan-container {
            max-width: 1200px;
            margin: 2rem auto;
            padding: 0 2rem;
        }
        
        .plan-section {
            background: rgba(6, 26, 56, 0.8);
            border-radius: 12px;
            padding: 2rem;
            margin-bottom: 2rem;
            border: 1px solid rgba(255,255,255,0.1);
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        
        .section-title {
            font-family: 'Space Grotesk', sans-serif;
            font-size: 1.8rem;
            color: var(--nasa-green);
            margin-bottom: 1.5rem;
            display: flex;
            align-items: center;
            gap: 0.8rem;
        }
        
        .section-subtitle {
            font-family: 'Space Grotesk', sans-serif;
            font-size: 1.3rem;
            color: var(--nasa-blue);
            margin: 1.5rem 0 1rem 0;
            border-left: 4px solid var(--nasa-green);
            padding-left: 1rem;
        }
        
        .overview-box {
            background: rgba(11, 61, 145, 0.3);
            padding: 1.5rem;
            border-radius: 8px;
            border-left: 4px solid var(--nasa-red);
            margin-bottom: 1.5rem;
        }
        
        .resource-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 1.5rem;
            margin: 1.5rem 0;
        }
        
        .resource-card {
            background: rgba(11, 61, 145, 0.2);
            padding: 1.5rem;
            border-radius: 8px;
            border: 1px solid rgba(255,255,255,0.1);
            transition: all 0.3s ease;
        }
        
        .resource-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.4);
            border-color: var(--nasa-green);
        }
        
        .resource-icon {
            font-size: 2rem;
            color: var(--nasa-green);
            margin-bottom: 1rem;
        }
        
        .resource-title {
            font-weight: 600;
            margin-bottom: 0.5rem;
            color: var(--nasa-light);
        }
        
        .resource-link {
            color: var(--nasa-green);
            text-decoration: none;
            font-weight: 500;
            display: inline-block;
            margin-top: 0.8rem;
            transition: color 0.3s ease;
        }
        
        .resource-link:hover {
            color: var(--nasa-orange);
        }
        
        .strategy-list {
            list-style: none;
            margin: 1.5rem 0;
        }
        
        .strategy-item {
            background: rgba(255,255,255,0.05);
            padding: 1.2rem;
            margin-bottom: 1rem;
            border-radius: 8px;
            border-left: 4px solid var(--nasa-blue);
            transition: all 0.3s ease;
        }
        
        .strategy-item:hover {
            background: rgba(255,255,255,0.08);
            transform: translateX(5px);
        }
        
        .strategy-number {
            display: inline-block;
            width: 30px;
            height: 30px;
            background: var(--nasa-blue);
            color: white;
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
            margin-right: 1rem;
            font-weight: bold;
        }
        
        .stakeholder-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 1.5rem;
            margin: 1.5rem 0;
        }
        
        .stakeholder-card {
            background: rgba(11, 61, 145, 0.2);
            padding: 1.5rem;
            border-radius: 8px;
            text-align: center;
        }
        
        .stakeholder-icon {
            font-size: 2.5rem;
            color: var(--nasa-orange);
            margin-bottom: 1rem;
        }
        
        .implementation-steps {
            counter-reset: step-counter;
            list-style: none;
        }
        
        .implementation-steps li {
            counter-increment: step-counter;
            background: rgba(11, 61, 145, 0.2);
            padding: 1.2rem 1.2rem 1.2rem 4rem;
            margin-bottom: 1rem;
            border-radius: 8px;
            position: relative;
            border-left: 4px solid var(--nasa-green);
        }
        
        .implementation-steps li::before {
            content: counter(step-counter);
            position: absolute;
            left: 1rem;
            top: 50%;
            transform: translateY(-50%);
            width: 2rem;
            height: 2rem;
            background: var(--nasa-green);
            color: white;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
        }
        
        .reference-list {
            list-style: none;
        }
        
        .reference-list li {
            margin-bottom: 0.8rem;
            padding-left: 1.5rem;
            position: relative;
        }
        
        .reference-list li::before {
            content: '🛰️';
            position: absolute;
            left: 0;
        }
        
        .data-visualization {
            background: rgba(11, 61, 145, 0.3);
            padding: 1.5rem;
            border-radius: 8px;
            margin: 1.5rem 0;
            border: 1px solid rgba(255,255,255,0.1);
        }
        
        .viz-placeholder {
            height: 200px;
            background: linear-gradient(135deg, var(--nasa-dark), #0A2342);
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #ccc;
            font-style: italic;
        }
        
        .nasa-footer {
            background: var(--nasa-dark);
            padding: 2rem;
            margin-top: 3rem;
            border-top: 1px solid rgba(255,255,255,0.1);
        }
        
        .footer-content {
            max-width: 1200px;
            margin: 0 auto;
            text-align: center;
            color: #888;
        }
        
        .disclaimer {
            font-size: 0.9rem;
            margin-top: 1rem;
            color: #666;
        }
        
        /* Animation classes */
        .fade-in {
            animation: fadeIn 0.8s ease-in;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .pulse {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(39, 174, 96, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(39, 174, 96, 0); }
            100% { box-shadow: 0 0 0 0 rgba(39, 174, 96, 0); }
        }
        
        /* Responsive Design */
        @media (max-width: 768px) {
            .plan-container {
                padding: 0 1rem;
            }
            
            .plan-section {
                padding: 1.5rem;
            }
            
            .main-title {
                font-size: 1.5rem;
            }
            
            .section-title {
                font-size: 1.5rem;
            }
            
            .resource-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <header class="nasa-header">
        <div class="header-content">
            <div class="logo-container">
                <div class="nasa-logo pulse">
                    UP
                </div>
                <div class="logo-text">
                    <h1 class="main-title">URBAN-PLANET BOLIVIA</h1>
                    <div class="tagline">NASA EARTH OBSERVATION RESILIENCE PLAN</div>
                </div>
            </div>
        </div>
    </header>

    <div class="plan-container">
        <!-- Overview Section -->
        <section class="plan-section fade-in">
            <h2 class="section-title">
                <i class="fas fa-globe-americas"></i>
                Resilient City Plan Informed by NASA Earth Observation Data
            </h2>
            
            <div class="overview-box">
                <p><strong>Overview:</strong></p>
                <p>Climate change adds new layers of complexity to maintaining the wellbeing of people and the environment in rapidly growing cities. Using <a href="https://www.earthdata.nasa.gov/" target="_blank" class="resource-link">NASA Earth observation data</a>, this plan outlines how urban planners can create sustainable development strategies that protect both citizens and ecosystems.</p>
            </div>
        </section>

        <!-- NASA Data Resources -->
        <section class="plan-section fade-in">
            <h2 class="section-title">
                <i class="fas fa-satellite"></i>
                Key NASA Data Resources
            </h2>
            
            <div class="resource-grid">
                <div class="resource-card">
                    <div class="resource-icon">🌡️</div>
                    <h3 class="resource-title">Urban Extreme Heat Dataset</h3>
                    <p>Estimates population exposure to extreme heat events through NASA Earthdata / SEDAC.</p>
                    <a href="https://www.earthdata.nasa.gov/news/feature-articles/urban-extreme-heat-dataset-offers-population-exposure-estimates-more-than" target="_blank" class="resource-link">
                        Access Dataset <i class="fas fa-external-link-alt"></i>
                    </a>
                </div>
                
                <div class="resource-card">
                    <div class="resource-icon">🌳</div>
                    <h3 class="resource-title">Forest Change Patterns</h3>
                    <p>Tracks forest cover loss and land-use dynamics in Bolivia through NASA Earth Observatory.</p>
                    <a href="https://earthobservatory.nasa.gov/images/150257/patterns-of-forest-change-in-bolivia" target="_blank" class="resource-link">
                        View Analysis <i class="fas fa-external-link-alt"></i>
                    </a>
                </div>
                
                <div class="resource-card">
                    <div class="resource-icon">🛰️</div>
                    <h3 class="resource-title">NASA Worldview / GIBS</h3>
                    <p>Real-time satellite imagery and environmental monitoring platform for global observation.</p>
                    <a href="https://worldview.earthdata.nasa.gov/" target="_blank" class="resource-link">
                        Explore Platform <i class="fas fa-external-link-alt"></i>
                    </a>
                </div>
                
                <div class="resource-card">
                    <div class="resource-icon">🌫️</div>
                    <h3 class="resource-title">Air Quality Monitoring</h3>
                    <p>Monitors air pollution and nitrogen dioxide concentrations in Santa Cruz de la Sierra.</p>
                    <a href="https://airquality.gsfc.nasa.gov/no2/world/south-and-central-america/santa-cruz-de-la-sierra" target="_blank" class="resource-link">
                        Check Air Quality <i class="fas fa-external-link-alt"></i>
                    </a>
                </div>
                
                <div class="resource-card">
                    <div class="resource-icon">🏞️</div>
                    <h3 class="resource-title">Urban Public Space Data</h3>
                    <p>SDG Indicator 11.7.1 datasets on access to public parks and greenspaces via SEDAC.</p>
                    <a href="https://www.earthdata.nasa.gov/data/catalog/sedac-ciesin-sedac-sdgi-upsae-2023-2023.00" target="_blank" class="resource-link">
                        Access SDG Data <i class="fas fa-external-link-alt"></i>
                    </a>
                </div>
            </div>
        </section>

        <!-- Strategic Components -->
        <section class="plan-section fade-in">
            <h2 class="section-title">
                <i class="fas fa-chess-board"></i>
                Main Strategic Components
            </h2>
            
            <ul class="strategy-list">
                <li class="strategy-item">
                    <span class="strategy-number">1</span>
                    <h3 class="section-subtitle">Mapping Urban Heat Exposure</h3>
                    <p>Using the Urban Extreme Heat Dataset, planners can identify neighborhoods with the highest exposure to heat stress and prioritize interventions such as green roofs, tree canopies, and cooling centers.</p>
                </li>
                
                <li class="strategy-item">
                    <span class="strategy-number">2</span>
                    <h3 class="section-subtitle">Monitoring Deforestation and Vegetation Change</h3>
                    <p>Satellite imagery from NASA's Earth Observatory enables urban planners to monitor forest loss near expanding cities and implement conservation boundaries to protect biodiversity and ecosystem services.</p>
                </li>
                
                <li class="strategy-item">
                    <span class="strategy-number">3</span>
                    <h3 class="section-subtitle">Assessing Air Quality and Pollution Hotspots</h3>
                    <p>NASA's Air Quality platform offers long-term data on NO₂ and particulate matter to locate industrial and traffic emission zones, helping cities design cleaner transport and emission policies.</p>
                </li>
                
                <li class="strategy-item">
                    <span class="strategy-number">4</span>
                    <h3 class="section-subtitle">Evaluating Access to Greenspace</h3>
                    <p>The SDG 11.7.1 dataset helps identify which communities lack access to public parks or recreation areas. Urban planners can then design micro-parks and community gardens to improve local wellbeing.</p>
                </li>
                
                <li class="strategy-item">
                    <span class="strategy-number">5</span>
                    <h3 class="section-subtitle">Visualization and Public Engagement</h3>
                    <p>By integrating datasets into the NASA Worldview platform, planners can visualize heat maps, pollution layers, and vegetation indices in real time. This supports transparent, data-driven decisions and allows citizens to participate by submitting local observations or feedback.</p>
                </li>
            </ul>
        </section>

        <!-- Stakeholder Roles -->
        <section class="plan-section fade-in">
            <h2 class="section-title">
                <i class="fas fa-users"></i>
                Stakeholder Roles
            </h2>
            
            <div class="stakeholder-grid">
                <div class="stakeholder-card">
                    <div class="stakeholder-icon"><i class="fas fa-landmark"></i></div>
                    <h3>Municipal Governments</h3>
                    <p>Use NASA data-driven analyses to prioritize public investments and climate adaptation measures.</p>
                </div>
                
                <div class="stakeholder-card">
                    <div class="stakeholder-icon"><i class="fas fa-tree"></i></div>
                    <h3>Parks & Public Health</h3>
                    <p>Implement green infrastructure projects, heat mitigation plans, and air quality programs.</p>
                </div>
                
                <div class="stakeholder-card">
                    <div class="stakeholder-icon"><i class="fas fa-hands-helping"></i></div>
                    <h3>Communities & NGOs</h3>
                    <p>Participate in data collection, citizen science, and monitoring of environmental quality.</p>
                </div>
            </div>
        </section>

        <!-- Implementation Steps -->
        <section class="plan-section fade-in">
            <h2 class="section-title">
                <i class="fas fa-tasks"></i>
                Implementation Steps
            </h2>
            
            <ol class="implementation-steps">
                <li>Download base datasets (UHE, Landsat/VIIRS, NO₂, SDG 11.7.1) from NASA Earthdata.</li>
                <li>Build GIS layers for vulnerability mapping and urban planning analysis.</li>
                <li>Develop pilot interventions (green corridors, rooftop gardens, cooling zones) and monitor impacts using NASA imagery.</li>
                <li>Launch a public online dashboard with visualized data, reports, and periodic updates.</li>
            </ol>
            
            <div class="data-visualization">
                <h3 class="section-subtitle">Data Integration Example</h3>
                <div class="viz-placeholder">
                    NASA Data Visualization: Heat Maps + Deforestation Layers + Air Quality Overlays
                </div>
            </div>
        </section>

        <!-- NASA References -->
        <section class="plan-section fade-in">
            <h2 class="section-title">
                <i class="fas fa-book"></i>
                NASA References
            </h2>
            
            <ul class="reference-list">
                <li><a href="https://www.earthdata.nasa.gov/news/feature-articles/urban-extreme-heat-dataset-offers-population-exposure-estimates-more-than" target="_blank" class="resource-link">Urban Extreme Heat Dataset (Earthdata / SEDAC)</a></li>
                <li><a href="https://earthobservatory.nasa.gov/images/150257/patterns-of-forest-change-in-bolivia" target="_blank" class="resource-link">Patterns of Forest Change (Earth Observatory)</a></li>
                <li><a href="https://worldview.earthdata.nasa.gov/" target="_blank" class="resource-link">NASA Worldview / GIBS Imagery Explorer</a></li>
                <li><a href="https://airquality.gsfc.nasa.gov/no2/world/south-and-central-america/santa-cruz-de-la-sierra" target="_blank" class="resource-link">NASA Air Quality — NO₂ Analysis</a></li>
                <li><a href="https://www.earthdata.nasa.gov/data/catalog/sedac-ciesin-sedac-sdgi-upsae-2023-2023.00" target="_blank" class="resource-link">SDG 11.7.1 — Access to Public Space (SEDAC)</a></li>
            </ul>
            
            <div class="disclaimer">
                <p><em>Note:</em> All links reference official NASA Earthdata resources. Ensure metadata, licensing, and data citations are included for operational or research use.</p>
            </div>
        </section>
    </div>

    <footer class="nasa-footer">
        <div class="footer-content">
            <p>Urban-Planet Bolivia · NASA Earth Observation Integration Platform</p>
            <p class="disclaimer">Data sourced from NASA Earthdata, SEDAC, and GIBS platforms</p>
        </div>
    </footer>

    <script>
        // Add interactive functionality
        document.addEventListener('DOMContentLoaded', function() {
            // Add smooth scrolling for anchor links
            document.querySelectorAll('a[href^="#"]').forEach(anchor => {
                anchor.addEventListener('click', function (e) {
                    e.preventDefault();
                    document.querySelector(this.getAttribute('href')).scrollIntoView({
                        behavior: 'smooth'
                    });
                });
            });
            
            // Add hover effects to resource cards
            const resourceCards = document.querySelectorAll('.resource-card');
            resourceCards.forEach(card => {
                card.addEventListener('mouseenter', function() {
                    this.style.transform = 'translateY(-5px)';
                });
                
                card.addEventListener('mouseleave', function() {
                    this.style.transform = 'translateY(0)';
                });
            });
            
            // Animate strategy items on scroll
            const observerOptions = {
                threshold: 0.1,
                rootMargin: '0px 0px -50px 0px'
            };
            
            const observer = new IntersectionObserver(function(entries) {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        entry.target.style.opacity = '1';
                        entry.target.style.transform = 'translateX(0)';
                    }
                });
            }, observerOptions);
            
            document.querySelectorAll('.strategy-item').forEach(item => {
                item.style.opacity = '0';
                item.style.transform = 'translateX(-20px)';
                item.style.transition = 'opacity 0.6s ease, transform 0.6s ease';
                observer.observe(item);
            });
        });
    </script>
</body>
</html>
