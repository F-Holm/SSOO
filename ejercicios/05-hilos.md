# 05 - Hilos (ejercicios)

Los prácticos con Gantt de ULTs/KLTs están en [04-planificacion.md](04-planificacion.md).

## Teóricos

### [Resumen] V/F: Usando semáforos en hilos ULT no se requiere cambio de modo para wait/signal

<details>
<summary>Ver respuesta</summary>

- **F**. wait y signal (del SO) son syscalls → cambio a modo kernel aunque sean ULTs

</details>

### [Resumen] V/F: Para compartir memoria entre procesos o entre KLTs se necesita intervención del SO; entre ULTs no

<details>
<summary>Ver respuesta</summary>

- **F**. Tanto KLTs como ULTs comparten memoria (variables globales, heap) sin intervención del SO. Entre procesos sí se necesita

</details>

### [Resumen] V/F: Usar KLTs en lugar de procesos, a pesar de ser más rápido su switch, puede generar memory leaks

<details>
<summary>Ver respuesta</summary>

- **V**. Los KLTs comparten el heap: lo que un hilo no libera queda ocupado hasta que termina todo el proceso

</details>

### [Resumen] Ventajas y desventajas de ULTs sobre KLTs

<details>
<summary>Ver respuesta</summary>

- + Velocidad en create/switch, planificador propio, portabilidad, bajo overhead (sin syscalls ni cambios de modo)
- − Sin paralelismo entre hilos del proceso; sin jacketing una syscall bloqueante bloquea a todo el proceso

</details>

### [Resumen / Final 2022-08-02] V/F: Los KLTs no pueden ocasionar memory leaks, dado que el TCB no tiene referencia al heap del proceso — 📘 respuesta del Resumen SO (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Aunque el TCB no apunte al heap, los KLTs lo comparten: si piden memoria y no la liberan hay memory leak

</details>

### [Resumen] V/F: Los KLT de un proceso pueden competir por el procesador; los ULT de un mismo proceso no

<details>
<summary>Ver respuesta</summary>

- **F**. Los ULTs compiten por ser elegidos por el planificador de la biblioteca (los KLTs por el del SO)

</details>

### [Resumen] Operaciones al cambiar de un KLT a otro del mismo proceso

<details>
<summary>Ver respuesta</summary>

- Se guarda PC y flags en el stack del sistema → el SO guarda el resto de los registros → decide el thread switch → mueve el contexto a la TCB → elige el próximo hilo → carga su TCB y vuelve a modo usuario
- No se tocan punteros a código/datos/heap (son del proceso)

</details>

### [Resumen] Compare procesos, KLTs y ULTs en overhead, multiprocesamiento y protección

<details>
<summary>Ver respuesta</summary>

| | Overhead | Multiprocesamiento | Protección |
|---|---|---|---|
| Procesos | Alto | Sí | Aislados por el SO |
| KLTs | Medio-alto (todo pasa por el SO) | Sí | Hilos manejados por el SO |
| ULTs | Bajo (el SO no los conoce) | No | Manejados por la biblioteca |

</details>

### [Resumen] ¿Cuándo convienen ULTs? Dos atributos del TCB

<details>
<summary>Ver respuesta</summary>

- Planificación propia, bajo overhead, portabilidad, operaciones de hilos rápidas
- TCB: TID, estado (también prioridad, contexto: PC, registros, puntero al stack)

</details>

### [Resumen] V/F: Usando jacketing en la biblioteca de ULTs, es lo mismo usar ULTs que KLTs

<details>
<summary>Ver respuesta</summary>

- **F**. El proceso no se bloquea, pero los ULTs siguen invisibles para el SO (sin paralelismo) y con otra planificación

</details>

### [Cuestionario 04-18] Respuestas

<details>
<summary>Ver respuesta</summary>

- Los ULTs se crean a nivel usuario sin necesidad de syscalls
- Crear y switchear ULTs del mismo KLT es más liviano que entre KLTs
- Jacketing SIMULA paralelismo
- Los KLTs no tienen que usar todos las mismas bibliotecas; si uso syscalls en ULTs conviene usar los wrappers
- Si se bloquean todos los ULTs de un KLT, suele devolver la CPU

</details>

### [Cuestionario 04-18] Un ULT realiza una operación bloqueante (E/S). ¿Cuáles de estas afirmaciones podrían ser ciertas?
Opciones:

- a) Por default se bloquea el KLT (a menos que se use jacketing)
- b) Podría bloquearse el KLT y también el proceso asociado
- c) Podría no bloquearse el KLT si se usa jacketing
- d) Al finalizar la operación bloqueante se vuelve a ejecutar el mismo ULT
- e) Al finalizar la operación bloqueante se ejecuta otro ULT

