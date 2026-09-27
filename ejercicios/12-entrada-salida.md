# 12 - Entrada/Salida y disco (ejercicios)

## Teóricos

### [Resumen] Diferencias entre E/S síncronas y asíncronas. ¿Función de los buffers en cada caso?
- Síncrona: el proceso necesita la respuesta para continuar. Asíncrona: sigue ejecutando mientras espera
- Síncrona: el buffer se llena y vacía con la operación. Asíncrona: el buffer puede ir llenándose y el SO lo vacía cuando le conviene (mejor performance)

### [Resumen] ¿Qué métrica optimizan los algoritmos de disco? Uno que la optimice y otro (no FIFO) que respete más el orden de llegada
- El tiempo de búsqueda (movimiento del brazo entre pistas)
- SSTF la optimiza: atiende siempre la pista más cercana (menor tiempo de espera promedio)
- N-step-SCAN respeta más el orden de llegada: agrupa en colas de hasta N pedidos (pierde algo de performance pero evita inanición)

### [Resumen] Compare SSTF, FSCAN y N-step-SCAN en tiempo de búsqueda y equidad. ¿Cuáles sufren inanición?
| | Tiempo de búsqueda | Equidad | Inanición |
|---|---|---|---|
| SSTF | Menor (siempre la más cercana) | Baja | Sí (ej: el cabezal en la pista 85 recibe pedidos para la 85 y cercanas → ignora las lejanas) |
| FSCAN | Mayor | Media | No |
| N-step-SCAN | Mayor, pero atiende todo | Alta | No |

### [Resumen] V/F
- Si un proceso necesita la respuesta de una E/S para continuar, probablemente se programe asíncrona → **F**, síncrona
- FSCAN y N-step-SCAN evitan la inanición → **V**: los pedidos nuevos van a otra cola que se atiende después
- Todos los algoritmos de disco pueden sufrir inanición → **F**: FIFO, FSCAN y N-step-SCAN no

### [Resumen] N-step-SCAN vs FSCAN. ¿Qué tienen de particular?
- N-step usa x colas de N pedidos; FSCAN 2 colas (activa/pasiva). Ambos atienden cada cola con SCAN
- A diferencia del resto (salvo FIFO), no generan inanición: garantizan que todos los pedidos se atienden

### [Resumen] Un caso de E/S asíncrona útil. ¿Cómo implementar una E/S síncrona no bloqueante?
- Asíncrona: cuando el proceso puede hacer otras tareas mientras espera (ej: pedir datos por red y seguir procesando)
- Síncrona no bloqueante: hacer la syscall no bloqueante y preguntar en loop si ya está (espera activa)

### [Final 2022-08-02] Luego de un `write(file, message, size)` exitoso, el mensaje podría no estar escrito en disco todavía, y en ese caso el proceso debería evitar modificar "message" por la E/S en curso — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Es cierto que puede no estar en disco aún, pero `write` copia los datos al buffer del kernel antes de retornar: el proceso puede modificar su variable libremente

### [Final 2023-03-07] La principal desventaja de la E/S con DMA es que la CPU debe encargarse de transferir la información entre el controlador y la memoria — ✅ solución oficial
- **F**. El DMA justamente libera a la CPU de esa tarea (oficial)

### [Final 2023-04-25] El DMA configurado bajo un bus de datos permite que el intercambio entre el DMA y el módulo de E/S use el bus de control — ✅ solución oficial
- **F**. El intercambio DMA ↔ E/S se hace por el bus de datos; por el bus de control pasan los datos que comparten la memoria y el procesador (oficial)

### [Final 2023-09-27] El método SCAN para la lectura de un disco puede tener inanición — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Si siguen llegando pedidos para la pista donde está el cabezal, los demás esperan indefinidamente

### [Final 2024-02-20] En entornos sin E/S asincrónicas, estas pueden simularse con hilos — ✅ solución oficial
- **V**. Un hilo hace la E/S sincrónica y el resto del proceso sigue ejecutando (oficial)

### [Final 2024-07-23] La ventaja de las E/S no bloqueantes es que los procesos obtendrán lo solicitado sin bloquearse — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Retornan enseguida pero pueden no tener el resultado (error / reintentar); el proceso tiene que volver a consultar. La ventaja es poder seguir ejecutando

## Prácticos

### [Final 2022-09-07] HDD con 900 cilindros, 10 sectores por pista y 10 cabezales. Pedidos (sectores lógicos): 600, 2218, 2220, 900, 6710, 10000, 4362, 541, 5890, 6524, 9801. Cabezal en el sector 2300 (antes leyó el 0), 1 ms entre pistas. a) Orden con SSTF, SCAN y C-SCAN b) Tiempo total de SCAN — ✅ solución oficial (con observaciones propias)
- Sectores por cilindro = 10 × 10 = 100 → cilindro = sector div 100
- 600 → 6, 2218 → 22, 2220 → 22, 900 → 9, 6710 → 67, 10000 → 100, 4362 → 43, 541 → 5, 5890 → 58, 6524 → 65, 9801 → 98. Cabezal en 23, subiendo (antes leyó el 0)

a) (oficial)
- SSTF: 2218 - 2220 - 900 - 600 - 541 - 4362 - 5890 - 6524 - 6710 - 9801 - 10000
- SCAN: 4362 - 5890 - 6524 - 6710 - 9801 - 10000 - 2218 - 2220 - 900 - 600 - 541
- C-SCAN: 4362 - 5890 - 6524 - 6710 - 9801 - 10000 - 541 - 600 - 900 - 2218 - 2220

b) Tiempo SCAN (la resolución oficial pega la vuelta en el último pedido, el cilindro 100):
- (43−23) + (58−43) + (65−58) + (67−65) + (98−67) + (100−98) + (100−22) + (22−9) + (9−6) + (6−5)
- = 20 + 15 + 7 + 2 + **31** + 2 + 78 + 13 + 3 + 1 = **172 ms**

> ⚠️ **Error en la solución oficial:** pone (98 − 67) = 21 en vez de 31 y da 162 ms. Además, lo que calcula como SCAN pega la vuelta en el último pedido (cilindro 100), que es en realidad LOOK.

- Si SCAN llega hasta el último cilindro (899) antes de volver: (899 − 23) + (899 − 5) = 876 + 894 = **1770 ms**
