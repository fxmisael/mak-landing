# MAK — Landing institucional

Web pública de **MAK** (Misael · Andrew · Kevin), compañía de tecnología en Santo Domingo, R.D.
Posicionamiento: «Tecnología que trabaja por tu negocio» · lema «Innovación que impulsa posibilidades».
Soluciones actuales: MAK Business, MAK AI y MAK Web (catálogo que crece: ver `SOLUTIONS`).

**Producción:** https://maksistemas.com (mak-landing.vercel.app y www redirigen al apex con 308 — Misael no quiere que el dominio vercel.app se vea; no volver a publicarlo · proyecto Vercel `mak-landing`, cuenta `fxmisael`)

---

## Archivos

| Archivo | Qué es |
|---|---|
| `MAK Landing.dc.html` | **La landing.** Fuente de verdad. Se publica como `index.html` |
| `support.js` | Runtime de Claude Design (generado). **Ver "Parches al runtime"** |
| `vendor/` | React 18.3.1 UMD servido desde nuestro dominio |
| `terminos.html` / `privacidad.html` | Páginas legales (HTML plano, sin runtime) |
| `assets/` | `hero.mp4`, `image-1-opt.jpg` (póster), `image-2-opt.jpg` (fondo contacto) |
| `assets/proyectos/` | Clips `.mp4` (H.264, sin audio) + póster `.jpg` de Asian Palace y Hermanos Britos, escritorio y celular |
| `api/demo.js` | Función de Vercel: IA real de la demo (OpenRouter). Ver «IA real de la demo» |
| `vercel.json` | Cabeceras de seguridad + CSP + `cleanUrls` + `maxDuration` de la función |
| `favicon.svg`, `apple-touch-icon.png` | Íconos |
| `uploads/` | Originales pesados. **No se despliegan** |

Los `.png` originales en `assets/` quedan como respaldo; el HTML referencia los `-opt.jpg`.

---

## Cómo está construido

Salida de **Claude Design**: HTML con estilos inline y un runtime propio (`support.js`) que
monta React. No hay build, no hay npm, no hay framework. Sintaxis del runtime:

- `{{ expr }}` — interpolación, resuelta contra lo que devuelve `renderVals()`
- `<sc-if value="{{ cond }}">` / `<sc-for list="{{ arr }}" as="x">`
- `style-hover="..."` / `style-active` / `style-focus` / `style-before` → reglas CSS generadas
- La lógica vive en el `<script type="text/x-dc">` al final: `class Component extends DCLogic`

### Parches al runtime (no se pierden al editar el HTML, pero sí si se regenera `support.js`)

1. **`style-hover` envuelto en `@media (hover:hover) and (pointer:fine)`** — sin esto el hover
   se queda pegado al tocar en celular. Está en `createPseudoSheet`.
2. **React local.** El runtime carga React desde `unpkg.com`; el CSP lo bloquea. En el `<head>`
   hay un `window.__resources` que remapea esas URLs a `/vendor/`. Los hashes SRI de los
   archivos descargados coinciden con los oficiales — verificar si se actualizan.

### Trampa conocida

`componentDidUpdate` recibe **un solo argumento** (`prevProps`), no `(prevProps, prevState)`.
El estado previo se guarda a mano en `_pMenuOpen` / `_pMobile`. Si alguien escribe
`componentDidUpdate(pp, ps)` vuelve el `TypeError` en cada render.

---

## Identidad — no cambiar

- **Fondo** `#060D1C` · **Superficies** `#0B1B35`, `#081427` · **Texto** `#fff`
- **Azul MAK** gradiente `#0A9BF5 → #0A3FC0` · **Acento** `#3AA8DB`
- **Sora** (300/400/500/600) titulares · **Inter** (300/400/500) cuerpo — Google Fonts, `display=swap`
- Vidrio líquido: `backdrop-filter: blur(18px) saturate(1.4)` + borde con máscara (`style-before`)
- Video hero pegajoso con paralaje y crossfade entre dos `<video>`

El escritorio se rediseñó el 2026-10-20 (tres productos + experiencias interactivas).
Ya no está congelado, pero cualquier cambio se verifica a 1280×800 y en los seis tamaños.

---

## Secciones y lógica (2026-10-20)

Orden: `#inicio` (hero de dos niveles, video + núcleo canvas con nodos Business/AI/Web que llevan a
la tarjeta de cada solución) → `#tecnologia` (5 capacidades) → `#flujo` (arquitectura conceptual) →
`#proceso` → `#soluciones` (catálogo + franja «Innovación continua» + panel de detalle `#detalle`) →
`#demo` → `#industrias` → `#proyectos` (clips en movimiento) → `#nosotros` (socios + filosofía) →
FAQ → `#diagnostico` → `#contacto`. **No hay sección de Planes** (se reemplazó por el catálogo).

