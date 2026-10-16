# SatGuide v2 - Registro de Cambios

## 2026-10-16 (v2.0.1)

### 16:00 - Mapa base satelital
- CARTO dejó de servir tiles sin API key: devolvía HTTP 200 con una imagen de marca de agua "API KEY REQUIRED"
- Reemplazado por Esri World Imagery (gratuito, sin API key) en `initMap()`
- Agregada capa "Límites y lugares" (Esri) activable desde un selector de capas
- Publicado el repositorio en GitHub

## 2026-04-05

### 22:30 - Restauración desde backup
- Restaurado archivo index.html desde "index - copia.html" debido a errores graves en el código
- La aplicación quedó funcionando correctamente

### 22:50 - Mejoras en página Acerca
- Agregada imagen del autor (img/jdgo.jpg)
- Reducido tamaño de badges de versiones (padding 2px 8px, font-size 13px)

### 23:30 - Historial dinámico desde JSON
- Agregado botón "Acerca de..." en navegación
- Creada página page-acerca con información del autor
- Historial de versiones ahora se genera dinámicamente desde versions.json
- Foto de perfil de 100px con borde de 2px

### 23:50 - Ajustes visuales
- Tamaño de nombres de colecciones en Búsqueda: 15px, font-weight 500

---

## Historial Anterior (por documentar)

### Antes de este registro
- Se creó la página "Acerca de..." con información del autor
- Se intentó implementar el historial de versiones desde JSON
- Múltples intentos de depuración causaron daños en el código