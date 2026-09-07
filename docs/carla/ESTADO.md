# Estado de Carla

**Última actualización:** 6 de septiembre de 2026 (hora de Lima)
**Mantener este archivo al día es parte del trabajo.** Es la memoria compartida entre chats: cualquier Claude que abra una sesión nueva sobre Carla debe leerlo antes de hacer nada, y dejarlo actualizado al terminar.

---

## 1. Léeme primero (30 segundos)

- **Carla** es la recepcionista virtual de WhatsApp del consultorio del **Dr. Luis Borja** — otorrinolaringólogo, especialista en rinoplastía, Pueblo Libre, Lima, Perú.
- **"Carla 3.0" NO es una versión del bot.** Es el nombre del chat / etapa de trabajo actual. No existe ningún archivo, artefacto ni repo llamado "Carla 3.0": no lo busques (ya se buscó en repo, artefactos y Drive — cero resultados).
- **Citas fantasma: el arreglo quedó mergeado** (reportado por el usuario, 05/09/2026). Fue el problema #1 de todo agosto — el 41% de las citas que Carla prometía nunca se creaban. **Falta verificarlo con datos reales de septiembre**; hasta entonces no lo des por cerrado.
- **Los números de este documento son de agosto y ya están vencidos.** Van al día 22 de agosto; hoy es 5 de septiembre. Toca volver a medir.
- **Carla no vive en este repo.** `luisedulive/rese-as-dr-borja` contiene el *Sistema de Reseñas*, que es otro proyecto del mismo consultorio. Carla corre sobre WhatsApp + GoHighLevel, fuera del alcance de Claude salvo que el usuario dé acceso.

---

## 2. Qué se sabe del sistema

| Pieza | Estado |
|---|---|
| Canal | WhatsApp |
| Origen de leads | Mayormente anuncios de Meta (Facebook/Instagram) |
| CRM / agenda | GoHighLevel — usa la etiqueta `agenda_sin_slot` |
| Ventana de reserva | El sistema solo puede reservar **14 a 21 días** hacia adelante |
| Audio y video | **Sí los procesa** (confirmado por el usuario en la sesión *Carla 3.0*) |
| Tiempo de respuesta | 5 s de mediana, 9 s en el 10% más lento |
| Stack | Node.js en Railway + GoHighLevel + OpenAI Responses API |
| Calendario donde caen las citas | `luisedulive@gmail.com` — el **primario** de la cuenta, o sea el destino por defecto de todo lo que el doctor crea sin elegir calendario. El calendario *"Agendamiento de Citas"* (`ca4d3be6dee…@group.calendar.google.com`) está **vacío**: cero eventos. Verificado 06/09/2026 |
| Citas de Carla, cómo se reconocen | Descripción con `widget/booking/28Gw88ixIkqf6V4uo5EY` (dominio `api.chatpartner.online` hasta ~23/08, `msgsndr.com` desde ~24/08) y título `🆕 NUEVO - Cita - Dr. Borja` |
| Duración de cita | 30 min, casi siempre entre 16:30 y 19:30 |

**No verificado desde Claude** (si lo necesitas, pídeselo al usuario): qué motor/prompt corre Carla, si el flujo vive en n8n u otro orquestador, acceso directo a los logs de conversación, y las credenciales de GoHighLevel.

---

## 3. Números — agosto 2026 (al día 22), vs julio

> ⚠️ **Vencidos.** Cierra agosto completo y mide septiembre antes de tomar decisiones con estas cifras.

| Métrica | Agosto | Julio |
|---|---|---|
| Personas que escribieron | 806 | 1.236 |
| Citas reales en el calendario | 32 | 41 |
| Conversión lead → cita | **4,0%** | 3,3% |
| Objetivo | 7% | — |
| Citas prometidas que no existen | ~20 en agosto · 53 acumuladas | — |

### El embudo, escalón por escalón

| Escalón | Personas | % sobre 806 |
|---|---|---|
| Escribieron por WhatsApp | 806 | 100% |
| Siguieron la conversación (2+ mensajes) | ≈505 | 63% |
| Conversación de verdad (6+ mensajes) | ≈230 | 29% |
| Carla ofreció día y hora puntual | ≈117 | 15% |
| Carla dijo "tu cita queda reservada" | ≈49 | 6,1% |
| **Cita real en el calendario** | **32** | **4,0%** |
| Confirmaron que asisten (respondieron recordatorio) | 12 | 1,5% |