- Todo el contenido comercial vive en constantes al inicio del script (`SOLUTIONS`, `INDUSTRIES`,
  `PARTNERS`, `PHILOSOPHY`, `AI_CAPS`, `DIAG`, `FLOW`, `PROJECTS`, `FAQS`…). Sumar una solución = un objeto
  nuevo en `SOLUTIONS` (+ su ícono en el marcado). `AI_CAPS` refleja lo verificado en `agente-mak` rama `main`: no subir
  un estado ("En desarrollo" → "Implementado") sin verificarlo en código desplegado.
- **Demo de IA = IA real + respaldo.** Decisión de Misael (2026-10-21): OpenRouter con `google/gemini-3.5-flash-lite`,
  ~$5/mes. Ver «IA real de la demo». `DemoEngine` (entre `/* ENGINE START */` y `/* ENGINE END */`) es el motor de
  reglas de respaldo: slots por escenario, sin red; nunca inventa precios, horarios ni confirma reservas.
- **`#demo` = MAK AI · Experience Lab.** 4 industrias (`SCENARIOS`: `dental`, `restaurante`, `comercio`,
  `servicios`) y dos modos (`LAB_MODES`): *conversa con MAK AI* (chat libre) y *ver cómo
  funciona* (panel «Detrás de escena»). La conversación guiada con guion se quitó a pedido de Misael (2026-10-21). El panel sale de `DemoEngine.insights(id, memoria)`: intención, etapas,
  ficha CRM, acciones y checklist del agente juez. Intención y ficha salen de lo que extrae el modelo; **acciones y
  evaluación son simuladas y lo dicen**. Las banderas `mem.flags.{price,info,contact,human}` y `mem.summarized` las
  pone `reply()` o `respond()` con lo que devuelve la función. Los textos con IA vienen en pares `[con IA, respaldo]`
  (`LAB_MODES`, `LAB_EXPLAIN`, elegidos con `DemoEngine.live()`): **nunca decir «IA» cuando responde el motor**.
  Probar el motor sin navegador: extraer `ENGINE START/END` y llamar `reply()` en Node.

### IA real de la demo (`api/demo.js`)

- Función de Vercel (CommonJS, sin dependencias) en el mismo dominio: la CSP no cambia (`connect-src 'self'`).
  `DEMO_ENDPOINT = '/api/demo'`. El cliente manda `{ scenario, messages }`; el prompt y los negocios ficticios
  viven **en el servidor** (`SCN`, mismas claves de campo que `DemoEngine`). Salida `json_schema` estricta:
  `{ text, slots, handoff, summarized, flags, suggestions }`; `respond()` arma la memoria y calcula `pending`
  con `next()` del motor.
- Variables de Vercel: `OPENROUTER_API_KEY` (obligatoria, solo Production) y `OPENROUTER_MODEL` (opcional).
  **Cambiar una variable exige redeploy.**
- Límites: 300 caracteres por mensaje, 24 mensajes de historial, 14 turnos por conversación, 30 peticiones
  por IP cada 10 min (en memoria, best effort), `max_tokens` 600, timeout 12 s (20 s con audio), `data_collection: 'deny'`.
  **El tope real de gasto es el límite de crédito de la key en OpenRouter.**
- Red de seguridad: si el texto trae una cifra de dinero (`MONEY`), se reemplaza por «el equipo lo confirma».
- **Personalidad** (pedido de Misael, 2026-10-21): humano, flow dominicano con medida, empático (reconoce el dolor o la urgencia
  antes de preguntar) e inteligente (no repite preguntas, toma todos los datos). `temperature` 0,7. Las reglas duras no cambian.
- **Demora:** `REPLY_DELAY = [8000, 10000]` ms (decisión de Misael). Se espera solo lo que falte tras la respuesta de la IA, con la
  burbuja de puntos y «Escribiendo…» en la cabecera. Las pruebas la acortan con `window.MAK_REPLY_DELAY`.
- **Notas de voz:** botón de micrófono (solo con IA en vivo y navegador con `MediaRecorder`), máximo 30 s. El audio se convierte en el
  navegador a WAV 16 kHz mono (`toWav`) y viaja como `audio: { data, format: 'wav' }` solo en el último mensaje (máx. 1,5 MB en
  base64). El modelo devuelve `heard` (transcripción), que reemplaza al audio en el historial. Sin IA no hay respaldo para voz: error
  honesto. Con audio: timeout 20 s y `maxDuration` 25. **`Permissions-Policy` lleva `microphone=(self)`**: si se vuelve a `()`, el
  micrófono deja de funcionar sin error visible.
