# Consultorio Dr. Luis Borja — guía para Claude

Otorrinolaringólogo, especialista en rinoplastía. Pueblo Libre, Lima, Perú.
El usuario escribe en español; respóndele en español.

## Si la sesión es sobre Carla → este repo no sirve

**Carla** es la recepcionista virtual de WhatsApp del consultorio. **No vive en este repo.**
Su fuente de verdad es `MAPA-CARLA.md` (y `BITACORA.md`) en el repo privado
`luisedulive/carla-whatsapp-server`. `docs/carla/ESTADO.md` está **archivado**: no lo
actualices ni escribas aquí estado, métricas ni datos de pacientes — este repo es público.

## Qué hay en este repo

`index.html` — **Sistema de Reseñas**, una página autocontenida (HTML + CSS + JS en un solo archivo, sin build ni dependencias) para gestionar la reputación del consultorio en Google Business: checklist de tareas con progreso en `localStorage`, generador de mensajes para pedir reseñas y generador de respuestas a reseñas negativas.

Es un proyecto **distinto de Carla**, aunque del mismo consultorio.

Detalle conocido a tener en cuenta: `callClaude()` (línea ~549) hace `fetch` directo a `api.anthropic.com` **sin cabecera de autenticación**, así que los dos generadores fallan si la página se abre en un navegador normal. No lo "arregles" metiendo una API key en el archivo — es una página estática y pública; la clave quedaría expuesta. La salida correcta es un proxy del lado del servidor.

## Convenciones

- Documentación y textos de producto en español.
- Cifras: si no tienen fuente, no entran. Marca las proyecciones con `≈` y di sobre qué muestra se proyectaron.
- Fechas: sácalas de `date`, no de la memoria ni de metadatos de la sesión.
- Este repo no tiene tests ni linters configurados.