### Por dónde entran los leads

| Puerta | Leads | Citas | Conversión |
|---|---|---|---|
| Anuncio · texto automático ("Hola, quisiera agendar una consulta!") | ≈505 (63%) | 19 | 3,8% |
| Anuncio · agrega algo propio | ≈182 (23%) | 6 | 3,3% |
| Audio o foto que no se pudo leer en el export | ≈61 (8%) | 0 | 0,0% ⚠️ ver §5 |
| Escribe por su cuenta (sin plantilla de anuncio) | ≈58 (7%) | 7 | **12,1%** |

**Cómo se midió:** 192 conversaciones de agosto leídas mensaje por mensaje y proyectadas sobre los 806 leads del mes. Toda cifra con `≈` es proyección, no conteo exacto. Las 32 citas sí están verificadas contra GoHighLevel.

---

## 4. Hallazgos

1. **Citas fantasma — 41% (ARREGLO MERGEADO, SIN VERIFICAR).**
   - **Qué pasaba:** en agosto Carla le dijo a ≈49 personas alguna versión de "tu cita queda reservada"; solo 29–32 existían. Las otras ~20 creían que tenían hora y nadie las esperaba.
   - **Causa:** Carla confirmaba horarios que no estaban entre los reales — sábados que no caen en sábado, o fechas a 30–40 días cuando el sistema solo reserva 14–21 días adelante.
   - **Mitigación previa:** el CRM dejó de marcarlas como confirmadas (quedan como `agenda_sin_slot`).
   - **Estado:** el usuario reporta que **el arreglo quedó mergeado el 05/09/2026**. No está verificado contra producción: ver §6.
   - **Casos que servían de prueba:** 20/08 Sandro ("30 de septiembre a las 4:30 pm" — es miércoles, sin cita); 20/08 Yuliana ("sábado 10:00 am para tu hijo" — sin cita); 22/08 Juan Carlos (Carla dio la dirección y cerró con "¡Te espero el sábado!" — sin cita). **Sirven de test de regresión: los tres casos deben ser imposibles ahora.**
   - **El premio:** con esas ~20 creadas, agosto habría cerrado en 52 citas = **6,5%**, casi el objetivo del 7%, sin gastar un sol más en publicidad. Eso es lo que hay que ver aparecer en septiembre.

2. **El precio no espanta; adelantarlo, sí. (ABIERTO)** Si el paciente pregunta el precio y Carla responde: 7,7% llega a cita. Si Carla lo menciona sin que se lo pidan: 4,6%. En agosto se dio el precio sin pedirlo a ~87 personas.

3. **La reactivación sirve, pero se cae en la ventana de 24 h. (ABIERTO)** ≈471 mensajes de "quedé pendiente de tu consulta"; ≈64 contestaron (13,6%). Pero ≈59 **nunca se entregaron**: WhatsApp bloquea texto libre pasadas 24 h si no va en plantilla aprobada.

4. **La velocidad no es el problema. (DESCARTADO)** 5 s de mediana. Nadie se va por demora ni por el horario — responde de madrugada igual que a mediodía. No inviertas tiempo aquí.

5. **Quien escribe con sus propias palabras convierte 3× mejor** (12,1% vs 3,8%). El 63% del volumen entra por el botón del anuncio con texto plantilla.

---

## 5. Correcciones al análisis (leer antes de citar cifras)

- **06/09/2026 — la cita de Angie NO era fantasma.** Verificada en el calendario: existe, jueves 3 de septiembre 19:00–19:30, creada el 1 de septiembre a las 19:52. Primera evidencia a favor del arreglo mergeado — un caso, no una prueba. Pero destapó **otro fallo**: ese día el doctor tenía `Guardia` y Carla agendó igual. Ver la cola en §6.
- **Audio y video.** El usuario confirmó que **Carla sí reconoce audio y video**. La fila "audio o foto que Carla no puede leer → 0% de conversión" del análisis original queda **en duda**: lo más probable es que sea un artefacto de la *exportación* de conversaciones (el multimedia no se descargó al leer los logs), no una falla del bot. **Pendiente verificar** con conversaciones reales antes de volver a usar ese dato.

---

## 6. Cola de desafíos

