# SatGuide · Imágenes Satelitales para SIG

Herramienta web interactiva que guía a profesionales y estudiantes de SIG en la selección, búsqueda y descarga de imágenes satelitales gratuitas según el tipo de análisis que necesitan realizar.

---

## Descripción

SatGuide es una aplicación de una sola página (SPA) sin backend propio. Combina un asistente de análisis con un buscador espacial que consulta APIs públicas abiertas de tres proveedores satelitales gratuitos: Copernicus Data Space (ESA), ASF Vertex (NASA/Alaska) y NASA Earthdata (USGS/LP DAAC).

**Funcionalidades principales:**

- Wizard de orientación: el usuario elige el tipo de análisis y recibe recomendaciones de sensor y bandas con justificación técnica.
- Búsqueda espacial: dibuja un bounding box sobre un mapa Leaflet y lanza búsquedas contra APIs STAC/GeoJSON en tiempo real.
- Resultados interactivos: muestra escenas con footprints en el mapa, metadatos (fecha, cobertura de nubes, tamaño) y acciones de descarga por escena.
- Panel de instrucciones de descarga: guías paso a paso específicas por proveedor y colección.
- Referencia técnica: tablas de bandas espectrales (Sentinel-2, Landsat 8/9), polarizaciones SAR y comparativa de DEMs.

---

## Requisitos previos

- Navegador web moderno con soporte para ES2017+ (`async/await`, Fetch API, `navigator.clipboard`).
- Conexión a internet (las búsquedas consultan APIs externas).
- No requiere Node.js, Python, ni ningún runtime adicional.
- No requiere servidor web: puede abrirse directamente desde el sistema de archivos con `file://` o servirse desde cualquier servidor HTTP estático.

**Cuentas externas requeridas solo para descarga** (no para búsqueda):

- [CDSE (Copernicus Data Space)](https://dataspace.copernicus.eu) — gratuita. Requerida para descargar Sentinel-1, Sentinel-2 y COP-DEM.
- [NASA Earthdata](https://urs.earthdata.nasa.gov) — gratuita. Requerida para descargar desde ASF Vertex (ALOS PALSAR) y EarthExplorer (Landsat).

---

## Instalación

No hay proceso de instalación. El proyecto es un único archivo HTML autocontenido.

```bash
# Opción 1: abrir directamente en el navegador
# (doble clic en index.html o arrastrar al navegador)

# Opción 2: servir con cualquier servidor HTTP estático
python -m http.server 8080
# luego abrir http://localhost:8080

# Opción 3: con Node.js
npx serve .
```

---

## Configuración

**NO DETECTADO** — No existen variables de entorno, archivos `.env`, ni configuración externa. Todos los parámetros están embebidos en el código fuente dentro de `index.html`:

- URLs de APIs: hardcodeadas en la función `doSearch()`.
- Datos de referencia (bandas, polarizaciones, DEMs): constantes JavaScript en el bloque `<script>`.
- Valores por defecto del formulario: rango de fechas inicializado a los últimos 90 días, cobertura de nubes al 20%.

---

## Ejecución

```bash
# Desarrollo local
python -m http.server 8080

# Producción: copiar index.html, logo.svg y logo.png a cualquier hosting estático
# (GitHub Pages, Netlify, Vercel, S3, etc.)
```

---

## Estructura de archivos

```
satguide/
├── index.html      # Aplicación completa (HTML + CSS + JS en un solo archivo)
├── logo.svg        # Logotipo vectorial
├── logo.png        # Logotipo rasterizado (fallback)
```

---

## Dependencias externas (CDN)

| Librería | Versión | Uso |
|---|---|---|
| Leaflet | 1.9.4 | Mapa interactivo base |
| Leaflet.draw | 1.0.4 | Herramienta de dibujo de bounding box |
| Google Fonts | — | Syne, Outfit, JetBrains Mono, Instrument Sans |

---

## APIs externas consultadas

| API | Protocolo | Autenticación | Uso |
|---|---|---|---|
| STAC CDSE | STAC (POST) | Ninguna (búsqueda pública) | Sentinel-1 GRD, Sentinel-2 L2A |
| OData CDSE Catalogue | REST (GET) | Ninguna (búsqueda pública) | Resolución nombre → ID de producto |
| ASF Search API | REST (GET) | Ninguna | ALOS PALSAR RTC |
| CMR STAC NASA Earthdata | STAC (GET) | Ninguna | HLS L30 (Landsat 8/9) |
| OpenTopography | URL redirect | Ninguna | SRTM, NASADEM, AW3D30 |

---

## Pruebas automatizadas

El proyecto no incluye suite de pruebas.
