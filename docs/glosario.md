# Glosario técnico

Términos, abreviaturas y convenciones utilizados en SatGuide.

---

## Sensores y plataformas

| Término | Definición |
|---|---|
| **Sentinel-2** | Constelación de satélites de observación óptica de ESA/Copernicus. Dos satélites (2A y 2B) con revisita de 5 días. |
| **S2-L2A** | Sentinel-2 Level-2A: producto con corrección atmosférica aplicada, expresado en reflectancia de superficie. Listo para cálculo de índices. |
| **Sentinel-1** | Satélite SAR de ESA/Copernicus con radar de apertura sintética en banda C (5.6 cm). |
| **S1-GRD** | Sentinel-1 Ground Range Detected: producto de amplitud SAR. Modo IW (Interferometric Wide) estándar sobre tierra. |
| **Landsat 8/9** | Satélites del programa Landsat (USGS/NASA). OLI (óptico) + TIRS (térmico). Archivo desde 1972. |
| **HLS L30** | Harmonized Landsat Sentinel-2 Level-30m: producto armonizado espectralmente entre Landsat y Sentinel-2, a 30 m de resolución, distribuido por NASA LP DAAC. |
| **ALOS** | Advanced Land Observing Satellite — satélite japonés (JAXA), activo 2006-2011. |
| **PALSAR** | Phased Array type L-band SAR — sensor radar de banda L (23 cm) a bordo de ALOS. Penetra el dosel vegetal. |
| **PALSAR RTC** | ALOS PALSAR Radiometrically Terrain Corrected: producto con corrección topográfica aplicada. Resolución 12.5 m. Distribuido por ASF Vertex. |

---

## Productos de elevación (DEM)

| Término | Definición |
|---|---|
| **DEM** | Digital Elevation Model — modelo de elevación digital. Representa la altura del terreno. |
| **DSM** | Digital Surface Model — modelo de superficie digital. Incluye edificios y vegetación. |
| **COP-DEM GLO-30** | Copernicus DEM Global 30m — basado en datos TanDEM-X de la misión interferométrica DLR/ESA. Resolución 30 m (~1 arc-second). El DEM libre de mayor calidad disponible globalmente. |
| **COP-DEM GLO-90** | Variante de 90 m (~3 arc-seconds) del Copernicus DEM. Cobertura global más completa. |
| **SRTM** | Shuttle Radar Topography Mission — DEM generado por la misión de lanzadera de la NASA en el año 2000. Resolución 30 m. |
| **NASADEM** | Reprocesamiento del SRTM (2020) con mejor calidad vertical. Distribuido por NASA/USGS. |
| **AW3D30** | ALOS World 3D 30m — DEM generado a partir del sensor óptico PRISM (ALOS) por estereoscopía. Versión v3.2 distribuida por OpenTopography. **Distinto de ALOS PALSAR RTC.** |
| **TanDEM-X** | Misión interferométrica DLR con dos satélites SAR. Base del COP-DEM. |
| **Tile** | Tesela geográfica. Los DEMs se distribuyen por cuadrículas regulares (tiles), no por escenas con fecha. |

---

## Teledetección y SIG

| Término | Definición |
|---|---|
| **AOI** | Area of Interest — área de interés. En SatGuide, definida por el bounding box dibujado en el mapa. |
| **Bbox / Bounding box** | Rectángulo delimitador definido por cuatro coordenadas: [West, South, East, North] en grados decimales WGS84. |
| **Banda** | Canal espectral de un sensor. Cada banda captura radiación en un rango de longitudes de onda específico. |
| **Banda clave** | Banda más relevante para un tipo de análisis específico. En SatGuide se resaltan con un color diferente. |
| **Polarización SAR** | Orientación de la onda electromagnética transmitida (T) y recibida (R). Notación: VV, VH, HH, HV. |
| **VV / VH** | Polarizaciones verticales usadas por Sentinel-1. VV sensible a superficie rugosa/humedad; VH sensible a volumen de vegetación. |
| **HH / HV** | Polarizaciones horizontales usadas por ALOS PALSAR. |
| **Banda C** | Longitud de onda ~5.6 cm (frecuencia ~5.4 GHz). Usada por Sentinel-1. Interacción con copa del dosel. |
| **Banda L** | Longitud de onda ~23 cm (frecuencia ~1.2 GHz). Usada por ALOS PALSAR. Penetra el dosel vegetal denso. |
| **Red Edge** | Bandas de Sentinel-2 (B05, B06, B07, B8A) en la región 705-865 nm, sensibles a la clorofila. |
| **SWIR** | Short-Wave Infrared — infrarrojo de onda corta (~1400-2500 nm). Sensible a humedad, arcillas y óxidos. |
| **NDVI** | Normalized Difference Vegetation Index. Índice de vegetación (B08-B04)/(B08+B04). |
| **NDBI** | Normalized Difference Built-up Index. Índice de zonas construidas (B11-B08)/(B11+B08). |
| **MNDWI** | Modified Normalized Difference Water Index. Índice de agua (B03-B11)/(B03+B11). |
| **L2A** | Level-2A: nivel de procesamiento con corrección atmosférica. Reflectancia de superficie. |
| **GRD** | Ground Range Detected: nivel de procesamiento SAR. Amplitud en geometría de proyección terrestre. |
| **RTC** | Radiometric Terrain Correction: corrección que elimina distorsiones topográficas en imágenes SAR. |
| **IW** | Interferometric Wide swath — modo de adquisición estándar de Sentinel-1 sobre tierra. |
| **Footprint** | Huella o contorno geográfico de una escena satelital. En SatGuide se dibuja como polígono en el mapa. |

