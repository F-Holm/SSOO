# 04 - Planificación (ejercicios)

Convención de los Gantt: columna `t` = intervalo [t, t+1). `X` = CPU, `io` = E/S, `F` = terminó (en el instante de esa columna).

**Desempate** (en todos los Gantt): 1° el proceso en ejecución (viene de CPU: fin de quantum/desalojo), 2° el que vuelve de E/S, 3° el nuevo. Dentro de la misma categoría, FIFO.

## Teóricos

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuáles de los siguientes planificadores impactan o modifican con sus acciones el grado de multiprogramación? (varias opciones)
Opciones:

- Planificador de corto plazo
- Planificador de mediano plazo
- Planificador de largo plazo

<details>
<summary>Ver respuesta</summary>

- **Mediano y largo plazo**
- Comentario de la cátedra: el de largo plazo, al admitir procesos, aumenta el grado de multiprogramación; el de mediano plazo, al mandar procesos a swap o traerlos, lo decrementa o incrementa.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuál es el planificador que es más importante que tenga menos overhead?
Opciones:

- El encargado de admitir nuevos procesos al sistema
- El encargado de hacer swapping
- El encargado de poner procesos en ejecución

<details>
<summary>Ver respuesta</summary>

- **El encargado de poner procesos en ejecución** (corto plazo)
- Comentario de la cátedra: se ejecuta muy seguido, así que tiene que decidir bien y con el menor overhead posible.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Qué es el tiempo de espera?
Opciones:

- El tiempo en que el proceso está en la cola de bloqueados
- El tiempo en que el proceso no está en ejecución
- El tiempo en que el proceso está en la cola de listos
- El tiempo en que el proceso está suspendido

<details>
<summary>Ver respuesta</summary>

- **El tiempo en que el proceso está en la cola de listos**
- Comentario de la cátedra: es el tiempo que le negamos la CPU: podría haber sido elegido pero el planificador eligió a otro.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuál/es de las siguientes afirmaciones son FALSAS sobre FIFO? (varias opciones)
Opciones:

- Podría permitir que un proceso monopolice la CPU
- Podría ser útil para correr procesos secuenciales
- Minimiza los cambios de contexto
- Todas
- Ninguna

<details>
<summary>Ver respuesta</summary>

- **Ninguna** (todas las afirmaciones son verdaderas)
- Comentario de la cátedra: al no tener desalojo un proceso podría no liberar nunca la CPU; para correr un lote de procesos minimizando overhead es buena opción; ejecutar uno tras otro minimiza la replanificación y los cambios de contexto.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuáles de las siguientes afirmaciones son correctas sobre SJF? (varias opciones)
Opciones:

- Puede implementarse con o sin desalojo
- Minimiza el tiempo de espera promedio
- Para poder optimizarlo se puede utilizar un promedio ponderado (media exponencial)
- Prioriza a los procesos CPU bound
- Es un algoritmo con poco overhead

<details>
<summary>Ver respuesta</summary>

- **Correctas**: puede implementarse con o sin desalojo (con desalojo = SRT); minimiza el tiempo de espera promedio
- La media exponencial no se toma como correcta porque no sirve para "optimizarlo": es **necesaria** para implementarlo (hay que estimar las ráfagas). Prioriza a los IO bound (no CPU bound) y tiene bastante overhead (más con desalojo)

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] HRRN podría implementarse con desalojo sin mayores desventajas
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Falso**
- Comentario de la cátedra: como el response ratio depende del tiempo de espera, con desalojo habría que recalcularlo para todos los listos en cada evento de replanificación → muchísimo overhead.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] En RR el SO lanza una interrupción para desalojar al proceso en ejecución y selecciona al siguiente proceso en LISTO
Opciones:

- Verdadero
- Falso

> ⚠️ **Error en el apunte (04-18):** ahí figuraba como respuesta "RR: lanza una interrupción para desalojar al proceso en ejecución y selecciona al siguiente en LISTO" (Verdadero). Según la cátedra es Falso: la interrupción la lanza el timer.

<details>
<summary>Ver respuesta</summary>

- **Falso**
- Comentario de la cátedra: no es el SO el que interrumpe sino el timer (programado por el planificador antes de poner a ejecutar al proceso), que lanza la interrupción de fin de quantum. Luego el SO la atiende y el planificador elige a otro proceso.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] Para un algoritmo de tipo "Feedback" no es suficiente saber que tiene dos colas de planificación y que ambas utilizan RR para poder implementarlo
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Verdadero**
- Comentario de la cátedra: también hay que saber a qué cola ingresan los procesos nuevos, cuál es el algoritmo entre colas, si hay desalojo entre ellas y cuál es el criterio para pasar de una cola a otra.

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] ¿Cuáles de los siguientes algoritmos podrían sufrir de inanición? (varias opciones)
Opciones:

- FIFO
- SJF
- Por prioridades
- RR
- VRR
- HRRN
- Feedback

<details>
<summary>Ver respuesta</summary>

- **SJF, por prioridades y Feedback**
- Comentario de la cátedra: FIFO atiende a todos en orden de llegada; SJF puede no elegir nunca a un proceso largo; prioridades, igual pero con la prioridad; RR es FIFO con quantum; VRR parece que podría por su cola prioritaria, pero Q' tiende a 0 y al consumir su Q vuelve a la cola menos prioritaria; HRRN incluye el tiempo de espera en la fórmula; Feedback depende de la configuración (puede haber inanición en una cola o entre colas).

</details>

### [Cuestionario Procesos, Hilos y Planificación (PDF)] Sabiendo que el SO planifica con SJF (α = 0,5) y que el estimado anterior de KAA fue 3, ¿cuáles de las siguientes afirmaciones son verdaderas? (varias opciones)
Estado del sistema (del gráfico del PDF, `X` = CPU, `io` = E/S):

| Proceso | Hilo | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|---|
| PA (KAA) | UAA1 | | X | X | io | io | io | io | |
| | UAA2 | | | | X | FIN | | | |
| | UAA3 | | | | | X | X | | io → (sigue) |
| PA (KAB) | | X | io | | | | | | |
| PB (KB) | | | | | | | | X | X |

Opciones:

