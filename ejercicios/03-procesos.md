# 03 - Procesos (ejercicios)

## Teóricos

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

### [Cuestionario 04-18] Respuestas

<details>
<summary>Ver respuesta</summary>

- La PCB siempre está en RAM
- Con fork cambia el PID, el parent (PPID), el retorno de fork(), etc.
- Si ocurre un cambio de proceso, hay al menos 2 cambios de contexto y más de un cambio de modo
- Si ocurre un cambio de contexto, puede no haber cambio de proceso (interrupciones, syscalls)
- Si ocurre un cambio de modo, hay un cambio de contexto. Puedo cambiar de contexto sin cambiar de modo
- Procesos: al finalizar liberan sus recursos; por defecto no comparten memoria (ni con el padre); la unidad mínima de planificación son los hilos; son más estables y confiables que los hilos

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
