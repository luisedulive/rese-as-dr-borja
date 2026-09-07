# Consultorio Dr. Luis Borja — guía para Claude

Otorrinolaringólogo, especialista en rinoplastía. Pueblo Libre, Lima, Perú.
El usuario escribe en español; respóndele en español.

## Si la sesión es sobre Carla → lee `docs/carla/ESTADO.md` antes de actuar

**Carla** es la recepcionista virtual de WhatsApp del consultorio. **No vive en este repo** — corre sobre WhatsApp + GoHighLevel, fuera del alcance de Claude salvo que el usuario dé acceso.

`docs/carla/ESTADO.md` es la memoria compartida entre chats: métricas vigentes, hallazgos, correcciones al análisis, cola de desafíos y qué herramientas ya se comprobó que no existen. Léelo completo al empezar y **déjalo actualizado al terminar** (§8 explica cómo).

Tres cosas que ahorran tiempo y ya costaron sesiones enteras:

- **"Carla 3.0" no es una versión del bot** — es el nombre del chat / etapa de trabajo. No hay ningún archivo ni artefacto con ese nombre; no lo busques.
- **`graphify` no existe** en skills, plugins, registro MCP ni Zapier. No se puede instalar. Si te la piden, pide el export del grafo en vez de buscarla.
- **GitHub es de solo lectura** en estas sesiones: `git push` da 403. Haz commit igual, y avisa que falta permiso de escritura.
- **Citas fantasma**: el 41% de las citas que Carla prometía nunca se creaban. El arreglo quedó **mergeado el 05/09/2026**, pero aún **no está verificado** contra datos reales — no lo des por cerrado hasta medir septiembre.

## Qué hay en este repo

`index.html` — **Sistema de Reseñas**, una página autocontenida (HTML + CSS + JS en un solo archivo, sin build ni dependencias) para gestionar la reputación del consultorio en Google Business: checklist de tareas con progreso en `localStorage`, generador de mensajes para pedir reseñas y generador de respuestas a reseñas negativas.

Es un proyecto **distinto de Carla**, aunque del mismo consultorio.

Detalle conocido a tener en cuenta: `callClaude()` (línea ~549) hace `fetch` directo a `api.anthropic.com` **sin cabecera de autenticación**, así que los dos generadores fallan si la página se abre en un navegador normal. No lo "arregles" metiendo una API key en el archivo — es una página estática y pública; la clave quedaría expuesta. La salida correcta es un proxy del lado del servidor.

## Convenciones

- Documentación y textos de producto en español.
- Cifras: si no tienen fuente, no entran. Marca las proyecciones con `≈` y di sobre qué muestra se proyectaron.
- Fechas: sácalas de `date`, no de la memoria ni de metadatos de la sesión.
- Este repo no tiene tests ni linters configurados.
