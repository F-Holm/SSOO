# 04 - Planificación (teoría)

## Conceptos
- Planificación = políticas y mecanismos del SO que gobiernan el orden de ejecución. Mueve procesos entre colas
- La ejecución es alternancia de ráfagas de CPU y ráfagas de E/S. No se sabe el largo de una ráfaga, se estima
- CPU bound (mucha CPU) / IO bound (mucha E/S)
- Grado de multiprogramación = cantidad de procesos activos en memoria (ready + running + blocked)
  - Bajo: CPU ociosa
  - Alto: CPU no ociosa pero menos % de CPU por proceso

## Tipos de planificadores
- Extra largo plazo
- Largo plazo: admisión (new → ready) y exit. Controla el grado de multiprogramación
- Mediano plazo: swapping (suspender / reanudar). Controla el grado de multiprogramación
- Corto plazo: ready → running (dispatcher). El que más frecuencia tiene → debe tener **menos overhead**. No modifica el grado de multiprogramación (solo trata procesos en RAM)
- Solo largo y mediano plazo modifican el grado de multiprogramación
- Corto plazo ejecuta cuando interviene el SO: interrupciones, syscalls, señales
- Funciones: dispatcher (da la CPU al elegido) + context switch

## Con / sin desalojo
- Sin desalojo: el SO interviene solo cuando se libera la CPU → syscall bloqueante, fin de proceso. Un proceso puede monopolizar la CPU
- Con desalojo: además cuando llega a ready uno de mayor prioridad (new → ready, blocked → ready, timeout)
- Running → ready es posible en todos los algoritmos con desalojo (no solo con quantum, ej: SRT)
- Sin desalojo en sistemas de tiempo compartido: un proceso que nunca se bloquea no libera nunca la CPU
- Con desalojo: más overhead pero más equidad / prioriza lo importante

## Criterios
- Cuantitativos (usuario): tiempo de ejecución/retorno, tiempo de espera, tiempo de respuesta
- Cuantitativos (sistema): tasa de procesamiento (throughput), % uso de CPU
- Cualitativos: previsibilidad, equidad, imposición de prioridades, equilibrio de recursos
- Tiempo de espera = tiempo en la cola de listos (= T final − T llegada − T CPU, si no hay E/S)
- Tiempo de respuesta = hasta la primera respuesta (la cátedra también usa T final − T llegada)
- NTT (normalized turnaround time) = (Σ tiempos en Ready + Σ ráfagas CPU) / Σ ráfagas CPU. Más alto = más perjudicado

## Algoritmos
- **FIFO / FCFS**: orden de llegada, sin desalojo
  - Un proceso largo se lleva todo el tiempo (no tiene en cuenta duración ni prioridad). Puede monopolizar la CPU
  - No monopolizan en general, útil para procesos secuenciales, minimiza cambios de contexto, overhead bajo
  - Starvation: no (salvo monopolización)
- **SJF / SPN** (sin desalojo): la ráfaga más corta. Prioriza a los IO bound. Mucho overhead (estimar ráfagas)
  - Minimiza el tiempo de espera promedio
  - Starvation si no paran de llegar procesos más cortos
  - Empate → FIFO (el que lleva más en ready)
- **SRT** (SJF con desalojo): si llega uno con ráfaga más corta que el restante del actual, lo desaloja
  - Empate → sigue el que estaba ejecutando
  - Starvation, overhead medio-alto (estimadores)
- **SJF con estimación**: `Est(n+1) = α·R(n) + (1−α)·Est(n)`, 0 ≤ α ≤ 1
  - Est = ráfaga estimada, R = ráfaga real
  - α alto: pesa lo reciente. α bajo: pesa la historia (conviene si el proceso es estable)
  - Siempre dan la estimación inicial o los datos para calcularla
  - Restar 1 a la estimación después de cada unidad ejecutada hasta terminar la ráfaga o llegar a 0