- El próximo estimado de KAA será 4
- KAA nunca llega a bloquearse
- KAA utiliza una biblioteca de ULTs con jacketing
- La biblioteca de ULTs de KAA utiliza SJF

<details>
<summary>Ver respuesta</summary>

- **Correctas**: el próximo estimado de KAA será 4; KAA usa una biblioteca de ULTs con jacketing
- Estimado: KAA ejecutó 5 unidades (1 a 6) hasta bloquearse → 0,5 × 5 + 0,5 × 3 = 4
- Sí se bloquea: en t6 todos sus ULTs están bloqueados o terminados, así que el KLT se bloquea (la biblioteca no tiene nada para ejecutar)
- Jacketing: UAA2 y UAA3 ejecutan mientras UAA1 hace E/S
- No se puede afirmar que la biblioteca use SJF: con estos datos también podría ser FIFO

</details>

### [Resumen] Los algoritmos con desalojo, ¿qué eventos tienen en cuenta para replanificar? ¿Por qué los sin desalojo no?

<details>
<summary>Ver respuesta</summary>

- Con desalojo: además de bloqueo/fin, la llegada de procesos a Ready (nuevo, fin de E/S, fin de quantum), porque si llega uno de mayor prioridad se desaloja al actual
- Sin desalojo: una vez asignada la CPU no se quita aunque llegue uno más prioritario

</details>

### [Resumen] Compare HRRN, SJF con y sin desalojo (criterio, desalojo, overhead, starvation, penalización)

<details>
<summary>Ver respuesta</summary>

| | Criterio | Desalojo | Overhead | Starvation | Penaliza |
|---|---|---|---|---|---|
| HRRN | Mayor (W+S)/S | No | Alto | No | Poco a los largos (el aging compensa) |
| SRT | Menor ráfaga restante | Sí | Medio-alto (estimadores) | Sí | Largos |
| SJF | Menor ráfaga | No | Menor entre los SJF | Sí | Largos |

</details>

### [Resumen] ¿Cómo podría el planificador de corto plazo generar una condición de carrera? Solución sin soporte del SO para multiprocesador

<details>
<summary>Ver respuesta</summary>

- Si un proceso está modificando una variable compartida (varias instrucciones) y una interrupción hace que el planificador elija otro que usa la misma variable
- Solución: sincronizar la SC con instrucciones atómicas (test_and_set) → sirven en multiprocesador

</details>

### [Resumen] ¿Es consciente el proceso de quedar bloqueado?

<details>
<summary>Ver respuesta</summary>

- No: se guarda su contexto al bloquearlo y se restaura al volver

</details>

### [Resumen] ¿Cómo afecta el tamaño del quantum? ¿Igual en RR y VRR?

<details>
<summary>Ver respuesta</summary>

- Chico → mucho overhead (el planificador interviene seguido) pero más equidad. Grande → RR se vuelve FIFO
- En VRR es similar, con el agregado de que la cola auxiliar puede crecer mucho

</details>

### [Resumen] Compare FIFO, RR y SJF en equidad, overhead y starvation

<details>
<summary>Ver respuesta</summary>

| | Equidad | Overhead | Starvation |
|---|---|---|---|
| FIFO | No (un proceso largo monopoliza) | Bajo | No (salvo monopolización) |
| RR | Sí (mismo quantum; favorece CPU bound) | Medio | No |
| SJF | No | Alto (comparar ráfagas/estimar) | Sí |

</details>

### [Resumen] Pasos cuando, ejecutando un proceso, llega una interrupción de fin de E/S de otro proceso bloqueado

<details>
<summary>Ver respuesta</summary>

- Llega la interrupción → al final de la instrucción se chequea → se guarda el contexto del actual → cambio a modo kernel → interrupt handler → el SO pasa el proceso bloqueado a ready → si el algoritmo tiene desalojo, se evalúa si el nuevo tiene más prioridad (si es así el actual vuelve a ready) → cambio a modo usuario → se restaura el contexto del que deba ejecutar

</details>

### [Resumen] Starvation: ¿qué es? Dos algoritmos con desalojo que la sufran y cómo solucionarla

<details>
<summary>Ver respuesta</summary>

- Se le niega la CPU indefinidamente porque siempre llegan otros con más prioridad
- SRT: los de ráfaga larga quedan siempre al final → aumentar la prioridad mientras espera (→ HRRN)
- Prioridades con desalojo: prioridades dinámicas (aging): si espera más de x tiempo sube su prioridad

</details>

### [Resumen] V/F: Con desalojo, al llegar una interrupción de fin de E/S de un KLT, será atendida solo si ese proceso tiene mayor prioridad que el hilo en ejecución

<details>
<summary>Ver respuesta</summary>

- **F**. La interrupción siempre se atiende al final de la instrucción; la prioridad solo decide si hay desalojo

</details>

### [Resumen] V/F: La transición Running → Ready solo es posible en algoritmos con quantum

<details>
<summary>Ver respuesta</summary>

- **F**. También en algoritmos con desalojo sin quantum (SRT, prioridades con desalojo)

</details>

### [Resumen] V/F: Un sistema con inversión de prioridades se soluciona cambiando el algoritmo por VRR

<details>
<summary>Ver respuesta</summary>

- **V** (según resumen). En VRR no hay prioridades fijas: todos ejecutan su quantum en algún momento, el de baja prioridad termina liberando el recurso

</details>

### [Resumen] Implicancias de un planificador sin desalojo en un SO de tiempo compartido

<details>
<summary>Ver respuesta</summary>

- Un proceso solo libera la CPU al terminar o bloquearse; si nunca lo hace (error o CPU bound) los demás no ejecutan

</details>

### [Resumen] Diferencias con y sin desalojo. ¿Dónde usar cada uno?

<details>
<summary>Ver respuesta</summary>

- Con desalojo: evalúan prioridades al llegar a ready y pueden quitar la CPU → más overhead, más equidad, priorizan lo importante (sistemas multitarea/interactivos)
- Sin desalojo: menos overhead (sistemas batch o donde minimizar overhead importa)

</details>

### [Resumen] V/F: Todos los planificadores (largo, mediano, corto) modifican el nivel de multiprogramación

<details>
<summary>Ver respuesta</summary>

- **F**. Solo largo y mediano plazo. El de corto plazo trata con procesos ya en RAM

