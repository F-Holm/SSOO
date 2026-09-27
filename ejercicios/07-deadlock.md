# 07 - Deadlock (ejercicios)

## Teóricos

### [Resumen] Diferencia entre prevención y detección. ¿Cuándo usar cada una?
- Prevención garantiza que no ocurra (rompe una condición); detección deja que ocurra y después recupera
- Prevención: sistemas donde el deadlock es inaceptable (aviones, dispositivos médicos). Detección: uso general (PC hogareña)

### [Resumen] V/F: En un deadlock una buena solución suele ser matar a todos los involucrados
- **F**. Es una opción pero no la mejor: conviene matar de a uno hasta que no haya deadlock

### [Resumen] V/F: Con el algoritmo de evasión podría ocurrir deadlock si todos piden su máximo
- **F**. El banquero asegura que no habrá deadlock

### [Resumen] Condiciones necesarias y suficientes
- Necesarias: mutua exclusión, sin desalojo, retención y espera. Suficiente: las 3 + espera circular

### [Resumen] V/F: Tanto prevención como detección y recuperación podrían generar starvation
- **V**. Prevención: retención y espera (esperar a tener todo). Recuperación: si siempre se elige la misma víctima

### [Resumen] Prevenir el deadlock atacando: a) retención y espera b) sin desalojo
- a) Pedir todos los recursos juntos; si alguno no está, se deniega todo
- b) Si A pide un recurso que tiene B y B está bloqueado, se le quita a B y se le da a A

### [Resumen] Explique en detalle la estrategia de prevención
- Garantiza que no haya deadlock impidiendo que se cumpla alguna de las 4 condiciones:
  - Mutua exclusión: no siempre se puede (recursos no compartibles)
  - Retención y espera: pedir todo junto al inicio (o no pedir teniendo algo asignado)
  - Sin desalojo: quitarle recursos a un proceso bloqueado que los retiene (poder retrotraerlo)
  - Espera circular: numerar los recursos y pedirlos en orden creciente

### [Resumen] V/F: No puede ocurrir deadlock en un SO que ejecuta procesos sin concurrencia
- **V**. Si ejecuta uno hasta que termina (y libera todo), no hay retención y espera entre procesos. Ojo con los hilos

### [Resumen] Evasión vs prevención: parecidos, diferencias y cuándo usar cada una
- Ambas aseguran que no haya deadlock. Prevención rompe una condición; evasión simula cada asignación y solo la hace si el estado queda seguro
- Prevención: poco overhead pero restrictiva (subutiliza recursos) → HW limitado, sistemas críticos simples. Evasión: más flexible pero mucho overhead y estructuras → cuando importa el aprovechamiento de recursos

### [Resumen] V/F: Con semáforos con espera activa puede darse un deadlock sin que ocurran las 4 condiciones
- **F**. El deadlock solo se da con las 4 condiciones

### [Resumen] V/F: Con procesador poco potente, muchos recursos y la necesidad de garantizar que no haya deadlocks conviene evasión
- **F**. Evasión genera mucho overhead y estructuras grandes con muchos recursos → conviene prevención

### [Resumen] Compare evasión y detección: frecuencia, overhead, criticidad, flexibilidad
| | Frecuencia | Overhead | Criticidad del sistema | Flexibilidad |
|---|---|---|---|---|
| Evasión | En cada solicitud | Muy alto | Alta (no puede haber deadlock) | Baja (deniega si queda inseguro) |
| Detección | Periódica / al sospechar | Bajo | Baja (matar procesos puede ser grave) | Alta (asigna libremente) |

### [Resumen] V/F: Con el banquero, si se detecta un deadlock se pueden desalojar recursos para solucionarlo
- **F**. El banquero no detecta deadlocks: si la asignación dejaría un estado inseguro, simplemente no la hace

### [Resumen] Banquero: ¿qué es estado seguro? ¿Podría quedar inseguro?
- Existe al menos una secuencia en la que todos los procesos pueden terminar
- Bien implementado nunca queda inseguro (rechaza la petición)

### [Resumen] V/F: Los ULTs de un mismo proceso pueden quedar en deadlock utilizando semáforos
- Resumen: **F**, porque con semáforos del SO un wait bloqueante bloquea a todos los ULTs y nunca se forma la espera circular
> ⚠️ **Aclaración sobre la respuesta del Resumen:** si la biblioteca provee sus propios semáforos (a nivel usuario) sí pueden quedar en deadlock (ULT1 tiene A y pide B, ULT2 tiene B y pide A). Con semáforos del SO el proceso puede igual quedar bloqueado para siempre si el signal lo debía hacer otro ULT del mismo proceso

### [Final 2022-09-07] La detección de deadlock requiere que cada proceso declare el máximo de recursos que puede necesitar — ✅ solución oficial
- **F**. Detección analiza la situación actual (peticiones y asignaciones). Evasión sí requiere los máximos (oficial)

