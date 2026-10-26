<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Logistics Network and Coverage - MELCO LOGISTICS DE MEXICO</title>

    <!-- Leaflet Stylesheets -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        :root {
            --primary: #0a2540;
            --bg-light: #f8f9fa;
            --text-dark: #1e293b;
            
            /* COLOR CODES */
            --port-color: #00b4d8;      /* Blue (Main Ports) */
            --warehouse-color: #10b981; /* Green (Warehouses) */
            --local-color: #6366f1;     /* Purple/Indigo (National Routes) */
            --intl-color: #f59e0b;      /* Amber (US Cross-Border) */
        }

        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            height: 100vh;
            background-color: var(--bg-light);
            overflow: hidden;
        }

        #sidebar {
            width: 380px;
            background: #ffffff;
            border-right: 1px solid #e2e8f0;
            display: flex;
            flex-direction: column;
            box-shadow: 4px 0 15px rgba(0,0,0,0.05);
            z-index: 1000;
        }

        /* HEADER AND LOGO */
        .sidebar-header {
            padding: 16px 20px 12px 20px;
            background: var(--primary);
            color: #ffffff;
            text-align: center;
        }

        .logo-container {
            background: #ffffff;
            padding: 10px;
            border-radius: 8px;
            margin-bottom: 8px;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 2px 8px rgba(0,0,0,0.15);
        }

        .logo-container img {
            max-width: 100%;
            height: auto;
            max-height: 55px;
            object-fit: contain;
        }

        .sidebar-header p {
            margin: 4px 0 0 0;
            font-size: 11px;
            opacity: 0.85;
            color: #cbd5e1;
        }

        .controls-bar {
            padding: 12px 20px;
            background: #f1f5f9;
            border-bottom: 1px solid #e2e8f0;
        }

        .btn-toggle-all {
            width: 100%;
            padding: 10px 12px;
            background: var(--primary);
            border: none;
            border-radius: 6px;
            color: #ffffff;
            font-weight: 700;
            font-size: 12px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .btn-toggle-all:hover {
            background: #0066cc;
        }

        .sidebar-scroll {
            flex: 1;
            overflow-y: auto;
            padding: 10px 15px;
        }

        /* ACCORDION */
        .accordion-section {
            margin-bottom: 12px;
            border-radius: 8px;
            overflow: hidden;
            border: 1px solid #e2e8f0;
        }

        .accordion-header {
            padding: 12px 14px;
            font-size: 12px;
            font-weight: 800;
            letter-spacing: 0.5px;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
            user-select: none;
            transition: background 0.2s;
        }

        /* CATEGORY COLORS */
        .accordion-section.sec-ports .accordion-header { background: #e0f2fe; color: #0369a1; border-left: 5px solid var(--port-color); }
        .accordion-section.sec-wh .accordion-header { background: #dcfce7; color: #15803d; border-left: 5px solid var(--warehouse-color); }
        .accordion-section.sec-intl .accordion-header { background: #fef3c7; color: #b45309; border-left: 5px solid var(--intl-color); }
        .accordion-section.sec-local .accordion-header { background: #e0e7ff; color: #4338ca; border-left: 5px solid var(--local-color); }

        .accordion-body {
            display: none;
            padding: 10px;
            background: #f8fafc;
        }

        .accordion-section.open .accordion-body {
            display: block;
        }

        .accordion-arrow {
            font-size: 10px;
            transition: transform 0.2s;
        }

        .accordion-section.open .accordion-arrow {
            transform: rotate(180deg);
        }

        /* CATEGORY SHOW ALL BUTTON */
        .btn-category-all {
            width: 100%;
            padding: 8px 10px;
            margin-bottom: 8px;
            border: 1px dashed;
            border-radius: 6px;
            font-size: 11px;
            font-weight: 700;
            cursor: pointer;
            text-align: center;
            transition: all 0.2s;
        }

        .sec-ports .btn-category-all { background: #ffffff; color: #0369a1; border-color: var(--port-color); }
        .sec-wh .btn-category-all { background: #ffffff; color: #15803d; border-color: var(--warehouse-color); }
        .sec-intl .btn-category-all { background: #ffffff; color: #b45309; border-color: var(--intl-color); }
        .sec-local .btn-category-all { background: #ffffff; color: #4338ca; border-color: var(--local-color); }

        .btn-category-all:hover { opacity: 0.85; transform: scale(0.99); }

        /* ITEM CARDS */
        .sub-card {
            background: #ffffff;
            border-radius: 6px;
            padding: 8px 12px;
            margin-bottom: 6px;
            border: 1px solid #cbd5e1;
            cursor: pointer;
            font-size: 12px;
            font-weight: 600;
            color: var(--text-dark);
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: all 0.2s ease;
        }

        .sub-card:hover {
            transform: translateX(3px);
            box-shadow: 0 2px 8px rgba(0,0,0,0.06);
        }

        .sub-card.active {
            font-weight: 700;
        }

        .sec-ports .sub-card.active { border-left: 4px solid var(--port-color); background: #f0f9ff; }
        .sec-wh .sub-card.active { border-left: 4px solid var(--warehouse-color); background: #f0fdf4; }
        .sec-intl .sub-card.active { border-left: 4px solid var(--intl-color); background: #fffbeb; }
        .sec-local .sub-card.active { border-left: 4px solid var(--local-color); background: #f5f3ff; }

        .sub-badge {
            font-size: 10px;
            color: #64748b;
            background: #f1f5f9;
            padding: 2px 6px;
            border-radius: 4px;
        }

        #map { flex: 1; height: 100%; z-index: 1; background: #e5e3df; }

        .leaflet-control-attribution a {
            color: #0a2540 !important;
            font-weight: 700 !important;
            text-decoration: none !important;
        }

        .leaflet-control-attribution a:hover {
            text-decoration: underline !important;
        }

        #details-panel {
            position: absolute;
            top: 20px;
            right: -420px;
            width: 380px;
            background: #ffffff;
            box-shadow: -5px 0 25px rgba(0,0,0,0.15);
            border-radius: 12px;
            z-index: 2000;
            transition: right 0.35s cubic-bezier(0.4, 0, 0.2, 1);
            padding: 24px;
            box-sizing: border-box;
            max-height: calc(100vh - 40px);
            overflow-y: auto;
        }

        #details-panel.open { right: 20px; }

        .close-btn {
            position: absolute;
            top: 15px;
            right: 15px;
            border: none;
            background: #f1f5f9;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            cursor: pointer;
            font-weight: bold;
            color: #64748b;
        }

        .panel-state-title {
            margin: 0 0 6px 0;
            font-size: 18px;
            color: var(--primary);
            border-bottom: 2px solid var(--primary);
            padding-bottom: 8px;
        }

        .units-container {
            display: flex;
            gap: 8px;
            margin: 12px 0 16px 0;
            flex-wrap: wrap;
        }

        .unit-pill {
            background: #f1f5f9;
            color: #334155;
            border: 1px solid #cbd5e1;
            padding: 5px 10px;
            border-radius: 6px;
            font-size: 11px;
            font-weight: 600;
        }

        .companies-list { list-style: none; padding: 0; margin: 0; }
        .companies-list li {
            padding: 10px 12px;
            background: #f8fafc;
            border-radius: 6px;
            margin-bottom: 6px;
            font-size: 12px;
            color: #334155;
            border-left: 3px solid #cbd5e1;
        }

        /* MAP MARKERS */
        .custom-hub-marker {
            background-color: #dc3545;
            border: 3px solid #ffffff;
            border-radius: 50%;
            box-shadow: 0 0 12px rgba(220,53,69,0.6);
        }

        .custom-port-marker {
            background-color: var(--port-color);
            border: 2px solid #ffffff;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(0, 180, 216, 0.8);
        }

        .custom-warehouse-marker {
            background-color: var(--warehouse-color);
            border: 2px solid #ffffff;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(16, 185, 129, 0.8);
        }

        .custom-local-marker {
            background-color: var(--local-color);
            border: 2px solid #ffffff;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(99, 102, 241, 0.8);
        }

        .custom-intl-marker {
            background-color: var(--intl-color);
            border: 2px solid #ffffff;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(245, 158, 11, 0.8);
        }
    </style>
</head>
<body>

    <div id="sidebar">
        <div class="sidebar-header">
            <div class="logo-container">
                <img src="https://melcologistics.com.mx/imagenes/MELCO_LOGO_2_ORIGINAL.png" alt="MELCO LOGISTICS" onerror="this.style.display='none'">
            </div>
            <p>Terrestrial Logistics Network</p>
        </div>

        <div class="controls-bar">
            <button class="btn-toggle-all" onclick="mostrarRedCompleta()">🌐 View Entire Logistics Network</button>
        </div>
        
        <div class="sidebar-scroll">
            <!-- 1. National and Local Routes -->
            <div class="accordion-section sec-local" id="sec-local">
                <div class="accordion-header" onclick="toggleAccordion('sec-local')">
                    <span>🇲🇽 National and Local Routes</span>
                    <span class="accordion-arrow">▼</span>
                </div>
                <div class="accordion-body">
                    <button class="btn-category-all" onclick="activarGrupoCapas('nacional')">🌐 View National Network</button>
                    <div id="sub-list-local"></div>
                </div>
            </div>

            <!-- 2. US Cross-Border -->
            <div class="accordion-section sec-intl" id="sec-intl">
                <div class="accordion-header" onclick="toggleAccordion('sec-intl')">
                    <span>🇺🇸 US Cross-Border</span>
                    <span class="accordion-arrow">▼</span>
                </div>
                <div class="accordion-body">
                    <button class="btn-category-all" onclick="activarGrupoCapas('intl')">🌐 View International Routes</button>
                    <div id="sub-list-intl"></div>
                </div>
            </div>

            <!-- 3. Main Ports -->
            <div class="accordion-section sec-ports" id="sec-ports">
                <div class="accordion-header" onclick="toggleAccordion('sec-ports')">
                    <span>⚓ Main Ports</span>
                    <span class="accordion-arrow">▼</span>
                </div>
                <div class="accordion-body">
                    <button class="btn-category-all" onclick="activarGrupoCapas('puertos')">🌐 View Port Network</button>
                    <div id="sub-list-ports"></div>
                </div>
            </div>

            <!-- 4. Warehouses -->
            <div class="accordion-section sec-wh" id="sec-wh">
                <div class="accordion-header" onclick="toggleAccordion('sec-wh')">
                    <span>🏭 Warehouses</span>
                    <span class="accordion-arrow">▼</span>
                </div>
                <div class="accordion-body">
                    <button class="btn-category-all" onclick="activarGrupoCapas('almacenes')">🌐 View Warehouse Network</button>
                    <div id="sub-list-wh"></div>
                </div>
            </div>
        </div>
    </div>

    <div id="map"></div>

    <div id="details-panel">
        <button class="close-btn" onclick="cerrarPanel()">✕</button>
        <h2 id="panel-title" class="panel-state-title">Details</h2>
        
        <div id="panel-units-section">
            <h4 style="margin: 15px 0 6px 0; color:#64748b; font-size:11px; text-transform:uppercase;">🚛 Available Units</h4>
            <div id="panel-units" class="units-container"></div>
        </div>

        <div id="panel-companies-container"></div>
    </div>

    <!-- Leaflet and Ant-Path Scripts -->
    <script src="https://cdn.jsdelivr.net/npm/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/leaflet-ant-path@1.3.0/dist/leaflet-ant-path.min.js"></script>

    <script>
        const baseOperativa = {
            nombre: "MELCO LOGISTICS DE MEXICO",
            ubicacion: "O'Donnell Industrial Park, Querétaro",
            coords: [20.5750, -100.1800]
        };

        const NODOS = {
            QRO: [20.5750, -100.1800],
            SLP: [22.1565, -100.9855],
            MATEHUALA: [23.6496, -100.6438],
            SALTILLO: [25.4232, -101.0053],
            MTY: [25.6866, -100.3161],
            TORREON: [25.5428, -103.4068],
            LAREDO: [27.5036, -99.5075],
            SAN_ANTONIO: [29.4241, -98.4936],
            DALLAS: [32.7767, -96.7970],
            INDIANAPOLIS: [39.7684, -86.1581],
            MASON_OH: [39.3601, -84.3099],
            CANTON_OH: [40.7989, -81.3784],
            LITENS_ON: [43.7911, -79.5447],
            LONDON_OH: [39.8864, -83.4483],
            DETROIT_MI: [42.3314, -83.0458],
            CYPRESS_CA: [33.8242, -118.0378],
            CELAYA: [20.5283, -100.8123],
            IRAPUATO: [20.6767, -101.3563],
            SILAO_GTO: [20.8500, -101.3200],
            AGS: [21.8823, -102.2826],
            LAGOS_JAL: [21.3551, -101.9168],
            GDL: [20.6597, -103.3496],
            COLIMA: [19.2433, -103.7250],
            MANZANILLO: [19.0522, -104.3158],
            LAZARO_CARDENAS: [17.9481, -102.1808],
            URUAPAN: [19.4170, -102.0620],
            MORELIA: [19.7060, -101.1950],
            VERACRUZ: [19.2003, -96.1422],
            PUEBLA: [19.0414, -98.2063],
            TEPOTZOTLAN: [19.7153, -99.2223],
            SJR: [20.3888, -99.9961],
            HERMOSILLO: [29.0729, -110.9559],
            TIJUANA: [32.5149, -116.9000],
            CDMX: [19.4326, -99.1332]
        };

        const almacenesVerdes = [
            { id: "WH-QRO1", nombre: "MELCO Main Warehouse (O'Donnell)", ciudad: "Querétaro", coords: [20.5750, -100.1800] },
            { id: "WH-QRO2", nombre: "Arso Park Warehouse", ciudad: "Querétaro", coords: [20.6120, -100.2850] },
            { id: "WH-QRO3", nombre: "Global Park Warehouse", ciudad: "Querétaro", coords: [20.5980, -100.1650] },
            { id: "WH-ZLO", nombre: "Manzanillo Warehouse (Cross-dock)", ciudad: "Manzanillo", coords: [19.0522, -104.3158] },
            { id: "WH-MTY", nombre: "Monterrey Warehouse (North Hub)", ciudad: "Monterrey", coords: [25.6866, -100.3161] },
            { id: "WH-LRD", nombre: "Laredo Warehouse (Border Cross-dock)", ciudad: "Laredo, TX / Nvo. Laredo", coords: [27.5036, -99.5075] }
        ];

        const puertosAzules = [
            {
                id: "PORT-ZLO",
                nombre: "Port of Manzanillo",
                estado: "Colima",
                frecuencia: "60 trips / month",
                coords: NODOS.MANZANILLO,
                waypoints: [NODOS.MANZANILLO, NODOS.COLIMA, NODOS.GDL, NODOS.LAGOS_JAL, NODOS.IRAPUATO, NODOS.CELAYA, NODOS.QRO],
                description: "Main Mexican Pacific port for containerized cargo import."
            },
            {
                id: "PORT-LZC",
                nombre: "Port of Lázaro Cárdenas",
                estado: "Michoacán",
                frecuencia: "10 trips / month",
                coords: NODOS.LAZARO_CARDENAS,
                waypoints: [NODOS.LAZARO_CARDENAS, NODOS.URUAPAN, NODOS.MORELIA, NODOS.CELAYA, NODOS.QRO],
                description: "Deep-water port in the Pacific, specialized in heavy machinery and containers."
            },
            {
                id: "PORT-VER",
                nombre: "Port of Veracruz",
                estado: "Veracruz",
                frecuencia: null,
                coords: NODOS.VERACRUZ,
                waypoints: [NODOS.VERACRUZ, NODOS.PUEBLA, NODOS.CDMX, NODOS.TEPOTZOTLAN, NODOS.SJR, NODOS.QRO],
                description: "Main port of the Gulf of Mexico for Atlantic and European routes."
            }
        ];

        const coberturaEstados = [
            {
                id: "EST-LAREDO-MX",
                categoria: "intl",
                estado: "Laredo (Border Cross-dock / Round Trip)",
                frecuencia: "24 trips / month",
                bajadas: "7 drop-offs / month in Querétaro",
                coords: NODOS.LAREDO,
                waypoints: [NODOS.QRO, NODOS.SLP, NODOS.MATEHUALA, NODOS.SALTILLO, NODOS.MTY, NODOS.LAREDO],
                esInternacional: true,
                tieneRetorno: true,
                excluirDeTotalRed: true,
                unidades: ["53ft DV (Mexican Chassis)"],
                empresas: ["Nuevo Laredo / Laredo TX Cross-dock Hub"]
            },
            {
                id: "EST-MASON",
                categoria: "intl",
                estado: "Mason, OH (USA)",
                frecuencia: "21 trips / month",
                coords: NODOS.MASON_OH,
                waypoints: [NODOS.LAREDO, NODOS.SAN_ANTONIO, NODOS.DALLAS, NODOS.INDIANAPOLIS, NODOS.MASON_OH],
                esInternacional: true,
                tieneRetorno: true,
                unidades: ["53ft DV (US Chassis)"],
                empresas: ["Mitsubishi Electric Automotive America (MEAA - Mason, OH)"]
            },
            {
                id: "EST-LONDON",
                categoria: "intl",
                estado: "London, OH (USA)",
                frecuencia: "5 trips / month",
                coords: NODOS.LONDON_OH,
                waypoints: [NODOS.LAREDO, NODOS.SAN_ANTONIO, NODOS.DALLAS, NODOS.INDIANAPOLIS, NODOS.LONDON_OH],
                esInternacional: true,
                unidades: ["53ft DV (US Chassis)"],
                empresas: ["London Industrial Area, Ohio"]
            },
            {
                id: "EST-MICHIGAN",
                categoria: "intl",
                estado: "Michigan (USA)",
                frecuencia: "5 trips / month",
                coords: NODOS.DETROIT_MI,
                waypoints: [NODOS.LAREDO, NODOS.SAN_ANTONIO, NODOS.DALLAS, NODOS.INDIANAPOLIS, NODOS.DETROIT_MI],
                esInternacional: true,
                unidades: ["53ft DV (US Chassis)"],
                empresas: ["Michigan Automotive and Industrial Corridor"]
            },
            {
                id: "EST-CANTON",
                categoria: "intl",
                estado: "Canton, OH (USA)",
                frecuencia: "4 trips / month",
                coords: NODOS.CANTON_OH,
                waypoints: [NODOS.LAREDO, NODOS.SAN_ANTONIO, NODOS.DALLAS, NODOS.CANTON_OH],
                esInternacional: true,
                unidades: ["53ft DV (US Chassis)"],
                empresas: ["Ohio Industrial Corridor"]
            },
            {
                id: "EST-LITENS",
                categoria: "intl",
                estado: "Litens (Canada / USA)",
                frecuencia: "As requested",
                coords: NODOS.LITENS_ON,
                waypoints: [NODOS.LAREDO, NODOS.SAN_ANTONIO, NODOS.DALLAS, NODOS.CANTON_OH, NODOS.LITENS_ON],
                esInternacional: true,
                unidades: ["53ft DV (US Chassis)"],
                empresas: ["Litens Automotive Group"]
            },
            {
                id: "EST-CALIFORNIA",
                categoria: "intl",
                estado: "California (USA)",
                frecuencia: "As requested",
                coords: NODOS.CYPRESS_CA,
                waypoints: [NODOS.LAREDO, [31.7619, -106.4850], [33.4484, -112.0740], NODOS.CYPRESS_CA],
                esInternacional: true,
                unidades: ["53ft DV (US Chassis)"],
                empresas: ["California / Cypress Industrial Corridor"]
            },
            {
                id: "EST-QRO",
                categoria: "nacional",
                estado: "Querétaro (Local / Matrix Base)",
                frecuencia: "25 trips / month",
                coords: NODOS.QRO,
                waypoints: [NODOS.QRO],
                esLocal: true,
                unidades: ["53ft DV", "Rabón (8t)", "3.5t", "1t"],
                empresas: ["MELMEX (Mitsubishi Electric Mex - SJR)", "ADVANEX MEXICO", "TERADAI MEXICO", "Sansho Mexicana", "JEM", "SUMITOMO"]
            },
            {
                id: "EST-NLR",
                categoria: "nacional",
                estado: "Nuevo León (Monterrey)",
                frecuencia: "1 trip / month",
                coords: NODOS.MTY,
                waypoints: [NODOS.QRO, NODOS.SLP, NODOS.MATEHUALA, NODOS.SALTILLO, NODOS.MTY],
                unidades: ["53ft DV", "Rabón (8t)", "3.5t"],
                empresas: ["REA", "AVO CARBON"]
            },
            {
                id: "EST-COAH",
                categoria: "nacional",
                estado: "Coahuila (Torreón)",
                frecuencia: "2 trips / month",
                coords: NODOS.TORREON,
                waypoints: [NODOS.QRO, NODOS.SLP, NODOS.MATEHUALA, NODOS.SALTILLO, NODOS.TORREON],
                unidades: ["53ft DV", "Rabón (8t)", "3.5t", "1t"],
                empresas: ["ESSEX GROUP, INC."]
            },
            {
                id: "EST-SLP",
                categoria: "nacional",
                estado: "San Luis Potosí",
                frecuencia: "1 trip / month",
                coords: NODOS.SLP,
                waypoints: [NODOS.QRO, [20.9992, -100.3831], NODOS.SLP],
                unidades: ["53ft DV", "Rabón (8t)", "3.5t", "1t"],
                empresas: ["NMB"]
            },
            {
                id: "EST-GTO",
                categoria: "nacional",
                estado: "Guanajuato (Silao / Salamanca / San José Iturbide / Celaya / Irapuato / Abasolo)",
                frecuencia: "31 trips / month",
                coords: NODOS.SILAO_GTO,
                waypoints: [NODOS.QRO, NODOS.CELAYA, NODOS.IRAPUATO, NODOS.SILAO_GTO],
                unidades: ["53ft DV", "Rabón (8t)", "3.5t", "1t"],
                empresas: ["KATOLEC (Guanajuato Capital)", "IKD", "SHOEI (Abasolo)", "NEDEC (San José Iturbide)", "Hitachi Astemo Silao", "HONDA CELAYA", "MAZDA Salamanca", "SCHAEFFLER IRAPUATO", "HISCOMEX"]
            },
            {
                id: "EST-AGS",
                categoria: "nacional",
                estado: "Aguascalientes",
                frecuencia: "1 trip / month",
                coords: NODOS.AGS,
                waypoints: [NODOS.QRO, NODOS.CELAYA, NODOS.IRAPUATO, NODOS.AGS],
                unidades: ["53ft DV", "Rabón (8t)", "3.5t"],
                empresas: ["NISSAN PLANT A1"]
            },
            {
                id: "EST-JAL",
                categoria: "nacional",
                estado: "Jalisco (Lagos de Moreno)",
                frecuencia: "1 trip / month",
                coords: NODOS.LAGOS_JAL,
                waypoints: [NODOS.QRO, NODOS.CELAYA, NODOS.IRAPUATO, NODOS.LAGOS_JAL],
                unidades: ["53ft DV", "Torton / Rabón", "3.5t", "1t"],
                empresas: ["STANLEY (Lagos de Moreno)"]
            },
            {
                id: "EST-TJ",
                categoria: "nacional",
                estado: "Baja California (Tijuana)",
                frecuencia: "11 trips / month",
                coords: NODOS.TIJUANA,
                waypoints: [NODOS.QRO, NODOS.CELAYA, NODOS.GDL, [21.5039, -104.8946], [23.2494, -106.4111], NODOS.HERMOSILLO, NODOS.TIJUANA],
                esNacional: true,
                tieneRetorno: true,
                unidades: ["53ft DV"],
                empresas: ["JFE (Tijuana)", "PIMS (Mexicali)", "ANA GLOBAL", "SUMITRONICS"]
            },
            {
                id: "EST-MEX",
                categoria: "nacional",
                estado: "Edomex / CDMX (Metropolitan Area)",
                frecuencia: "1 trip / month",
                coords: NODOS.CDMX,
                waypoints: [NODOS.QRO, NODOS.SJR, NODOS.TEPOTZOTLAN, NODOS.CDMX],
                unidades: ["53ft DV", "Rabón", "3.5t"],
                empresas: ["SCHAEFFLER", "DCC"]
            }
        ];

        // Map Initialization
        const map = L.map('map', { 
            center: [23.6345, -102.5528], 
            zoom: 5, 
            zoomControl: false,
            attributionControl: false
        });

        L.control.zoom({ position: 'topleft' }).addTo(map);

        L.control.attribution({
            position: 'bottomright',
            prefix: '<a href="https://melcologistics.com.mx/" target="_blank">MELCO LOGISTICS DE MEXICO</a>'
        }).addTo(map);

        // Esri World Street Map Base Layer
        L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}', {
            maxZoom: 19,
            attribution: 'Tiles &copy; Esri &mdash; Source: Esri, DeLorme, NAVTEQ, USGS, Intermap, iPC, NRCAN, Esri Japan, METI, Esri China (Hong Kong), Esri (Thailand), TomTom'
        }).addTo(map);

        const capaBase = L.featureGroup().addTo(map);
        const capaAlmacenes = L.featureGroup().addTo(map);
        const capaPuertos = L.featureGroup().addTo(map);
        const capaIntl = L.featureGroup().addTo(map);
        const capaNacional = L.featureGroup().addTo(map);
        const capaAnimada = L.featureGroup().addTo(map);

        L.marker(baseOperativa.coords, {
            icon: L.divIcon({ className: 'custom-hub-marker', iconSize: [18, 18], iconAnchor: [9, 9] })
        }).addTo(capaBase).bindTooltip(`<b>${baseOperativa.nombre}</b><br>Matrix Logistics Center (O'Donnell)`, { direction: 'top' });

        const mapAlmacenesMarkers = new Map();
        const mapPuertosMarkers = new Map();
        const mapRutas = new Map();
        const subCardsMap = new Map();

        function interpolarRutaCarretera(waypoints, pasosPorTRAMO = 15) {
            const puntosFinales = [];
            for (let i = 0; i < waypoints.length - 1; i++) {
                const p0 = waypoints[i];
                const p1 = waypoints[i + 1];
                for (let j = 0; j <= pasosPorTRAMO; j++) {
                    const t = j / pasosPorTRAMO;
                    puntosFinales.push([p0[0] + (p1[0] - p0[0]) * t, p0[1] + (p1[1] - p0[1]) * t]);
                }
            }
            return puntosFinales;
        }

        almacenesVerdes.forEach(a => {
            const m = L.marker(a.coords, {
                icon: L.divIcon({ className: 'custom-warehouse-marker', iconSize: [14, 14], iconAnchor: [7, 7] })
            }).addTo(capaAlmacenes).bindTooltip(`<b>🏭 ${a.nombre}</b> (${a.ciudad})`, { direction: 'top' });
            
            m.on('click', () => seleccionarAlmacenIndividual(a));
            mapAlmacenesMarkers.set(a.id, m);
        });

        puertosAzules.forEach(p => {
            p.puntosRuta = interpolarRutaCarretera(p.waypoints);
            const tooltipText = `<b>⚓ ${p.nombre}</b> (${p.estado})`;

            const m = L.marker(p.coords, {
                icon: L.divIcon({ className: 'custom-port-marker', iconSize: [14, 14], iconAnchor: [7, 7] })
            }).addTo(capaPuertos).bindTooltip(tooltipText, { direction: 'top' });
            
            m.on('click', () => seleccionarPuertoIndividual(p));
            mapPuertosMarkers.set(p.id, m);
        });

        coberturaEstados.forEach((item) => {
            const puntosTrazo = item.esLocal ? [item.coords] : interpolarRutaCarretera(item.waypoints);
            const destinoCapas = item.esInternacional ? capaIntl : capaNacional;

            mapRutas.set(item.id, { puntos: puntosTrazo, item: item });

            if (!item.esLocal) {
                const markerClass = item.esInternacional ? 'custom-intl-marker' : 'custom-local-marker';
                const m = L.marker(item.coords, {
                    icon: L.divIcon({ className: markerClass, iconSize: [12, 12], iconAnchor: [6, 6] })
                }).addTo(destinoCapas);
                m.on('click', () => activarRutaEspecifica(item));
            }
        });

        const subListPorts = document.getElementById('sub-list-ports');
        puertosAzules.forEach(p => {
            const card = document.createElement('div');
            card.className = 'sub-card';
            card.innerHTML = `<span>⚓ ${p.nombre}</span> <span class="sub-badge">${p.estado}</span>`;
            card.onclick = () => seleccionarPuertoIndividual(p);
            subListPorts.appendChild(card);
            subCardsMap.set(p.id, card);
        });

        const subListWH = document.getElementById('sub-list-wh');
        almacenesVerdes.forEach(a => {
            const card = document.createElement('div');
            card.className = 'sub-card';
            card.innerHTML = `<span>🏭 ${a.nombre}</span> <span class="sub-badge">${a.ciudad}</span>`;
            card.onclick = () => seleccionarAlmacenIndividual(a);
            subListWH.appendChild(card);
            subCardsMap.set(a.id, card);
        });

        const subListIntl = document.getElementById('sub-list-intl');
        const subListLocal = document.getElementById('sub-list-local');

        coberturaEstados.forEach(item => {
            const card = document.createElement('div');
            card.className = 'sub-card';
            let badgeText = `${item.empresas.length} destination(s)`;
            if (item.tieneRetorno) {
                badgeText = `🔄 Round Trip | ${badgeText}`;
            }
            card.innerHTML = `<span>${item.estado}</span> <span class="sub-badge">${badgeText}</span>`;
            card.onclick = () => activarRutaEspecifica(item);
            subCardsMap.set(item.id, card);

            if (item.esInternacional) subListIntl.appendChild(card);
            else subListLocal.appendChild(card);
        });

        function toggleAccordion(id) {
            const sec = document.getElementById(id);
            const estaAbierto = sec.classList.contains('open');
            document.querySelectorAll('.accordion-section').forEach(s => s.classList.remove('open'));
            if (!estaAbierto) sec.classList.add('open');
        }

        function ocultarTodasLasCapas() {
            capaAnimada.clearLayers();
            if (map.hasLayer(capaAlmacenes)) map.removeLayer(capaAlmacenes);
            if (map.hasLayer(capaPuertos)) map.removeLayer(capaPuertos);
            if (map.hasLayer(capaIntl)) map.removeLayer(capaIntl);
            if (map.hasLayer(capaNacional)) map.removeLayer(capaNacional);
            subCardsMap.forEach(c => c.classList.remove('active'));
        }

        function crearRutaConFlechas(puntos, colorHex, reverse = false) {
            if (L.polyline.antPath && typeof L.polyline.antPath === 'function') {
                return L.polyline.antPath(puntos, {
                    delay: 800,
                    dashArray: [15, 25],
                    weight: 4,
                    color: colorHex,
                    pulseColor: '#ffffff',
                    paused: false,
                    reverse: reverse,
                    hardwareAcceleration: true
                });
            } else {
                return L.polyline(puntos, {
                    weight: 4,
                    color: colorHex,
                    opacity: 0.8
                });
            }
        }

        function activarGrupoCapas(tipo) {
            ocultarTodasLasCapas();

            if (tipo === 'almacenes') {
                map.addLayer(capaAlmacenes);
                document.getElementById('panel-title').innerText = "Warehouses";
                document.getElementById('panel-units-section').style.display = 'none';

                let htmlHTML = `<ul class="companies-list">`;
                almacenesVerdes.forEach(a => { htmlHTML += `<li><b>🏭 ${a.nombre}</b><br><small>Location: ${a.ciudad}</small></li>`; });
                htmlHTML += `</ul>`;

                document.getElementById('panel-companies-container').innerHTML = htmlHTML;
                document.getElementById('details-panel').classList.add('open');
                if (capaAlmacenes.getBounds().isValid()) map.fitBounds(capaAlmacenes.getBounds(), { padding: [40, 40] });
            } else if (tipo === 'puertos') {
                map.addLayer(capaPuertos);
                document.getElementById('panel-title').innerText = "Import Seaports";
                document.getElementById('panel-units-section').style.display = 'none';

                puertosAzules.forEach(p => {
                    if (p.puntosRuta) {
                        crearRutaConFlechas(p.puntosRuta, '#00b4d8').addTo(capaAnimada);
                    }
                });

                let htmlHTML = `<ul class="companies-list">`;
                puertosAzules.forEach(p => { 
                    htmlHTML += `<li><b>⚓ ${p.nombre}</b> (${p.estado})<br><small>Corridor: Port ➔ Querétaro Land Connection</small></li>`; 
                });
                htmlHTML += `</ul>`;

                document.getElementById('panel-companies-container').innerHTML = htmlHTML;
                document.getElementById('details-panel').classList.add('open');
                if (capaPuertos.getBounds().isValid()) map.fitBounds(capaPuertos.getBounds(), { padding: [40, 40] });
            } else if (tipo === 'intl') {
                map.addLayer(capaIntl);
                document.getElementById('panel-title').innerText = "US Cross-Border Network";
                document.getElementById('panel-units-section').style.display = 'none';

                mapRutas.forEach((datos) => {
                    if (datos.item.esInternacional) {
                        const color = '#f59e0b';
                        crearRutaConFlechas(datos.puntos, color, false).addTo(capaAnimada);
                        if (datos.item.tieneRetorno) {
                            crearRutaConFlechas(datos.puntos, '#ef4444', true).addTo(capaAnimada);
                        }
                    }
                });

                let htmlHTML = `<ul class="companies-list">`;
                coberturaEstados.filter(i => i.esInternacional).forEach(i => {
                    htmlHTML += `<li><b>🇺🇸 ${i.estado}</b><br><small style="display:block; margin-top:4px;">${i.empresas.join(', ')}</small></li>`;
                });
                htmlHTML += `</ul>`;

                document.getElementById('panel-companies-container').innerHTML = htmlHTML;
                document.getElementById('details-panel').classList.add('open');
                if (capaIntl.getBounds().isValid()) map.fitBounds(capaIntl.getBounds(), { padding: [40, 40] });
            } else if (tipo === 'nacional') {
                map.addLayer(capaNacional);
                document.getElementById('panel-title').innerText = "National and Local Transport Network";
                document.getElementById('panel-units-section').style.display = 'none';

                mapRutas.forEach((datos) => {
                    if (!datos.item.esInternacional && !datos.item.esLocal) {
                        const color = '#6366f1';
                        crearRutaConFlechas(datos.puntos, color, false).addTo(capaAnimada);
                        if (datos.item.tieneRetorno) {
                            crearRutaConFlechas(datos.puntos, '#ef4444', true).addTo(capaAnimada);
                        }
                    }
                });

                let htmlHTML = `<ul class="companies-list">`;
                coberturaEstados.filter(i => !i.esInternacional).forEach(i => {
                    htmlHTML += `<li><b>🇲🇽 ${i.estado}</b><br><small>${i.empresas.length} destination(s)</small></li>`;
                });
                htmlHTML += `</ul>`;

                document.getElementById('panel-companies-container').innerHTML = htmlHTML;
                document.getElementById('details-panel').classList.add('open');
                if (capaNacional.getBounds().isValid()) map.fitBounds(capaNacional.getBounds(), { padding: [40, 40] });
            }
        }

        function seleccionarPuertoIndividual(p) {
            ocultarTodasLasCapas();
            map.addLayer(capaPuertos);

            const card = subCardsMap.get(p.id);
            if (card) card.classList.add('active');

            document.getElementById('panel-title').innerText = p.nombre;
            document.getElementById('panel-units-section').style.display = 'none';

            let htmlContenido = '';
            if (p.frecuencia) {
                htmlContenido += `
                <div style="margin: 12px 0; background: #e0f2fe; border: 1px solid #00b4d8; border-radius: 8px; padding: 10px 12px;">
                    <div style="display: flex; align-items: center; justify-content: space-between;">
                        <span style="font-size: 11px; font-weight: 700; color: #0369a1; text-transform: uppercase;">🔄 Operational Frequency</span>
                        <span style="font-size: 12px; font-weight: 800; color: #0369a1; background: #ffffff; padding: 3px 8px; border-radius: 6px; border: 1px solid #cbd5e1;">${p.frecuencia}</span>
                    </div>
                </div>`;
            }

            htmlContenido += `<ul class="companies-list"><li><b>State:</b> ${p.estado}</li><li>${p.description}</li></ul>`;

            document.getElementById('panel-companies-container').innerHTML = htmlContenido;
            document.getElementById('details-panel').classList.add('open');

            if (p.puntosRuta) {
                map.fitBounds(L.latLngBounds(p.puntosRuta), { padding: [40, 40], maxZoom: 11, animate: false });
                crearRutaConFlechas(p.puntosRuta, '#00b4d8').addTo(capaAnimada);
            }
        }

        function seleccionarAlmacenIndividual(a) {
            ocultarTodasLasCapas();
            map.addLayer(capaAlmacenes);

            const card = subCardsMap.get(a.id);
            if (card) card.classList.add('active');

            document.getElementById('panel-title').innerText = a.nombre;
            document.getElementById('panel-units-section').style.display = 'none';
            document.getElementById('panel-companies-container').innerHTML = `<ul class="companies-list"><li><b>Location:</b> ${a.ciudad}</li><li>Operational storage and distribution facility.</li></ul>`;
            document.getElementById('details-panel').classList.add('open');

            map.flyTo(a.coords, 12, { duration: 1.2 });
        }

        function activarRutaEspecifica(item) {
            ocultarTodasLasCapas();

            const cardSeleccionada = subCardsMap.get(item.id);
            if (cardSeleccionada) cardSeleccionada.classList.add('active');

            if (item.esInternacional) map.addLayer(capaIntl);
            else map.addLayer(capaNacional);

            actualizarPanelDetalles(item);

            if (item.esLocal) {
                map.flyTo(item.coords, 11, { duration: 1.2 });
                return;
            }

            const datosRuta = mapRutas.get(item.id);
            if (!datosRuta || !datosRuta.puntos || datosRuta.puntos.length === 0) return;

            map.fitBounds(L.latLngBounds(datosRuta.puntos), { padding: [40, 40], maxZoom: 11, animate: false });

            const colorLinea = item.esInternacional ? '#f59e0b' : '#6366f1';
            
            crearRutaConFlechas(datosRuta.puntos, colorLinea, false).addTo(capaAnimada);
            if (item.tieneRetorno) {
                crearRutaConFlechas(datosRuta.puntos, '#ef4444', true).addTo(capaAnimada);
            }
        }

        function extraerNumeroViajes(frecuenciaStr) {
            if (!frecuenciaStr) return 0;
            const match = frecuenciaStr.match(/(\d+)/);
            return match ? parseInt(match[1], 10) : 0;
        }

        function mostrarRedCompleta() {
            ocultarTodasLasCapas();

            map.addLayer(capaAlmacenes);
            map.addLayer(capaPuertos);
            map.addLayer(capaIntl);
            map.addLayer(capaNacional);

            document.querySelectorAll('.accordion-section').forEach(s => s.classList.add('open'));
            subCardsMap.forEach(c => c.classList.remove('active'));

            mapRutas.forEach((datos) => {
                if (!datos.item.esLocal) {
                    const color = datos.item.esInternacional ? '#f59e0b' : '#6366f1';
                    crearRutaConFlechas(datos.puntos, color, false).addTo(capaAnimada);
                    if (datos.item.tieneRetorno) {
                        crearRutaConFlechas(datos.puntos, '#ef4444', true).addTo(capaAnimada);
                    }
                }
            });

            puertosAzules.forEach(p => {
                if (p.puntosRuta) {
                    crearRutaConFlechas(p.puntosRuta, '#00b4d8').addTo(capaAnimada);
                }
            });

            document.getElementById('panel-title').innerText = "Network General Summary";
            document.getElementById('panel-units-section').style.display = 'none';

            const totalPuertos = puertosAzules.length;
            const totalAlmacenes = almacenesVerdes.length;
            const rutasIntl = coberturaEstados.filter(i => i.esInternacional).length;
            const rutasNac = coberturaEstados.filter(i => !i.esInternacional).length;

            let totalDestinos = 0;
            coberturaEstados.forEach(i => totalDestinos += i.empresas.length);

            let estimacionViajesPuertos = 0;
            puertosAzules.forEach(p => { estimacionViajesPuertos += extraerNumeroViajes(p.frecuencia); });

            let estimacionViajesCrossBorder = 0;
            coberturaEstados.filter(i => i.esInternacional && !i.excluirDeTotalRed).forEach(i => {
                estimacionViajesCrossBorder += extraerNumeroViajes(i.frecuencia);
            });
            estimacionViajesCrossBorder = Math.max(0, estimacionViajesCrossBorder - 5);

            let estimacionViajesNacionales = 70;

            let htmlSummary = `
                <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px;">
                    <div style="background: #e0e7ff; padding: 12px; border-radius: 8px; border-left: 4px solid var(--local-color); text-align: center;">
                        <span style="font-size: 22px; font-weight: 800; color: #4338ca; display: block;">${rutasNac}</span>
                        <span style="font-size: 11px; font-weight: 700; color: #4338ca; text-transform: uppercase;">National Routes</span>
                    </div>
                    <div style="background: #fef3c7; padding: 12px; border-radius: 8px; border-left: 4px solid var(--intl-color); text-align: center;">
                        <span style="font-size: 22px; font-weight: 800; color: #b45309; display: block;">${rutasIntl}</span>
                        <span style="font-size: 11px; font-weight: 700; color: #b45309; text-transform: uppercase;">Cross-Border Routes</span>
                    </div>
                    <div style="background: #e0f2fe; padding: 12px; border-radius: 8px; border-left: 4px solid var(--port-color); text-align: center;">
                        <span style="font-size: 22px; font-weight: 800; color: #0369a1; display: block;">${totalPuertos}</span>
                        <span style="font-size: 11px; font-weight: 700; color: #0369a1; text-transform: uppercase;">Seaports</span>
                    </div>
                    <div style="background: #dcfce7; padding: 12px; border-radius: 8px; border-left: 4px solid var(--warehouse-color); text-align: center;">
                        <span style="font-size: 22px; font-weight: 800; color: #15803d; display: block;">${totalAlmacenes}</span>
                        <span style="font-size: 11px; font-weight: 700; color: #15803d; text-transform: uppercase;">Warehouses / Hubs</span>
                    </div>
                </div>

                <div style="margin: 15px 0; background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 8px; padding: 14px;">
                    <h4 style="margin: 0 0 10px 0; font-size: 12px; color: var(--primary); text-transform: uppercase;">📊 Consolidated Monthly Summary</h4>
                    <div style="display: flex; justify-content: space-between; font-size: 12px; margin-bottom: 6px; padding-bottom: 6px; border-bottom: 1px solid #e2e8f0;">
                        <span style="color: #64748b;">Customers / Industrial Destinations:</span>
                        <span style="font-weight: bold; color: var(--text-dark);">${totalDestinos} companies</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; font-size: 12px; margin-bottom: 6px; padding-bottom: 6px; border-bottom: 1px solid #e2e8f0;">
                        <span style="color: #64748b;">National / Local Trips:</span>
                        <span style="font-weight: bold; color: #4338ca;">~${estimacionViajesNacionales} / month</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; font-size: 12px; margin-bottom: 6px; padding-bottom: 6px; border-bottom: 1px solid #e2e8f0;">
                        <span style="color: #64748b;">US Cross-Border Trips:</span>
                        <span style="font-weight: bold; color: #b45309;">~${estimacionViajesCrossBorder} / month</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; font-size: 12px;">
                        <span style="color: #64748b;">Estimated Maritime Trips:</span>
                        <span style="font-weight: bold; color: #0369a1;">~${estimacionViajesPuertos} / month</span>
                    </div>
                </div>

                <div style="text-align: center; margin-top: 15px;">
                    <a href="https://melcologistics.com.mx/" target="_blank" style="font-size: 11px; color: #0066cc; text-decoration: none; font-weight: 700;">Visit official website of MELCO LOGISTICS DE MEXICO ➔</a>
                </div>
            `;

            document.getElementById('panel-companies-container').innerHTML = htmlSummary;
            document.getElementById('details-panel').classList.add('open');

            const bounds = L.latLngBounds([20.5750, -100.1800], [32.5149, -116.9000]);
            map.fitBounds(bounds, { padding: [40, 40], maxZoom: 5, animate: false });
        }

        function actualizarPanelDetalles(item) {
            document.getElementById('panel-title').innerText = item.estado;

            const unitsDiv = document.getElementById('panel-units');
            unitsDiv.innerHTML = '';
            if (item.unidades && item.unidades.length > 0) {
                document.getElementById('panel-units-section').style.display = 'block';
                item.unidades.forEach(u => {
                    const pill = document.createElement('div');
                    pill.className = 'unit-pill';
                    pill.innerText = `🚛 ${u}`;
                    unitsDiv.appendChild(pill);
                });
            } else {
                document.getElementById('panel-units-section').style.display = 'none';
            }

            let htmlContent = '';
            
            if (item.tieneRetorno) {
                htmlContent += `
                <div style="margin: 0 0 12px 0; background: #fef2f2; border: 1px solid #ef4444; border-radius: 8px; padding: 10px 12px; display: flex; align-items: center; gap: 8px;">
                    <span style="font-size: 16px;">🔄</span>
                    <div>
                        <strong style="font-size: 11px; color: #b91c1c; display: block; text-transform: uppercase;">Active Bidirectional Route</strong>
                        <span style="font-size: 11px; color: #7f1d1d;">Features confirmed flows for both outbound and return (Laredo ⇄ Querétaro round trip).</span>
                    </div>
                </div>`;
            }

            if (item.frecuencia) {
                let badgeColorBg = item.esInternacional ? '#fef3c7' : '#e0e7ff';
                let badgeColorText = item.esInternacional ? '#b45309' : '#4338ca';
                let borderColor = item.esInternacional ? '#f59e0b' : '#6366f1';
                
                htmlContent += `
                <div style="margin: 12px 0; background: ${badgeColorBg}; border: 1px solid ${borderColor}; border-radius: 8px; padding: 10px 12px;">
                    <div style="display: flex; align-items: center; justify-content: space-between;">
                        <span style="font-size: 11px; font-weight: 700; color: ${badgeColorText}; text-transform: uppercase;">🔄 Operational Frequency</span>
                        <span style="font-size: 12px; font-weight: 800; color: ${badgeColorText}; background: #ffffff; padding: 3px 8px; border-radius: 6px; border: 1px solid #cbd5e1;">${item.frecuencia}</span>
                    </div>`;
                if (item.bajadas) {
                    htmlContent += `
                    <div style="margin-top: 6px; font-size: 11px; font-weight: 600; color: #334155; border-top: 1px dashed rgba(0,0,0,0.1); padding-top: 6px;">
                        📥 ${item.bajadas}
                    </div>`;
                }
                htmlContent += `</div>`;
            }

            if (item.empresas && item.empresas.length > 0) {
                htmlContent += `<h4 style="margin: 15px 0 6px 0; color:var(--primary); font-size:11px; text-transform:uppercase;">🏢 Associated Plants and Industries</h4>`;
                htmlContent += `<ul class="companies-list">`;
                item.empresas.forEach(emp => {
                    htmlContent += `<li>🏢 ${emp}</li>`;
                });
                htmlContent += `</ul>`;
            }

            document.getElementById('panel-companies-container').innerHTML = htmlContent;
            document.getElementById('details-panel').classList.add('open');
        }

        function cerrarPanel() {
            document.getElementById('details-panel').classList.remove('open');
            subCardsMap.forEach(c => c.classList.remove('active'));
        }

        // Automatic initial network loading and dimension readjustment
        window.onload = function() {
            mostrarRedCompleta();
            setTimeout(function() {
                map.invalidateSize();
            }, 300);
        };
    </script>
</body>
</html>