- **Prioridades**: con o sin desalojo. Starvation → se soluciona con **aging** (envejecimiento: subir la prioridad mientras espera)
- **HRRN** (Highest Response Ratio Next): RR = (W + S) / S = 1 + W/S
  - W = tiempo esperando en Ready, S = duración de la próxima ráfaga
  - Adaptación de SJF sin inanición (aging implícito). Favorece ráfagas cortas y a los que esperan mucho
  - Suele ser sin desalojo → puede monopolizar la CPU. Mucho overhead. Con desalojo habría que recalcular el ratio de todos los listos en cada evento (muchísimo overhead)
- **RR** (Round Robin): FIFO + quantum (interrupción de clock), siempre con desalojo
  - Al vencer el quantum, el **timer** (programado por el planificador) lanza la interrupción; el SO la atiende y el planificador elige al siguiente en listos (no es el SO el que interrumpe)
  - Con n procesos y quantum q, nadie espera más de q·(n−1)
  - No minimiza la espera, la hace previsible. Equitativo. Sin starvation. Overhead medio
  - Quantum muy grande → FIFO. Muy chico → mucho overhead
  - Si se bloquea antes de terminar el quantum, **no se acumula**
  - Favorece a CPU bound, perjudica a IO bound
- **VRR** (Virtual RR): cola multinivel, soluciona el problema de RR con ráfagas chicas (favorece IO bound)
  - Cola READY normal con quantum Q (vienen por fin de quantum o nuevos)
  - Cola AUX de mayor prioridad con quantum Q' = Q − lo que ya usó (vienen de E/S)
  - Es un RR normal hasta que una ráfaga termina antes del quantum; al desbloquearse va a la aux y ejecuta lo que le faltaba del quantum
  - Sin desalojo por prioridad de cola (la aux se atiende cuando se libera la CPU). Ambas FIFO
  - Sin starvation. Overhead alto
- **Colas multinivel (CMN)**: una cola por prioridad (ej: sistema, usuarios, tipo R), cada una con su algoritmo
  - Prioridad estática. Puede generar starvation
  - Desalojo entre algunas, todas o ninguna cola
- **CMN retroalimentado (feedback)**: los procesos cambian de cola (ej: bajan al agotar quantum; timers para subir a los que esperan mucho)
  - No basta con saber cuántas colas y su algoritmo; hay que definir:
    - Número de colas
    - Algoritmo de cada cola
    - A qué cola llegan los nuevos
    - Criterio para pasar de una cola a otra
    - Algoritmo entre colas (con o sin desalojo)
  - Con aging no tiene inanición. Algunos feedback (ej: VRR) no generan inanición

### Algoritmos con inanición
- SJF, SRT, prioridades, colas multinivel, feedback (según implementación)

### Resumen
| Algoritmo | Desalojo | Starvation | Overhead | Otros |
|---|---|---|---|---|
| FIFO | No | No* | Bajo | Monopoliza |
| SJF | No | Sí | Medio | Min. espera promedio |
| SRT | Sí | Sí | Medio-alto | |
| HRRN | No | No | Alto | Monopoliza |
| RR | Sí (clock) | No | Medio | Favorece CPU bound |
| VRR | Sí (clock) | No | Alto | Favorece IO bound |
| Prioridades | Depende | Sí (sin aging) | | |
| CMN | Depende | Sí | | |

## Simultaneidad de eventos en ready
- Desempate: 1° el proceso en ejecución (fin de quantum / desalojo, interrupción de clock), 2° el que vuelve de E/S (fin de evento), 3° el proceso nuevo (syscall)
- Se usa para cualquier empate en los Gantt (llegada simultánea o misma prioridad/ráfaga). Dentro de la misma categoría, FIFO
- Importa en FIFO y RR (el orden de llegada define la prioridad). En SJF no. En VRR los que vuelven de E/S van a la aux
- Las E/S no se planifican: se atienden por FIFO

## Condiciones de carrera y el planificador
- El planificador puede interrumpir en medio de una operación que debía ser atómica → condición de carrera (aún en monoprocesador). Si están bien sincronizados el resultado es correcto sin importar el orden
- El proceso no sabe que fue bloqueado/desalojado (se guarda y restaura su contexto)

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
