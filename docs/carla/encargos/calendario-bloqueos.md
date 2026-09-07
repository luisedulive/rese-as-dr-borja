# Encargo: Carla debe respetar el calendario del doctor

**Proyecto A — Prevención.** Que Carla no ofrezca ni agende en días u horas en que el doctor no atiende.

---

## 1. Contexto del sistema

Carla: Node.js en Railway + GoHighLevel (GHL) + OpenAI Responses API.
Fuente de verdad: rama `main` / carpeta local. **No trabajar sobre scratchpads.**
Este cambio **se suma** a `fix/citas-fantasma` (ya mergeado), no lo reemplaza.

---

## 2. Lo ya verificado — no lo vuelvas a averiguar

Comprobado el 5–6 de septiembre de 2026 leyendo Google Calendar directamente. Son datos, no supuestos:

**a) El calendario donde caen las citas es el PRINCIPAL de la cuenta.** Todas las citas creadas por GHL, y también las actividades propias del doctor, están en `luisedulive@gmail.com` — el calendario primario de esa cuenta de Google, no un calendario aparte. Esto es central para §4: **su id es la dirección de correo y no cambia nunca**, y **es el destino por defecto de todo evento que el doctor cree sin elegir calendario.**

**b) El calendario "Agendamiento de Citas" está vacío.** Existe un calendario llamado *Dr. Luis Borja* / descripción *"Agendamiento de Citas"* (`ca4d3be6dee7217de06259b53511e28497627557c7491b1898a9dae22ac76fc9@group.calendar.google.com`). **Cero eventos** — consultado agosto, septiembre y sin filtro de fechas.

**c) Las citas de Carla son distinguibles.** Las que crea GHL traen esta descripción:

```
Phone:- 971 324 907
Email:-
Nombre:- Angie Castillo Pangalima
Reschedule:- https://msgsndr.com/widget/booking/28Gw88ixIkqf6V4uo5EY?event_id=TDXEidNA16tg0nTpJawH
Cancel:-     https://msgsndr.com/widget/cancel-booking?event_id=TDXEidNA16tg0nTpJawH
```

y el título `🆕 NUEVO - Cita - Dr. Borja` (variante: `🔵 OPERADO - Cita - Dr. Borja`).
El id `28Gw88ixIkqf6V4uo5EY` aparece en **todas** las citas del bot → es, con alta confianza, el calendar id de GHL. **Dos dominios según la fecha:** `api.chatpartner.online` hasta el ~23 de agosto, `msgsndr.com` desde el ~24. Parsea los dos.

**d) Los eventos del doctor no traen ese patrón.** Título libre: `Otoplastia Postigo`, `Guardia`, `Atención dental`, `Consulta Carlos Enrique Campos V.`, `Virtual Magno Cubas Medina 954152192`. Varios de esos **también son pacientes**, agendados a mano por el doctor: ocupan slot igual.

**e) Las citas duran 30 minutos**, casi siempre entre las 16:30 y las 19:30.

---

## 3. El problema, con el caso real

Carla ofrece y agenda sin mirar la disponibilidad real del doctor.

**Caso verificado — 3 de septiembre, Angie Castillo Pangalima (971 324 907):**

| | |
|---|---|
| Cita creada | 1 sep 19:52 (Lima) · evento Google `9eag9ua8t3km36io3ig3g7p9ek` · GHL `TDXEidNA16tg0nTpJawH` |
| Para | jueves 3 sep, 19:00–19:30 |
| Ese día el doctor tenía | `Guardia`, evento de todo el día |

La cita **sí se creó** (no fue cita fantasma). El fallo es otro: se creó **en un día bloqueado**. Según lo reportado, en la misma conversación Carla llegó a ofrecer también las 5 pm de ese día.

**El detalle que decide el diseño:** el `Guardia` está marcado en Google como **`transparency: transparent` / `AVAILABILITY_FREE`**, o sea "libre". Cualquier implementación que consulte free/busy (`freeBusy`, `availability`) **no lo verá como ocupado**. La regla tiene que ignorar cómo esté marcado el evento y mirar solo si existe.