</details>

### [Resumen] ¿Qué es el aging? Ejemplo

<details>
<summary>Ver respuesta</summary>

- Aumentar la prioridad de un proceso a medida que espera, para evitar starvation. Ej: HRRN (tiene en cuenta W)

</details>

### [Final 2023-02-14] En los algoritmos FIFO y HRRN, un proceso podría monopolizar el procesador de manera permanente — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Ambos son sin desalojo: si un proceso nunca se bloquea ni termina (ej: loop de CPU), nunca libera la CPU

</details>

### [Final 2023-02-28] Todos los algoritmos tipo Feedback que implementan prioridades entre colas pueden generar inanición — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. VRR es un feedback sin inanición (los nuevos entran por la cola de menor prioridad). También uno con aging (oficial)

</details>

### [Final 2023-08-01 / 2024-05-10] En VRR algunos procesos con ráfagas de CPU muy largas pueden sufrir inanición si el quantum es muy corto — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Una característica de VRR es no generar inanición (oficial)

</details>

### [Final 2023-09-27] El quantum es un intervalo indivisible, por lo tanto durante su duración el proceso no podrá ser interrumpido — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Durante el quantum pueden llegar interrupciones (fin de E/S, etc.) que se atienden al final de cada instrucción; además el proceso puede bloquearse antes

</details>

### [Final 2023-12-12] Todos los algoritmos sin desalojo presentan el riesgo de que el SO pierda el control del sistema — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V** (con matiz). Sin desalojo el SO solo recupera la CPU cuando el proceso se bloquea o termina: si nunca lo hace, monopoliza la CPU y el SO no puede replanificar. Las interrupciones igual se siguen atendiendo (el SO no pierde el control del HW, pierde el control de la planificación)

</details>

### [Final 2023-12-19] Los planificadores de corto plazo RR son tan justos para los KLT como para los ULT — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. El SO solo ve KLTs: los ULTs de un proceso se reparten el quantum de su KLT (un proceso con 10 ULTs recibe lo mismo que uno con 1 hilo) y el reparto entre ULTs lo decide la biblioteca

</details>

### [Final 2024-03-05] En los algoritmos sin desalojo un proceso puede monopolizar la CPU, pero si esto nunca ocurre no se genera inanición — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Con SJF o prioridades sin desalojo puede haber inanición por la constante aparición de procesos más prioritarios (oficial)

</details>

### [Final 2025-02-11 / 2025-12-09] El algoritmo HRRN resuelve el problema de starvation — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. La prioridad (W+S)/S crece con el tiempo de espera W (aging implícito): tarde o temprano todo proceso es elegido

</details>

### [Final 2025-09-25] Un algoritmo con prioridades fijas podría generar inanición pero no monopolización de la CPU — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Si es sin desalojo (o el proceso más prioritario nunca se bloquea) también puede monopolizar la CPU

</details>

### [Final 2025-12-02] RR mejora el tiempo de respuesta promedio con respecto a FIFO — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V** (en general). Los procesos cortos/interactivos no esperan a que terminen los largos: todos reciben CPU dentro de q·(n−1)

</details>

### [Final 2026-02-10] De todos los planificadores, solo los de mediano y largo plazo influyen en el nivel de multiprogramación — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Son los que ingresan/sacan procesos de memoria (admisión, swap) (oficial)

</details>

### [Final 2025-02-18] Los estados que involucran a los planificadores de largo plazo sólo cumplen la función de iniciar y finalizar procesos — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. El de largo plazo también regula el grado de multiprogramación: decide cuándo admitir (puede demorar procesos en New) o rechazar

</details>

## Prácticos

### [Final 2022-08-02 / 2023-04-25] RR Q=2, 2 procesos con ULTs, biblioteca SJF sin desalojo que maneja sus E/S. Gantt. ¿Hay simultaneidad de eventos? — ✅ solución oficial
| Hilo | Llegada | CPU | E/S | CPU |
|---|---|---|---|---|
| P1-u1 | 7 | 1 | 1 | 1 |
| P1-u2 | 1 | 2 | 3 | 2 |
| P2-u1 | 0 | 1 | 1 | 1 |
| P2-u2 | 0 | 2 | 2 | 2 |

| | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| P1-u1 | | | | | | | | | X | io | | X | F |
| P1-u2 | | X | X | io | io | io | X | X | F | | | | |
| P2-u1 | X | io | | X | F | | | | | | | | |
| P2-u2 | | | | | X | X | io | io | | X | X | F | |

<details>
<summary>Ver respuesta</summary>

- t0: SJF en P2 → u1 (1 < 2). t1: E/S de u1 → P2 se bloquea (sin jacketing) → P1 (u2)
- t3: P1 se bloquea; P2 (volvió en t2): u1 (1) < u2 (2) → u1 termina; sigue u2. t5 fin de quantum de P2 pero no hay nadie más listo → sigue
- t6: P1 vuelve de E/S; en t7 llega u1 a la biblioteca de P1
- **t8: simultaneidad**: fin de quantum de P1 (u2 terminó y queda u1 listo) y fin de E/S de P2 → gana el proceso en ejecución → sigue P1 (oficial)

</details>

### [Final 2022-12-06] RR q=3. Gantt, NTT de cada proceso, ¿alguno perjudicado? Proponer otro algoritmo con desalojo y ráfagas limitadas a 3 que lo mejore — ✍️ respuesta propia (el PDF del final no trae solución)
| | Llegada | CPU | E/S | CPU | E/S | CPU |
|---|---|---|---|---|---|---|
| P1 | 0 | 2 | 5 | 2 | 4 | 1 |
| P2 | 1 | 8 | 1 | 5 | - | - |
| P3 | 2 | 8 | - | - | - | - |

> ⚠️ **Aclaración de la consigna:** no dice si hay uno o varios dispositivos de E/S. Se asume que las E/S se hacen en paralelo (no hay cola de E/S).

`NTT = (Σ tiempos en Ready + Σ ráfagas CPU) / Σ ráfagas CPU`

