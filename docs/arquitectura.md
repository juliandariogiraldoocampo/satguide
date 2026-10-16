# Arquitectura del sistema

## Visión general

SatGuide es una SPA (Single-Page Application) sin backend. Toda la lógica reside en el cliente (navegador). No existe base de datos propia ni servidor de aplicaciones; el estado de la sesión vive en memoria JavaScript.

```
┌─────────────────────────────────────────────────────────────────┐
│                        NAVEGADOR (cliente)                       │
│                                                                  │
│   ┌──────────┐   ┌──────────────┐   ┌──────────────────────┐   │
│   │  Página  │   │   Página     │   │      Página          │   │
│   │Orientac. │   │  Búsqueda    │   │    Referencia        │   │
│   └────┬─────┘   └──────┬───────┘   └──────────────────────┘   │
│        │                │                                        │
│        │         ┌──────┴──────────────────────┐                │
│        │         │         Estado global        │                │
│        │         │  bbox, selectedCollection,   │                │
│        │         │  results, activeResult, ...  │                │
│        │         └──────┬──────────────────────-┘                │
│        │                │                                        │
│        │         ┌──────┴──────────────────────┐                │
│        │         │        doSearch()            │                │
│        │         │  Router de búsqueda por      │                │
│        │         │  colección seleccionada      │                │
│        │         └──┬───────┬──────────┬───────┘                │
│        │            │       │          │                         │
└────────┼────────────┼───────┼──────────┼─────────────────────── ┘
         │            │       │          │
         ▼            ▼       ▼          ▼
    [Leaflet +   [STAC CDSE] [ASF API] [CMR STAC]
     Draw]       Sentinel    PALSAR    Landsat
                 S1 / S2
```

---

## Diagrama de componentes (Mermaid)

```mermaid
graph TD
    subgraph Browser["Navegador (SPA)"]
        Header["Header / Navegación<br/>showPage()"]

        subgraph PageOrientacion["Página: Orientación"]
            AnalysisGrid["Grid de tipos de análisis<br/>renderAnalysisGrid()"]
            RecPanel["Panel de recomendación<br/>showRecommendation()"]
        end

        subgraph PageBusqueda["Página: Búsqueda"]
            SearchPanel["Panel de controles<br/>(colección, fechas, nubes, bbox)"]
            MapArea["Área del mapa<br/>Leaflet + Leaflet.draw"]
            ResultsBar["Barra de resultados<br/>renderResults()"]
        end

        subgraph PageReferencia["Página: Referencia"]
            BandTables["Tablas de bandas S2 / Landsat"]
            PolGrid["Grid de polarizaciones SAR"]
            DEMComp["Comparativa de DEMs"]
            FAQ["FAQ accordion"]
        end

        State["Estado global<br/>{ bbox, selectedCollection,<br/>selectedDem, results,<br/>resultNames, activeResult }"]

        DoSearch["doSearch()<br/>Router de búsqueda"]
        RenderResults["renderResults()"]
        BuildDirectLink["buildDirectLink()"]
        OpenCDSEDownload["openCDSEDownload()"]
        ShowDEMInstructions["showDEMInstructions()"]
        ShowDownloadPanel["showDownloadPanel()"]
    end

    subgraph ExternalAPIs["APIs externas"]
        STACCDSE["STAC CDSE<br/>stac.dataspace.copernicus.eu<br/>Sentinel-1 / Sentinel-2"]
        ODataCDSE["OData CDSE Catalogue<br/>catalogue.dataspace.copernicus.eu<br/>Resolución de nombre → ID"]
        ASFApi["ASF Search API<br/>api.daac.asf.alaska.edu<br/>ALOS PALSAR RTC"]
        CMRSTAC["CMR STAC NASA<br/>cmr.earthdata.nasa.gov<br/>HLS L30 Landsat 8/9"]
    end

    subgraph ExternalPortals["Portales externos (redirect)"]
        CopernicusBrowser["Copernicus Browser"]
        ASFVertex["ASF Vertex"]
        EarthdataSearch["NASA Earthdata Search"]
        EarthExplorer["USGS EarthExplorer"]
        OpenTopo["OpenTopography"]
    end

    Header -->|"showPage()"| PageOrientacion
    Header -->|"showPage()"| PageBusqueda
    Header -->|"showPage()"| PageReferencia

    AnalysisGrid -->|"selectAnalysis()"| RecPanel
    RecPanel -->|"goSearch()"| PageBusqueda

    SearchPanel --> DoSearch
    MapArea -->|"bbox dibujado"| State
    State --> DoSearch

    DoSearch -->|"col === 'sentinel-*'"| STACCDSE
    DoSearch -->|"col === 'palsar'"| ASFApi
    DoSearch -->|"col === 'landsat'"| CMRSTAC
    DoSearch -->|"col === 'dem'"| ShowDEMInstructions

    STACCDSE --> RenderResults
    ASFApi --> RenderResults
    CMRSTAC --> RenderResults

    RenderResults --> ResultsBar
    RenderResults -->|"footprints GeoJSON"| MapArea
    ResultsBar -->|"activateResult()"| BuildDirectLink

    BuildDirectLink -->|"Sentinel"| OpenCDSEDownload
    OpenCDSEDownload --> ODataCDSE
    OpenCDSEDownload --> ShowDownloadPanel

    ShowDownloadPanel -->|"window.open"| CopernicusBrowser
    ShowDEMInstructions -->|"window.open"| CopernicusBrowser
    ShowDEMInstructions -->|"window.open"| OpenTopo
    BuildDirectLink -->|"Landsat"| EarthExplorer
    BuildDirectLink -->|"PALSAR"| ASFVertex
```