- Si la IA no está disponible responde **200 `{ unavailable, reason }`** (no 5xx, para no ensuciar la consola);
  el cliente usa el motor de reglas, muestra «Modo respaldo» y, si el motivo es `no_key|credit|rate|turns`,
  no vuelve a llamar en esa visita. Nunca se loguea el contenido de los mensajes.
- Privacidad §2 d) y §5 nombran a OpenRouter y Google como destinatarios. Si cambia el proveedor o se deja de
  usar `data_collection: 'deny'`, actualizar la política.
- Pruebas (scripts temporales, se rehacen si se pierden): handler con `fetch` simulado, y Playwright con `page.route('**/api/demo')`
  (IA simulada o «sin key»; `LIVE=1` deja pasar la función real). Voz: Chrome con `--use-fake-device-for-media-stream
  --use-file-for-fake-audio-capture=<wav>` y un WAV generado con `say -v Paulina`.
- **Formulario:** `LEAD_ENDPOINT` vacío → redacta la solicitud en WhatsApp y la pantalla final dice
  que aún no llegó. Con valor → POST JSON real con estados de carga/error.
- Selectores CSS: el runtime envuelve cada `{{ }}` en un `<span>`; usar `>` o clases, nunca
  `.x span` genérico. No interpolar `d=` de `<path>` (error de consola): íconos estáticos + `sc-if`.
- Pruebas: Playwright de `agente-mak/node_modules` con `channel:'chrome'` (no hay navegadores de
  Playwright instalados).

## Reglas de celular

Todo lo móvil vive en el bloque `@media (max-width: 767px)` del `<style>` del `<helmet>`,
con clases `mak-*` y `g-*` puestas sobre los elementos.

| Clase | Qué controla |
|---|---|
| `mak-h1`, `mak-br` | Titular del hero; los `<br>` se ocultan (llevan espacio antes, si no se pegan las palabras) |
| `mak-hero`, `mak-vspacer`, `mak-hero-video`, `mak-scrim` | Hero y encuadre del video |
| `mak-fab`, `mak-form-label` | Botón de WhatsApp (safe-area) y etiquetas del formulario |
| `mk-sol`, `mk-card`, `mk-ptn` | Catálogo de soluciones, tarjetas de capacidades/industrias (riel en móvil) y socios (3 en fila siempre) |
| `menu-link` | Escala reducida del menú móvil |

Bloques adicionales: `@media (orientation: landscape) and (max-height: 560px)` (celular
horizontal: oculta el separador del video, refuerza el scrim) y `768–1023px` (tablet).

Invariantes: `100svh` nunca `100vh` · `overflow-x: clip` en `html` y `.mak-root` · toques ≥44px ·
cuerpo ≥16px · cero desbordes a 360, 375, 390, 844×390, 768 y 1280.

---

## Datos reales vs. placeholders

**Reales:** nombre MAK, los tres socios y sus roles, los tres servicios, el proceso de 5 etapas,
Santo Domingo R.D., WhatsApp **+1 (787) 902-6590** (`WA_NUMBER = '17879026590'`), correo **info@maksistemas.com**
(Zoho Mail; los MX/SPF/DKIM van en el DNS de Vercel), clips de Asian Palace y Hermanos Britos (**con permiso**, 2026-10-21).
RNC, razón social y dirección exacta **se quitaron a propósito** (decisión de Misael, 2026-10-21): no volver a ponerlos.

**Placeholders — nunca inventar valores:**

- Instagram y LinkedIn — columna «Redes» del footer **quitada** (2026-10-21) hasta tener cuentas reales
- 3 fotos de socios — recuadros con la paleta, intencionales
- Cifras de precios, plazos y soporte en el FAQ — `[confirmar]`
- Testimonios — sección oculta por la prop `showTestimonials` hasta tener reales

**El formulario no tiene backend.** Al enviar abre WhatsApp con los datos redactados
(`submit()` en el script). Es un puente, no la solución: conectarlo a correo o CRM.

---

## Legal

Redactado para **República Dominicana**: Ley 172-13 (datos personales, derechos ARCO —
5 días hábiles acceso, 10 para rectificación/cancelación/oposición), Ley 358-05 (consumidor,
Pro Consumidor), Ley 126-02, Ley 310-14 (anti-spam), Ley 53-07, Ley 65-00.

**No hay `cookies.html` a propósito:** el sitio no tiene analítica, píxeles ni cookies no
esenciales, y la política de privacidad lo dice explícitamente.