<details>
<summary>Ver respuesta</summary>

- **Todas pueden ser ciertas**
- d) pasa si se hizo la syscall directa (la biblioteca no replanifica); e) si se usó el wrapper de la biblioteca y replanificó eligiendo otro ULT

</details>

### [Final 2022-09-07] Los procesos con KLTs pueden ejecutar en modo kernel y los que usan ULTs en modo usuario — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Todos los procesos ejecutan en modo usuario; la diferencia es quién provee la biblioteca de hilos (oficial)

</details>

### [Final 2022-12-21] Los permisos de un hilo sobre archivos son iguales para todos los hilos del proceso — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Los permisos sobre archivos abiertos son a nivel proceso (tabla de archivos abiertos por proceso) (oficial)

</details>

### [Final 2023-02-28] En un servidor que usa una biblioteca externa con posibles memory leaks, lo ideal es crear un hilo por solicitud para mitigarlo — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Los hilos comparten el heap: el leak se acumula en el proceso. Conviene un **proceso** por solicitud (o grupo) para que al terminar se libere todo (oficial)

</details>

### [Final 2023-03-07] ULTs con jacketing tienen los mismos beneficios que KLTs, solo que son más portables — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Son más portables, pero los KLTs permiten ejecución en paralelo y los ULTs no (oficial)

</details>

### [Final 2023-04-25] El problema de los ULTs de no permitir multiprocesamiento se soluciona con jacketing — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Jacketing evita que se bloquee el proceso/KLT, pero no permite ejecutar ULTs en paralelo (oficial)

</details>

### [Final 2023-08-01] Con dos ULTs en un mismo KLT, ULT2 puede modificar el heap de ULT1 pero no su stack — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. El heap es compartido; el stack es propio de cada hilo (oficial)

</details>

### [Final 2023-09-27] El uso de wait y signal requiere cambios de modo, tanto en KLTs como en ULTs — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Son syscalls del SO (salvo que la biblioteca de ULTs implemente sus propios semáforos en modo usuario)

</details>

### [Final 2023-12-12] Una syscall es necesaria para comunicar dos hilos KLTs — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Comparten memoria (datos, heap): se comunican escribiendo/leyendo variables compartidas sin el SO (para sincronizar sí pueden usar semáforos)

</details>

### [Final 2024-03-05] Si dos hilos del mismo proceso operan el mismo archivo, los resultados podrían no ser consistentes, sean ULTs o KLTs — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Los archivos abiertos son del proceso y comparten las estructuras (puntero de posición) → condición de carrera si no usan locks (oficial)

</details>

### [Final 2025-02-11 / 2025-12-09] Ciertas implementaciones de hilos permiten algoritmos de planificación no soportados por el SO — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Las bibliotecas de ULTs planifican con su propio algoritmo

</details>

### [Final 2025-05-20] Dos ULTs del mismo proceso pueden ejecutar en distintos procesadores si se usa jacketing — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. El SO solo ve al KLT: los ULTs nunca ejecutan en paralelo, con o sin jacketing

</details>

### [Final 2025-07-29] Se pueden usar hilos sólo si el SO implementa procesos multihilo — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Los ULTs se implementan con una biblioteca en espacio de usuario sin soporte del SO

</details>

### [Final 2025-09-25] Jacketing podría permitir que dos ULTs del mismo KLT ejecuten en paralelo con 2+ procesadores — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Mismo motivo: el KLT es la unidad que el SO asigna a una CPU

</details>

### [Final 2025-12-09] Los KLTs de un mismo proceso solo comparten los Datos — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Comparten código, datos, heap y recursos (archivos, PCB); solo el stack y el TCB son propios

</details>

### [Final 2025-12-16] Es posible invocar fork() desde un ULT — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Los ULTs pueden hacer syscalls: el KLT/proceso hace el pedido al SO (oficial)

</details>

### [Final 2023-07-25 N°1] ChatGPT, desventajas de ULTs vs KLTs: 1) falta de soporte multiprocesador, 2) bloqueo por llamadas al sistema, 3) mayor latencia de planificación porque para planificar hay que pasar a modo kernel. Evaluar — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- 1) Correcta (en realidad nunca, no "generalmente": el SO no puede asignar ULTs a distintas CPUs)
- 2) Correcta pero incompleta: solo si no se usa jacketing
- 3) **Incorrecta**: la planificación de ULTs la hace la biblioteca en modo usuario, sin cambio de modo → es justamente una ventaja (menor latencia/overhead)

</details>
