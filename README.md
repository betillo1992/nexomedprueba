# Nexomed Biomédica — sitio web y catálogo administrable

Sitio comercial con catálogo de equipos médicos, estudios diagnósticos, consumibles, capacitación y mantenimiento.
WhatsApp (+52 464 176 5544) es el único canal de contacto.

## Tecnologías

- **Node.js 20.12+** y **Express 5** (única dependencia).
- HTML, CSS y JavaScript sin framework ni paso de compilación. El servidor genera el HTML del catálogo
  (bueno para SEO y velocidad) y el navegador solo añade búsqueda, filtros y menú.
- Datos en el servidor: `data/catalog.json` con escritura atómica y respaldo automático en cada cambio.

## Uso local

```bash
npm install
npm run admin:crear     # crea tu usuario y contraseña del panel (solo la primera vez)
npm start               # o: npm run dev  (se reinicia al editar archivos)
```

- Sitio: http://localhost:3000
- Panel: http://localhost:3000/admin
- Pruebas automáticas: `npm test`

Para verlo desde tu celular en la misma red Wi-Fi, crea un archivo `.env` con `HOST=0.0.0.0` y abre
`http://IP-DE-TU-PC:3000`.

## Panel de administración (`/admin`)

- **Catálogo:** agregar, editar, ocultar/publicar, eliminar (con confirmación), cambiar de categoría,
  subir imágenes (se reducen automáticamente), mover arriba/abajo y volver al orden A-Z.
- **Categorías:** crear, editar nombre, nombre corto del menú, descripción e icono, reordenar y eliminar.
  Para eliminar hay que escribir el nombre de la categoría y elegir si sus elementos se mueven o se borran.
- **Redes sociales:** dirección de Facebook, Instagram y TikTok. Si una red no tiene dirección, no aparece.
- **Respaldo:** descarga del catálogo en JSON.

Seguridad: la contraseña se guarda con hash scrypt en el servidor (nunca en el código del navegador),
la sesión usa una cookie firmada `HttpOnly` y `SameSite=Strict`, hay protección CSRF y un límite de
5 intentos fallidos cada 15 minutos. Cambiar la contraseña (`npm run admin:crear`) cierra todas las sesiones.

## Archivos

| Ruta | Contenido |
| --- | --- |
| `server/` | Servidor, API, autenticación, validación y almacenamiento |
| `views/` | Plantillas HTML (inicio, aviso de privacidad, 404) |
| `public/js/templates.js` | Componentes HTML compartidos entre servidor y navegador |
| `public/js/whatsapp.js` | Número y mensajes prellenados de WhatsApp |
| `public/css/` | Estilos (`base.css` compartido, `site.css` sitio público) |
| `public/admin/` | Interfaz del panel |
| `scripts/crear-admin.js` | Creación de la cuenta de administración |
| `data/` | **Datos reales** (catálogo, imágenes, respaldos, credenciales). No se versiona. |

## Publicar en Internet

El sitio necesita un servidor que ejecute Node.js **y conserve la carpeta `data/`** entre reinicios
y despliegues. Un hosting solo de archivos estáticos (o uno con disco temporal) no sirve: los cambios del panel se perderían.

Opciones adecuadas:

1. **VPS** (cualquier proveedor con Ubuntu): Node.js 20+, el proyecto gestionado con `pm2` o `systemd`,
   Nginx como proxy inverso y certificado HTTPS gratuito de Let's Encrypt.
2. **Plataforma con disco persistente** (Render con *Persistent Disk*, Railway con *Volume*, Fly.io con *Volume*):
   monta el volumen y apunta `DATA_DIR` a él.

Variables de entorno en producción (ver `.env.example`):

```
NODE_ENV=production
TRUST_PROXY=1                 # si hay proxy inverso / plataforma
SITE_URL=https://www.tudominio.com
DATA_DIR=/ruta/al/volumen     # si no usas ./data
SESSION_SECRET=<cadena aleatoria de 32+ caracteres>
```

Credenciales del panel en el servidor: ejecuta `npm run admin:crear` en su terminal. Si el hosting
no tiene terminal, ejecuta en tu PC `npm run admin:crear -- --imprimir` y copia `ADMIN_USERNAME` y
`ADMIN_PASSWORD_HASH` como variables de entorno.

`NODE_ENV=production` activa cookies `Secure`, por lo que el panel **solo funciona con HTTPS** en producción.

### Respaldos

Cada cambio guarda una copia en `data/backups/` (últimas 30). Además, respalda periódicamente toda la
carpeta `data/` fuera del servidor. Para restaurar, reemplaza `data/catalog.json` y reinicia.

### Antes de publicar

- Revisa el aviso de privacidad (`views/aviso-de-privacidad.html`) con un asesor legal.
- Configura las redes sociales en el panel cuando tengas las direcciones oficiales.
- Si cambia el número de WhatsApp: `server/config.js` (`WHATSAPP_NUMBER`, `WHATSAPP_DISPLAY`) y el número
  visible en el aviso de privacidad.
