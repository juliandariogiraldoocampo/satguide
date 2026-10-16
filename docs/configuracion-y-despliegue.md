# Configuración y despliegue

## Variables de entorno

**NO DETECTADO** — El proyecto no usa variables de entorno. No existe `.env`, `.env.example`, ni ningún mecanismo de configuración por entorno.

Todos los parámetros configurables están hardcodeados en `index.html`:

| Parámetro | Valor actual | Ubicación en código |
|---|---|---|
| URL STAC CDSE | `https://stac.dataspace.copernicus.eu/v1/search` | `doSearch()` línea ~1700 |
| URL OData CDSE | `https://catalogue.dataspace.copernicus.eu/odata/v1/Products` | `openCDSEDownload()` |
| URL ASF API | `https://api.daac.asf.alaska.edu/services/search/param` | `doSearch()` línea ~1588 |
| URL CMR STAC | `https://cmr.earthdata.nasa.gov/stac/LPCLOUD/collections/HLSL30.v2.0/items` | `doSearch()` línea ~1627 |
| Concept ID Landsat | `C2021957657-LPCLOUD` | `doSearch()` |
| Límite de resultados | `20` | `doSearch()` (todas las ramas) |
| Rango de fechas por defecto | Últimos 90 días | `init()` |
| Nubosidad por defecto | `20%` | Atributo `value` del `<input type=range>` |

---

## Build system

**NO DETECTADO** — No hay proceso de build. No existen:

- `package.json`
- `webpack.config.js` / `vite.config.js` / `rollup.config.js`
- `Makefile`
- Scripts de compilación

El archivo `index.html` es el artefacto de producción directamente.

---

## Docker

**NO DETECTADO** — No existe `Dockerfile`, `docker-compose.yml` ni configuración de contenedores.

Si se quisiera contenerizar, bastaría con un servidor nginx/caddy que sirva el archivo estático:

```dockerfile
# Ejemplo NO incluido en el repositorio — solo referencial
FROM nginx:alpine
COPY index.html logo.svg logo.png /usr/share/nginx/html/
```

---

## CI/CD

**NO DETECTADO** — No existen pipelines de CI/CD. No hay:

- `.github/workflows/`
- `.gitlab-ci.yml`
- `Jenkinsfile`
- `netlify.toml` / `vercel.json`

---

## Opciones de despliegue

El proyecto es un sitio estático. Cualquier hosting de archivos estáticos es válido.

### GitHub Pages

```bash
# 1. Crear repositorio en GitHub
# 2. Subir los archivos
git init
git add index.html logo.svg logo.png
git commit -m "initial"
git remote add origin https://github.com/usuario/satguide.git
git push -u origin main

# 3. En Settings → Pages → Source: Deploy from branch → main → / (root)
```

### Netlify (drag & drop)

Arrastrar la carpeta del proyecto al panel de Netlify en app.netlify.com/drop.

### Servidor nginx

```nginx
server {
    listen 80;
    root /var/www/satguide;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Headers CORS no necesarios — todas las APIs son de terceros
    # Caché agresivo para los assets estáticos
    location ~* \.(svg|png)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

### Python (desarrollo local)

```bash
cd satguide
python -m http.server 8080
# → http://localhost:8080
```

---

## Consideraciones de red

- Las búsquedas hacen peticiones `fetch()` a APIs externas desde el navegador del usuario (no desde servidor).
- Todas las APIs consultadas tienen CORS abierto para búsqueda pública.
- No se requiere proxy inverso ni configuración especial de CORS en el servidor.
- La descarga real de los productos satelitales ocurre directamente desde los portales externos (CDSE, ASF, EarthExplorer), no a través de SatGuide.

---

## Rendimiento y caché

- No hay service worker ni caché offline.
- Las librerías Leaflet y Leaflet.draw se cargan desde cdnjs (CDN externo) en cada visita.
- Las fuentes de Google Fonts se cargan externamente.
- El mapa base es Esri World Imagery (satelital, sin API key) con una capa opcional de límites y lugares de Esri. Se debe conservar la atribución de Esri visible en el mapa.
