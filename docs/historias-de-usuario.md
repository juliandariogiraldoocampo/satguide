# Historias de usuario inferidas

Inferidas a partir del código fuente, los datos de referencia y los flujos de la aplicación.

---

## HU-01 · Orientación por tipo de análisis

**Como** profesional o estudiante de SIG que no sabe qué imagen satelital necesita,
**quiero** seleccionar el tipo de análisis que voy a realizar (vegetación, suelo, agua, urbano, etc.),
**para** recibir una recomendación del sensor, las bandas específicas y una justificación técnica antes de buscar imágenes.

**Criterios de aceptación:**
- El wizard muestra 8 tipos de análisis con icono, nombre y descripción.
- Al seleccionar uno, aparece un panel con los sensores recomendados ordenados por idoneidad.
- Cada sensor muestra: nombre, badge, portal de origen, bandas clave resaltadas y razón técnica.
- Si el análisis tiene limitaciones (ej: PALSAR solo cubre 2006-2011), se muestra una advertencia visible.
- El botón "Buscar escenas disponibles →" pre-selecciona la colección recomendada en la página de búsqueda.

---

## HU-02 · Búsqueda espacial de escenas

**Como** usuario que ya sabe qué colección necesita,
**quiero** dibujar un área de interés en un mapa y buscar escenas disponibles,
**para** obtener la lista de imágenes que cubren mi zona de estudio con sus metadatos.

**Criterios de aceptación:**
- El mapa carga con una herramienta de dibujo de rectángulo.
- Al dibujar, las coordenadas del bbox se muestran en el panel de control.
- La búsqueda muestra un spinner de carga y mensajes de estado en el log.
- Los resultados se muestran como tarjetas con: nombre, fecha, nubosidad (badge de color) y tamaño.
- Los footprints de cada escena se dibujan sobre el mapa.
- Si no hay resultados, se muestra un mensaje sugiriendo ampliar los filtros.

---

## HU-03 · Filtrado por fecha y cobertura de nubes

**Como** analista que necesita imágenes en condiciones específicas,
**quiero** filtrar las escenas por rango de fechas y porcentaje máximo de nubosidad,
**para** obtener solo imágenes útiles para mi análisis óptico.

**Criterios de aceptación:**
- El rango de fechas se inicializa por defecto a los últimos 90 días.
- El filtro de nubosidad es un slider 0-100%, con valor por defecto 20%.
- Para colecciones SAR (Sentinel-1, PALSAR) y DEM, la nubosidad no aplica y no se envía como filtro.
- El badge de nubosidad en cada resultado es verde (<20%), amarillo (20-50%) o rojo (>50%).

---

## HU-04 · Descarga guiada de escenas Sentinel

**Como** usuario con cuenta CDSE que encontró la escena que necesita,
**quiero** recibir instrucciones paso a paso para descargar la escena desde Copernicus Browser,
**para** obtener el archivo GeoTIFF sin tener que navegar el portal por mi cuenta.

**Criterios de aceptación:**
- Al hacer clic en una tarjeta de resultado, aparecen los botones de acción.
- El botón "📋 Copiar nombre" copia el nombre exacto de la escena al portapapeles.
- El botón "📥 Instrucciones de descarga" consulta el catálogo CDSE y muestra un modal con: nombre, tamaño, estado online/offline y pasos numerados.
- Si el producto está offline, el modal advierte que debe solicitarse en el Data Workspace.
- El botón "↗ Copernicus Browser" abre el portal centrado en el área de la búsqueda.

---

## HU-05 · Descarga guiada de PALSAR (ASF Vertex)

**Como** analista de radar que necesita datos ALOS PALSAR RTC,
**quiero** buscar escenas y ser redirigido a ASF Vertex con mi área de interés pre-cargada,
**para** descargar datos SAR de banda L sin configurar la búsqueda manualmente.

**Criterios de aceptación:**
- Al seleccionar la colección PALSAR y hacer búsqueda, la app consulta la ASF API.
- Los resultados muestran las escenas con fecha (sin nubosidad — SAR).
- El botón "↗ Abrir en ASF Vertex (PALSAR)" incluye el polígono WKT del AOI en la URL.
- Si la API falla, se abre ASF Vertex automáticamente como fallback.
- Cada tarjeta ofrece link de preview de la escena cuando está disponible.

---

## HU-06 · Descarga de DEM con instrucciones por portal

**Como** usuario que necesita un modelo de elevación digital,
**quiero** seleccionar el DEM más adecuado y recibir instrucciones específicas para descargarlo,
**para** obtener el GeoTIFF de elevación sin confundirme entre los distintos portales y formatos.

**Criterios de aceptación:**
- Se puede elegir entre GLO-30, GLO-90 (CDSE), SRTM, NASADEM y AW3D30 (OpenTopography).
- Al buscar un DEM, aparece un modal con instrucciones paso a paso según el portal.
- Para CDSE: los pasos explican cómo navegar el Copernicus Browser hasta los tiles.
- Para OpenTopography: el link incluye las coordenadas del AOI para descarga directa sin cuenta.
- El modal advierte que los DEMs se distribuyen por tiles geográficos, no por escena con fecha.
- Se distingue claramente AW3D30 (DEM óptico) de ALOS PALSAR RTC (imagen SAR).

---

## HU-07 · Referencia técnica de bandas y polarizaciones

**Como** analista que necesita recordar para qué sirve cada banda o polarización,
**quiero** consultar una tabla de referencia con todas las bandas de Sentinel-2 y Landsat 8/9,
**para** identificar rápidamente qué bandas usar en mi índice o clasificación.

**Criterios de aceptación:**
- La tabla Sentinel-2 muestra las 13 bandas con: nombre, longitud de onda, resolución, nombre del rango y aplicaciones.
- La resolución se resalta visualmente (verde=10m, amarillo=20m, gris=60m).
- La tabla Landsat 8/9 sigue el mismo formato con sus 11 bandas OLI/TIRS.
- Las polarizaciones SAR (VV, VH, HH, HV) se muestran en tarjetas con descripción y aplicaciones.
- La comparativa de DEMs muestra resolución, nota técnica, portal y badge RECOMENDADO para GLO-30.

---

## HU-08 · Aclaración de dudas frecuentes (FAQ)

**Como** usuario con dudas técnicas sobre los productos satelitales,
**quiero** consultar una sección de preguntas frecuentes expandibles,
**para** resolver dudas comunes sin necesidad de salir de la herramienta.

**Criterios de aceptación:**
- El FAQ usa un accordion: cada pregunta se expande/colapsa de forma independiente.
- Las respuestas pueden incluir HTML enriquecido (negritas, código).

---

## HU-09 · Búsqueda de Landsat con fallback a Earthdata Search

**Como** analista que necesita series temporales largas con Landsat,
**quiero** buscar escenas Landsat HLS L30 desde SatGuide,
**para** identificar imágenes disponibles y acceder directamente al portal de descarga con mi AOI pre-cargado.

**Criterios de aceptación:**
- La búsqueda consulta CMR STAC con filtro de bbox, fechas y nubosidad CQL2.
- Los resultados se muestran como tarjetas con enlace a EarthExplorer para descarga.
- El botón "↗ Buscar en NASA Earthdata Search" lleva al portal con AOI, fechas y colección correcta pre-seleccionados.
- Si la API CMR falla, se abre automáticamente NASA Earthdata Search como fallback.