- [ ] **Carla agenda en días bloqueados del doctor** — encargo escrito en `docs/carla/encargos/calendario-bloqueos.md`. Caso: 3 sep, Angie Castillo Pangalima, cita a las 19:00 en un día con `Guardia`. Es un fallo **distinto** de las citas fantasma: la cita sí se crea, pero en un momento en que el doctor no atiende.
- [ ] **Verificar que las citas fantasma se cerraron** — con el arreglo mergeado, medir en septiembre la brecha entre "Carla prometió cita" y "cita existe en el calendario". Debe ser ~0. Los tres casos de agosto (§4.1) son el test de regresión.
- [ ] **Cerrar agosto y medir septiembre** — las cifras de §3 van solo al 22 de agosto.
- [ ] **Verificar audio/video** — sacar la conversión real de los ~61 leads que llegaron como multimedia. Requiere acceso a logs.
- [ ] **Reactivación >24 h** — crear/aprobar plantilla de WhatsApp para que esos ~59 mensajes se entreguen.
- [ ] **Precio** — dejar de adelantarlo cuando nadie lo pregunta.
- [ ] **graphify** — definir de dónde sale (ver §7) o descartarla.

---

## 7. Herramientas: qué hay y qué no

**Disponible en las sesiones de Claude:** Google Drive / Gmail / Calendar, GitHub (solo `luisedulive/rese-as-dr-borja`, **y solo lectura** — ver abajo), Windsor.ai (tiene conector de GoHighLevel, Meta Ads, GA4 — no probado aún para Carla), Zapier, Higgsfield.

**GitHub es de solo lectura.** La Claude GitHub App está instalada pero sin permiso de escritura: `git push` devuelve 403 y la API responde `Resource not accessible by integration`. En GitHub solo existe la rama `main`. Los commits de las sesiones se quedan en el contenedor hasta que alguien dé permiso de escritura en https://github.com/apps/claude/installations/select_target.

**NO disponible: `graphify`.** Verificado en los cuatro sitios posibles — skills de claude.ai, catálogo de plugins de la cuenta, registro de conectores MCP y skills guardadas de Zapier. **No existe y no se puede instalar.** Vive fuera de Claude. Ya se perdió tiempo con esto en dos sesiones (también el 27/08 en *Davinci TouchApp*): si te la piden, no vuelvas a buscarla — pide directamente (a) el export del grafo, (b) la URL/API de graphify, o (c) el `owner/repo` del marketplace si es un plugin de GitHub.

**Artefactos publicados relacionados:**
- Estado de Carla (este documento, como página) — https://claude.ai/code/artifact/c338ed97-2a0a-47df-89e6-c8eb324eba30
- Embudo de Carla (análisis completo de agosto, fuente de casi todo este documento) — https://claude.ai/code/artifact/2b4e7d9c-55e1-4369-8f7e-6b765430bd97
- Bot de reseñas v2 — https://claude.ai/code/artifact/111aa9a5-87ba-488e-b382-6a8c32cca396
- Encuesta y Reseñas Dr. Borja — https://claude.ai/code/artifact/aa04edb7-16c3-4f26-b6df-21ef1b23d3f0
- Carta de la Ley General de Salud — https://claude.ai/code/artifact/e631227b-2c3a-4567-976c-2fa05b154f85

**Datos en Drive:**
- `Pacientes Dr. Borja` (Sheets, del usuario) — `1llnsPWgV7js9ggaSle0Uy_KnsELySaTZOuO9HBq5aiw`
- `Datos` (Sheets, compartido por reylaser100@gmail.com) — `11wcZ8BcbvCT71Z4tjATKjkFc_ApQvwjgWR_aKNAVe4U`

---

## 8. Cómo actualizar este archivo

Al cerrar una sesión de trabajo sobre Carla:

1. Cambia la fecha de **Última actualización** (usa la fecha real del sistema: `date`, no la que recuerdes).
2. Mueve a §4 lo que se confirmó, y a §5 lo que se desmintió — con fecha y quién lo confirmó.
3. Tacha o quita de §6 lo cerrado; agrega lo nuevo.
4. Anota en §7 toda herramienta que resultó no existir o no servir, para que la próxima Claude no repita la búsqueda.
5. Commit en la rama de trabajo, y republica el artefacto de §7 para que coincida.

Regla: **si una cifra no tiene fuente, no entra.** Marca las proyecciones con `≈` y di sobre qué muestra se proyectaron. Un arreglo **reportado** no es un arreglo **verificado**: dilo distinto hasta que los datos lo confirmen.
