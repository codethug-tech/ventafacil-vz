# Desplegar VentaFácil VZ

App estática: `index.html` + `manifest.json` + `sw.js` + íconos. **Sin build, sin
framework, sin backend.** Cualquier host estático sirve los archivos tal cual.

Requisitos para que la app se pueda **instalar** en el teléfono (PWA):

- El sitio debe abrirse por **HTTPS** (los tres hosts de abajo lo dan gratis).
- `index.html` debe estar en la **raíz** del sitio publicado.
- Las rutas son relativas (`./manifest.json`, `sw.js`), así que también funciona
  en un subdirectorio (por ejemplo `usuario.github.io/ventafacil-vz/`).

Archivos de configuración incluidos, uno por host (no estorban entre sí):

| Archivo | Para qué |
| --- | --- |
| `netlify.toml` | Netlify: publica la raíz y evita cachear `sw.js` e `index.html`. |
| `vercel.json` | Vercel: los mismos encabezados de cache. |
| `_headers` | Cloudflare Pages (y Netlify): los mismos encabezados. |
| `.nojekyll` | GitHub Pages: no procesar con Jekyll. |

> `sw.js` sin cache es importante: si el navegador guarda una versión vieja del
> service worker, el teléfono sigue mostrando la app anterior.

---

## Opción 1 · Netlify (la más rápida, 2 minutos)

1. Entra a <https://app.netlify.com> e inicia sesión con GitHub.
2. **Add new site → Import an existing project → Deploy with GitHub**.
3. Autoriza a Netlify si es la primera vez y elige el repositorio
   `codethug-tech/ventafacil-vz`.
   - Si el repo es privado, Netlify pedirá permiso para verlo; acéptalo.
4. En la configuración del sitio:
   - **Branch to deploy:** `main`
   - **Build command:** *(vacío)*
   - **Publish directory:** `.` *(un punto: la raíz del repo)*
5. **Deploy site**. Da la URL en ~30 segundos, algo como
   `https://ventafacil-vz.netlify.app`.
6. **Site configuration → Change site name** para elegir un nombre fijo:
   `https://ventafacil.netlify.app`.
7. Cada `git push` a `main` vuelve a publicar automáticamente. Los pull requests
   generan URLs de vista previa.

## Opción 2 · Vercel

1. Entra a <https://vercel.com/new> e inicia sesión con GitHub.
2. **Import Git Repository** → elige `codethug-tech/ventafacil-vz`
   (autoriza el acceso al repo si es privado).
3. En **Configure Project**:
   - **Framework Preset:** `Other`
   - **Build Command:** *(vacío / override off)*
   - **Output Directory:** *(vacío / override off)*
   - **Root Directory:** `./`
4. **Deploy**. URL del tipo `https://ventafacil-vz.vercel.app`.
5. **Settings → Domains** para renombrar el proyecto o conectar un dominio propio.
6. Cada push a `main` redespliega solo.

## Opción 3 · Cloudflare Pages

1. Entra a <https://dash.cloudflare.com/?to=/:account/workers-and-pages> →
   **Workers & Pages → Create → Pages → Connect to Git**.
2. Autoriza GitHub y elige `codethug-tech/ventafacil-vz`.
3. En **Set up builds and deployments**:
   - **Production branch:** `main`
   - **Framework preset:** `None`
   - **Build command:** *(vacío)*
   - **Build output directory:** `/` *(la raíz)*
4. **Save and Deploy**. URL del tipo
   `https://ventafacil-vz.pages.dev`.
5. **Custom domains** para usar un dominio propio.
6. Cada push a `main` publica de nuevo; las ramas generan previews.

## Opción 4 · GitHub Pages (sin cuenta nueva)

Pages en cuentas gratuitas exige un repositorio **público**.

1. **Settings → General → Danger Zone → Change visibility → Public**.
2. **Settings → Pages → Build and deployment**:
   - **Source:** `Deploy from a branch`
   - **Branch:** `main` · carpeta `/ (root)`
3. **Save**. En 1–2 minutos queda en
   `https://codethug-tech.github.io/ventafacil-vz/`
   (la primera publicación puede tardar unos minutos y el primer intento a veces
   da 404: recarga).

Con la CLI de GitHub ya autenticada en este equipo:

```sh
gh repo edit codethug-tech/ventafacil-vz --visibility public --accept-visibility-change-consequences
gh api -X POST repos/codethug-tech/ventafacil-vz/pages \
  -f 'source[branch]=main' -f 'source[path]=/'
```

---

## Instalar en el teléfono (cliente)

1. Abre la URL en **Chrome (Android)** o **Safari (iPhone)**.
2. **Android:** menú ⋮ → *Instalar app* (o el botón **Instalar VentaFácil** en
   Ajustes de la app).
   **iPhone:** Compartir → *Añadir a pantalla de inicio*.
3. La primera visita necesita conexión: es cuando el service worker guarda la
   interfaz. Después abre sin internet con la última tasa guardada.

## Verificar el despliegue

- `https://TU-URL/` carga la app y muestra la tasa BCV.
- `https://TU-URL/manifest.json` devuelve JSON (no 404 ni HTML).
- `https://TU-URL/sw.js` responde 200 con `Cache-Control: no-cache`.
- En Chrome de escritorio, **DevTools → Application → Manifest**: sin errores y
  con los íconos 192/512 y uno *maskable*.
- **Application → Service workers:** estado `activated` y sin errores.
- Genera un **Recibo PDF** (necesita conexión para jsPDF desde el CDN).

## Problemas comunes

| Síntoma | Causa / solución |
| --- | --- |
| No aparece “Instalar app” | El sitio no está en HTTPS, o `index.html` no está en la raíz publicada. |
| Publica 404 | Publish/output directory mal configurado: debe ser la raíz (`.` o `/`). |
| Sigue la versión vieja tras un cambio | El service worker quedó cacheado: `sw.js` debe servirse con `no-cache` (ya está en los tres archivos de config) o subir `CACHE` en `sw.js`. |
| La tasa no carga | Sin internet o DolarAPI caída: se usa la última tasa guardada; se puede cargar una tasa manual en Ajustes. |
| “jsPDF no disponible sin conexión” al generar el recibo | El CDN de jsPDF necesita red la primera vez. |