**Consigna:**
- a) Realice el diagrama de Gantt
- b) Calcule el NTT de cada proceso y, en base a dicha métrica, mencione si existió algún proceso perjudicado y por qué
- c) Proponga otro algoritmo de planificación (con desalojo y que limite las ráfagas de los procesos en 3 unidades) que mejore el rendimiento del proceso afectado, y justifique realizando los dos puntos anteriores nuevamente

<details>
<summary>Ver respuesta</summary>

a) RR q=3 (E/S en paralelo)
```
t      0         1         2
       01234567890123456789012345
P1     XXiiiii----XXiiii---XF
P2      -XXX---XXX-----XXi--XXXXXF
P3       ---XXX-----XXX--XXF
```
(`-` = en ready). Colas: t5 [P3,P2] → t7 llega P1 → t8 [P2,P1,P3] → t11 [P1,P3,P2] → t13 [P3,P2] → t16 [P2,P3] + P1 en t17 → t19 P2 → [P1,P2]

b) NTT
- P1: ready 7-11 + 17-20 = 7; CPU 5 → (7+5)/5 = **2,4**
- P2: ready 1-2 + 5-8 + 11-16 + 19-21 = 11; CPU 13 → 24/13 = **1,85**
- P3: ready 2-5 + 8-13 + 16-18 = 10; CPU 8 → 18/8 = **2,25**
- El perjudicado es **P1** (IO bound, ráfagas cortas): RR favorece a los CPU bound; cada vez que vuelve de E/S va al final de la cola

c) Propuesta: colas multinivel realimentadas con desalojo por quantum: cola 1 (prioridad) para los que vuelven de E/S con RR q=3, cola 2 para nuevos/fin de quantum con RR q=3, sin desalojo entre colas
```
t      0         1         2
       01234567890123456789012345
P1     XXiiiii-XXiiii--XF
P2      -XXX-----XXX----XXi-XXXXXF
P3       ---XXX-----XXX---XXF
```
- P1: ready 7-8 + 14-16 = 3 → (3+5)/5 = **1,6** (mejora)
- P2: 1 + 5 + 4 + 1 = 11 → **1,85**; P3: 3 + 5 + 3 = 11 → 19/8 = **2,375**
- Ojo: con VRR "puro" (cola aux con q' = 3 − 2 = 1) P1 ejecuta 1 sola unidad al volver de E/S, se corta y vuelve al final de la cola → NTT = (8+5)/5 = 2,6 (no mejora con estos datos)

</details>

### [Final 2023-02-14] RR Q=3; dos procesos con ULTs, biblioteca SJF sin desalojo — ✍️ respuesta propia (el PDF del final no trae solución)
| Proceso | Hilo | Llegada | CPU | E/S | CPU |
|---|---|---|---|---|---|
| 1 | ULT1.A | 0 | 2 | 2 | 1 |
| 1 | ULT1.B | 0 | 2 | 2 | 3 |
| 2 | ULT2.C | 1 | 2 | 2 | 1 |
| 2 | ULT2.D | 1 | 3 | 1 | 2 |

> ⚠️ **Error en la consigna:** en el PDF los dos hilos del proceso 2 se llaman "ULT2.C". Se toma el segundo como ULT2.D.

**Consigna:**
1. Si la biblioteca de hilos utiliza jacketing:
   - a) ¿Qué hilo continúa ejecutando en t=2?
   - b) Luego de ejecutar el hilo respondido en el punto anterior, ¿qué hilo debe ejecutar?
   - c) Este último hilo, ¿en qué instante comenzará a ejecutar?
2. Si la biblioteca NO utiliza jacketing:
   - a) ¿Qué hilo continúa ejecutando en t=2?
   - b) Si la biblioteca replanifica luego de cada E/S, ¿qué hilo debe ejecutar luego del hilo respondido en el punto anterior?

<details>
<summary>Ver respuesta</summary>

- t0-2 ejecuta ULT1.A (empate 2 = 2 con B, ambos nuevos → FIFO/orden de tabla). En t2 A pide E/S
1. Con jacketing:
   - a) En t2 sigue **ULT1.B** (P1 no se bloquea y le queda 1 ut de quantum)
   - b) Luego (t3, fin de quantum de P1) ejecuta **ULT2.C** (P2: SJF, C=2 < D=3)
   - c) En **t = 3**
2. Sin jacketing:
   - a) La E/S bloquea a P1 → en t2 ejecuta **ULT2.C**
   - b) C corre 2-4 y pide E/S (P2 se bloquea). En t4 vuelve P1 (E/S de A 2-4); la biblioteca replanifica: A (1) < B (2) → **ULT1.A**

</details>

### [Final 2023-02-28] SJF sin desalojo; KLTs que acceden a archivos con lock exclusivo que no cierran. Cada acceso 1 ut + 1 ut si el archivo nunca fue abierto. El dispositivo no permite accesos en paralelo. Todos listos en t0 — ✅ solución oficial
| | | CPU | E/S | CPU | E/S | CPU |
|---|---|---|---|---|---|---|
| P1 | KLT1 | 1 | arch1 | 1 | arch2 | 1 |
| P1 | KLT2 | 3 | arch2 | 2 | arch1 | 1 |
| P2 | KLT3 | 2 | arch2 | 2 | arch1 | 1 |

**Consigna:**
- a) Realice el diagrama de Gantt e indique los tiempos de finalización de cada hilo y/o todo problema que ocurra
- b) ¿En qué instante cambiaría el diagrama si los archivos fueran abiertos mediante lock de lectura (compartido)?

<details>
<summary>Ver respuesta</summary>

a)
```
t     0123456789
K1    XiiX.            (t4 pide arch2: lo tiene P2 → bloqueado)
K2        XXX.         (t7 pide arch2 → bloqueado)
K3     XXii--XX.       (t9 pide arch1: lo tiene P1 → bloqueado)
```
- t0 K1 (ráfaga 1); t1-3 E/S arch1 (abre: 1+1) → lock arch1 para P1. t1 K3 (2 < 3); t3-5 E/S arch2 (abre) → lock arch2 para P2
- t3 K1, t4 K2 (sin desalojo corre 4-7 aunque K3 vuelve en t5), t7 K3
- **Deadlock**: P1 (K1 y K2) espera arch2 que tiene P2; P2 (K3) espera arch1 que tiene P1

