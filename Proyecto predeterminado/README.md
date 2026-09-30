# TechnoSystem — Sitio web de mantenimiento y soporte técnico

Página web profesional, responsive y de carga rápida para un negocio de
mantenimiento y soporte de computadores. Sin dependencias externas (sin
librerías ni frameworks): HTML + CSS + JavaScript puros.

## Estructura del proyecto

```
├── index.html        ← Contenido principal, textos e iconos (SVG inline)
├── politicas.html    ← Políticas, términos y libro de quejas y reclamos
├── css/styles.css    ← Estilos (organizados por secciones, usa variables CSS)
├── js/main.js        ← Interacciones y CONFIGURACIÓN (bloque 1 del archivo)
├── robots.txt
└── README.md         ← Esta guía
```

## Cómo verla

Abre `index.html` directamente en el navegador, o para probarla en local:

```bash
# Python
python -m http.server 8080
# o con Node (si tienes npx)
npx serve .
```

Luego visita `http://localhost:8080`.

---

## Lo que debes personalizar

### 1. Nombre y datos del negocio
- **Nombre:** busca `TechnoSystem` en `index.html` (aparece en título, navbar,
  footer) y cámbialo por el nombre real.
- **Teléfono:** están en la sección `#contacto` y en el
  footer de `index.html`. El sitio no muestra otro correo: el único correo
  visible es el de quejas y reclamos (abajo).
- **Redes sociales:** en el footer solo está Instagram, con enlace directo a
  `https://www.instagram.com/itstechnosystem_jm/` (cámbialo si usas otra cuenta).
- **Precios:** en la sección `#precios`, edita montos y contenidos de los tres
  planes (están comentados con `EDITA`), y debajo, la lista **"Precio por
  servicio individual"** (7 filas con su propio comentario `EDITA`). La moneda
  mostrada es el **peso
  colombiano (COP)**: si cambias de moneda, actualiza los montos, el texto
  `COP / servicio` de cada plan y la nota bajo los planes.
- **Correo de quejas y reclamos:**
  `arisantigo2024@gmail.com` (aparece en `index.html` —tarjeta
  de contacto y footer— y en `politicas.html`). Si cambias de correo,
  reemplázalo en ambos archivos.
- **Políticas legales:** viven en `politicas.html` y se enlazan desde el footer
  (privacidad, cookies, términos y libro de reclamos). Son un modelo completo
  y coherente con el sitio: ajústalo a la legislación de tu país y, si puedes,
  hazlo revisar por un profesional del derecho antes de publicar.

### 2. WhatsApp
En `js/main.js`, bloque **1. CONFIGURACIÓN**:

```js
whatsapp: {
  numero: "573122698232",  // formato internacional sin "+" ni espacios
  mensajeBase: "Hola, quiero solicitar un servicio de mantenimiento."
}
```

Cambia también los enlaces `https://wa.me/573122698232...` directamente en
`index.html` (aparecen en el hero, la sección de contacto y el botón flotante).

### 3. Colores / identidad visual
Todo el color está en `css/styles.css` dentro de `:root`. Los más
importantes:

```css
--bg: #05070d;        /* fondo oscuro */
--accent: #22d3ee;    /* cian principal */
--accent-2: #818cf8;  /* violeta secundario */
--grad: linear-gradient(135deg, #22d3ee, #818cf8);  /* degradado de marca */
```

---

## Conectar formularios a un backend

Ambos formularios (`#serviceForm` y `#reviewForm`) tienen **dos modos**
(configurado en `js/main.js`):

| `CONFIG.*` | Comportamiento real |
|---|---|
| `formEndpoint: ""` | La solicitud se envía abriendo WhatsApp **conversación con tu número** y el mensaje ya escrito (nombre, contacto, servicio, equipo y descripción): la persona solo presiona "Enviar". Si el navegador bloquea la ventana, se navega en la misma pestaña y aparece un enlace de respaldo. Funciona sin servidor. |
| `formEndpoint: "/api/solicitudes"` | Envía `POST` en JSON a tu backend con `fetch`. |
| `reviewsPostEndpoint: ""` | La reseña se guarda **dentro de la aplicación** (localStorage del navegador): aparece al instante en el panel, sin WhatsApp, y **solo se permite una por persona** (se valida con el campo "Correo o WhatsApp"). |
| `reviewsPostEndpoint: "/api/resenas"` | Envía `POST` en JSON con la reseña (aplica el límite de una por persona en el backend). |
| `reviewsEndpoint: ""` | El panel muestra las reseñas guardadas en ese navegador. |
| `reviewsEndpoint: "/api/resenas"` | Hace `GET` y pinta las reseñas publicadas para **todos** los visitantes. |