También hay casos reportados de reagendamiento donde no se borra la cita anterior, dejando duplicados.

---

## 4. El modelo de calendarios

### 4.1 Lo que decidió el doctor

- Renombrar el calendario actual a **"Carla Citas"** — sin migrar nada ni reconfigurar GHL, que sigue escribiendo donde siempre.
- Crear un **calendario nuevo y personal** para sus bloqueos: guardias, vacaciones, cirugías, cursos, enfermedad.

### 4.2 El problema de ese modelo — léelo antes de ejecutarlo

El calendario que se renombraría **es el primario de la cuenta** (§2a). Eso trae dos consecuencias que rompen la separación:

1. **Sigue siendo el destino por defecto.** Todo evento que el doctor cree sin elegir calendario a mano —desde el celular, desde Gmail, aceptando una invitación— cae en "Carla Citas". Google **no permite** poner un calendario secundario como destino por defecto. La separación pasa a depender de que el doctor se acuerde de cambiar el calendario **cada vez**.
2. **Cuando se olvide, el fallo es silencioso y es exactamente el que estamos arreglando:** el bloqueo queda en "Carla Citas", Carla solo lee el personal nuevo, no lo ve, y agenda encima. Igual que con Angie.

Además queda un calendario llamado "Carla Citas" lleno del historial personal del doctor (`Atención dental`, `Otoplastia Postigo`…), que confunde a cualquiera que lo abra.

### 4.3 La alternativa recomendada: invertir

En vez de renombrar, **mover a Carla**: reconfigurar GHL para que escriba en el calendario "Agendamiento de Citas" que ya existe y está vacío (§2b).

- El doctor **no cambia ningún hábito**: sus eventos siguen cayendo por defecto en su calendario de siempre, que pasa a ser el de bloqueos. La regla trabaja **a favor** de los defaults, no contra ellos.
- Cuesta **un cambio de configuración en GHL**, una sola vez, contra "acordarse para siempre".
- El historial se queda quieto: nada se migra.

**Recomendación: invertir.** Si aun así se elige renombrar, que sea con la regla de §4.4, que aguanta el olvido.

### 4.4 La regla que funciona en los dos escenarios

**Cualquier evento, en cualquiera de los dos calendarios, bloquea ese tiempo.**

Esto es más simple que discriminar y más seguro que leer un solo calendario:

- No hay que clasificar eventos ni parsear URLs de descripción — esa fragilidad desaparece.
- Si el doctor pone una guardia en el calendario equivocado, **igual bloquea**. El olvido de §4.2 deja de ser un fallo silencioso.
- Es correcta por definición: una cita de otro paciente **también** ocupa el slot, así que no hay caso en que un evento deba ignorarse.

Escríbela como una función que recibe **una lista de calendarios a vigilar**, no dos ids fijos. Así, si mañana se invierte el modelo o se agrega un tercer calendario, se cambia una constante y nada más.

### 4.5 Histórico y eventos recurrentes

Los bloqueos pasados no importan: Carla solo agenda hacia adelante (ventana de 14–21 días). Pero dos cosas sí:

- Los bloqueos que ya estén anotados **dentro de la ventana** (próximas ~3 semanas). Con la regla de §4.4 quedan cubiertos aunque no se muevan.
- **Los eventos recurrentes.** Una guardia semanal creada en el calendario viejo sigue generando instancias futuras dentro de la ventana **para siempre**. Hay que moverlos aunque la instancia de esta semana ya haya pasado.

### 4.6 Qué NO corrige este fix

Las citas ya creadas en días bloqueados (Angie el 3 sep) **no se migran ni se corrigen solas**: quedan para revisión manual del doctor o para el futuro Revisor (§11). Esto previene errores nuevos, no limpia el pasado.

---

## 5. Tareas del doctor (en este orden)

Nada de esto lo hace el código; sin esto, el código no sirve. Y el paso 3 es el que suele olvidarse y bloquea todo.