---

## APIs y protocolos

| Término | Definición |
|---|---|
| **STAC** | SpatioTemporal Asset Catalog — estándar abierto para catalogar datos geoespaciales. Define endpoints `/search` con filtros por bbox, datetime, colección y propiedades. |
| **OData** | Open Data Protocol — protocolo REST para consultar catálogos con filtros tipo SQL (`$filter`, `$select`, `$top`). Usado por el catálogo CDSE. |
| **CQL2** | Common Query Language 2 — lenguaje de filtros estándar OGC para STAC. Existe en variante texto (`cql2-text`) y JSON (`cql2-json`). |
| **WKT** | Well-Known Text — formato de texto para geometrías geoespaciales. SatGuide lo usa para construir el polígono de búsqueda en ASF. |
| **GeoJSON** | Formato JSON estándar para datos geoespaciales. Las APIs devuelven `FeatureCollection` con los metadatos de cada escena. |
| **CORS** | Cross-Origin Resource Sharing — mecanismo de seguridad del navegador. Las APIs consultadas por SatGuide tienen CORS abierto para búsqueda pública. |
| **CMR** | Common Metadata Repository — sistema de catálogo de NASA para sus colecciones de datos. |
| **LP DAAC** | Land Processes Distributed Active Archive Center — centro de datos NASA/USGS que distribuye HLS Landsat y otros productos de superficie terrestre. |

---

## Portales externos

| Abreviatura | Nombre completo | URL |
|---|---|---|
| **CDSE** | Copernicus Data Space Ecosystem | dataspace.copernicus.eu |
| **ASF** | Alaska Satellite Facility | search.asf.alaska.edu |
| **ASF Vertex** | Interfaz de búsqueda y descarga de ASF | search.asf.alaska.edu |
| **USGS EarthExplorer** | Portal de búsqueda/descarga del USGS | earthexplorer.usgs.gov |
| **NASA Earthdata Search** | Portal de búsqueda de colecciones NASA | search.earthdata.nasa.gov |
| **OpenTopography** | Portal de distribución de DEMs de acceso libre | portal.opentopography.org |
| **JAXA** | Japan Aerospace Exploration Agency — agencia japonesa que produce AW3D30 | — |

---

## Convenciones del código fuente

| Convención | Descripción |
|---|---|
| `state.*` | Propiedades del objeto global de estado de la sesión |
| `UPPER_SNAKE_CASE` | Constantes de datos de referencia (`ANALYSIS_TYPES`, `COLLECTIONS`, `S2_BANDS`, etc.) |
| `camelCase` | Funciones y variables (`doSearch`, `renderResults`, `state.bbox`) |
| `kebab-case` | IDs de elementos HTML y clases CSS (`page-busqueda`, `results-bar`) |
| `sp-` | Prefijo de clases CSS del Search Panel |
| `rc-` | Prefijo de clases CSS de Result Card |
| `dl-` | Prefijo de clases CSS de Download buttons |
| `bt-` | Prefijo de clases CSS de Band Table |
| `acc-` | Prefijo de clases CSS de Accordion |
| `pol-` | Prefijo de clases CSS de Polarization |
| `slog(msg, type)` | Función de logging en UI. `type`: `'ok'` \| `'err'` \| `'inf'` |
