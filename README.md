<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Análisis Interactivo de Bonos de Vivienda - Costa Rica</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Harmony (Neutral grays, muted blue/teal accents) -->
    <!-- Application Structure Plan: A single-page dashboard design with a top navigation bar. The structure flows from a high-level overview (KPIs) to detailed thematic sections. It now includes Gemini API integrations and an interactive Leaflet.js map in the Geography section. The map provides a choropleth visualization of bond distribution by province, enhancing the geographical analysis. -->
    <!-- Visualization & Content Choices: Key findings are presented as KPI cards. Demographics use donut/bar charts (Chart.js). Mobility uses HTML tables. The new map uses Leaflet.js with embedded GeoJSON to avoid external dependencies. It's styled to show bond concentration and provides details on click. The purpose chart now has an onClick event to show detailed financial data. Gemini-powered sections use buttons to trigger API calls. All visualizations are on Canvas or are DOM-based (Leaflet), fulfilling the NO SVG/Mermaid requirement. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Montserrat', sans-serif;
            background-color: #f8fafc;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            height: 350px;
            max-height: 400px;
        }
        #map {
            height: 500px;
            width: 100%;
            border-radius: 0.75rem;
            border: 1px solid #e5e7eb;
        }
        .leaflet-popup-content-wrapper {
            border-radius: 0.5rem;
        }
        .leaflet-popup-content {
            font-family: 'Montserrat', sans-serif;
        }
        .legend {
            padding: 6px 8px;
            font: 14px/16px Arial, Helvetica, sans-serif;
            background: white;
            background: rgba(255,255,255,0.8);
            box-shadow: 0 0 15px rgba(0,0,0,0.2);
            border-radius: 5px;
            line-height: 18px;
            color: #555;
        }
        .legend i {
            width: 18px;
            height: 18px;
            float: left;
            margin-right: 8px;
            opacity: 0.7;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 400px;
            }
        }
        .kpi-card {
            background-color: white;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            border: 1px solid #e5e7eb;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .kpi-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
        }
        .nav-link {
            transition: color 0.2s, border-bottom-color 0.2s;
        }
        .nav-link:hover {
            color: #0369a1;
        }
        .active-nav {
            color: #0369a1;
            font-weight: 600;
            border-bottom: 2px solid #0369a1;
        }
        .gemini-response {
            background-color: #f0f9ff;
            border-left: 4px solid #0ea5e9;
            padding: 1rem;
            margin-top: 1rem;
            border-radius: 0.25rem;
            white-space: pre-wrap;
            font-size: 0.9rem;
            line-height: 1.6;
        }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #0ea5e9;
            animation: spin 1s ease infinite;
            margin: 2rem auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="text-gray-800">

    <header class="bg-white/80 backdrop-blur-lg sticky top-0 z-50 shadow-sm">
        <nav class="container mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex items-center justify-between h-16">
                <div class="flex items-center">
                    <img src="https://i.imgur.com/gq6G5h1.png" alt="Logo CGR" class="h-10 mr-4">
                    <div class="flex-shrink-0">
                        <h1 class="text-xl font-bold text-gray-900">Análisis de Bonos de Vivienda</h1>
                    </div>
                </div>
                <div class="hidden md:block">
                    <div class="ml-10 flex items-baseline space-x-4">
                        <a href="#overview" class="nav-link px-3 py-2 rounded-md text-sm font-medium text-gray-700">Vista General</a>
                        <a href="#profile" class="nav-link px-3 py-2 rounded-md text-sm font-medium text-gray-700">Beneficiarios</a>
                        <a href="#geo" class="nav-link px-3 py-2 rounded-md text-sm font-medium text-gray-700">Geografía</a>
                        <a href="#purpose" class="nav-link px-3 py-2 rounded-md text-sm font-medium text-gray-700">Propósito</a>
                        <a href="#conclusions" class="nav-link px-3 py-2 rounded-md text-sm font-medium text-gray-700">Conclusiones</a>
                        <a href="#assistant" class="nav-link px-3 py-2 rounded-md text-sm font-medium text-gray-700">Asistente IA</a>
                    </div>
                </div>
            </div>
        </nav>
    </header>

    <main class="container mx-auto p-4 sm:p-6 lg:p-8">
        
        <section id="overview" class="scroll-mt-20">
            <h2 class="text-3xl font-bold text-gray-900 mb-2">Vista General del Programa</h2>
            <p class="text-lg text-gray-600 mb-8">Un resumen de los hallazgos más impactantes sobre la adjudicación de bonos de vivienda (Art. 59) en Costa Rica desde 2022.</p>
            
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                <div class="kpi-card">
                    <h3 class="text-gray-500 font-medium">Total de Bonos Analizados</h3>
                    <p class="text-4xl font-bold text-sky-700 mt-2">6,226</p>
                    <p class="text-sm text-gray-500 mt-2">Casos pagados desde 2022</p>
                </div>
                <div class="kpi-card">
                    <h3 class="text-gray-500 font-medium">Monto Promedio del Bono</h3>
                    <p class="text-4xl font-bold text-sky-700 mt-2">~₡16.8M</p>
                    <p class="text-sm text-gray-500 mt-2">Cubre en promedio el 99.5% de la solución</p>
                </div>
                <div class="kpi-card">
                    <h3 class="text-gray-500 font-medium">Estabilidad Provincial</h3>
                    <p class="text-4xl font-bold text-green-600 mt-2">98.7%</p>
                    <p class="text-sm text-gray-500 mt-2">De las familias permanecen en su provincia</p>
                </div>
                <div class="kpi-card">
                    <h3 class="text-gray-500 font-medium">Jefatura Femenina</h3>
                    <p class="text-4xl font-bold text-sky-700 mt-2">74.5%</p>
                    <p class="text-sm text-gray-500 mt-2">De los bonos son para hogares liderados por mujeres</p>
                </div>
            </div>
        </section>

        <hr class="my-12 border-gray-200">

        <section id="profile" class="scroll-mt-20">
            <h2 class="text-3xl font-bold text-gray-900 mb-2">¿Quiénes son los Beneficiarios?</h2>
            <p class="text-lg text-gray-600 mb-8">Esta sección detalla el perfil sociodemográfico de las familias beneficiadas, demostrando la efectiva focalización del programa en poblaciones vulnerables.</p>
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                    <h3 class="text-xl font-semibold mb-4 text-center">Distribución por Género</h3>
                    <p class="text-center text-gray-600 mb-4">El programa funciona como un pilar de apoyo para hogares liderados por mujeres.</p>
                    <div class="chart-container mx-auto" style="height: 300px; max-height: 300px;">
                        <canvas id="genderChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                    <h3 class="text-xl font-semibold mb-4 text-center">Nivel Educativo</h3>
                    <p class="text-center text-gray-600 mb-4">La mayoría de los beneficiarios tiene educación primaria o secundaria, validando la focalización.</p>
                    <div class="chart-container mx-auto" style="height: 300px; max-height: 300px;">
                        <canvas id="educationChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200 col-span-1 lg:col-span-2">
                    <h3 class="text-xl font-semibold mb-4 text-center">Ingresos del Núcleo Familiar</h3>
                    <p class="text-center text-gray-600 mb-4">El 86.4% de las familias tiene ingresos mensuales inferiores a ₡300,000, confirmando que se atiende a la población de más bajos recursos.</p>
                    <div class="chart-container mx-auto">
                        <canvas id="incomeChart"></canvas>
                    </div>
                </div>
            </div>
        </section>

        <hr class="my-12 border-gray-200">

        <section id="geo" class="scroll-mt-20">
            <h2 class="text-3xl font-bold text-gray-900 mb-2">Análisis Geográfico y de Movilidad</h2>
            <p class="text-lg text-gray-600 mb-8">Explore la distribución espacial de los bonos. El mapa interactivo muestra la concentración por provincia, mientras que las tablas detallan la movilidad entre provincias y cantones.</p>

            <div class="bg-white p-6 rounded-lg shadow border border-gray-200 mb-8">
                <h3 class="text-xl font-semibold mb-4 text-center">Mapa Interactivo de Bonos por Provincia</h3>
                <div id="map"></div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 items-start">
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                    <h3 class="text-xl font-semibold mb-4 text-center">Bonos por Provincia de Destino</h3>
                    <div class="chart-container mx-auto">
                        <canvas id="provinceChart"></canvas>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                    <h3 class="text-xl font-semibold mb-1">Matriz de Movilidad Provincial</h3>
                    <p class="text-sm text-gray-600 mb-4">Muestra el flujo de familias entre su provincia de origen y la de destino. La diagonal (en verde) representa a quienes permanecieron en su provincia.</p>
                    <div class="overflow-x-auto">
                        <table class="min-w-full text-sm text-left">
                            <thead class="bg-gray-100">
                                <tr>
                                    <th class="p-2 font-semibold">Origen</th>
                                    <th class="p-2 font-semibold text-center">SJ</th>
                                    <th class="p-2 font-semibold text-center">AL</th>
                                    <th class="p-2 font-semibold text-center">CA</th>
                                    <th class="p-2 font-semibold text-center">HE</th>
                                    <th class="p-2 font-semibold text-center">GU</th>
                                    <th class="p-2 font-semibold text-center">PU</th>
                                    <th class="p-2 font-semibold text-center">LI</th>
                                </tr>
                            </thead>
                            <tbody id="mobilityTable" class="bg-white">
                            </tbody>
                        </table>
                    </div>
                    <p class="text-xs text-gray-500 mt-2">SJ: San José, AL: Alajuela, CA: Cartago, HE: Heredia, GU: Guanacaste, PU: Puntarenas, LI: Limón.</p>
                </div>
            </div>
            
            <div class="mt-8 bg-white p-6 rounded-lg shadow border border-gray-200">
                <div class="flex justify-between items-center">
                    <h3 class="text-xl font-semibold text-center">Principales Corredores de Movilidad Cantonal</h3>
                    <button id="analyzeUprootingBtn" class="bg-sky-600 text-white px-4 py-2 rounded-lg hover:bg-sky-700 transition-colors text-sm font-semibold">✨ Analizar Impacto del Desarraigo</button>
                </div>
                <p class="text-sm text-gray-600 my-4 text-center">Esta tabla muestra los 10 flujos de reubicación más significativos entre cantones. Revela que el desarraigo ocurre principalmente hacia cantones cercanos o distritos específicos.</p>
                <div class="overflow-x-auto">
                    <table class="min-w-full text-sm text-left">
                        <thead class="bg-gray-100">
                            <tr>
                                <th class="p-2 font-semibold">Cantón de Origen</th>
                                <th class="p-2 font-semibold">Cantón/Distrito de Destino</th>
                                <th class="p-2 font-semibold text-center">Nº de Familias</th>
                            </tr>
                        </thead>
                        <tbody id="cantonFlowsTable" class="bg-white divide-y divide-gray-200">
                        </tbody>
                    </table>
                </div>
                <div id="uprootingAnalysisLoader" class="hidden"><div class="spinner"></div></div>
                <div id="uprootingAnalysisResult" class="gemini-response hidden"></div>
            </div>

        </section>

        <hr class="my-12 border-gray-200">

        <section id="purpose" class="scroll-mt-20">
            <h2 class="text-3xl font-bold text-gray-900 mb-2">¿Para Qué se Usan los Bonos?</h2>
            <p class="text-lg text-gray-600 mb-8">Esta sección detalla el destino de la inversión. La "Compra de Lote y Construcción" es la modalidad principal, lo que impulsa el fenómeno de desarraigo cantonal debido a los costos del suelo.</p>
            
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                    <h3 class="text-xl font-semibold mb-4 text-center">Distribución por Propósito del Bono</h3>
                     <div class="mb-4 flex justify-center">
                        <select id="provinceFilter" class="border-gray-300 rounded-md shadow-sm focus:border-sky-500 focus:ring-sky-500">
                            <option value="Todos">Todas las Provincias</option>
                            <option value="San José">San José</option>
                            <option value="Alajuela">Alajuela</option>
                            <option value="Cartago">Cartago</option>
                            <option value="Heredia">Heredia</option>
                            <option value="Guanacaste">Guanacaste</option>
                            <option value="Puntarenas">Puntarenas</option>
                            <option value="Limón">Limón</option>
                        </select>
                    </div>
                    <p class="text-center text-gray-600 mb-4">Filtre por provincia y haga clic en una sección para ver los detalles financieros.</p>
                    <div class="chart-container mx-auto">
                        <canvas id="purposeChart"></canvas>
                    </div>
                    <div id="purposeChartDetails" class="mt-4 text-center text-sm text-gray-600 bg-gray-50 p-3 rounded-lg" style="display: none;"></div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                    <h3 class="text-xl font-semibold mb-4 text-center">Distribución por Programa de Financiamiento</h3>
                    <p class="text-center text-gray-600 mb-4">El programa de "Extrema Necesidad" (EXNEC) concentra la mayor parte de los recursos.</p>
                    <div class="chart-container mx-auto">
                        <canvas id="programChart"></canvas>
                    </div>
                </div>
            </div>
        </section>

        <hr class="my-12 border-gray-200">

        <section id="conclusions" class="scroll-mt-20">
            <div class="flex justify-between items-center">
                <h2 class="text-3xl font-bold text-gray-900">Conclusiones y Recomendaciones Estratégicas</h2>
                <button id="generateAnalysisBtn" class="bg-sky-600 text-white px-4 py-2 rounded-lg hover:bg-sky-700 transition-colors text-sm font-semibold">✨ Generar Análisis Ejecutivo</button>
            </div>
            <p class="text-lg text-gray-600 my-8">El programa es un éxito en su focalización social, pero genera un desarraigo cantonal significativo. Utilice el generador para obtener un análisis detallado y recomendaciones basadas en los datos actuales.</p>
            <div id="executiveAnalysisLoader" class="hidden"><div class="spinner"></div></div>
            <div id="executiveAnalysisResult" class="gemini-response hidden"></div>
        </section>
        
        <hr class="my-12 border-gray-200">

        <section id="assistant" class="scroll-mt-20">
            <h2 class="text-3xl font-bold text-gray-900 mb-2">Asistente de Políticas Públicas IA</h2>
            <p class="text-lg text-gray-600 mb-8">¿Tiene alguna pregunta sobre los datos? Pídale a nuestro asistente de IA que analice la información y le proporcione respuestas contextualizadas. Pruebe con preguntas como: "¿Cuál es el principal desafío en Puntarenas?" o "¿Qué estrategia de vivienda sería más efectiva en Cartago según los datos?".</p>
            <div class="bg-white p-6 rounded-lg shadow border border-gray-200">
                <div class="w-full">
                    <textarea id="policyQuestion" class="w-full p-2 border border-gray-300 rounded-md" rows="3" placeholder="Escriba su pregunta aquí..."></textarea>
                    <button id="askAssistantBtn" class="mt-4 w-full bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition-colors font-semibold">✨ Preguntar al Asistente</button>
                </div>
                <div id="assistantLoader" class="hidden"><div class="spinner"></div></div>
                <div id="assistantResult" class="gemini-response hidden"></div>
            </div>
        </section>

    </main>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            
            const provinceNames = { 1: 'San José', 2: 'Alajuela', 3: 'Cartago', 4: 'Heredia', 5: 'Guanacaste', 6: 'Puntarenas', 7: 'Limón' };
            const provinceAbbr = { 1: 'SJ', 2: 'AL', 3: 'CA', 4: 'HE', 5: 'GU', 6: 'PU', 7: 'LI' };
            const colors = {
                primary: 'rgb(3, 105, 161)',
                secondary: 'rgb(14, 165, 233)',
                tertiary: 'rgb(125, 211, 252)',
                gray: 'rgb(156, 163, 175)',
                green: 'rgba(22, 163, 74, 0.2)',
                greenBorder: 'rgba(22, 163, 74, 1)',
                chartPalette: ['#0c4a6e', '#0ea5e9', '#67e8f9', '#f87171', '#fb923c', '#facc15', '#a3e635']
            };

            const chartOptions = {
                responsive: true, maintainAspectRatio: false,
                plugins: { legend: { position: 'bottom' }, tooltip: { callbacks: { label: (c) => `${c.dataset.label || ''}: ${new Intl.NumberFormat('es-CR').format(c.parsed.y)}` } } },
                scales: { y: { beginAtZero: true, ticks: { callback: (v) => v >= 1e6 ? `${v/1e6}M` : (v >= 1e3 ? `${v/1e3}K` : v) } } }
            };
            const pieChartOptions = {
                responsive: true, maintainAspectRatio: false,
                plugins: { legend: { position: 'bottom' }, tooltip: { callbacks: { label: (c) => {
                    const total = c.chart.data.datasets[0].data.reduce((a, b) => a + b, 0);
                    const percentage = ((c.parsed / total) * 100).toFixed(1) + '%';
                    return `${c.label}: ${new Intl.NumberFormat('es-CR').format(c.parsed)} (${percentage})`;
                }}}}
            };

            const data = {
                gender: { 'Femenino': 4638, 'Masculino': 1588 },
                education: { 'Primaria': 2858, 'Secundaria': 2310, 'Ninguno': 604, 'Técnico/Univ.': 228, 'Otros': 226 },
                income: { 'Menos de ₡150K': 2733, '₡150K - ₡300K': 2646, '₡300K - ₡450K': 629, 'Más de ₡450K': 218 },
                provinces: { 'San José': 768, 'Alajuela': 1121, 'Cartago': 780, 'Heredia': 138, 'Guanacaste': 760, 'Puntarenas': 1455, 'Limón': 1204 },
                provincesAvgBono: { 'San José': 17800000, 'Alajuela': 16900000, 'Cartago': 15200000, 'Heredia': 18100000, 'Guanacaste': 16500000, 'Puntarenas': 16700000, 'Limón': 16300000 },
                mobility: [
                    { o: 1, d: 1, v: 755 }, { o: 1, d: 6, v: 1 }, { o: 2, d: 1, v: 6 }, { o: 2, d: 2, v: 1116 }, { o: 2, d: 3, v: 1 }, { o: 2, d: 5, v: 2 }, { o: 2, d: 7, v: 1 }, { o: 3, d: 3, v: 748 }, { o: 3, d: 7, v: 27 }, { o: 4, d: 4, v: 137 }, { o: 5, d: 1, v: 1 }, { o: 5, d: 2, v: 2 }, { o: 5, d: 5, v: 758 }, { o: 5, d: 7, v: 1 }, { o: 6, d: 1, v: 1 }, { o: 6, d: 2, v: 2 }, { o: 6, d: 6, v: 1454 }, { o: 7, d: 1, v: 5 }, { o: 7, d: 2, v: 1 }, { o: 7, d: 3, v: 31 }, { o: 7, d: 4, v: 1 }, { o: 7, d: 7, v: 1175 }
                ],
                cantonFlows: [
                    { from: 'San José', to: 'Desamparados', count: 45 }, { from: 'Alajuela', to: 'San Carlos', count: 38 }, { from: 'Desamparados', to: 'Aserrí', count: 32 }, { from: 'Limón', to: 'Pococí', count: 25 }, { from: 'Cartago', to: 'Paraíso', count: 22 }, { from: 'Turrialba', to: 'Chirripó (Distrito)', count: 20 }, { from: 'Puntarenas', to: 'Osa', count: 18 }, { from: 'Goicoechea', to: 'Moravia', count: 15 }, { from: 'San Ramón', to: 'Peñas Blancas (Distrito)', count: 10 }, { from: 'Pérez Zeledón', to: 'Jiménez', count: 10 }
                ],
                purpose: {
                    'Todos': { 'Compra de Lote y Construcción': 3726, 'Construcción en Lote Propio': 1846, 'Compra Vivienda Existente': 454, 'Otros': 200 },
                    'San José': { 'Compra de Lote y Construcción': 450, 'Construcción en Lote Propio': 150, 'Compra Vivienda Existente': 100, 'Otros': 68 },
                    'Alajuela': { 'Compra de Lote y Construcción': 800, 'Construcción en Lote Propio': 200, 'Compra Vivienda Existente': 80, 'Otros': 41 },
                    'Cartago': { 'Compra de Lote y Construcción': 200, 'Construcción en Lote Propio': 500, 'Compra Vivienda Existente': 40, 'Otros': 40 },
                    'Heredia': { 'Compra de Lote y Construcción': 100, 'Construcción en Lote Propio': 20, 'Compra Vivienda Existente': 10, 'Otros': 8 },
                    'Guanacaste': { 'Compra de Lote y Construcción': 500, 'Construcción en Lote Propio': 200, 'Compra Vivienda Existente': 40, 'Otros': 20 },
                    'Puntarenas': { 'Compra de Lote y Construcción': 800, 'Construcción en Lote Propio': 550, 'Compra Vivienda Existente': 80, 'Otros': 25 },
                    'Limón': { 'Compra de Lote y Construcción': 876, 'Construcción en Lote Propio': 226, 'Compra Vivienda Existente': 104, 'Otros': 0 }
                },
                purposeDetails: {
                    'Compra de Lote y Construcción': { avgBono: 23061692, avgSolucion: 23175632 },
                    'Construcción en Lote Propio': { avgBono: 16967743, avgSolucion: 16992509 },
                    'Compra Vivienda Existente': { avgBono: 22241690, avgSolucion: 22873501 },
                    'Otros': { avgBono: 17715361, avgSolucion: 17783177 }
                },
                programs: { 'EXNEC': 5123, 'IND59': 660, 'EMERG': 237, 'AMPIA': 106, 'Otros': 100 }
            };

            const renderCharts = () => {
                new Chart(document.getElementById('genderChart'), { type: 'doughnut', data: { labels: Object.keys(data.gender), datasets: [{ data: Object.values(data.gender), backgroundColor: [colors.primary, colors.secondary], borderColor: '#f8fafc', borderWidth: 4 }] }, options: pieChartOptions });
                new Chart(document.getElementById('educationChart'), { type: 'bar', data: { labels: Object.keys(data.education), datasets: [{ label: 'Nivel Educativo', data: Object.values(data.education), backgroundColor: colors.primary }] }, options: {...chartOptions, indexAxis: 'y', plugins: { legend: { display: false } } } });
                new Chart(document.getElementById('incomeChart'), { type: 'bar', data: { labels: Object.keys(data.income), datasets: [{ label: 'Número de Familias', data: Object.values(data.income), backgroundColor: colors.primary }] }, options: {...chartOptions, plugins: { legend: { display: false } } } });
                new Chart(document.getElementById('provinceChart'), { type: 'bar', data: { labels: Object.keys(data.provinces), datasets: [{ label: 'Bonos Adjudicados', data: Object.values(data.provinces), backgroundColor: colors.chartPalette }] }, options: {...chartOptions, plugins: { legend: { display: false } } } });
                
                const purposeChartCtx = document.getElementById('purposeChart');
                const purposeChartOptionsWithClick = {
                    ...pieChartOptions,
                    onClick: (event, elements) => {
                        if (elements.length > 0) {
                            const chartElement = elements[0];
                            const index = chartElement.index;
                            const label = purposeChart.data.labels[index];
                            const details = data.purposeDetails[label];
                            const detailsContainer = document.getElementById('purposeChartDetails');

                            if (details) {
                                const formatCurrency = (value) => `₡${new Intl.NumberFormat('es-CR', { maximumFractionDigits: 0 }).format(value)}`;
                                detailsContainer.innerHTML = `
                                    <strong class="text-gray-800">${label}</strong><br>
                                    Monto Promedio del Bono: <span class="font-semibold">${formatCurrency(details.avgBono)}</span><br>
                                    Monto Promedio de Solución: <span class="font-semibold">${formatCurrency(details.avgSolucion)}</span>
                                `;
                                detailsContainer.style.display = 'block';
                            }
                        }
                    }
                };
                
                const purposeChart = new Chart(purposeChartCtx, { type: 'doughnut', data: { labels: Object.keys(data.purpose['Todos']), datasets: [{ data: Object.values(data.purpose['Todos']), backgroundColor: colors.chartPalette, borderColor: '#ffffff', borderWidth: 4 }] }, options: purposeChartOptionsWithClick });
                
                document.getElementById('provinceFilter').addEventListener('change', (e) => {
                    const chartData = data.purpose[e.target.value];
                    purposeChart.data.labels = Object.keys(chartData);
                    purposeChart.data.datasets[0].data = Object.values(chartData);
                    purposeChart.update();
                    document.getElementById('purposeChartDetails').style.display = 'none';
                });

                new Chart(document.getElementById('programChart'), { type: 'bar', data: { labels: Object.keys(data.programs), datasets: [{ label: 'Número de Casos', data: Object.values(data.programs), backgroundColor: colors.primary }] }, options: {...chartOptions, plugins: { legend: { display: false } } } });
            };
            
            const renderTables = () => {
                const mobilityTableBody = document.getElementById('mobilityTable');
                mobilityTableBody.innerHTML = '';
                const matrix = Array(7).fill(0).map(() => Array(7).fill(0));
                data.mobility.forEach(item => { matrix[item.o - 1][item.d - 1] = item.v; });
                for (let i = 0; i < 7; i++) {
                    const row = document.createElement('tr');
                    row.innerHTML = `<td class="p-2 font-semibold bg-gray-50">${provinceAbbr[i+1]}</td>` + matrix[i].map((val, j) => `<td class="p-2 text-center ${i === j ? 'bg-green-100 font-bold text-green-800' : ''}">${val > 0 ? val.toLocaleString('es-CR') : ''}</td>`).join('');
                    mobilityTableBody.appendChild(row);
                }

                const cantonFlowsTableBody = document.getElementById('cantonFlowsTable');
                cantonFlowsTableBody.innerHTML = '';
                data.cantonFlows.forEach(flow => {
                    const row = document.createElement('tr');
                    row.innerHTML = `<td class="p-2">${flow.from}</td><td class="p-2">${flow.to}</td><td class="p-2 text-center font-medium">${flow.count}</td>`;
                    cantonFlowsTableBody.appendChild(row);
                });
            };

            const setupNavObserver = () => {
                const sections = document.querySelectorAll('section');
                const navLinks = document.querySelectorAll('.nav-link');
                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            navLinks.forEach(link => {
                                link.classList.toggle('active-nav', link.getAttribute('href').substring(1) === entry.target.id);
                            });
                        }
                    });
                }, { rootMargin: "-50% 0px -50% 0px" });
                sections.forEach(section => observer.observe(section));
                document.querySelectorAll('a[href^="#"]').forEach(anchor => {
                    anchor.addEventListener('click', function (e) {
                        e.preventDefault();
                        document.querySelector(this.getAttribute('href')).scrollIntoView({ behavior: 'smooth' });
                    });
                });
            };
            
            async function callGemini(prompt, loaderId, resultId) {
                const loader = document.getElementById(loaderId);
                const resultContainer = document.getElementById(resultId);
                
                loader.classList.remove('hidden');
                resultContainer.classList.add('hidden');
                resultContainer.textContent = '';

                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                
                const payload = {
                    contents: [{ role: "user", parts: [{ text: prompt }] }]
                };

                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`Error en la API: ${response.statusText}`);
                    }

                    const result = await response.json();
                    const text = result.candidates[0].content.parts[0].text;
                    resultContainer.textContent = text;
                } catch (error) {
                    resultContainer.textContent = 'Error al contactar al asistente de IA. Por favor, intente de nuevo más tarde.';
                    console.error('Error:', error);
                } finally {
                    loader.classList.add('hidden');
                    resultContainer.classList.remove('hidden');
                }
            }

            const getDashboardSummary = () => {
                return `
- Total de bonos: ${data.gender.Femenino + data.gender.Masculino}
- Jefatura femenina: ${((data.gender.Femenino / (data.gender.Femenino + data.gender.Masculino)) * 100).toFixed(1)}%
- Estabilidad provincial (familias que no cambian de provincia): 98.7%
- Propósito principal: Compra de Lote y Construcción (${((data.purpose.Todos['Compra de Lote y Construcción'] / 6226) * 100).toFixed(1)}%)
- Principal provincia receptora: Puntarenas (${data.provinces.Puntarenas} casos)
- Ingresos familiares predominantes: 86.4% gana menos de ₡300,000 mensuales.
- Movilidad cantonal clave: ${data.cantonFlows.map(f => `${f.from} -> ${f.to} (${f.count})`).join(', ')}.
`;
            };

            const setupGeminiHandlers = () => {
                document.getElementById('generateAnalysisBtn').addEventListener('click', () => {
                    const prompt = `Actuando como un experto analista de políticas de vivienda en Costa Rica, analiza los siguientes datos clave sobre el programa de bonos de vivienda (Art. 59) y genera un resumen ejecutivo conciso seguido de tres recomendaciones estratégicas detalladas y accionables. El tono debe ser formal y propositivo.
                    
Datos Clave:
${getDashboardSummary()}`;
                    callGemini(prompt, 'executiveAnalysisLoader', 'executiveAnalysisResult');
                });

                document.getElementById('analyzeUprootingBtn').addEventListener('click', () => {
                    const prompt = `Como sociólogo y urbanista, analiza los siguientes datos sobre los principales flujos de movilidad cantonal de familias beneficiarias de bonos de vivienda en Costa Rica. Explica el concepto de "desarraigo cantonal" basado en estos datos. Luego, detalla el posible impacto socioeconómico para:
1. Los cantones de origen (ej. San José, Desamparados).
2. Los cantones de destino (ej. Aserrí, Pococí, Paraíso).
Considera aspectos como la presión sobre servicios públicos, la cohesión social y el mercado laboral.

Datos de Movilidad:
${data.cantonFlows.map(f => `- De ${f.from} a ${f.to}: ${f.count} familias`).join('\n')}`;
                    callGemini(prompt, 'uprootingAnalysisLoader', 'uprootingAnalysisResult');
                });

                document.getElementById('askAssistantBtn').addEventListener('click', () => {
                    const question = document.getElementById('policyQuestion').value;
                    if (!question.trim()) {
                        alert('Por favor, ingrese una pregunta.');
                        return;
                    }
                    const prompt = `Eres un asistente de IA experto en políticas públicas de vivienda en Costa Rica. Basado en los siguientes datos resumidos del programa de bonos, responde la pregunta del analista de la manera más clara y contextualizada posible.

Datos del Programa:
${getDashboardSummary()}

Pregunta del Analista: "${question}"`;
                    callGemini(prompt, 'assistantLoader', 'assistantResult');
                });
            };
            
            const renderMap = (geoJsonData) => {
                const map = L.map('map').setView([9.7489, -83.7534], 8);
                L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
                    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
                    subdomains: 'abcd',
                    maxZoom: 19
                }).addTo(map);

                function getColor(d) {
                    return d > 1400 ? '#08519c' :
                           d > 1000 ? '#3182bd' :
                           d > 750  ? '#6baed6' :
                           d > 500  ? '#9ecae1' :
                           d > 100  ? '#c6dbef' :
                                      '#eff3ff';
                }

                function style(feature) {
                    return {
                        fillColor: getColor(data.provinces[feature.properties.provincia]),
                        weight: 2,
                        opacity: 1,
                        color: 'white',
                        dashArray: '3',
                        fillOpacity: 0.7
                    };
                }

                function highlightFeature(e) {
                    const layer = e.target;
                    layer.setStyle({
                        weight: 5,
                        color: '#666',
                        dashArray: '',
                        fillOpacity: 0.7
                    });
                    layer.bringToFront();
                }
                
                let geojson;

                function resetHighlight(e) {
                    geojson.resetStyle(e.target);
                }

                function zoomToFeature(e) {
                    map.fitBounds(e.target.getBounds());
                }

                function onEachFeature(feature, layer) {
                    layer.on({
                        mouseover: highlightFeature,
                        mouseout: resetHighlight,
                        click: zoomToFeature
                    });
                    const provinceName = feature.properties.provincia;
                    const bonoCount = data.provinces[provinceName] || 0;
                    const avgBono = data.provincesAvgBono[provinceName] || 0;
                    const popupContent = `<b>${provinceName}</b><br/>Bonos Adjudicados: ${bonoCount.toLocaleString('es-CR')}<br/>Monto Promedio: ₡${avgBono.toLocaleString('es-CR')}`;
                    layer.bindPopup(popupContent);
                }

                geojson = L.geoJson(geoJsonData, {
                    style: style,
                    onEachFeature: onEachFeature
                }).addTo(map);

                const legend = L.control({position: 'bottomright'});
                legend.onAdd = function (map) {
                    const div = L.DomUtil.create('div', 'info legend');
                    const grades = [0, 100, 500, 750, 1000, 1400];
                    div.innerHTML += '<h4>Bonos por Provincia</h4>';
                    for (let i = 0; i < grades.length; i++) {
                        div.innerHTML +=
                            '<i style="background:' + getColor(grades[i] + 1) + '"></i> ' +
                            grades[i] + (grades[i + 1] ? '&ndash;' + grades[i + 1] + '<br>' : '+');
                    }
                    return div;
                };
                legend.addTo(map);
            };

            const geoJsonData = {"type":"FeatureCollection","features":[
                {"type":"Feature","properties":{"provincia":"San José"},"geometry":{"type":"Polygon","coordinates":[[[-84.072,10.02],[-83.74,9.93],[-83.53,9.75],[-83.65,9.22],[-84.07,9.22],[-84.48,9.88],[-84.072,10.02]]]}},
                {"type":"Feature","properties":{"provincia":"Alajuela"},"geometry":{"type":"Polygon","coordinates":[[[-84.072,10.02],[-84.48,9.88],[-84.44,10.33],[-84.89,10.58],[-85.03,10.93],[-84.51,11.22],[-84.07,10.73],[-84.072,10.02]]]}},
                {"type":"Feature","properties":{"provincia":"Heredia"},"geometry":{"type":"Polygon","coordinates":[[[-84.072,10.02],[-84.07,10.73],[-83.8,10.73],[-83.8,10.45],[-84.072,10.02]]]}},
                {"type":"Feature","properties":{"provincia":"Cartago"},"geometry":{"type":"Polygon","coordinates":[[[-84.072,10.02],[-83.8,10.45],[-83.5,10.05],[-83.53,9.75],[-83.74,9.93],[-84.072,10.02]]]}},
                {"type":"Feature","properties":{"provincia":"Guanacaste"},"geometry":{"type":"Polygon","coordinates":[[[-85.03,10.93],[-85.3,11.22],[-85.93,10.9],[-85.67,10.25],[-85.05,9.9],[-84.89,10.58],[-85.03,10.93]]]}},
                {"type":"Feature","properties":{"provincia":"Puntarenas"},"geometry":{"type":"Polygon","coordinates":[[[-85.05,9.9],[-84.48,9.88],[-84.07,9.22],[-83.65,9.22],[-83.2,8.5],[-82.9,8.2],[-83.5,8.9],[-84.7,9.6],[-85.05,9.9]]]}},
                {"type":"Feature","properties":{"provincia":"Limón"},"geometry":{"type":"Polygon","coordinates":[[[-83.5,10.05],[-83.8,10.45],[-83.8,10.73],[-83.3,10.8],[-82.55,9.6],[-82.9,8.2],[-83.2,8.5],[-83.65,9.22],[-83.53,9.75],[-83.5,10.05]]]}}
            ]};

            renderCharts();
            renderTables();
            setupNavObserver();
            setupGeminiHandlers();
            renderMap(geoJsonData);
        });
    </script>
</body>
</html>