1. Crear el calendario nuevo de bloqueos (si se va por §4.3, en vez de esto: reconfigurar GHL para que escriba en "Agendamiento de Citas").
2. Mover ahí los bloqueos de las próximas ~3 semanas **y todos los recurrentes** (§4.5).
3. **Compartir el/los calendario(s) con la credencial que use Carla en producción**, con permiso de solo lectura. Si se usa una service account, hay que compartírselo a su dirección de correo — de lo contrario la API responde 404 y parece un bug del código.
4. Confirmar con quien implemente **antes** de renombrar nada (§7.3).

---

## 6. Las reglas, exactas

Un slot se ofrece **solo si**:

1. **No solapa ningún evento** de los calendarios vigilados (§4.4), sin importar tipo ni cómo esté marcado (libre / ocupado / transparent).
2. **No solapa otra cita de paciente** ya existente.

Precisiones que evitan bugs:

- **Evento de todo el día** (guardia, vacaciones) → bloquea **el día completo** en hora de Lima.
- **Evento con hora** (cirugía de 8 a 12) → bloquea ese rango.
- **Eventos recurrentes.** Pide las instancias expandidas (`singleEvents: true` en la API de Google) y ordénalas por fecha. Sin eso la API devuelve el evento maestro y **todas las repeticiones futuras se pierden** — una guardia semanal quedaría invisible. Este es el error más fácil de cometer aquí.
- **Cancelados o rechazados NO bloquean.** Hay eventos con `status: "cancelled"` y con `attendees[].responseStatus: "declined"` — uno real el 5 de septiembre (Carlos Andrés Montalvo Zambrano, declinado). Sin filtrarlos, Carla arrastra bloqueos que ya no existen.
- **Zona horaria.** Todo se compara en `America/Lima` (UTC−5, sin horario de verano). Los eventos de todo el día llegan como fecha sin hora y con **fin exclusivo**: el `Guardia` del 3 de septiembre vino como `start.date = 2026-09-03`, `end.date = 2026-09-04`, y significa *solo el 3*. Convertirlo a UTC ingenuamente corre el bloqueo medio día. Aquí es donde se cuela el bug.
- **Rango a consultar:** el mismo que la ventana de reserva (14–21 días). Una lectura por conversación es suficiente; **no** una por mensaje.

**Doble validación:**
- Al **ofrecer**: la lista de horarios ya sale filtrada.
- Al **agendar**: revalidar justo antes de crear, aunque el slot se haya ofrecido hace minutos. Si ya no está libre → **no crear** y usar el mensaje suave que ya existe para "ese horario no me quedó confirmado".

**Si la lectura del calendario falla** (API caída, timeout, credencial vencida) — decidido, no al azar:
- Al **agendar**: **fallar cerrado.** No crear la cita a ciegas; decir al paciente que se confirma en un momento. Crear sin poder verificar es el error que se está arreglando.
- Al **ofrecer**: no inventar disponibilidad. Pedir que espere la confirmación.
- Registrar el fallo de forma visible en logs: si pasa seguido, el doctor tiene que enterarse.

**Si el día pedido está bloqueado**, Carla ofrece el siguiente disponible en vez de decir solo que no. No hace falta explicarle al paciente por qué el doctor no atiende.

**Reagendamiento:** respetar los bloqueos **y eliminar la cita anterior**. Dos cuidados:
- Guardar el identificador de la cita vieja **antes** de crear la nueva.
- **Cancelarla por GHL, no borrando el evento en Google.** La fuente de verdad es GHL: borrar el evento del lado de Google puede dejar la cita viva en GHL, con sus links de reschedule/cancel activos, y el sync puede recrearla. Si el borrado falla, que no pase en silencio.

---

## 7. Antes de codear: el plan primero

No implementes hasta responder por escrito:

