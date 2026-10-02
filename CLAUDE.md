# EnerGym Suplementos — sitio web

Sitio estático (sin build step) para EnerGym Suplementos, tienda de suplementos deportivos en Formosa, Argentina. Todo vive en un único `index.html` con datos de productos/combos embebidos en arrays JS, más la carpeta `images/`.

## Stack y despliegue

- Un solo archivo `index.html` (HTML + CSS + JS inline, sin frameworks ni bundler).
- `images/` tiene cada foto en `.jpg` (fallback) y `.webp` (preferido), vía `<picture>`.
- `_headers` configura cache y headers de seguridad para Cloudflare (reemplazó a un viejo `netlify.toml`, ya no se usa Netlify).
- Repo en GitHub: `energymsuplementosfsa-art/energym-web` (cuenta de GitHub propia del negocio, **no** la cuenta personal/empresarial del dueño).
- Deploy automático: Cloudflare Pages/Workers está conectado a este repo vía "Connect to Git". Cada push a `main` dispara un redeploy solo. URL actual: `https://energym-web.energym-suplementos-fsa.workers.dev` (sin dominio propio todavía).
- No hay backend. El carrito y el formulario de contacto arman un link de `https://wa.me/5493704348191?text=...` y abren WhatsApp — ahí se cierran las ventas, no hay pasarela de pago ni base de datos.

## Estructura de datos (dentro de index.html)

- `PRODUCTS`: array de productos individuales (`id`, `cat`, `name`, `qty`, `price`, `img`). `img` es el nombre base del archivo en `images/` (sin extensión).
- `COMBOS`: array de combos (`id`, `badge`, `name`, `price`, `save`, `desc`, `imgs: [...]`). Los combos reutilizan las mismas imágenes de `PRODUCTS`, no tienen fotos propias — si mejorás una foto de producto, el combo se actualiza solo.
- Categorías: `muscular` (Ganancia Muscular), `energia` (Energía), `recuperacion` (Recuperación).

## Convención de fotos de catálogo

La mayoría de las fotos de producto se generaron recortando las plantillas de Instagram Story (formato 1080x1920) que están en `STOCK/Nueva carpeta/` (carpeta del usuario, fuera de este repo). Esas plantillas tienen el producto siempre en la misma posición relativa. El recorte que da buen resultado:

- Box de crop: `(160, 485, 950, 1205)` sobre la imagen original 1080x1920.
- Resize final a 700px de ancho, manteniendo aspect ratio.
- Guardar `.jpg` calidad 85 y `.webp` calidad 82 (método 6) con el mismo nombre base que usa `img:` en `PRODUCTS`.
- Fondo: las plantillas ya traen un degradé azul marino que combina con el tema oscuro del sitio — no hace falta quitarlo ni reemplazarlo.

Pendiente conocido: `creatine-frutos-rojos.jpg/webp` sigue con una foto de menor calidad porque no existe una plantilla "story" de esa variedad en STOCK. Si aparece una fuente mejor, conviene re-procesarla con el mismo pipeline para que quede consistente con el resto.

`hydro-max-660g` muestra "SPORT DRIN" cortado y `hydro-max-660g`/`hydro-max-1320g` tienen fondo ligeramente distinto a propósito — es una limitación de la foto original del proveedor, no un bug de recorte.

## Pendientes / backlog de UI (pedido por el dueño, Valentin, como "PO")

- Sumar prueba social (testimonios de clientes reales). La sección `#testimonios` ya existe y se muestra sola cuando el array `TESTIMONIALS` (en `index.html`) tiene elementos `{ name, detail, text }`. Falta que el dueño pase reseñas reales — **nunca inventar testimonios**.
- Evaluar dominio propio en vez de `workers.dev` (mejora SEO y preview de links compartidos).
- ~~Agregar `sitemap.xml`/`robots.txt`~~ (hecho). Si se cambia de dominio, actualizar la URL en: `robots.txt`, `sitemap.xml`, `<link rel="canonical">`, `og:url`/`og:image`/`twitter:image` y `SITE_URL` en `index.html`.
- Aclarar si hay diferencia de precio entre transferencia y efectivo en la sección de medios de pago.
- ~~Rediseño visual~~ (hecho y aprobado por el dueño, ya en `main`).

## Sistema visual (rediseño)

- Header navy con el logo real (`images/energym-logo-header.png`, recortado del banner con fondo transparente — es blanco, solo sirve sobre fondos oscuros).
- Color de marca = el azul marino del logo y de los fondos de las fotos (`--navy`, `--photo-bg`). Fondo general claro tipo papel (`--paper`), texto `--ink`, un único acento `--blue`. Tema fijo, no cambia con modo oscuro.
- Tipografía: una sola familia, `Archivo` variable (ejes `wdth` 62–125 e itálica). Títulos `font-stretch:62%` itálica 850–900 en mayúsculas; precios y números `font-stretch:125%` (contraste condensada/expandida); texto en ancho normal. Acento extra `--volt` (lima de los envases) solo en detalles: selector del hero, sticker de ahorro en combos, rombos del ticker.
- Interacción: el hero tiene selector de objetivo (`HERO_GOALS` en el JS, rota solo cada 4,5s hasta que la persona elige), ticker de datos/marcas, aparición al scrollear (`.reveal` + IntersectionObserver) y "bump" del botón del carrito. Todo se desactiva con `prefers-reduced-motion` (ojo: Chrome headless lo reporta activado — emular `no-preference` para probar animaciones).
- El título del hero es `nowrap` y su tamaño está calculado para que entre "TU RECUPERACIÓN." (la palabra más larga). Si se agrega un objetivo con una palabra más larga, recalcular.
- Evitar volver a: neón/glow cian, grillas de fondo, fuente monoespaciada decorativa, emojis como íconos, badges repetidos en cada card. Íconos = SVG inline de trazo fino.
- Fotos de producto con `object-fit:contain` sobre `--photo-bg` (hay fotos verticales: Bro's, creatina frutos rojos, collagen plus) — no usar `cover` en catálogo porque las recorta.
- El cartel de precio del hero se toma de `PRODUCTS` por JS (no hardcodear); los contadores de las categorías también.

## Cuidado con las cuentas

El dueño tiene dos cuentas de GitHub en su navegador: `energymsuplementosfsa-art` (la de este proyecto) y otra personal/empresarial (`valentinnzarate`, usada para otro proyecto en curso). **Nunca** pushear ni autenticar con la cuenta personal para este repo. Si `git push` falla pidiendo credenciales de forma no interactiva, es normal — Git Credential Manager en Windows necesita un login interactivo en navegador; pedirle al dueño que corra el push él mismo desde una terminal abierta en esta carpeta.

## Caché de imágenes

`_headers` cachea `/images/*` 1 día (con `stale-while-revalidate`), **no** `immutable`: las fotos se re-procesan con el mismo nombre y además Cloudflare aplica ese header también a los 404 — con `immutable` un 404 visto durante un deploy quedaba cacheado un año en el navegador (pasó con el logo del header). Si se reemplaza una imagen y tiene que verse ya, cambiarle el nombre.

## Favicon

Monograma "E" itálica blanca + swoosh celeste sobre navy (opción elegida por el dueño entre 4). Los PNG (16/32/48/192/512, `apple-touch-icon` 180 cuadrado sin redondear) y `favicon.ico` (PNG 16/32/48 empaquetados) se generaron renderizando el diseño con Chrome headless, con la fuente Archivo. A 16px se omite el swoosh porque no se distingue. Los links llevan `?v=N`: subir N si se cambia el favicon, porque los navegadores lo cachean mucho.
