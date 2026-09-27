# 04 - Planificación (práctica: diagramas de Gantt)

## Llevar al final
- Template de ejercicio de planificación de al menos 30 tiempos, 8 procesos/hilos, tabla de datos de entrada y espacio para múltiples colas (5+)

## Cómo armar el Gantt
1. Una fila por proceso/hilo, columnas = unidades de tiempo. Marcar CPU, E/S (o bloqueado) y F (fin)
2. Debajo, en cada instante, anotar la cola de Ready (y la aux en VRR, colas de E/S si hay un solo dispositivo)
3. En cada evento (llegada, fin de ráfaga, fin de quantum, fin de E/S) re-evaluar según el algoritmo
4. Revisar simultaneidad de eventos en cada instante

## Convenciones de la cátedra
- **Desempate (siempre que haya empate, en llegada simultánea a ready o en el criterio del algoritmo)**:
  1. El proceso en ejecución (el que viene de CPU: fin de quantum o desalojo)
  2. El proceso que vuelve de E/S
  3. El proceso nuevo
  - Dentro de la misma categoría: FIFO (el que espera desde antes)
- SJF con empate de ráfaga entre uno que vuelve de E/S y uno que nunca ejecutó → el que vuelve de E/S
- SRT con empate: no desaloja
- RR: el quantum no se acumula. Si queda solo un proceso listo al terminar su quantum, sigue ejecutando
- Las E/S se atienden FIFO. Si el dispositivo "no permite accesos en paralelo" hay que hacer cola de E/S; si no dice nada, suelen asumirse en paralelo
- Si un recurso de E/S tiene N instancias, hasta N procesos a la vez
- En los ejercicios se asume **sin jacketing** salvo que diga lo contrario
- Si dicen "cada involucramiento del SO / process switch insume X ut", dibujar una fila del SO

## Métricas
- Tiempo de espera = tiempo en ready
- Tiempo de retorno = T fin − T llegada
- NTT = (Σ ready + Σ CPU) / Σ CPU
- % uso CPU = tiempo útil / tiempo total

## Estimación SJF
- `Est(n+1) = α·R(n) + (1−α)·Est(n)`
- HRRN: `R = (W + S) / S` → elegir el mayor. Calcular cada vez que se libera la CPU

## VRR
- Vuelve de E/S → cola aux (prioridad) con Q' = Q − lo que usó en su última ejecución antes de bloquearse
- Fin de quantum o nuevo → cola ready normal con Q completo
- Si usa todo Q' → vuelve a la cola normal
- La aux no desaloja al que está ejecutando; se elige de la aux cuando se libera la CPU
- Ojo: si la ráfaga siguiente es más larga que Q', el proceso se corta y vuelve al final de la cola normal (puede terminar perjudicándolo)

## Hilos en el Gantt (ULT / KLT)
- El SO planifica **KLTs** (o procesos monohilo). No conoce a los ULTs
- Un proceso con ULTs se ve como un solo KLT → el quantum es del KLT y se reparte entre sus ULTs
- La biblioteca planifica los ULTs dentro del tiempo de CPU que el SO le da al KLT (doble planificación)
- La biblioteca solo toma el control cuando la invocan: creación/llegada de un hilo, wrapper de E/S, yield, fin de hilo. **No** se entera del fin de quantum
- Fin de quantum del KLT: el ULT que estaba ejecutando queda "a medias"; cuando el KLT vuelve, sigue ese mismo ULT (salvo biblioteca con desalojo que replanifique al retomar)
- Biblioteca sin desalojo: el ULT sigue hasta terminar la ráfaga o bloquearse
- Biblioteca con desalojo (ej: SRT): puede desalojar cuando llega un ULT nuevo más corto o vuelve uno de E/S (la biblioteca se entera por el wrapper)

### E/S de un ULT
- **Sin jacketing**: la E/S bloquea a todo el KLT/proceso. El SO pasa a otro KLT
  - Si la E/S es por **wrapper de la biblioteca**: la biblioteca guarda el contexto y al volver **replanifica** (elige con su algoritmo)
  - Si es una **syscall directa**: la biblioteca no se entera → al volver sigue el **mismo ULT**
- **Con jacketing**: la biblioteca hace la E/S no bloqueante → el KLT no se bloquea y sigue con otro ULT dentro de su quantum. Al terminar la E/S el ULT vuelve a ready de la biblioteca
- Jacketing no permite paralelismo entre ULTs (solo evita bloquear al KLT). Si no hay otro ULT listo, el KLT devuelve la CPU
- `exit()` desde un ULT termina todo el proceso (todos sus hilos)
- Llamadas a la biblioteca (sin syscall): crear hilo, replanificar. Con syscall: E/S (el wrapper llama a la syscall)

### Pasos típicos para preguntas "¿en qué instante cambia si hubiese jacketing?"
- Buscar el primer instante en que un ULT pide E/S **y al KLT todavía le quedaba quantum y otro ULT listo** → ahí cambia
- Si la E/S coincide con fin de quantum, el KLT igual iba a salir: el cambio se ve recién después

### KLTs
- Cada KLT se planifica por separado (como procesos). La E/S de un KLT solo bloquea a ese KLT
- Con varias CPUs, KLTs del mismo proceso pueden ejecutar en paralelo; ULTs del mismo KLT no

## Gantt "inverso" (deducir el algoritmo)
- SO: mirar cuánto ejecuta cada KLT seguido → si corta siempre en el mismo valor = RR con ese Q. Si un KLT que volvió de E/S pasa antes que otro que esperaba desde antes y ejecuta menos que Q → VRR (Q' = Q − usado). Sin cortes → FIFO/SJF
- Biblioteca: comparar la elección en cada instante con SJF (ráfaga más corta), FIFO (el que espera desde antes), HRRN ((W+S)/S). Si un ULT recién llegado corta al que ejecutaba → con desalojo
- Jacketing: si un ULT hace E/S y el mismo KLT sigue ejecutando otro ULT → hay jacketing. Si el KLT se bloquea teniendo quantum restante y ULTs listos → no hay

## Semáforos / locks en el Gantt
- `wait` sobre semáforo en 0 → el hilo pasa a bloqueado; `signal` lo pasa a ready (al final de la cola)
- Si wait/signal son atómicos deshabilitando interrupciones → el fin de quantum no los corta (se atiende al terminar la syscall)
- Locks exclusivos que no se liberan + pedidos cruzados → deadlock (marcar en el Gantt)
- Recurso con N instancias: si están todas ocupadas, el proceso espera (cola del recurso)
