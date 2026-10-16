# API Reference

SatGuide no expone una API propia. Actúa como cliente de cuatro APIs externas públicas. Esta sección documenta los contratos de cada integración tal como están implementados en el código.

---

## 1. STAC Copernicus Data Space (CDSE)

**Usado para:** Sentinel-2 L2A, Sentinel-1 GRD

### Endpoint
```
POST https://stac.dataspace.copernicus.eu/v1/search
Content-Type: application/json
```

### Request body
```json
{
  "collections": ["sentinel-2-l2a"],
  "bbox": [-74.5, 4.2, -73.8, 4.9],
  "datetime": "2024-01-01T00:00:00Z/2024-03-31T23:59:59Z",
  "limit": 20,
  "filter": {
    "op": "<",
    "args": [{ "property": "eo:cloud_cover" }, 20]
  },
  "filter-lang": "cql2-json"
}
```

**Nota:** El campo `filter` (nubosidad) solo se aplica cuando la colección es `sentinel-2-l2a`. Para `sentinel-1-grd` se omite (SAR no reporta nubosidad).

### Colecciones válidas
| ID | Sensor |
|---|---|
| `sentinel-2-l2a` | Sentinel-2 Level-2A (reflectancia de superficie) |
| `sentinel-1-grd` | Sentinel-1 GRD (amplitud SAR) |

### Response (GeoJSON FeatureCollection)
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "id": "S2A_MSIL2A_20240315T151701_N0510_R025_T18NXM_20240315T211958",
      "geometry": { "type": "Polygon", "coordinates": [[...]] },
      "properties": {
        "datetime": "2024-03-15T15:17:01Z",
        "eo:cloud_cover": 4.5,
        "file:size": 1234567890
      }
    }
  ]
}
```

### Campos utilizados por SatGuide
| Campo | Uso en UI |
|---|---|
| `feature.id` | Nombre de la escena (producto) |
| `feature.geometry` | Footprint dibujado en el mapa |
| `feature.properties.datetime` | Fecha mostrada en la tarjeta |
| `feature.properties.eo:cloud_cover` | Badge de nubosidad |
| `feature.properties.file:size` | Tamaño en GB |

---

## 2. OData CDSE Catalogue

**Usado para:** Resolución del nombre de escena Sentinel a su ID OData (para instrucciones de descarga)

### Endpoint
```
GET https://catalogue.dataspace.copernicus.eu/odata/v1/Products
    ?$filter=Name eq '{nombre}.SAFE'
    &$select=Id,Name,Online,ContentLength
    &$top=1
```

### Estrategia de búsqueda por nombre
SatGuide intenta dos variantes del nombre para manejar la discrepancia entre el ID STAC (sin extensión) y el nombre OData (con `.SAFE`):

1. `{id_stac}.SAFE`
2. Si el nombre termina en `_COG`: también prueba sin `_COG.SAFE`
3. Si no termina en `_COG`: también prueba con `_COG.SAFE`

### Response
```json
{
  "value": [
    {
      "Id": "uuid-producto",
      "Name": "S2A_MSIL2A_..._COG.SAFE",
      "Online": true,
      "ContentLength": 1234567890
    }
  ]
}
```

### Campos utilizados
| Campo | Uso en UI |
|---|---|
| `Online` | `true` → "Disponible inmediatamente" / `false` → aviso de producto offline |
| `ContentLength` | Tamaño en MB mostrado en el modal |
| `Name` | Nombre canónico para copiar y buscar en Copernicus Browser |

---

## 3. ASF Search API (Alaska Satellite Facility)

**Usado para:** ALOS PALSAR RTC

### Endpoint
```
GET https://api.daac.asf.alaska.edu/services/search/param
    ?platform=ALOS
    &processingLevel=RTC_HI_RES
    &intersectsWith={WKT_POLYGON}
    &start={fecha}T00:00:00UTC
    &end={fecha}T23:59:59UTC
    &maxResults=20
    &output=geojson
Accept: application/json
```

### Construcción del polígono WKT
A partir del bbox `[W, S, E, N]`:
```
POLYGON((W S, E S, E N, W N, W S))
```

### Response (GeoJSON FeatureCollection)
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "id": "ALPSRP...",
      "geometry": { "type": "Polygon", "coordinates": [[...]] },
      "properties": {
        "granuleName": "ALPSRP...",
        "fileName": "ALPSRP...zip",
        "startTime": "2009-03-15T10:22:15UTC",
        "sceneDateString": "2009-03-15",
        "downloadUrl": "https://datapool.asf.alaska.edu/...",
        "browse": ["https://datapool.asf.alaska.edu/.../browse.png"]
      }
    }
  ]
}
```

### Campos utilizados
| Campo | Uso en UI |
|---|---|
| `properties.granuleName` / `properties.fileName` | Nombre de la escena |
| `properties.startTime` / `properties.sceneDateString` | Fecha |
| `properties.downloadUrl` | **No utilizado directamente** — se redirige a ASF Vertex |
| `properties.browse[0]` | Link de vista previa |
| `feature.geometry` | Footprint en el mapa |