b) Con lock compartido cambia en **t = 4/5**: en t4 K1 puede abrir arch2 (ya abierto por P2 → 1 ut) y hace la E/S en 5-6 (en t4 el dispositivo sigue ocupado por K3) (oficial)

</details>

### [Final 2023-05-23 / 2024-02-27] RR Q=3. El proceso A tiene 2 ULTs (biblioteca SJF sin desalojo). a) Gantt. b) Con jacketing, ¿cuándo empieza KLTB1? — ✅ solución oficial
| Proceso | Hilo | Arribo | CPU | E/S | CPU | E/S | CPU | E/S | CPU |
|---|---|---|---|---|---|---|---|---|---|
| A | ULTA1 | 0 | 2 | 1 | 1 | 2 | 1 | 1 | 4 |
| A | ULTA2 | 0 | 3 | 2 | 1 | - | - | - | - |
| B | KLTB1 | 3 | 4 | 2 | 2 | - | - | - | - |
| B | KLTB2 | 16 | 5 | - | - | - | - | - | - |

> ⚠️ **Error en la consigna:** en el PDF el segundo hilo del proceso A se llama "ULTB1" (parece del proceso B). Es un ULT de A: acá se lo llama ULTA2 (la resolución oficial lo dibuja como ULTA2).

<details>
<summary>Ver respuesta</summary>

a)
```
t        0         1         2
         0123456789012345678901234
ULTA1    XXiXii.Xi......XX...XXF
ULTA2    .........XXXiiXF
KLTB1       -XXX.Xii.XXF
KLTB2                    -XXX..XXF
```
- t0 A1 (2 < 3). t2-3 E/S de A1 → A bloqueado y B todavía no llegó → CPU ociosa
- t3: vuelve A (fin E/S) y llega KLTB1 (nuevo) → primero A (vuelve de E/S > nuevo). A1 (1 < 3) → 3-4, E/S 4-6
- KLTB1 4-7 (quantum), A 7-8 (A1: 1), KLTB1 8-9 → E/S 9-11
- t9 A: A1 (4) vs A2 (3) → A2 9-12, E/S 12-14. KLTB1 12-14 termina
- t14 A: A2 (1) < A1 (4) → A2 termina en 15; A1 15-17. t16 llega KLTB2. B2 17-20, A1 20-22 termina, B2 22-24 termina

b) Con jacketing KLTB1 empieza en **t = 6**: en t2 A no se bloquea (sigue A2 hasta t3), en t3 llegan a listos A (fin de quantum) y KLTB1 (nuevo) → A primero 3-6 (oficial)

</details>

### [Final 2023-07-25 N°10] RR Q=2. P1: CPU(3), E/S(1), CPU(1); P2: CPU(1), E/S(1), CPU(3). Listos en orden P1, P2. Cada intervención del SO = 1 ut. ChatGPT dice que el SO intervino 5 veces. Evaluar con un Gantt — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

```
t     0         1
      012345678901234
P1    XX...X.....X
P2       X...X.X...X
SO      S.S.S.S.S.S.S
```
- 2-3 SO: fin de quantum de P1 → P2
- 4-5 SO: syscall E/S de P2 (E/S 5-6) → P1
- 6-7 SO: syscall E/S de P1 (E/S 7-8) + fin de E/S de P2 → P2
- 8-9 SO: interrupción fin de E/S de P1 (P2 vuelve a ejecutar 9-10)
- 10-11 SO: fin de quantum de P2 → P1
- 12-13 SO: exit de P1 → P2
- 14-15 SO: exit de P2
- **7 intervenciones** (8 si se cuenta el dispatch inicial) → la respuesta de ChatGPT (5) es **incorrecta**: no cuenta las interrupciones de fin de E/S ni las finalizaciones

</details>

### [Final 2023-09-27 / 2024-03-05] RR Q=2, biblioteca de ULTs SJF sin desalojo, E/S a través de la biblioteca — ✅ solución oficial
| Proceso | Hilo | Arribo | CPU | E/S | CPU |
|---|---|---|---|---|---|
| 1 | ULT1.1 | 0 | 4 | 1 | 2 |
| 1 | ULT1.2 | 1 | 1 | 1 | 1 |
| 2 | KLT2 | 1 | 3 | 1 | 2 |

**Consigna:**
- a) Realice el diagrama de Gantt correspondiente a la ejecución de los hilos
- b) (2024-03-05) Mencione los instantes en los que se produce una llamada o ejecución de la biblioteca de hilos (aclarando si también ocurre una syscall o no)
- c) Sin rehacer el diagrama, ¿a partir de qué instante cambiaría si la biblioteca de hilos utilizase jacketing?
- d) (2024-03-05) ¿Quiénes seguirían ejecutando si el ULT 1.1 ejecutara la syscall `exit()` en el instante 1?

<details>
<summary>Ver respuesta</summary>

a)
```
t        0         1
         01234567890123
ULT1.1   XX..XXi....XXF
ULT1.2    ......Xi.XF
KLT2      .XX..XiXXF
```
- ULT1.1 sigue en 4-6 aunque llegó ULT1.2 (sin desalojo). t7 la biblioteca replanifica: 1.2 (1) < 1.1 (2). t10: 1.2 (1) < 1.1 (2)

b) Llamadas a la biblioteca (oficial): t1 (creación de ULT1.2, sin syscall), t6 (wrapper de E/S → syscall), t7 (replanifica), t8 (wrapper de E/S → syscall), t10 (replanifica), t11 (fin de ULT1.2)

c) Con jacketing cambia en **t = 8**: ULT1.2 pide E/S y a P1 le queda 1 ut de quantum → seguiría ULT1.1 (en t6 no cambia porque coincide con el fin de quantum)

d) Si ULT1.1 hace `exit()` en t1 termina todo el proceso 1 → solo sigue el proceso 2

</details>

### [Final 2023-12-19] VRR Q=3, 2 procesos con ULTs (biblioteca SJF sin desalojo, maneja sus E/S). Un solo disco. a) Gantt b) ¿En qué instantes se aplica SJF? c) ¿Cuándo cambia con jacketing? — ✍️ respuesta propia (el PDF del final no trae solución)
| Proceso | Hilo | Arribo | CPU | Disco | CPU |
|---|---|---|---|---|---|
| A | ULTA1 | 5 | 1 | 1 | 2 |
| A | ULTA2 | 2 | 2 | 1 | 2 |
| B | ULTB1 | 1 | 3 | 1 | 2 |
| B | ULTB2 | 0 | 2 | 2 | 3 |