Si se agrega Google Analytics, Meta Pixel o cualquier cookie no esencial → hace falta banner
con "Aceptar" y "Rechazar" del mismo peso, cargar los scripts solo tras aceptar, crear
`cookies.html` y actualizar la sección 9 de privacidad. Si se añade un backend al formulario
o newsletter, actualizar privacidad (destinatarios) y pedir consentimiento aparte para promos.

Los documentos no han sido validados por un abogado colegiado.

---

## Seguridad

`vercel.json` fija: `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`,
`Permissions-Policy`, HSTS y **CSP**.

La CSP solo permite: propio dominio, `fonts.googleapis.com` (CSS) y `fonts.gstatic.com`
(fuentes). **Cualquier script, imagen o fetch externo nuevo queda bloqueado en silencio** —
hay que añadir el origen a mano. Preferir alojar local, como se hizo con React.

`script-src` lleva `'unsafe-inline'` y `'unsafe-eval'` porque el runtime los necesita.

---

## Desplegar

```bash
# desde una carpeta limpia con index.html, support.js, vendor/, assets/, api/,
# terminos.html, privacidad.html, favicon.svg, apple-touch-icon.png, vercel.json
npx vercel link --project mak-landing --yes
npx vercel deploy --prod --yes
```

`MAK Landing.dc.html` se copia como `index.html`. **Sin `vercel link` se crea un proyecto
nuevo y la URL cambia** (ya pasó una vez: quedó un proyecto vacío `mak-deploy` por borrar).

`cleanUrls: true` → los enlaces internos van a `/terminos`, `/privacidad` y `/`, sin `.html`.
Por eso el proyecto no se puede probar bien con `python3 -m http.server`: esas rutas dan 404
en local pero funcionan en Vercel.

### Verificar antes de dar por cerrado

Cero errores de consola (leerlos en una pestaña nueva, el buffer es acumulativo) · cero
recursos fallidos · solo dos orígenes externos · sin desbordes en los seis tamaños ·
escritorio idéntico a 1280×800.

---

## Herramientas ausentes en esta Mac

No hay `ffmpeg`, `cwebp`, `PIL` ni `playwright`. Las imágenes se optimizaron con `sips`
(PNG → JPEG; **`sips` no soporta webp** en esta versión de macOS). El video no se puede
recodificar aquí.

El hero es **848×480** y en escritorio se muestra a ~1600px (1.9× de upscale). Hace falta un
master 1920×1080 y un corte 9:16 de 1080×1920 para celular. `og:image` debería ser 1200×630.

---

## Nosotros, misión y visión

- Socios: tarjetas 3/4 con la inicial (M · A · K) en gradiente MAK; sin fotos por ahora. Para poner una foto:
  llenar `foto` en `PARTNERS` (p. ej. `assets/socios/m.jpg`) y subir el archivo.
- `MISION` y `VISION` (textos que pasó Misael el 2026-10-21) se muestran con la prop `showMision` (activa por defecto) y solo si tienen texto.

## Proyectos en movimiento

`assets/proyectos/{asian,hb}-{desktop,movil}.mp4` son **grabaciones de pantalla** de los sitios (Asian Palace publicado;
Hermanos Britos desde su build local, aún sin publicar). Se reproducen solo mientras el escenario está a la vista
(`setupProjVideo`, `IntersectionObserver`); con `prefers-reduced-motion` queda el póster. El `poster` se aplica por JS
desde `data-poster` (un `poster="{{ }}"` lo pide el navegador antes de que el runtime interpole → 404).
Para regenerarlos: Playwright en Chrome con `Page.startScreencast` (CDP) y `ffmpeg-static` (en carpeta temporal,
no en el proyecto): `-an -c:v libx264 -profile:v main -crf 30 -movflags +faststart`, 1280 px de ancho escritorio y 560 móvil.
Asian Palace y Hermanos Britos dieron permiso para mostrarlos (2026-10-21).

## Contención horizontal en celular (zoom y paneo lateral)

Capas: `html{overflow-x:clip}` + `.mak-root{overflow-x:clip}` + `body{touch-action:pan-y pinch-zoom}` en ≤859px
(los rieles `.mk-rail/.mk-tl/.mk-flow/.mk-scn/.mk-sugg/.mk-side` llevan `touch-action:pan-x pan-y`) +
`-webkit-text-size-adjust:100%`. Los rieles **no** usan márgenes negativos. No usar `maximum-scale` (bloquea el zoom accesible).
Evitar nombres de clase genéricos: `.mk-cap` ya existía (pie de foto) y rompió una tarjeta; las tarjetas nuevas son `.mk-card`.
Playwright no corre WebKit en esta Mac (macOS 12): la verificación en iPhone real es manual.