---

## Diagrama de flujo de datos

```mermaid
sequenceDiagram
    actor Usuario
    participant UI as UI (HTML/CSS/JS)
    participant State as Estado global
    participant API as APIs externas

    Usuario->>UI: Selecciona tipo de análisis
    UI->>State: state.selectedAnalysis, state.recommendedCollections
    UI-->>Usuario: Muestra recomendación de sensores/bandas

    Usuario->>UI: Navega a Búsqueda / elige colección
    UI->>State: state.selectedCollection

    Usuario->>UI: Dibuja bounding box en el mapa
    UI->>State: state.bbox = [W, S, E, N]

    Usuario->>UI: Configura fechas y filtro de nubes
    Usuario->>UI: Clic "Buscar escenas"

    UI->>UI: doSearch() — enruta por colección

    alt Sentinel-1 / Sentinel-2
        UI->>API: POST https://stac.dataspace.copernicus.eu/v1/search<br/>{collections, bbox, datetime, filter}
        API-->>UI: GeoJSON FeatureCollection (≤20 items)
    else ALOS PALSAR RTC
        UI->>API: GET https://api.daac.asf.alaska.edu/services/search/param<br/>?platform=ALOS&intersectsWith=WKT_POLYGON&...
        API-->>UI: GeoJSON FeatureCollection (≤20 items)
    else Landsat HLS L30
        UI->>API: GET https://cmr.earthdata.nasa.gov/stac/LPCLOUD/<br/>collections/HLSL30.v2.0/items?bbox=...&filter=cloud_cover<=N
        API-->>UI: GeoJSON FeatureCollection (≤20 items)
    else DEM
        UI-->>Usuario: Muestra modal con instrucciones por portal
    end

    UI->>State: state.results = items
    UI->>UI: renderResults() → cards + footprints en mapa

    Usuario->>UI: Clic en escena
    UI-->>Usuario: Expande acciones de descarga

    opt Sentinel (CDSE)
        Usuario->>UI: Clic "Instrucciones de descarga"
        UI->>API: GET catalogue.dataspace.copernicus.eu/odata/v1/Products<br/>?$filter=Name eq 'nombre.SAFE'
        API-->>UI: {Id, Name, Online, ContentLength}
        UI-->>Usuario: Modal con pasos + tamaño + estado online/offline
    end
```

---

## Patrones de diseño detectados

- **SPA sin router**: navegación entre páginas mediante clases CSS (`.page.active`) y la función `showPage()`. No hay hash routing ni History API.
- **Estado centralizado en objeto plano**: el objeto global `state` actúa como store mínimo (sin reactividad, sin framework).
- **Render functions**: las funciones `renderAnalysisGrid()`, `renderCollections()`, `renderResults()`, `renderReference()` generan HTML mediante template literals y `innerHTML`. No se usa virtual DOM.
- **Data constants**: todos los datos de referencia (bandas, polarizaciones, DEMs, FAQ) son arrays/objetos de constantes declarados antes de las funciones.
- **Fallback degradado**: cada búsqueda tiene manejo de error con `try/catch` que abre el portal externo correspondiente como alternativa cuando la API falla.
- **Copy-to-clipboard con fallback**: `copyProductName()` intenta `navigator.clipboard.writeText()` y cae a `document.execCommand('copy')` para compatibilidad.

---

## Tecnologías y versiones

| Capa | Tecnología |
|---|---|
| Lenguaje | HTML5, CSS3, JavaScript ES2017+ (vanilla, sin transpilación) |
| Mapa | Leaflet 1.9.4 |
| Dibujo espacial | Leaflet.draw 1.0.4 |
| Tipografías | Google Fonts (Syne, Outfit, JetBrains Mono, Instrument Sans) |
| Build system | **NO DETECTADO** (no hay bundler, no hay compilación) |
| Framework JS | **NO DETECTADO** (vanilla JS) |
| Framework CSS | **NO DETECTADO** (CSS custom properties propio) |