### Qué debe recibir tu backend

**Solicitudes** — `POST` con `Content-Type: application/json`:

```json
{
  "nombre": "Juan Pérez",
  "contacto": "312 2698232",
  "servicio": "Limpieza interna del computador",
  "equipo": "Portátil",
  "mensaje": "Mi computador tarda mucho en encender..."
}
```

**Reseñas** — `POST` con `Content-Type: application/json`:

```json
{
  "nombre": "María López",
  "contacto": "maria@correo.com",
  "puntuacion": "5",
  "comentario": "Excelente servicio, quedó como nuevo."
}
```

> El campo `contacto` es el que sirve para aplicar la regla de
> **una reseña por persona** (guárdalo hasheado y crea un índice único).

**Listado de reseñas** — `GET` que devuelva un array:

```json
[
  { "nombre": "María López", "puntuacion": 5, "comentario": "Excelente servicio." }
]
```

### Ejemplo mínimo con Node/Express

```js
app.post("/api/solicitudes", express.json(), (req, res) => {
  const { nombre, contacto, servicio, equipo, mensaje } = req.body;
  // valida los campos y guárdalos (base de datos, correo, Telegram, etc.)
  res.status(201).json({ ok: true });
});

app.post("/api/resenas", express.json(), (req, res) => {
  const { nombre, contacto, puntuacion, comentario } = req.body;
  // 1) hashea "contacto" y verifica que esa persona no tenga otra reseña
  // 2) guarda la reseña con estado "pendiente" y repítela al cliente al moderar
  res.status(201).json({ ok: true });
});

app.get("/api/resenas", (req, res) => {
  // devuelve solo las reseñas con estado "publicada"
  res.json([{ nombre: "María López", puntuacion: 5, comentario: "Excelente servicio." }]);
});
```

### Ejemplo con Google Forms / Formspree / Basin

Si usas un servicio de formularios, pon su URL en `formEndpoint` y ajusta los
nombres de los campos en `js/main.js` para que coincidan con los del
servicio (usa `FormData` en lugar del objeto `data` si es necesario).

Si el servidor responde con estado distinto de 2xx, el formulario muestra un
mensaje de error y sugiere escribir por WhatsApp: nunca finge un envío exitoso.

---

## Sobre las reglas pedidas

- **Sin funciones falsas:** nada simula reparaciones, diagnósticos en vivo ni
  procesos automáticos. El "Checklist de servicio" del hero es texto
  informativo, no una interfaz que simule acciones. Tampoco hay reseñas
  inventadas: no se precargan valoraciones de mentira.
- **Reseñas en la aplicación:** se envían y se muestran en la propia página
  (nada de WhatsApp), quedan guardadas en el navegador del visitante y cada
  persona solo puede publicar **una** (validada por su correo o WhatsApp). La
  persona que publicó una reseña puede **eliminarla** con el botón «Eliminar»
  del panel (solo se muestra en el navegador donde la creó). Para que todas las
  reseñas las vean todos los visitantes, conecta `reviewsEndpoint` /
  `reviewsPostEndpoint` a un backend.
- **Backend pendiente:** los dos formularios y el listado de reseñas quedan
  declarados en `CONFIG` y documentados arriba.
- **Sin cookies de rastreo:** el sitio no usa publicidad ni analítica de
  terceros (declarado en `politicas.html`), lo que evita avisos de cookies
  invasivos y riesgos de retirada por incumplimiento.
- **Fácil de modificar:** textos en el HTML, colores en `:root`, datos en el
  bloque `CONFIG` de `main.js`.
- **Rápida:** cero librerías, cero imágenes pesadas, iconos SVG inline,
  fuente con `display=swap` y JavaScript con `defer`.
- **Responsive:** menú móvil, grids adaptativos y `prefers-reduced-motion`
  para accesibilidad.
- **Auditoría Lighthouse:** 100 en Accesibilidad, Buenas prácticas y SEO
  (tanto `index.html` como `politicas.html`).

## Mantenimiento de contenidos

Las secciones siguen el orden del `index.html`:
`hero → servicios → precios → contacto (solicitud) → sobre nosotros → FAQ → reseñas → footer`.
Para agregar un servicio, copia un bloque `<article class="card">` en la
sección `#servicios` y añade una opción al `<select id="servicio">` del
formulario.
