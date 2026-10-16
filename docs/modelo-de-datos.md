# Modelo de datos

SatGuide no tiene base de datos propia. Los datos son de dos tipos: **constantes estáticas** embebidas en el código y **estado en memoria** de la sesión activa.

---

## Estado de la aplicación (`state`)

Objeto JavaScript global que actúa como store de sesión. Se resetea al recargar la página.

```javascript
const state = {
  selectedAnalysis:       null,    // string | null — ID del tipo de análisis seleccionado
  recommendedCollections: null,    // string[] | null — colecciones sugeridas por el wizard
  selectedCollection:     null,    // string | null — colección activa en búsqueda
  selectedDem:            null,    // string | null — ID del DEM seleccionado
  bbox:                   null,    // [W, S, E, N] (float[4]) | null — área de interés
  results:                [],      // GeoJSON Feature[] — escenas encontradas
  resultNames:            [],      // string[] — nombres de escenas (índice sincronizado con results)
  activeResult:           null,    // number | null — índice de escena expandida
}
```

---

## Constantes de datos de referencia

### `ANALYSIS_TYPES` — Tipos de análisis

```typescript
interface AnalysisType {
  id:       string;          // Identificador único ('clasificacion', 'vegetacion', etc.)
  icon:     string;          // Emoji representativo
  name:     string;          // Nombre legible
  desc:     string;          // Descripción corta
  color:    string;          // Color hex del acento
  sensors:  SensorRec[];     // Sensores recomendados para este análisis
  warning:  string | null;   // Advertencia técnica (ej: cobertura temporal limitada)
}

interface SensorRec {
  id:         string;        // ID interno del sensor ('s2l2a', 's1', 'ls', 'palsar', 'copdem', 'srtm')
  name:       string;        // Nombre completo
  badge:      string;        // Etiqueta corta ('S2-L2A', 'S1-GRD', etc.)
  color:      string;        // Color hex
  portal:     string;        // Portal de descarga ('CDSE', 'ASF Vertex (NASA)', etc.)
  collection: string;        // ID de colección para búsqueda (llave en COLLECTIONS)
  why:        string;        // Justificación técnica de la recomendación
  key_bands:  string[];      // Bandas principales para el análisis
  all_bands:  string[];      // Todas las bandas relevantes
}
```

**Valores del campo `id` en `ANALYSIS_TYPES`:**

| id | Nombre |
|---|---|
| `clasificacion` | Clasificación de coberturas |
| `vegetacion` | Estado de vegetación |
| `suelo` | Análisis de suelo / geología |
| `agua` | Cuerpos de agua / inundaciones |
| `urbano` | Análisis urbano |
| `radar` | Análisis SAR / bajo dosel |
| `dem` | Modelo de elevación |
| `temporal` | Serie temporal / cambio |

---

### `COLLECTIONS` — Colecciones buscables

```typescript
interface Collection {
  name:   string;   // Nombre legible
  res:    string;   // Resolución espacial
  portal: string;   // Portal de origen
  color:  string;   // Color hex
  type:   'optical' | 'sar' | 'dem';  // Tipo de dato
}
```

| ID | name | res | portal | type |
|---|---|---|---|---|
| `sentinel-2-l2a` | Sentinel-2 L2A | 10–60 m | CDSE | optical |
| `sentinel-1-grd` | Sentinel-1 GRD | 10–40 m | CDSE | sar |
| `landsat` | HLS L30 (Landsat 8/9) | 30 m | NASA Earthdata | optical |
| `dem` | COP-DEM GLO-30 | 30 m | CDSE | dem |
| `palsar` | ALOS PALSAR | 12.5 m | ASF Vertex | sar |

---

### `DEM_OPTIONS` — Opciones de DEM

```typescript
interface DemOption {
  id:   string;   // 'copdem30' | 'copdem90' | 'srtm' | 'nasadem' | 'aw3d30'
  name: string;   // Nombre del producto
  res:  string;   // Resolución ('30 m', '90 m', etc.)
  note: string;   // Nota descriptiva corta
}
```

| id | name | res | Portal descarga |
|---|---|---|---|
| `copdem30` | GLO-30 | 30 m | Copernicus Browser (CDSE) |
| `copdem90` | GLO-90 | 90 m | Copernicus Browser (CDSE) |
| `srtm` | SRTM GL1 | 30 m | OpenTopography |
| `nasadem` | NASADEM | 30 m | OpenTopography |
| `aw3d30` | AW3D30 v3.2 | 30 m | OpenTopography |

---

### `S2_BANDS` — Bandas Sentinel-2

```typescript
interface Band {
  band:  string;   // Identificador ('B01', 'B02', ..., 'B12', 'B8A')
  nm:    string;   // Longitud de onda central en nm ('443', '490', etc.)
  res:   string;   // Resolución ('10m', '20m', '60m')
  name:  string;   // Nombre del rango espectral ('Aerosol costero', 'Azul', etc.)
  use:   string;   // Aplicaciones principales
  color: string;   // Color hex para la muestra visual
}
```

13 bandas (B01–B12 + B8A), resoluciones 10m / 20m / 60m.

---

### `LS_BANDS` — Bandas Landsat 8/9

Misma interfaz que `S2_BANDS`. Cubre B1–B11 (OLI + TIRS), resoluciones 15m / 30m / 100m.

---

### `POL_DATA` — Polarizaciones SAR

```typescript
interface Polarization {
  code:  string;     // 'VV' | 'VH' | 'HH' | 'HV'
  desc:  string;     // Descripción del modo de transmisión/recepción
  uses:  string[];   // Aplicaciones típicas
  color: string;     // Color hex
}
```

---

### `DEM_COMP_DATA` — Comparativa de DEMs

```typescript
interface DEMComp {
  name:       string;
  res:        string;        // Resolución numérica
  unit:       string;        // Unidad ('m')
  note:       string;        // Descripción técnica
  warn:       boolean;       // true → "No usar como DEM" (ej: PALSAR RTC)
  restricted: boolean;       // true → "No libre" (ej: TanDEM-X 12m)
  best:       boolean;       // true → badge RECOMENDADO
  portal:     string;
}
```

---

### `FAQ_DATA` — Preguntas frecuentes

```typescript
interface FAQ {
  q: string;   // Pregunta
  a: string;   // Respuesta (puede contener HTML)
}
```

---

### `BROWSER_INSTRUCTIONS` — Instrucciones por colección

Mapa `{ [collectionId: string]: string }`. Define el texto de instrucción que aparece bajo cada tarjeta de resultado en la página de Búsqueda.

Colecciones con instrucción definida: `sentinel-2-l2a`, `sentinel-1-grd`.

---

## Migraciones de base de datos

**NO DETECTADO** — El proyecto no tiene base de datos ni migraciones.