### [Final 2022-12-06] La estrategia de detección asegura que no se producirá deadlock, analizando cada solicitud antes de asignar — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Eso describe a la evasión. Detección deja que ocurra y lo detecta periódicamente

### [Final 2022-12-21] Un proceso cuyos hilos sufren condición de carrera es menos propenso a que sufran deadlock entre sí — ✅ solución oficial
- **V**. Si hay condición de carrera es porque no hay mutua exclusión, una de las condiciones necesarias (oficial)

### [Final 2023-02-28] Si los procesos deben solicitar todos los recursos al iniciarse, nunca ocurrirá interbloqueo — ✅ solución oficial
- **V**. Es prevención: se rompe la retención y espera (oficial)

### [Final 2023-03-07] No es recomendable usar evasión en un sistema sobre hardware de bajas prestaciones — ✅ solución oficial
- **V**. Evasión genera mucho overhead (oficial)

### [Final 2023-04-25 / 2023-05-23] Un conjunto A de procesos no comparte recursos con otro conjunto B. Si A entra en deadlock no afecta a B, pero sí lo haría si fuese un livelock — ✅ solución oficial
- **V**. En deadlock están bloqueados (no usan CPU); en livelock ocupan CPU indefinidamente (oficial)

### [Final 2023-05-23] Un deadlock es un bloqueo permanente entre procesos debido a que éstos solicitaron recursos en común — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Pedir recursos en común no alcanza: deben darse las 4 condiciones (mutua exclusión, retención y espera, sin desalojo, espera circular)

### [Final 2023-09-27] Si varios procesos solicitan un recurso no compartible de una instancia y a su vez retienen otros, los que no accedan quedarán en deadlock — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Quedan bloqueados esperando, pero falta la espera circular: si el que tiene el recurso no espera nada de los otros, termina y lo libera

### [Final 2023-12-19 / 2024-05-10] El algoritmo del banquero permite que el sistema reaccione inmediatamente ante la ocurrencia de deadlock — ✅ solución oficial
- **F**. Con el banquero el sistema siempre está en estado seguro y nunca ocurre deadlock (oficial)

### [Final 2024-02-20] Si un recurso puede ser accedido en paralelo por un conjunto de procesos, no puede causar deadlock pero sí livelock — ✅ solución oficial
- **F**. Si es compartido no hay mutua exclusión, necesaria tanto para deadlock como para livelock (oficial)

### [Final 2024-02-27] Si en el grafo de asignación hay espera circular, se puede concluir que hay deadlock sin más análisis — ✅ solución oficial
- **V** (oficial), siempre que todos los recursos del ciclo tengan una sola instancia (se ve en el mismo grafo)

### [Final 2024-03-05] Tanto en prevención como en evasión se finaliza a los procesos en espera circular — ✅ solución oficial
- **F**. En prevención la espera circular no se produce; en evasión no se finaliza a nadie, se bloquean antes al simular la asignación (oficial)

### [Final 2024-07-23] Si el banquero determina un futuro estado inseguro, tarde o temprano ocurrirá deadlock — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Inseguro = posibilidad de deadlock, no certeza (los procesos pueden no pedir su máximo). Además el banquero no hace esa asignación

### [Final 2024-07-30] Recuperarse de un deadlock eliminando los procesos de baja prioridad puede generar starvation — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Si siempre se elige como víctima al de menor prioridad, puede no terminar nunca

### [Final 2025-02-25] La evasión es la más eficiente (en uso de recursos del sistema) en sistemas críticos donde no puede existir deadlock — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Entre las que garantizan que no haya deadlock (prevención y evasión), evasión aprovecha mejor los recursos: no obliga a pedir todo junto ni a un orden fijo. El costo es overhead de CPU en cada solicitud

### [Final 2025-05-20] Si se impide que un proceso adquiera un recurso mientras tiene asignado otro, nunca ocurrirá deadlock — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Se rompe la retención y espera (prevención)

### [Final 2025-07-15] Si se quiere minimizar el overhead y asegurar que nunca haya deadlock, evasión es lo más apropiado — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Evasión tiene mucho overhead (algoritmo en cada solicitud); lo apropiado es prevención

### [Final 2025-07-29] Para evitar deadlock es fundamental usar semáforos mutex para proteger los recursos — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. El mutex impone mutua exclusión, que es justamente una condición necesaria del deadlock; mal usado lo provoca. Para evitarlo se usa prevención o evasión

### [Final 2025-09-25] No usar ninguna técnica de tratamiento de deadlocks es aceptable si la criticidad es baja — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Es lo que hace la mayoría de los SO (se evita el overhead)

### [Final 2025-12-02 / 2025-12-09] En evasión por denegación de recursos (estados seguro/inseguro) el sistema nunca podrá quedar en deadlock — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Solo se asigna si el estado resultante es seguro