<details>
<summary>Ver respuesta</summary>

Supuestos: al volver de E/S el proceso va a la cola aux con Q' = 3 − lo que ejecutó antes de bloquearse; si agota Q' vuelve a la cola normal. Desempate: 1° el que estaba en ejecución, 2° el que vuelve de E/S, 3° el nuevo (FIFO dentro de la misma categoría).

a)
```
t        0         1
         012345678901234567
ULTB2    XXiiX.XXF
ULTA2      XXi....XXF
ULTB1     .......X..XXi.XXF
ULTA1         Xi......XXF
```

Detalle:
| t | Ejecuta | Motivo | Aux | Ready |
|---|---|---|---|---|
| 0-2 | B (B2) | único hilo listo (B1 llega en 1) | | |
| 2-4 | A (A2) | B2 al disco 2-4 (B usó 2 → Q'=1) | | |
| 4-5 | B (B2) | A2 al disco 4-5 (A Q'=1); B vuelve → aux Q'=1. Empate SJF B1 (3) / B2 (3) → **B2 porque vuelve de E/S** (B1 es nuevo, nunca ejecutó) | | |
| 5-6 | A (A1) | B agota Q' → ready; A vuelve → aux Q'=1; SJF A1 (1) < A2 (2) | | B |
| 6-8 | B (B2) | A1 al disco 6-7; B2 sigue (sin desalojo) y termina en 8 | A (Q'=2, t7) | |
| 8-9 | B (B1) | queda solo B1; en t9 fin de quantum de B | A | |
| 9-11 | A (A2) | A desde aux (Q'=2): empate A1/A2 (ambos vuelven de E/S) → FIFO A2 (lista desde t5), termina en 11 | | B |
| 11-13 | B (B1) | A agotó Q' → ready; B1 termina la ráfaga → disco 13-14 (B Q'=1) | | A |
| 13-15 | A (A1) | A1 termina en 15 | B (Q'=1, t14) | |
| 15-17 | B (B1) | desde aux 15-16; agota Q' pero no hay nadie más → sigue; termina en 17 | | |

b) SJF decide en **t4** (empate B1/B2 → B2 por volver de E/S), **t5** (A1=1 vs A2=2), t8 (solo queda B1), **t9** (empate A1/A2 → A2 por FIFO). El instante donde se ve claro SJF: t5

c) Con jacketing cambia en **t = 2**: B2 pide el disco con 1 ut de quantum restante → B no se bloquea y ejecuta B1 en 2-3

</details>

### [Final 2025-02-11] RR Q=2 para dos procesos. B usa ULTs con biblioteca SJF **con** desalojo que maneja sus E/S. t0 llega ULTB1; t1 ULTB2 y KLTA2; t4 KLTA1 — ✍️ respuesta propia (el PDF del final no trae solución)
| Proceso | Hilo | CPU | E/S | CPU |
|---|---|---|---|---|
| A | KLTA1 | 1 | 1 | 2 |
| A | KLTA2 | 3 | 1 | 1 |
| B | ULTB1 | 3 | 1 | 1 |
| B | ULTB2 | 1 | 2 | 2 |

**Consigna:**
- a) Realice el diagrama de Gantt según la traza de ejecución
- b) ¿En qué instante cambiaría el Gantt si la biblioteca de hilos utilizara jacketing?

<details>
<summary>Ver respuesta</summary>

a)
```
t        0         1
         012345678901234
ULTB1    X....XXi.XF
ULTB2     Xii......X..XF
KLTA2     .XXXi..XF
KLTA1        ...Xi..XXF
```
- t1: llega ULTB2 (1) a la biblioteca → desaloja a B1 (restan 2). t2 B2 pide E/S → B bloqueado (y fin de quantum)
- t4: simultaneidad: fin de quantum KLTA2, fin E/S de B, llega KLTA1 → cola [KLTA2 (en ejecución), B (vuelve de E/S), KLTA1 (nuevo)]
- t5 B: empate B1 (2) / B2 (2) → B1, porque es el que venía ejecutando (fue desalojado en t1) y B2 vuelve de E/S. t7 B1 E/S → B bloqueado
- t9 B: B1 (1) < B2 (2) → B1 termina en 10; B2 10-11, fin de quantum; KLTA1 11-13; B2 13-14

b) Con jacketing cambia en **t = 4**: en t2 B no se bloquea (igual sale por fin de quantum) → en t4 B está primero en la cola y ejecuta antes que KLTA2

</details>

### [Final 2025-02-18] SO con FIFO, biblioteca de hilos SJF sin desalojo — ✍️ respuesta propia (el PDF del final no trae solución)
| Proceso | Hilo | Arribo | CPU | E/S | CPU |
|---|---|---|---|---|---|
| 1 | KLT1 | 0 | 2 | 1 | 3 |
| 1 | KLT2 | 2 | 3 | 2 | 1 |
| 2 | ULT1 | 1 | 3 | 2 | 3 |
| 2 | ULT2 | 3 | 2 | 1 | 2 |

**Consigna:** (justificando cada respuesta)
- a) ¿Qué proceso ejecuta a partir del instante 5?
  - i) Si ULT1 realizó una llamada al sistema
  - ii) Si ULT1 realizó la E/S a través de la biblioteca y no hay jacketing
  - iii) Si ULT1 realizó la E/S a través de la biblioteca y hay jacketing
- b) Cuando vuelva a ejecutar el proceso 2, ¿qué hilo continuará ejecutando?
  - i) Si ULT1 realizó la E/S a través de la biblioteca
  - ii) Si ULT1 realizó una llamada al sistema

<details>
<summary>Ver respuesta</summary>

- t0-2 KLT1, t2-5 proceso 2 (ULT1, único ULT en t2). En t3 llega ULT2 (a la biblioteca) y vuelve KLT1. En t5 ULT1 hace E/S
- a) ¿Qué proceso ejecuta a partir de t5?
  - i) ULT1 hizo una syscall → se bloquea todo el proceso 2 → ejecuta el **proceso 1** (KLT2, primero en la cola FIFO)
  - ii) E/S por la biblioteca sin jacketing → el wrapper igual termina en una syscall bloqueante → **proceso 1** (KLT2)
  - iii) Con jacketing → el proceso 2 no se bloquea → sigue el **proceso 2** con ULT2
