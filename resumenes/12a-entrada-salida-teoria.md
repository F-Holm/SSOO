# 12 - Entrada / Salida (teoría)

## Dispositivos
- Tipos: comprensibles por el usuario (mouse, teclado, impresora), por el sistema (disco), de comunicación (placa de red, módem)
- Diferencias: velocidad de transferencia, uso, complejidad, unidad de transferencia (carácter / bloque), condiciones de error, bloqueante o no, síncrono o asíncrono, acceso secuencial o aleatorio, compartible o dedicado, lectura/escritura

## Bloqueante / no bloqueante, síncrona / asíncrona
- Bloqueante: el proceso se bloquea hasta la interrupción de fin de evento
- No bloqueante: sigue ejecutando; cuando necesita el resultado puede estar o no (si no, error/reintentar → tiene que volver a consultar). No garantiza obtener lo pedido sin bloquearse
- Síncrona: el proceso necesita la respuesta para seguir → si la necesita para continuar se programa síncrona
- Asíncrona: no necesita la respuesta ya, hace otras cosas mientras
- Combinaciones: síncrona-bloqueante (la más común), asíncrona-no bloqueante
- Síncrona no bloqueante: espera activa preguntando si ya está
- Si no hay E/S asíncrona se puede simular con hilos (un hilo hace la E/S síncrona y el resto sigue)
- Para programadores inexpertos: syscalls bloqueantes (código secuencial simple; el proceso pasa a Blocked y no consume CPU)

## Estructura
- SO (syscall `read`) → módulo de E/S del SO → **driver** (SW del fabricante, uno por SO) → **controladora** (HW) → dispositivo
- Objetivo: tratar a todos los dispositivos igual (interfaz uniforme)

## Técnicas de transferencia
- **E/S programada**: la CPU hace todo y pregunta en loop si el dispositivo está listo → espera activa, desperdicio de CPU
- **E/S por interrupciones**: el dispositivo avisa con una interrupción, pero la CPU sigue copiando los datos
- **DMA**: módulo aparte que transfiere entre dispositivo y memoria; la CPU le indica tipo de operación, dirección del dispositivo, cantidad de bytes y ubicación en memoria. Interrumpe al terminar
  - Libera a la CPU de la transferencia (esa era la desventaja de las anteriores)
  - Comparte el bus con la CPU → "roba ciclos" de bus (impacto mínimo por las cachés)
  - Con DMA configurado sobre el bus de datos, el intercambio DMA ↔ E/S usa el bus de datos (no el de control)
  - Todas las PC actuales lo usan

## Buffering
- Buffers = espacios de memoria del kernel para datos en tránsito
- Adaptar unidad de transferencia (leo 25 bytes de un bloque de 50) y diferencias de velocidad
- Síncrona: el buffer se llena y vacía con la operación. Asíncrona: el SO lo vacía cuando le conviene
- `write()` copia los datos al buffer del kernel y retorna → el proceso puede modificar su variable aunque aún no esté en disco
- Buffering de páginas (víctimas en buffer → se escriben en grupo, si se piden de nuevo no hay acceso a disco)
- Leer más bytes por syscall reduce syscalls, no la cantidad de bloques leídos de disco

## Disco
- Plato(s) con 1 o 2 caras, un cabezal por cara, pistas, sectores. Cilindro = mismas pistas de todas las caras
- Dirección física: CHS (cilindro, cabezal, sector). Lógica: sectores numerados 0..N (se numera completando cilindros)
- Tiempo de acceso: **TA = TB + LR + TT**
  - TB (tiempo de búsqueda): mover el brazo a la pista → lo que optimiza la planificación
  - LR (latencia rotacional): esperar que el sector pase bajo la cabeza
  - TT (transferencia): el único "constante"

## Algoritmos de planificación de disco (optimizan el tiempo de búsqueda)
| Algoritmo | Cómo | Dirección | Topes | Colas | Starvation |
|---|---|---|---|---|---|
| FCFS/FIFO | Orden de llegada | No | - | 1 | No |
| SSTF | El más cercano al cabezal | Solo para desempatar | - | 1 | Sí |
| SCAN (ascensor) | Barre en un sentido hasta el tope y vuelve | Sí | Va hasta la última pista | 1 | Sí |
| C-SCAN | Atiende en un solo sentido, llega al tope y salta al otro extremo | Siempre la misma | Sí | 1 | Sí |
| LOOK | Como SCAN pero hasta el último pedido | Sí | No | 1 | Sí |
| C-LOOK | Como C-SCAN pero hasta el último pedido y salta al primero | Siempre la misma | No | 1 | Sí |
| FSCAN | 2 colas: activa (se atiende con SCAN) y pasiva (llegan los nuevos). Al vaciarse la activa, se intercambian | Sí | Sí | 2 | No |
| N-step-SCAN | Colas de hasta N pedidos, cada una con SCAN | Sí | Sí | x colas de N | No |
- SSTF mejora el tiempo de espera promedio (menor búsqueda), baja equidad
- SCAN tiene inanición cuando siguen llegando pedidos de la pista actual
- FSCAN y N-step-SCAN privilegian el orden de llegada, sin inanición (más tiempo de búsqueda, más equidad)
- Sin inanición: FIFO, FSCAN, N-step-SCAN

## RAID
- Array de discos en paralelo; por SW (SO) o HW (controladora)
- RAID 0: striping (bandas), sin redundancia, no tolera fallos, overhead bajo
- RAID 1 / 0+1: espejado (copia de cada disco), tolera la caída de una copia de cada disco
- RAID 3/4: bandas + disco de paridad dedicado (cuello de botella), tolera 1 disco
- RAID 5: paridad distribuida en todos los discos, tolera 1 disco (cualquiera), overhead medio
- RAID 6: doble paridad distribuida, tolera 2 discos

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