**Nota:** PALSAR no reporta nubosidad. El badge muestra "SAR" en lugar del porcentaje.

---

## 4. CMR STAC NASA Earthdata (LP DAAC)

**Usado para:** HLS L30 — Landsat 8/9 harmonizado

### Endpoint
```
GET https://cmr.earthdata.nasa.gov/stac/LPCLOUD/collections/HLSL30.v2.0/items
    ?bbox={W},{S},{E},{N}
    &datetime={fecha_inicio}T00:00:00Z/{fecha_fin}T23:59:59Z
    &limit=20
    &filter=eo:cloud_cover <= {porcentaje}
    &filter-lang=cql2-text
Accept: application/json
```

### Concept ID de la colección
`HLSL30.v2.0` = Landsat 8/9 OLI Surface Reflectance Daily Global 30m v2.0
ID externo LPCLOUD: `C2021957657-LPCLOUD`

### Response
```json
{
  "features": [
    {
      "id": "HLS.L30.T18NXM.2024075T152007.v2.0",
      "geometry": { "type": "Polygon", "coordinates": [[...]] },
      "properties": {
        "datetime": "2024-03-15T15:20:07Z",
        "eo:cloud_cover": 12.3
      },
      "links": [
        { "rel": "self", "href": "https://cmr.earthdata.nasa.gov/stac/.../items/..." }
      ]
    }
  ]
}
```

### Campos utilizados
| Campo | Uso en UI |
|---|---|
| `feature.id` | Nombre de la escena |
| `feature.geometry` | Footprint en el mapa |
| `feature.properties.datetime` | Fecha |
| `feature.properties.eo:cloud_cover` | Badge de nubosidad |
| `feature.links[rel=self].href` | Link a metadatos (botón secundario) |

---

## 5. Portales externos (redirects)

Estos portales no se consultan vía API; SatGuide construye URLs con parámetros y las abre en una nueva pestaña.

### Copernicus Browser
```
https://dataspace.copernicus.eu/browser/
  ?zoom=10
  &lat={lat_centro}
  &lng={lng_centro}
  [&fromTime={fecha}T00:00:00.000Z]
  [&toTime={fecha}T23:59:59.999Z]
  [&dateMode=SINGLE]
```

### ASF Vertex
```
https://search.asf.alaska.edu/#/
  ?zoom=7
  &center={lng},{lat}
  &polygon={WKT_URL_ENCODED}
  &dataset=ALOS
```

### NASA Earthdata Search
```
https://search.earthdata.nasa.gov/search/granules
  ?p=C2021957657-LPCLOUD
  &pg[0][v]=f
  &pg[0][gsk]=-start_date
  &sb[0]={W}%2C{S}%2C{E}%2C{N}
  [&qt={fecha_inicio}T00:00:00.000Z%2C{fecha_fin}T23:59:59.999Z]
```

### OpenTopography (SRTM / NASADEM / AW3D30)
```
https://portal.opentopography.org/raster
  ?opentopoID={DATASET_ID}
  &minX={W}&minY={S}&maxX={E}&maxY={N}
```

| DEM | opentopoID |
|---|---|
| SRTM GL1 | `OTSRTM.082015.4326.1` |
| NASADEM | `OTSRTM.042013.4326.1` |
| AW3D30 v3.2 | `OTALOS.112016.4326.2` |

---

## Funciones públicas JavaScript (API interna)

Estas funciones están en el scope global y son invocadas directamente desde atributos `onclick` en el HTML.

| Función | Parámetros | Descripción |
|---|---|---|
| `showPage(id, btn)` | `id`: string, `btn`: HTMLElement | Activa una página por su ID |
| `goSearch()` | — | Navega a Búsqueda con colecciones recomendadas pre-seleccionadas |
| `selectAnalysis(id)` | `id`: string | Selecciona un tipo de análisis y muestra la recomendación |
| `selectCollection(id)` | `id`: string | Selecciona una colección en el panel de búsqueda |
| `selectDem(id, url)` | `id`: string, `url`: string | Selecciona un modelo DEM |
| `doSearch()` | — | Lanza la búsqueda según la colección activa |
| `clearBbox()` | — | Elimina el área de interés del mapa |
| `openBrowser()` | — | Abre Copernicus Browser centrado en el AOI |
| `openASF()` | — | Abre ASF Vertex con el polígono AOI |
| `activateResult(i)` | `i`: number | Expande/colapsa los controles de descarga de una escena |
| `copyProductName(idx, btn)` | `idx`: number, `btn`: HTMLElement | Copia el nombre de escena al portapapeles por índice |
| `copyProductNameDirect(name, btn)` | `name`: string, `btn`: HTMLElement | Copia un nombre directamente al portapapeles |
| `openCDSEDownload(productName, btn)` | `productName`: string, `btn`: HTMLElement | Consulta OData CDSE y muestra el panel de descarga |
| `toggleAcc(i)` | `i`: number | Abre/cierra un item del accordion de FAQ |
| `clearCollectionFilter()` | — | Limpia el filtro de colecciones recomendadas |