- b) Cuando vuelve a ejecutar el proceso 2 (t11, sin jacketing), ¿qué hilo sigue?
  - i) E/S por la biblioteca → replanifica con SJF: ULT2 (2) < ULT1 (3) → **ULT2**
  - ii) Syscall directa → la biblioteca no se enteró → sigue **ULT1**

</details>

### [Final 2025-05-20] RR Q=2; Gantt incluyendo al SO, señalando interrupciones. A: CPU 6, E/S 2, CPU 2. B: CPU 1, E/S 1, CPU 1. Ambos en ready (A primero). Process switch = 2 ut; Blocked→Ready y Run→Exit = 1 ut — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

Supuestos: el despacho inicial de A no se cuenta; si no hay otro proceso listo la CPU queda ociosa; despachar desde ociosa también es un process switch.
```
t     0         1         2
      01234567890123456789012345
A     XX......XX......XXii...XX
B         Xi......X
SO      SS.SSS..SS.ESS....BSS..E
```
(S = SO haciendo switch / atendiendo, E = Run→Exit, B = Blocked→Ready)
| t | Qué pasa |
|---|---|
| 0-2 | A |
| 2 | **Int. clock** → 2-4 SO: switch A → B |
| 4-5 | B |
| 5 | **Syscall E/S** de B (E/S 5-6) → 5-7 SO: switch B → A |
| 6 | **Int. fin E/S** de B (durante el SO) → 7-8 SO: B blocked → ready |
| 8-10 | A |
| 10 | **Int. clock** → 10-12 SO: switch A → B |
| 12-13 | B termina → 13-14 SO: Run → Exit (**syscall exit**) → 14-16 SO: switch → A |
| 16-18 | A (completa las 6 ut) |
| 18 | **Syscall E/S** de A (E/S 18-20), no hay otro proceso → CPU ociosa |
| 20 | **Int. fin E/S** → 20-21 SO: blocked → ready → 21-23 SO: switch → A |
| 23-25 | A → 25-26 SO: Run → Exit |

- Hay otras convenciones posibles (ej: no contar el switch desde ociosa); lo importante es justificar cada intervención

</details>

### [Final 2025-07-15] Dado el Gantt, deducir el algoritmo del SO y de cada biblioteca (y si tiene jacketing). En t0 llegan KA, KB, KC en ese orden; los ULTs están listos (orden desconocido) — ✍️ respuesta propia (el PDF del final no trae solución)
Gantt del enunciado (X = CPU, io = E/S):
| | | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| KA | UA1 | | X | X | | | | | X | io | X | | | | X | F | | | | | | |
| | UA2 | X | io | | | | | | | X | F | | | | | | | | | | | |
| | UA3 | | | | | | | | | | | | | | | X | X | | | X | X | F |
| KB | UB1 | | | | | | | | | | | | | | | | | | X | F | | |
| | UB2 | | | | X | X | X | | | | | | | X | io | | | X | F | | | |
| KC | | | | | | | | X | io | | | X | X | F | | | | | | | | |

**Consigna:** (justificando)
- a) ¿Qué algoritmo de planificación utiliza el SO?
- b) ¿Qué algoritmo utiliza cada biblioteca de hilos de usuario? Indicar para cada una si implementa jacketing o no
- Nota: en caso que corresponda, asuma que el planificador conoce la duración de las ráfagas de CPU

<details>
<summary>Ver respuesta</summary>

a) SO: **VRR con Q = 3**
- Los KLTs ejecutan como máximo 3 seguidas (KA 0-3, KB 3-6, KA 7-10, KA 13-16) → quantum 3
- t10: ejecuta KC (volvió de E/S en 8) antes que KB (esperaba desde 6) y solo 2 ut (= 3 − 1 usada) → cola auxiliar de VRR. Igual KB en t16: vuelve de E/S en 14 y ejecuta 2 ut antes que nadie

b) Biblioteca de KA: **SJF sin desalojo, con jacketing**
- Jacketing: t1-2 UA2 hace E/S y KA sigue ejecutando UA1; t8 UA1 hace E/S y sigue UA2
- SJF: t0 elige UA2 (ráfaga 1) sobre UA1 (3) y UA3 (4); t9 elige UA1 (2) sobre UA3 (4) aunque UA3 espera desde t0 (descarta FIFO)
- Sin desalojo: t2 vuelve UA2 (ráfaga 1) y UA1 (le quedan 2) no es desalojado; en t7 sigue UA1

c) Biblioteca de KB: **FIFO, sin jacketing**
- t3 elige UB2 (ráfaga 4) teniendo a UB1 (ráfaga 1) → no es SJF (UB2 habría llegado primero)
- t13 UB2 hace E/S y KB se bloquea (ejecuta KA) → sin jacketing; al volver (t16) sigue el mismo UB2

</details>

### [Final 2025-09-25] Dado el Gantt, deducir el algoritmo del SO y de cada biblioteca — ✍️ respuesta propia (el PDF del final no trae solución)
| Hilos | | Arribo | CPU | E/S | CPU |
|---|---|---|---|---|---|
| KLTA | ULT1 | 0 | 5 | 1 | 4 |
| | ULT2 | 1 | 2 | 3 | 2 |
| KLTB | ULT3 | 3 | 6 | 2 | 4 |
| | ULT4 | 4 | 6 | 3 | 1 |
| | ULT5 | 6 | 3 | 1 | 1 |

El Gantt del enunciado está como imagen en `finales/Final 2025-09-25.pdf` (en la respuesta está la traza reconstruida).

**Consigna:** (justificando en todos los casos con al menos un instante en el que se aprecie claramente)
- a) ¿Qué algoritmo de corto plazo está utilizando el SO?
- b) ¿Qué algoritmo utiliza la biblioteca de ULTs del KLTA? ¿Tiene jacketing?
- c) ¿Qué algoritmo utiliza la biblioteca de ULTs del KLTB? ¿Tiene jacketing?