1. **¿Cómo obtiene hoy Carla los horarios que ofrece?** Revisa `slots.detector.js` y de dónde saca los slots de GHL. ¿Consulta solo el calendario de citas, o algo más?
2. **¿Por qué vía lee Carla EN PRODUCCIÓN los calendarios de bloqueo?** Distinción clave: que se haya podido leer el calendario para *verificar* el caso de Angie desde un entorno de trabajo **no** es lo mismo que el servidor en Railway leyéndolo *automáticamente, 24/7, en cada agendamiento*. ¿GHL lo expone, o hay que integrar la API de Google Calendar aparte —credenciales, OAuth o service account, alcance de solo lectura, dónde viven los secretos en Railway—? **Esto decide la viabilidad y puede ser un sub-proyecto: dilo antes de implementar.**
3. **¿Cómo está conectado GHL con Google Calendar hoy?** ¿Sincronización de dos vías, o GHL empuja a Google? De esto depende si cancelar por GHL borra el evento de Google, y si borrar en Google cancela en GHL (§6, reagendamiento). Verifícalo, no lo asumas.
4. **§4.3 o §4.1: ¿invertir o renombrar?** Con la razón. Si se renombra: confirma que el id del calendario no cambia (no debería: el primario se identifica por la dirección de correo) y que GHL sigue escribiendo igual. **Si esto no se puede confirmar, el doctor no debe renombrar todavía.**

---

## 8. Alcance

**Dentro:** filtrado de slots al ofrecer, revalidación al agendar, cancelación de la cita anterior al reagendar, la función pura de solapamiento con sus tests.

**Fuera:** el Revisor de citas (§11). Cambiar el tono o los textos de Carla más allá del mensaje de "no me quedó confirmado" que ya existe. Tocar recordatorios, ruteo de confirmación, clasificación de pacientes o detección de comprobante/audio — **todo eso está vivo y no se rompe.**

---

## 9. Calidad y pruebas

La lógica de *"¿este slot choca con un bloqueo?"* va como **función pura**, sin red ni I/O: recibe el slot y la lista de eventos, devuelve si está libre. El servicio solo trae los datos.

Tests obligatorios — varios salen de casos reales:

1. Slot en un día con evento de **todo el día** marcado como **libre/transparent** → bloqueado. *(el caso Angie: 3 sep 19:00 con `Guardia`)*
2. Slot que solapa un evento **con hora** → bloqueado.
3. Slot sin nada alrededor → libre.
4. **Borde de medianoche en Lima**: un evento de todo el día del 3 de septiembre no bloquea ni el 2 a las 23:00 ni el 4 a las 00:30.
5. Evento **cancelado o declinado** → **no** bloquea.
6. Slot ocupado por otra cita de paciente → bloqueado.
7. Evento que empieza justo cuando el slot termina (19:30 con slot 19:00–19:30) → **no** bloquea. Bordes cerrados/abiertos explícitos.
8. **Instancia de un evento recurrente** dentro de la ventana → bloquea. *(la guardia semanal de §4.5)*
9. **Bloqueo puesto en el calendario "equivocado"** → bloquea igual. *(la garantía de §4.4; si este test falla, el olvido del doctor vuelve a ser un fallo silencioso)*

`node --check` sobre lo tocado y la suite en verde. Rama propia, commits en español, PR pequeño.

---

## 10. Entregables y criterio de aceptación

1. **Plan** (§7) — antes de tocar código.
2. **Implementación**, tras aprobación del plan.
3. **Prueba en vivo:** poner un evento de prueba (ej. `Guardia`) en el calendario de bloqueos un día cualquiera dentro de la ventana; pedirle horarios a Carla para ese día y verificar que **no ofrece ninguno**; forzar un agendamiento y verificar que **no crea la cita**. Repetir poniendo el evento en el **otro** calendario: debe bloquear igual. Y un reagendamiento comprobando que la cita anterior queda cancelada en GHL **y** desaparece de Google.

**Está hecho cuando:** un día con guardia no se ofrece ni se agenda —esté el bloqueo en el calendario que esté—, ningún reagendamiento deja duplicados, y los 9 tests de §9 pasan.

---

## 11. Nota sobre el futuro (Proyecto B — NO ahora)

Después de esta prevención viene un **"Revisor de citas"**: agente independiente que corre a las 6 am, lee las conversaciones de GHL de las citas de los próximos días y reporta al doctor por WhatsApp (996047938) los errores que se hayan escapado, en formato estructurado pensado para que a futuro otro agente lo consuma.

**No construir ahora.** Se menciona para que las decisiones de hoy no lo compliquen después: si la lectura de calendarios y la función de solapamiento (§4.4, §9) quedan reutilizables, el Revisor podrá usar las mismas.
