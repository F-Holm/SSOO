# 07 - Deadlock (práctica)

## Matrices
- Recursos totales (vector)
- Asignados (MA): lo que tiene cada proceso
- Máximas necesidades (MM): lo que puede llegar a pedir (solo evasión)
- Necesidades pendientes = MM − MA
- Peticiones actuales: lo que está pidiendo ahora (detección)
- Disponibles = Totales − Σ asignados (columna por columna)

## Algoritmo del banquero (¿estado seguro?)
1. Pendientes = MM − MA
2. Calcular Disponibles
3. Buscar un proceso cuyas pendientes ≤ Disponibles (componente a componente)
4. "Finalizarlo": Disponibles += su fila de asignados
5. Repetir. Si terminan todos → estado seguro (anotar la secuencia segura). Al final Disponibles = Totales (sirve para chequear)
6. Si quedan procesos que no pueden terminar → inseguro

## Simular una solicitud (¿se le puede dar X a P?)
1. ¿Solicitud ≤ pendientes de P? (si no, error: pide más que su máximo)
2. ¿Solicitud ≤ Disponibles? (si no, P espera)
3. Simular: Disponibles −= solicitud; MA[P] += solicitud; Pendientes[P] −= solicitud
4. Correr el algoritmo de estado seguro. Seguro → se asigna. Inseguro → se deniega (P espera)
- Si varios piden en orden FCFS, evaluar de a uno actualizando el estado cada vez que se asigna

## Algoritmo de detección
1. Disponibles = Totales − Σ asignados
2. Descartar los procesos sin recursos asignados (no pueden estar en deadlock: a lo sumo en inanición)
3. Finalizar los que no piden nada → sumar sus asignados a Disponibles
4. Buscar uno con Peticiones ≤ Disponibles → finalizarlo y sumar sus asignados
5. Repetir hasta que no se pueda. Los que quedan (con recursos asignados) están en **deadlock**
- Un proceso sin recursos asignados que no puede avanzar → bloqueado/inanición, no deadlock

## Recuperación
- Elegir víctima según criterio del enunciado (ej: "más recursos asignados = más cerca de terminar" → matar al que menos tiene)
- Sumar sus asignados a Disponibles y volver a correr detección → ¿se resolvió o hay que matar otro?

## Grafo
- Dibujar P → R (pide) y R → P (asignado), un punto por instancia
- Ciclo con recursos de 1 instancia = deadlock

## Deadlock con semáforos / locks
- Buscar dos procesos/hilos que tomen dos recursos en orden inverso (wait(A); wait(B) vs wait(B); wait(A))
- Encontrar una traza: P1 toma A, se lo interrumpe (ej: RR o FIFO con E/S), P2 toma B, P2 pide A (bloq), P1 pide B (bloq)
- Solución: mismo orden de pedido en todos (prevención de espera circular), achicar la SC, o sacar el mutex si ya hay orden garantizado
- Con semáforos con bloqueo el deadlock no consume CPU (no afecta a otros procesos). Con espera activa → livelock que sí consume CPU