<details>
<summary>Ver respuesta</summary>

Traza (reconstruida del Gantt):
| t | Ejecuta | Nota |
|---|---|---|
| 0-1 | ULT1 | |
| 1-3 | ULT2 | llega ULT2 (2) y desaloja a ULT1 (le quedan 4) |
| 3-7 | ULT3 | ULT2 hace E/S (3-6) → KLTA bloqueado aunque le quedaba 1 ut |
| 7-8 | ULT2 | KLTA vuelve de E/S (aux) con Q' = 4 − 3 = 1 |
| 8-10 | ULT3 | termina su ráfaga de 6 → E/S 10-12 |
| 10-12 | ULT5 | KLTB sigue ejecutando (jacketing), elige ULT5 sobre ULT4 |
| 12-13 | ULT2 | termina |
| 13-16 | ULT1 | fin de quantum de KLTA |
| 16-17 | ULT5 | termina ráfaga → E/S 17-18 |
| 17-20 | ULT4 | KLTB sigue (jacketing); elige ULT4 sobre ULT3 |
| 20-21 | ULT1 | termina ráfaga de 5 → E/S 21-22 (KLTA bloqueado) |
| 21-24 | ULT4 | termina ráfaga → E/S 24-27 |
| 24-25 | ULT5 | termina |
| 25-28 | ULT1 | KLTA desde la aux con Q' = 4 − 1 = 3 |
| 28-32 | ULT3 | termina |
| 32-33 | ULT1 | termina |
| 33-34 | ULT4 | termina |

a) SO: **VRR con Q = 4**. Los KLTs ejecutan hasta 4 seguidas (3-7, 8-12, 12-16, 16-20, 21-25, 28-32). En t7 KLTA ejecuta solo 1 ut (había usado 3 antes de bloquearse en t3) y en t25 ejecuta 3 (había usado 1) → cola auxiliar con Q − usado

b) KLTA: **SJF con desalojo (SRT), sin jacketing**. t1: ULT2 (2) desaloja a ULT1 (restan 4). t3: ULT2 hace E/S y KLTA se bloquea teniendo quantum y a ULT1 listo → sin jacketing

c) KLTB: **HRRN, con jacketing**. Jacketing: t10 y t17 un ULT hace E/S y KLTB sigue con otro
- t10: ULT4 R = (6+6)/6 = 2; ULT5 R = (4+3)/3 = 2,33 → ULT5 (FIFO habría elegido ULT4)
- t17: ULT4 R = (13+6)/6 = 3,17; ULT3 R = (5+4)/4 = 2,25 → ULT4 (SJF habría elegido ULT3)
- t24: ULT5 R = (6+1)/1 = 7; ULT3 R = (12+4)/4 = 4 → ULT5. Sin desalojo (t8 sigue ULT3)

</details>

### [Final 2025-12-02] 2 CPUs, SO FIFO, ULTs SJF sin desalojo. a) Gantt b) ¿Cuándo cambia con jacketing? — ✍️ respuesta propia (el PDF del final no trae solución)
| KLT | Hilo | Llegada | CPU | E/S |
|---|---|---|---|---|
| KLTA | ULT1 | 0 | 3 | - |
| KLTA | ULT2 | 0 | 2 | - |
| KLTB | ULT3 | 2 | 1 | 2 |
| KLTB | ULT4 | 2 | 2 | - |

> ⚠️ **Aclaración de la consigna:** ULT3 termina con una E/S (no tiene ráfaga de CPU después): se considera terminado al finalizar esa E/S.

<details>
<summary>Ver respuesta</summary>

a)
```
t        01234567
CPU1     22111           (ULT2 0-2, ULT1 2-5)
CPU2       3ii44         (ULT3 2-3, E/S 3-5 → termina, ULT4 5-7)
```
- Los ULTs de un mismo KLT no pueden ejecutar en paralelo: KLTA usa una CPU, KLTB la otra
- t3: ULT3 hace E/S → KLTB bloqueado (CPU2 ociosa 3-5)

b) Con jacketing cambia en **t = 3**: KLTB no se bloquea y ejecuta ULT4 en 3-5 (terminaría en 5)

</details>

### [Final 2026-02-10] RR Q=2, dos recursos de E/S (Red y Disco). a) Gantt b) Interrupciones — ✅ solución oficial
| Proceso | Arribo | CPU | E/S | CPU |
|---|---|---|---|---|
| A | 0 | 2 | Red (5) | 3 |
| B | 1 | 2 | Disco (4) | 3 |
| C | 2 | 2 | Red (4) | 3 |

<details>
<summary>Ver respuesta</summary>

a) (oficial)
```
t     0         1
      0123456789012345
A     XXrrrrrXX..X
B       XXdddd.XX.X
C         XX.rrrr..XXX
```
- C pide Red en t6 pero la usa A hasta t7 → espera y la usa 7-11. CPU ociosa 6-7
- t11: fin de quantum de B y fin de E/S de C → primero B (en ejecución) y después C (vuelve de E/S) → cola [A, B, C]

b) Interrupciones: t7 fin E/S (A), t8 fin E/S (B), t9 clock, t11 clock, t11 fin E/S (C), t15 clock

</details>

### [Final 2026-05-19] SJF con desalojo, un recurso de E/S con dos instancias. a) Gantt b) Grafo de asignación en t4 — ✅ solución oficial
| Proceso | Arribo | CPU | E/S | CPU |
|---|---|---|---|---|
| A | 0 | 2 | 3 | 1 |
| B | 2 | 1 | 4 | 1 |
| C | 3 | 1 | 1 | 3 |

<details>
<summary>Ver respuesta</summary>

a) (oficial)
```
t     01234567890
A     XXiiiXF
B       XiiiiXF
C        X.iX.XXF
```
(C: CPU 3-4, espera instancia 4-5, E/S 5-6, CPU 6-7, desalojado por B 7-8, CPU 8-10 F)
- t4: C pide E/S pero las dos instancias están ocupadas (A 2-5, B 3-7) → espera hasta 5
- t7: vuelve B (ráfaga 1) < lo que le queda a C (2) → desaloja a C

b) Grafo en t4: R (2 instancias) → asignado a A y a B; C → R (solicitud)

</details>
