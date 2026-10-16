# Guía de contribución

---

## Estructura del código

Todo el código vive en `index.html` organizado en tres bloques claramente separados:

```
index.html
├── <head>
│   ├── Meta + dependencias CDN (Leaflet, Leaflet.draw, Google Fonts)
│   └── <style>  ← CSS completo, organizado por secciones con comentarios ═══
│
├── <body>
│   ├── <header>          ← Navegación entre páginas
│   ├── #page-orientacion ← Wizard + panel de recomendación (markup estático)
│   ├── #page-busqueda    ← Panel de controles + mapa + barra de resultados
│   └── #page-referencia  ← Tablas + grids de referencia (contenido renderizado por JS)
│
└── <script>              ← JS completo, organizado por secciones con comentarios ══
    ├── DATOS             ← Constantes: ANALYSIS_TYPES, COLLECTIONS, S2_BANDS, etc.
    ├── Navigation        ← showPage(), goSearch()
    ├── Orientación       ← renderAnalysisGrid(), selectAnalysis(), showRecommendation()
    ├── Búsqueda          ← renderCollections(), renderDemOptions(), initMap(), doSearch()
    ├── Results           ← renderResults(), activateResult(), buildDirectLink()
    ├── Download          ← openCDSEDownload(), showDownloadPanel(), showDEMInstructions()
    ├── Referencia        ← renderReference(), toggleAcc()
    ├── Helpers           ← setSpinner(), slog(), getBboxCenter(), fallbackCopy()
    └── INIT              ← init()
```

---

## Convenciones de código detectadas

### CSS
- Se usa un sistema de **design tokens** con CSS custom properties en `:root` bajo el bloque `/* TOKENS */`.
- Cada sección CSS está delimitada con comentarios `/* ═══════════════════ NOMBRE ═══════════════════ */`.
- Los nombres de clases siguen **BEM-like** con prefijos semánticos: `.sp-` (search panel), `.rc-` (result card), `.dl-` (download), `.bt-` (band table), `.pol-` (polarization), `.dem-` (DEM), `.acc-` (accordion).
- El diseño es **dark-first** — no hay modo claro.

### JavaScript
- Código vanilla ES2017+, sin transpilación.
- Las secciones JS están delimitadas con `// ══════════════════════════════════════════════════`.
- Las funciones de render usan **template literals** para generar HTML; no se usa innerHTML para datos de usuario sin sanear.
- Las funciones asíncronas siguen el patrón `setSpinner(true) → try/catch/finally → setSpinner(false)`.
- El log de búsqueda (`slog()`) usa tres niveles: `'ok'` (verde), `'err'` (rojo), `'inf'` (amarillo).
- Los datos de constantes se declaran con `const` en PascalCase para objetos/arrays de datos (`ANALYSIS_TYPES`, `COLLECTIONS`, `S2_BANDS`).
- Las variables de estado y variables de instancia del mapa son `let` en camelCase global.

### HTML
- Atributos `onclick` inline para event listeners (no `addEventListener`).
- IDs de página: `page-{nombre}` (ej: `page-orientacion`, `page-busqueda`).
- Secciones marcadas con comentarios `<!-- ═══════════════ NOMBRE ═══════════════ -->`.

---

## Cómo agregar un nuevo tipo de análisis

1. Agregar un objeto al array `ANALYSIS_TYPES` en el bloque `// DATOS` del `<script>`:

```javascript
{
  id: 'mi-analisis',           // ID único, snake_case
  icon: '🔬',                  // Emoji
  name: 'Nombre del análisis',
  desc: 'Descripción corta',
  color: '#hex',               // Color del acento (usar tokens existentes si es posible)
  sensors: [
    {
      id: 'sensor-id',
      name: 'Nombre del sensor',
      badge: 'BADGE',
      color: '#hex',
      portal: 'Portal',
      collection: 'id-en-COLLECTIONS',  // debe existir en COLLECTIONS
      why: 'Justificación técnica',
      key_bands: ['B01'],
      all_bands: ['B01', 'B02'],
    }
  ],
  warning: null   // o string con advertencia
}
```

2. Si la colección es nueva, agregarla también a `COLLECTIONS`.

---

## Cómo agregar una nueva colección buscable

1. Agregar a `COLLECTIONS`:

```javascript
'mi-coleccion': {
  name: 'Nombre',
  res: 'XX m',
  portal: 'Portal',
  color: '#hex',
  type: 'optical' | 'sar' | 'dem'
}
```

2. Agregar la lógica de búsqueda en `doSearch()` con el patrón:

```javascript
if (col === 'mi-coleccion') {
  setSpinner(true);
  slog('Consultando...', 'inf');
  clearResults();
  try {
    const resp = await fetch(url, options);
    if (!resp.ok) throw new Error(`HTTP ${resp.status}`);
    const data = await resp.json();
    const items = data.features || [];
    slog(`${items.length} escenas encontradas.`, items.length ? 'ok' : 'inf');
    renderResults(items, 'mi-coleccion');
  } catch(e) {
    slog(`Error: ${e.message}`, 'err');
  } finally {
    setSpinner(false);
  }
  return;
}
```

3. Agregar la lógica de `buildDirectLink()` para la nueva colección.
4. Si tiene instrucciones específicas, agregar a `BROWSER_INSTRUCTIONS`.

---

## Cómo agregar una nueva opción de DEM

1. Agregar a `DEM_OPTIONS`:

```javascript
{ id: 'mi-dem', name: 'Nombre', res: '30 m', note: 'Descripción corta' }
```

2. Agregar la configuración de descarga en `showDEMInstructions()` dentro del objeto `demConfig`.

---

## Cómo agregar preguntas al FAQ

Agregar un objeto `{ q: '...', a: '...' }` al array `FAQ_DATA`. El campo `a` acepta HTML.

---

## Checklist antes de integrar cambios

- [ ] La aplicación carga sin errores en consola con `file://` y con servidor HTTP.
- [ ] Funciona en Chrome, Firefox y Safari (mobile incluido).
- [ ] Los textos nuevos están en español (el proyecto es completamente en español).
- [ ] No se usan `alert()`, `prompt()` ni `console.log()` en producción.
- [ ] Los enlaces externos usan `target="_blank"`.
- [ ] El estado global (`state`) se actualiza de forma consistente.
- [ ] Las búsquedas tienen manejo de errores con fallback al portal externo.

---

## Control de versiones

**NO DETECTADO** — El directorio no es un repositorio git. No existen convenciones de commits documentadas.

Se recomienda usar el formato convencional:
```
feat: agrega soporte para colección MODIS
fix: corrige cálculo de bbox cuando el rectángulo cruza el antimeridiano
refactor: extrae buildDownloadUrl como función independiente
docs: actualiza referencia de bandas Landsat con banda pancromática
```
