# Estructura del proyecto

## Árbol de archivos

```
satguide/
│
├── index.html          # Aplicación completa — único archivo de producción
│                         Contiene HTML + CSS (~830 líneas) + JS (~1400 líneas)
│                         Organizado internamente en secciones comentadas
│
├── logo.svg            # Logotipo vectorial del proyecto (cargado en el <header>)
├── logo.png            # Logotipo rasterizado (alternativa/fallback del SVG)
│
└── docs/               # Documentación técnica (generada)
    ├── README.md
    ├── arquitectura.md
    ├── estructura-del-proyecto.md  ← este archivo
    ├── api-reference.md
    ├── modelo-de-datos.md
    ├── configuracion-y-despliegue.md
    ├── historias-de-usuario.md
    ├── guia-de-contribucion.md
    └── glosario.md
```

---

## Estructura interna de `index.html`

El archivo está organizado con comentarios delimitadores. Se describe cada bloque:

### Bloque `<head>`

| Elemento | Responsabilidad |
|---|---|
| Meta tags | Charset UTF-8, viewport responsivo |
| Leaflet CSS/JS (CDN) | Librería de mapas interactivos |
| Leaflet.draw CSS/JS (CDN) | Plugin para dibujar formas en el mapa |
| Google Fonts | Carga de las 4 familias tipográficas |
| `<style>` | CSS completo de la aplicación (~830 líneas) |

### Bloque `<style>` (secciones CSS)

| Sección | Clases principales | Responsabilidad |
|---|---|---|
| TOKENS | `:root` | Variables globales de color, tipografía y espaciado |
| SCROLLBAR | `::-webkit-scrollbar` | Scrollbar personalizado |
| LAYOUT | `#app` | Grid raíz de la app (header + contenido) |
| HEADER | `header`, `.logo-mark`, `.nav-btn` | Barra superior con logo y navegación |
| PAGES | `.page`, `.page.active` | Sistema de visibilidad de páginas |
| PAGE: ORIENTACIÓN | `.wizard-step`, `.analysis-card`, `.rec-*`, `.sensor-rec`, `.cta-btn` | Wizard de orientación y panel de recomendación |
| PAGE: BÚSQUEDA | `#search-panel`, `.coll-card`, `.dem-btn`, `#map-area`, `.result-card`, `.dl-*` | Panel lateral + mapa + resultados |
| PAGE: REFERENCIA | `.band-table`, `.pol-card`, `.dem-comp`, `.acc-item` | Tablas de referencia y FAQ |
| SPINNER | `.spin-wrap`, `.spin` | Indicador de carga |
| TOOLTIP | `.tip-wrap`, `.tip-box` | Tooltips de ayuda contextual |
| MAP INSTRUCTION | `.map-instruction` | Etiqueta flotante sobre el mapa |

### Bloque `<body>` (estructura HTML)

| Elemento | ID / clase | Responsabilidad |
|---|---|---|
| Header | `<header>` | Logo + botones de navegación entre páginas |
| Página Orientación | `#page-orientacion` | Wizard paso 1 (grid análisis) + panel recomendación |
| Página Búsqueda | `#page-busqueda` | Panel controles izquierdo + área mapa derecha |
| Panel de controles | `#search-panel` | Colección, DEM, fechas, nubes, bbox, botones de búsqueda |
| Área del mapa | `#map-area` | Contenedor Leaflet + spinner + barra de resultados |
| Página Referencia | `#page-referencia` | Tablas S2/Landsat + polarizaciones SAR + DEMs + FAQ |

### Bloque `<script>` (secciones JS)

| Sección | Constantes / funciones | Responsabilidad |
|---|---|---|
| DATOS | `ANALYSIS_TYPES`, `COLLECTIONS`, `DEM_OPTIONS`, `S2_BANDS`, `LS_BANDS`, `POL_DATA`, `DEM_COMP_DATA`, `FAQ_DATA`, `BROWSER_INSTRUCTIONS` | Todos los datos de la aplicación como constantes estáticas |
| Navigation | `showPage()`, `goSearch()` | Sistema de páginas SPA |
| Orientación | `renderAnalysisGrid()`, `selectAnalysis()`, `showRecommendation()`, `renderCollections()`, `renderDemOptions()`, `clearCollectionFilter()`, `selectCollection()`, `selectDem()`, `updatePanelVisibility()` | Wizard + panel de recomendación + sincronización con búsqueda |
| Mapa | `initMap()`, `clearBbox()`, `updateBboxDisplay()` | Inicialización Leaflet y manejo del bounding box |
| Búsqueda | `doSearch()`, `buildASFUrl()` | Consultas a las 4 APIs externas con routing por colección |
| Resultados | `renderResults()`, `activateResult()`, `buildBrowserUrl()`, `buildDirectLink()`, `getBboxCenter()`, `openBrowser()`, `openASF()`, `showBrowserButton()`, `clearResults()` | Renderizado de tarjetas y footprints + lógica de acciones |
| Descarga | `openCDSEDownload()`, `showDownloadPanel()`, `showDEMInstructions()`, `copyProductName()`, `copyProductNameDirect()`, `fallbackCopy()` | Paneles modales de instrucciones + clipboard |
| Referencia | `renderReference()`, `toggleAcc()` | Renderizado dinámico de tablas y FAQ |
| Helpers | `setSpinner()`, `slog()` | Utilidades de UI transversales |
| INIT | `init()` | Inicialización al cargar la página |

---

## Variables globales del mapa

Estas variables se declaran en el scope global para ser compartidas entre funciones del mapa y de resultados:

| Variable | Tipo | Descripción |
|---|---|---|
| `map` | `L.Map` | Instancia del mapa Leaflet |
| `drawnItems` | `L.FeatureGroup` | Capa que contiene el rectángulo dibujado |
| `footprintLayers` | `L.GeoJSON[]` | Array de layers de footprints de escenas resultantes |

---

## Archivos NO presentes (ausencias notables)

| Archivo | Motivo de ausencia |
|---|---|
| `package.json` | No hay dependencias npm ni scripts |
| `*.test.js` / `*.spec.js` | No hay suite de pruebas |
| `.gitignore` | El directorio no es un repositorio git |
| `Dockerfile` | No hay contenerización |
| `.env` | No hay configuración por entorno |
| `CHANGELOG.md` | No hay historial de versiones documentado |
