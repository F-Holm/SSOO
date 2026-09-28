# 03 - Procesos (ejercicios)

## Teóricos

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Qué partes de la imagen de un proceso deben estar siempre en RAM? (varias opciones)
Opciones:

- Stack
- Heap
- Código
- Datos
- PCB
- Todo debe estar siempre en memoria
- Ninguno necesita estar siempre en memoria

<details>
<summary>Ver respuesta</summary>

- **PCB**
- Comentario de la cátedra: el PCB siempre tiene que estar en memoria porque es la estructura que usa el SO para administrar al proceso en todo momento. El resto de la imagen puede pasarse a swap (suspender el proceso); con memoria virtual se podrá mandar a swap "de a partes".

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] Si un proceso A ejecuta fork() creando un proceso B, inmediatamente luego de la llamada lo único que cambiará en ambas imágenes es el PID
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Falso**
- Comentario de la cátedra: también cambia el resultado de la llamada a fork() (guardado en una variable: en el stack si es local o en datos si es global). Eso permite distinguir si ejecuta el padre o el hijo.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuáles de las siguientes afirmaciones son correctas? (varias opciones)
Opciones:

- Si ocurre un cambio de proceso ⇒ va a ocurrir más de un cambio de contexto
- Si ocurre un cambio de proceso ⇒ va a ocurrir más de un cambio de modo
- Si ocurre un cambio de contexto ⇒ va a ocurrir un cambio de proceso
- Si ocurre un cambio de modo ⇒ va a ocurrir un cambio de proceso
- Si ocurre un cambio de modo ⇒ va a ocurrir un cambio de contexto
- Si ocurre un cambio de contexto ⇒ va a ocurrir un cambio de modo

<details>
<summary>Ver respuesta</summary>

- **Correctas**: cambio de proceso ⇒ más de un cambio de contexto; cambio de proceso ⇒ más de un cambio de modo; cambio de modo ⇒ cambio de contexto
- Comentario de la cátedra: al cambiar de proceso ejecuta en el medio el planificador: usuario → kernel → usuario, y en cada paso cambia el contexto. No es cierto que cambiar de contexto implique cambiar de modo (interrupción anidada: ya está en kernel), ni que cambiar de contexto o de modo implique cambiar de proceso (una syscall, o atender una interrupción y volver al mismo proceso).

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuáles de las siguientes afirmaciones sobre procesos son FALSAS? (varias opciones)
Opciones:

- Al finalizar, se liberan los recursos que tenía asignados
- Por default comparten memoria con otros procesos para poder comunicarse
- Por default comparten memoria con su proceso padre para poder comunicarse
- Es la mínima unidad de planificación para el SO
- Posee un PCB que siempre debe estar en RAM
- Pueden comunicarse con otros procesos con paso de mensajes
- Son menos estables y seguros que los hilos (KLTs)
- Ninguna

<details>
<summary>Ver respuesta</summary>

- **Falsas**: comparten memoria con otros procesos por default; comparten memoria con su padre por default; es la mínima unidad de planificación; son menos estables y seguros que los KLTs
- Comentario de la cátedra: los procesos son independientes: no comparten memoria de forma inherente (hace falta memoria compartida vía syscall o paso de mensajes). Al finalizar el SO libera sus recursos. El PCB siempre está en memoria aunque el proceso esté suspendido. Por su aislamiento son más estables y seguros que los KLTs del mismo proceso. La mínima unidad de planificación (si el SO soporta hilos) es el KLT.

</details>

### [Resumen] ¿Quiénes pueden crear o finalizar procesos? ¿Cómo? ¿Qué le pasa al hijo si el padre finaliza inesperadamente?

<details>
<summary>Ver respuesta</summary>

- Los crea el SO u otro proceso, siempre por syscall. El SO: asigna PID, reserva espacio para estructuras, inicializa la PCB, la ubica en las colas de planificación
- Finalizan por exit (normal/anormal), kill del SO u otro proceso, abort del padre
- Si el padre finaliza, el hijo puede seguir ejecutando (en Linux lo adopta init)

</details>

### [Resumen] ¿Qué comparten un proceso padre y un hijo? ¿Y dos hilos del mismo proceso?

<details>
<summary>Ver respuesta</summary>

- Padre e hijo: por defecto no comparten memoria (el hijo es una copia); el hijo conoce el PPID (y hereda los archivos abiertos)
- Dos hilos: comparten PCB, código, datos, heap y recursos (cada uno tiene su stack y TCB)

</details>

### [Resumen] Ejemplos de transiciones

<details>
<summary>Ver respuesta</summary>

- Running → Ready: fin de quantum (clock) o desalojo por llegada de uno de mayor prioridad
- Susp/Ready → Ready: baja el grado de multiprogramación y se trae un proceso de disco a RAM
- Ready → Exit: se lo mata sin que estuviera ejecutando
- Ready → Blocked: no existe

</details>

### [Final 2024-02-27] Dentro de una aplicación de varios procesos monohilo, en un sistema monoprocesador, no hay concurrencia — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Hay concurrencia: los procesos están activos en el mismo intervalo de tiempo aunque haya un solo procesador (oficial). No hay paralelismo

</details>

### [Final 2024-07-30] Los procesos pueden cambiar del estado bloqueado a listo sólo si se ejecuta el planificador de corto plazo — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. El pasaje bloqueado → listo lo hace el SO al atender la interrupción de fin de evento (fin de E/S, signal de un semáforo). El planificador de corto plazo decide listo → ejecutando

</details>

### [Final 2025-07-15] Si se quiere priorizar seguridad y estabilidad, es mejor una arquitectura multiproceso antes que multihilo — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Los procesos están aislados: la falla o el memory leak de uno no afecta a los demás; los hilos comparten memoria y un hilo puede tirar abajo todo el proceso

</details>

### [Final 2026-02-24] Evaluar la respuesta del LLM — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- Afirmación: "Una arquitectura multi-proceso será siempre recomendable sobre una multi-hilo"
- LLM: "¡Exacto! Da mayor estabilidad y además mejora el rendimiento por la velocidad de creación y facilidad de comunicación entre procesos"
- **Incorrecta**. Los procesos son más seguros y estables, pero los hilos son más rápidos de crear y se comunican sin intervención del SO (oficial)

</details>

---
[⬆ Volver al índice de ejercicios](00-indice.md) · [Índice general](../README.md)