### [Final 2025-12-16] Prevención tiene menos overhead que evasión y permite más flexibilidad en el uso de recursos que detección y recuperación — ✅ solución oficial
- **F**. Menos overhead que evasión sí, pero es más restrictiva en el uso de recursos que detección (oficial)

### [Final 2026-02-10] Los procesos que no forman parte de un deadlock no resultan afectados — ✅ solución oficial
- **F**. Pueden verse afectados si necesitan recursos retenidos por los procesos del deadlock (oficial)

### [Final 2026-05-19] El SO siempre asignará un recurso dependiendo únicamente de su disponibilidad actual — ✅ solución oficial
- **F**. Con el banquero puede rechazar una solicitud aunque el recurso esté disponible (oficial)

### [Final 2023-07-25 N°5] ChatGPT: "Sí, uno de los procesos en interbloqueo puede romper la espera circular aplicando prevención o resolución, por ejemplo con el algoritmo del banquero". Evaluar — ✍️ respuesta propia (el PDF del final no trae solución)
- **Incorrecta**. Los procesos en deadlock están bloqueados: no pueden ejecutar nada para liberar recursos. La resolución viene de afuera (el SO mata o expropia). Además el banquero es evasión: evita entrar en estados inseguros, no resuelve un deadlock existente

## Prácticos

### [Final 2022-12-06] a) Disponibles (1,0,9,4): ¿estado seguro? b) Con FCFS (orden = nro de proceso), ¿se satisfacen las 3 primeras peticiones (cada uno pide todo lo pendiente)? — ✍️ respuesta propia (el PDF del final no trae solución)
| | MA R1 | R2 | R3 | R4 | | MM R1 | R2 | R3 | R4 |
|---|---|---|---|---|---|---|---|---|---|
| P1 | 0 | 3 | 1 | 3 | | 2 | 3 | 2 | 5 |
| P2 | 1 | 1 | 3 | 2 | | 1 | 2 | 6 | 3 |
| P3 | 0 | 2 | 1 | 0 | | 0 | 2 | 4 | 5 |
| P4 | 2 | 0 | 2 | 0 | | 3 | 0 | 5 | 2 |
| P5 | 1 | 3 | 5 | 2 | | 3 | 4 | 5 | 4 |

Pendientes = MM − MA: P1 (2,0,1,2), P2 (0,1,3,1), P3 (0,0,3,5), P4 (1,0,3,2), P5 (2,1,0,2)

a)
| Paso | Termina | Disponibles |
|---|---|---|
| inicio | | (1,0,9,4) |
| 1 | P4 (1,0,3,2 ≤ disp) | + (2,0,2,0) = (3,0,11,4) |
| 2 | P1 | + (0,3,1,3) = (3,3,12,7) |
| 3 | P2 | + (1,1,3,2) = (4,4,15,9) |
| 4 | P3 | + (0,2,1,0) = (4,6,16,9) |
| 5 | P5 | + (1,3,5,2) = (5,9,21,11) = totales ✓ |
- Secuencia segura P4, P1, P2, P3, P5 → **estado seguro**

b) Con disponibles (1,0,9,4):
- P1 pide (2,0,1,2): R1 2 > 1 → no hay recursos → espera
- P2 pide (0,1,3,1): R2 1 > 0 → espera
- P3 pide (0,0,3,5): R4 5 > 4 → espera
- **Ninguna** de las tres se satisface (ni siquiera hace falta simular el estado seguro). La única atendible sería la de P4

### [Final 2023-08-01] Detección y recuperación. Totales [R1..R4] = [4,5,7,5]. Se asume que más recursos asignados = más cerca de terminar — ✅ solución oficial
| | Pet R1 | R2 | R3 | R4 | | Asig R1 | R2 | R3 | R4 |
|---|---|---|---|---|---|---|---|---|---|
| P1 | 1 | 0 | 2 | 2 | | 0 | 0 | 0 | 0 |
| P2 | 0 | 2 | 1 | 0 | | 1 | 1 | 2 | 0 |
| P3 | 2 | 1 | 0 | 1 | | 1 | 2 | 3 | 1 |
| P4 | 0 | 0 | 0 | 0 | | 1 | 1 | 1 | 1 |
| P5 | 1 | 0 | 2 | 0 | | 1 | 1 | 1 | 2 |

- Disponibles = [4,5,7,5] − [4,5,7,4] = [0,0,0,1]
- P4 no pide nada → termina → [1,1,1,2]. Nadie más puede: P1 (R3), P2 (R2), P3 (R1), P5 (R3)
- a) P4 sin problemas; **P2, P3 y P5 en deadlock**; **P1 en inanición** (no tiene recursos asignados, no forma parte de la espera circular) (oficial)
- b) Matar a **P2** (el de menos recursos asignados entre los del deadlock → el que está menos cerca de terminar)
- c) [1,1,1,2] + [1,1,2,0] = [2,2,3,2] → P3 termina → [3,4,6,3] → P5 → [4,5,7,5] → P1 → **deadlock solucionado**
