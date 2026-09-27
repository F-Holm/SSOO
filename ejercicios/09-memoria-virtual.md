# 09 - Memoria virtual (ejercicios)

## Teóricos

### [Resumen] Dos maneras que tiene el HW para mejorar la eficiencia de la gestión de memoria
- MMU (traducción rápida por HW), TLB (evita accesos a la tabla de páginas), disco de swap, bits de uso/modificado actualizados por HW

### [Resumen] V/F: La corrupción total de la tabla de páginas no impediría seguir ejecutando si hay VM, el proceso no escribió páginas y hay RAM libre
- **V**. El SO podría crear una tabla nueva e ir cargando las páginas desde disco en marcos nuevos (hace falta RAM libre porque los marcos viejos quedan sin poder liberarse, y que no haya escrituras para que la copia en disco esté actualizada)

### [Cuestionario 05-30] Asignación y sustitución
- Asignación fija con sustitución global no tiene sentido
- Asignación dinámica con sustitución local: en cada PF hay que decidir si darle otro frame o sustituir; si siempre se le da, se puede llevar más frames de los que debería
- No se pueden tener menos PF que con el óptimo (ve el futuro, no implementable)
- LRU suele ser el mejor algoritmo implementable

### [Final 2022-08-02] Si la utilización de CPU es muy baja, siempre es recomendable aumentar el grado de multiprogramación
- **F**. Si la causa es thrashing (procesos bloqueados por PF), aumentar la multiprogramación lo empeora: hay que disminuirla

### [Final 2022-09-07] Una referencia puede provocar uno o más page faults con paginación jerárquica y VM
- **V**. Un PF para traer la porción de la tabla de páginas y otro para la página (oficial)

### [Final 2023-02-14] En segmentación, al traer o remover un segmento alcanza con actualizar el bit de presencia
- **F**. Al traerlo hay que actualizar la base (se ubica donde haya lugar, tamaño variable) y al removerlo, si fue modificado, escribirlo a disco

### [Final 2023-02-28 / 2024-05-10] Paginación multinivel con VM hace que el tamaño de las tablas en RAM sea en general menor que con paginación tradicional
- **V**. Solo se cargan las partes de la tabla que se usan (oficial)

### [Final 2023-08-01] No es posible que un proceso entre en thrashing si es el único en ejecución
- **F**. Si tiene menos frames asignados que los que necesita puede generar thrashing igual (oficial)

### [Final 2024-02-20] Con una alta tasa de TLB hits, es posible que un page fault no genere accesos a disco
- **F**. Un PF siempre accede a disco al menos una vez para traer la página (oficial)

### [Final 2024-07-30] La sobrepaginación puede detectarse cuando el consumo del procesador es alto pero los procesos permanecen bloqueados
- **F**. En thrashing el uso de CPU es **bajo** (los procesos están bloqueados esperando PF) y la actividad de disco es alta

### [Final 2025-02-11 / 2025-12-09] El principio de localidad es indispensable para el funcionamiento óptimo de la VM
- **V**. Sin localidad habría PF constantemente (thrashing)

### [Final 2025-02-18] El uso de memoria virtual permite que los procesos estén bloqueados menos tiempo
- **F**. Agrega bloqueos por PF (acceso a disco). Su ventaja es la multiprogramación y ejecutar procesos más grandes que la RAM

### [Final 2025-02-25] Paginación con VM garantiza que el tamaño de las estructuras administrativas en memoria sea siempre el mínimo posible
- **F**. La tabla de páginas (proporcional al espacio virtual) sigue en memoria; multinivel o invertida la reducen pero no garantizan un mínimo

### [Final 2025-05-20] En VM, los algoritmos de reemplazo y el thrashing están relacionados
- **V**. Un mal algoritmo de reemplazo (saca páginas que se van a usar) aumenta los PF y es una de las causas del thrashing

### [Final 2025-07-15] Si los procesos nunca son más grandes que la RAM, la VM no genera ningún beneficio
- **F**. Permite más grado de multiprogramación (cargar solo lo que se usa de muchos procesos), carga más rápida, compartir páginas

### [Final 2025-12-09] LRU necesita la colaboración del hardware para implementarse
- **V**. Hay que registrar el instante de cada referencia en cada acceso a memoria: solo es viable con soporte de HW (contador/timestamp o bits de referencia); por software implicaría al SO en cada acceso

### [Final 2026-02-10] Con paginación simple y VM, la MMU y la TLB favorecen el rendimiento porque generan menos page faults
- **F**. Favorecen el rendimiento por otro motivo: la MMU traduce por HW y la TLB reduce los accesos a memoria. No reducen PF (oficial)

### [Final 2023-07-25 N°6] ChatGPT: "No sería posible lograr los beneficios de la VM sin direcciones lógicas: la VM es una abstracción donde el espacio que ve el proceso es distinto del físico". Evaluar y relacionar con los PF
- **Correcta**. El proceso siempre usa la misma DL; la MMU la traduce a (página, offset) y busca la página: si P = 0 lanza un PF, el SO la trae a cualquier marco libre y actualiza la tabla. Como el proceso solo conoce la DL, no le importa en qué marco quedó ni si estuvo en disco. Sin DL no se podría reubicar una página en otro marco al volver de disco

## Prácticos

### [Clase] Cadena 2 3' 2 1 5' 2 4' 5 3 2' 5 2 con 3 frames (LRU, Clock, Clock modificado)
- Ver [resumen 09b](../resumenes/09b-memoria-virtual-practica.md): LRU 7 PF / 9 accesos, Clock 8 PF / 11 accesos, Clock modificado 9 PF / 12 accesos

### [Final 2022-08-02] Segmentación paginada con VM, punteros de 16 bits, marcos de 4 KiB, TLB de 4 entradas, máximo 4 segmentos (solo existen 0, 1 y 2)
| Seg 0 (RW-) | Seg 1 (-X-) | Seg 2 (R--) | TLB |
|---|---|---|---|
| p0: F6, P1 U0 | p0: F9, P1 U0 | p0: FA, P0 U0 | B → F8 |
| p1: F9, P0 U0 | p1: FB, P1 U1 | p1: F1, P0 U0 | 3 → F1 |
| p2: F11, P1 U1 | p2: F7, P1 U0 | p2: FC, P0 U0 | (resto vacío) |

Formato: segmento 2 bits (4 segmentos) | página 2 bits | offset 12 bits (4 KiB). La TLB usa como etiqueta seg+página (4 bits): B = 10 11 → seg 2 pág 3; 3 = 00 11 → seg 0 pág 3

a) Escrituras (Clock, reemplazo global, asignación dinámica)
- 3C38h = 00 11 C38 → seg 0, pág 3 → TLB hit → marco 1 → seg 0 es RW → **DF = 1C38h**
- FFACh = 11 11 FAC → seg 3 → el proceso no tiene segmento 3 → **segmentation fault** (no genera dirección)
- AA00h = 10 10 A00 → seg 2, pág 2 → seg 2 es R-- → escritura no permitida → **error de protección** (no genera dirección; ni siquiera se trata el PF)
- No hay reemplazos: Clock no llega a usarse

b) ¿Uno de los segmentos puede ser una biblioteca compartida usada con muy poca frecuencia?
- El código de una biblioteca necesita permiso X → solo podría ser el seg 1, pero tiene todas sus páginas presentes y una con U = 1 → se usa seguido
- El seg 2 tiene sus páginas ausentes (poco uso) pero no es ejecutable → no puede ser código de biblioteca (a lo sumo datos de solo lectura)
- → No parece ser así

### [Final 2023-02-28 / 2024-02-27 / 2024-05-10] Paginación bajo demanda, DL de 16 bits, hasta 4 frames por proceso, sustitución local LRU (oficial)
| Página | 0 | 1 | 2 | 3 | 4 | 5 | 6-9 |
|---|---|---|---|---|---|---|---|
| Frame | 5 | - | - | 9 | 7 | 1 | - |
| Últ. ref | 200 | - | - | 400 | 300 | 100 | - |

Referencias: A000h, 0300h, 3500h, 1000h, 4100h, A001h. Todas válidas, A000h no produce PF, los frames se cargaron en orden ascendente

a) A000h = 1010 0000 0000 0000: la única forma de que no dé PF es 3 bits de página (101 = 5) y 13 de offset → A000h p5, 0300h p0, 3500h p1, 1000h p0, 4100h p2, A001h p5

| | 5 | 0 | 1 | 0 | 2 | 5 |
|---|---|---|---|---|---|---|
| Fr 1 | **5** | 5 | 5 | 5 | 5 | **5** |
| Fr 5 | 0 | **0** | 0 | **0** | 0 | 0 |
| Fr 7 | 4 | 4 | **1** | 1 | 1 | 1 |
| Fr 9 | 3 | 3 | 3 | 3 | **2** | 2 |
| | | | PF | | PF | |

- p1: LRU entre 5 (t nuevo), 0 (nuevo), 4 (300), 3 (400) → sale la 4. p2: LRU → sale la 3

b) Con FIFO cambia en la referencia a la página 1 (3500h): FIFO reemplaza por orden de carga (frames cargados en orden ascendente → el más viejo es el frame 1, página 5), sin importar que se acaba de usar

### [Final 2023-05-23] Referencias 1R, 2R, 3W, 2R, 3R, 4R, 1R, 2R, 6R, 5R, 3R, 1W, 2R con 3 frames. a) Estado de memoria con FIFO, LRU y Óptimo b) Tiempo de atención: PF sin remoción 3 ms, con remoción 20 ms, acceso a memoria 100 ns, sin TLB
(* = modificada)

FIFO
| Ref | 1 | 2 | 3W | 2 | 3 | 4 | 1 | 2 | 6 | 5 | 3 | 1W | 2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | 1 | 1 | 1 | 1 | 1 | 4 | 4 | 4 | 6 | 6 | 6 | 1* | 1* |
| F2 | | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 1 | 5 | 5 | 5 | 2 |
| F3 | | | 3* | 3* | 3* | 3* | 3* | 2 | 2 | 2 | 3 | 3 | 3 |
| PF | PF | PF | PF | | | PF | PF | PF (escribe 3) | PF | PF | PF | PF | PF |
- **11 PF** (3 sin remoción, 8 con remoción), 1 escritura

LRU
| Ref | 1 | 2 | 3W | 2 | 3 | 4 | 1 | 2 | 6 | 5 | 3 | 1W | 2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | 1 | 1 | 1 | 1 | 1 | 4 | 4 | 4 | 6 | 6 | 6 | 1* | 1* |
| F2 | | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 1 | 5 | 5 | 5 | 2 |
| F3 | | | 3* | 3* | 3* | 3* | 3* | 2 | 2 | 2 | 3 | 3 | 3 |
| PF | PF | PF | PF | | | PF | PF | PF (escribe 3) | PF | PF | PF | PF | PF |
- Queda igual que FIFO: **11 PF** (3 + 8), 1 escritura (la cadena no reusa páginas recientes)

Óptimo
| Ref | 1 | 2 | 3W | 2 | 3 | 4 | 1 | 2 | 6 | 5 | 3 | 1W | 2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1* | 1* |
| F2 | | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| F3 | | | 3* | 3* | 3* | 4 | 4 | 4 | 6 | 5 | 3 | 3 | 3 |
| PF | PF | PF | PF | | | PF (escribe 3) | | | PF | PF | PF | | |
- **7 PF** (3 sin remoción, 4 con remoción), 1 escritura

b) Cada referencia sin PF = 2 accesos (tabla + dato) = 200 ns; con PF = acceso a la tabla + PF + reejecución (2 accesos)
- FIFO y LRU: 3 × 3 ms + 8 × 20 ms = 169 ms + (13 × 200 ns + 11 × 100 ns = 3,7 µs) ≈ **169,004 ms**
- Óptimo: 3 × 3 + 4 × 20 = 89 ms + (2,6 µs + 0,7 µs) ≈ **89,003 ms**

### [Final 2023-08-01] Paginación bajo demanda, TLB de 6 entradas (= 1% de los frames) con reemplazo LRU; tablas con Clock modificado y sustitución local. PB tiene 3 páginas en memoria (último acceso: escritura en DL A55093h). Marcos de 1 MiB (oficial)
| Pág PB | Marco | P | U | M |
|---|---|---|---|---|
| 0 | 10 | 0 | 0 | 0 |
| 1 | 11 | 1 | 1 | 0 |
| 2 | 13 | 1 | 0 | 1 |
| 3 | 12 | 0 | 0 | 0 |
| 4 | 12 | 0 | 0 | 0 |
| 5 | - | 0 | - | - |

| TLB | Pr | Pág | Marco | TUR |
|---|---|---|---|---|
| 0 | PA | 1 | 10 | 100 |
| 1 | PB | 1 | 11 | 120 |
| 2 | PA | 12 | 12 | 90 |
| 3 | PB | 2 | 13 | 60 |
| 4 | PB | 10 | 14 | 121 |
| 5 | PC | 20 | 16 | 80 |

- Marco de 1 MiB → offset 20 bits. A55093h → pág Ah = 10 (entrada 4 de la TLB, marco 14, TUR más alto): es la 3ra página presente de PB, con U = 1 y M = 1
- a) Asignación **dinámica**: PB tuvo los frames 10 y 12 que ahora tiene PA (según la TLB)
- b) 1400001h → pág 14h = 20; 511102h → pág 5. No están presentes → Clock modificado reemplaza en orden a la pág 1 y a la pág 2 → la pág 20 va al **marco 11** y la pág 5 al **marco 13**
- c) Por LRU de la TLB: el marco 13 queda en la **posición 3** y el marco 11 en la **posición 5**
- d) 6 entradas = 1% → 600 frames → bitmap de 600 bits = **75 bytes**

### [Final 2024-02-20] P1: 10 11 12 13 14 15 16 17 (repite); P2: 10 11 0 3 4 3 11 4 0 3 4 3 0 11 (repite). 4 frames por proceso, LRU local, TLB de 2 entradas FIFO. a) ¿Mejora una TLB de 4 entradas? b) ¿Thrashing? (oficial)
- a) Solo mejora P2: repite pocas páginas (0, 3, 4, 11) varias veces → con 4 entradas hay muchos más hits y se ahorran accesos a la tabla. P1 recorre 8 páginas distintas en ciclo → nunca hay hit
- b) P1 genera thrashing: su localidad es de 8 páginas y tiene 4 frames → con LRU todas las referencias son PF. Se soluciona asignándole 8 frames. P2 usa 5 páginas distintas por ciclo, pero casi todas sus referencias son a 0, 3, 4 y 11 → pocos PF por ciclo (solo al pasar por la 10), no entra en thrashing

### [Final 2025-02-11] DL de 32 bits, RAM de 32 MiB en 8192 frames, asignación fija de 3 frames, sustitución local. Referencias: 8100 (E), 9000 (L), 40990 (E), 3500 (L). a) LRU b) Clock modificado
| Marco | Página | U | M | Inst. carga | Inst. última ref |
|---|---|---|---|---|---|
| 2 | 10 | 1 | 1 | 29 | 29 |
| 6 (←puntero) | 17 | 0 | 0 | 11 | 11 |
| 8 | 19 | 1 | 0 | 12 | 12 |

Tamaño de frame = 32 MiB / 8192 = 4 KiB → pág = DL div 4096: 8100 → p1 (E), 9000 → p2 (L), 40990 → p10 (E), 3500 → p0 (L)

a) LRU
| Ref | Inicial | p1 (E) | p2 (L) | p10 (E) | p0 (L) |
|---|---|---|---|---|---|
| M2 | 10* (29) | 10* | 10* | 10* (32) | 10* |
| M6 | 17 (11) | 1* (30) | 1* | 1* | 0 (33) |
| M8 | 19 (12) | 19 | 2 (31) | 2 | 2 |
| | | PF | PF | hit | PF (escribe p1) |
- 3 PF; se escribe a disco la **página 1**

b) Clock modificado (orden circular M2 → M6 → M8, puntero en M6). Notación página(U,M)
| Ref | M2 | M6 | M8 | Puntero | Nota |
|---|---|---|---|---|---|
| Inicial | 10 (1,1) | 17 (0,0) | 19 (1,0) | M6 | |
| p1 (E) | 10 (1,1) | 1 (1,1) | 19 (1,0) | M8 | PF: 1ra pasada encuentra (0,0) en M6 |
| p2 (L) | 10 (0,1) | 1 (0,1) | 2 (1,0) | M2 | PF: 1ra pasada nada; 2da pone U=0 en M8, M2, M6 sin encontrar (0,1) antes de dar la vuelta; 3ra pasada: M8 (0,0) víctima (p19 sin escribir) |
| p10 (E) | 10 (1,1) | 1 (0,1) | 2 (1,0) | M2 | hit |
| p0 (L) | 10 (0,1) | 0 (1,0) | 2 (1,0) | M8 | PF: 1ra pasada nada; 2da: M2 → U=0, M6 (0,1) víctima → **escribe p1** |
- 3 PF; se escribe a disco la **página 1**

### [Final 2025-02-25] Paginación bajo demanda, asignación fija de 4 frames, sustitución local, DL de 32 bits, fragmentación máxima 8191 B, proceso de 159 KB. Algoritmo Clock
| Puntero | Marco | Página | U | M | Inst. ref |
|---|---|---|---|---|---|
| | 1 | 14 | 1 | 1 | 28 |
| | 3 | 17 | 1 | 0 | 3 |
| | 5 | 19 | 1 | 1 | 15 |
| → | 8 | - | - | - | - |

- Frag. máx 8191 → página de 8 KiB (8192). El proceso tiene ⌈159 / 8⌉ = 20 páginas (0 a 19)
- Referencias: 100 (L) → p0; 122950 (E) → p15; 98306 (L) → p12; 139264 (E) → p17; 122880 (L) → p15; 155650 (E) → p19; 172100 (L) → p21 → **inválida**; 100 (L) → no llega a ejecutarse

| Ref | M1 | M3 | M5 | M8 | Puntero | PF | Escritura |
|---|---|---|---|---|---|---|---|
| Inicial | 14 (1,1) | 17 (1,0) | 19 (1,1) | - | M8 | | |
| p0 (L) | 14 (1,1) | 17 (1,0) | 19 (1,1) | 0 (1,0) | M1 | PF (marco libre) | |
| p15 (E) | 15 (1,1) | 17 (0,0) | 19 (0,1) | 0 (0,0) | M3 | PF (da la vuelta: U=0 en todos, víctima p14) | p14 |
| p12 (L) | 15 (1,1) | 12 (1,0) | 19 (0,1) | 0 (0,0) | M5 | PF (víctima p17) | |
| p17 (E) | 15 (1,1) | 12 (1,0) | 17 (1,1) | 0 (0,0) | M8 | PF (víctima p19) | p19 |
| p15 (L) | 15 (1,1) | 12 (1,0) | 17 (1,1) | 0 (0,0) | M8 | hit | |
| p19 (E) | 15 (1,1) | 12 (1,0) | 17 (1,1) | 19 (1,1) | M1 | PF (víctima p0) | |
| p21 (L) | - | - | - | - | - | dirección inválida → se finaliza el proceso | |
- **5 PF**, páginas escritas a disco: **14 y 19**

### [Final 2025-05-20] Proceso de 21 KiB, paginación bajo demanda, asignación fija, reemplazo local, máximo 6 frames por proceso; otra instancia del mismo programa. Páginas 0 y 1 = código y constantes (siempre usadas); el resto, una estructura que se recorre de inicio a fin repetidas veces. Memoria de 40 KiB. El proceso completo ocuparía 6 páginas
- 21 KiB en 6 páginas → páginas de 4 KiB → la memoria tiene 10 frames → cada proceso tiene 5 frames (no 6): páginas 0 y 1 fijas + 3 frames para las 4 páginas de datos recorridas en ciclo
- a)
  1. **Cierta**: recorrido cíclico de 4 páginas con 3 frames (LRU/FIFO) → cada acceso a una página de datos es PF, en los dos procesos
  2. **Cierta** (si los datos pueden compartirse): con hilos de un mismo proceso el código y las constantes se cargan una sola vez y comparten la estructura → 6 páginas en 6 frames sin PF. Si cada hilo necesitara su propia estructura, un único proceso quedaría limitado a 6 frames y empeoraría
  3. **Cierta**: el proceso restante puede tener sus 6 frames → entra entero → sin PF (menos accesos a disco)
- b) **Compartir las páginas de código/constantes** (0 y 1) entre las dos instancias (solo lectura): ambas tablas apuntan a los mismos marcos → 2 + 4 + 4 = 10 frames = toda la memoria, cada proceso con sus 6 páginas residentes → sin PF
```
Proceso 1: p0 → M0, p1 → M1, p2 → M2, p3 → M3, p4 → M4, p5 → M5
Proceso 2: p0 → M0, p1 → M1, p2 → M6, p3 → M7, p4 → M8, p5 → M9   (M0 y M1 compartidos, R-X)
```

### [Final 2025-07-29] Paginación simple con VM, Clock, asignación variable, reemplazo local. 3 marcos por proceso, sin marcos libres. A: 0 4 0 4 0 4 ... (lecturas); B: 1 2 3 4 1 2 3 4 ...
a) PF en las primeras 9 referencias
- A: solo las dos primeras (0 y 4) → **2 PF**
- B: 1, 2, 3 (carga); 4: todos con U = 1 → da la vuelta y saca al 1; 1 saca al 2; 2 saca al 3; 3: da la vuelta, saca al 4; 4 saca al 1; 1 saca al 2 → **9 PF** (todas)

b) Conjunto residente: A usa 2 páginas → darle 2 marcos (liberar 1); B necesita 4 → darle 4 marcos. B tendría solo los 4 PF iniciales

c) `fork()` de A crea C: con **copy-on-write**, C comparte los marcos de A (solo se copian si alguien escribe; A solo lee) → no hace falta cargar nada para C
```
A: p0 → M10 (P=1), p4 → M11 (P=1)
C: p0 → M10 (P=1, solo lectura/COW), p4 → M11 (P=1, solo lectura/COW)
```

d) B luego de la 9ª referencia: M20 = 3 (U=1), M21 = 4 (U=1), M22 = 1 (U=1), puntero en M20. Referencias 5, 2, 1, 7:
- 5: PF, da la vuelta (U=0 en todos), víctima 3 → M20 = 5, puntero M21
- 2: PF, M21 (4, U=0) víctima → M21 = 2, puntero M22
- 1: hit (U=1)
- 7: PF, M22 (U=1 → 0), M20 (U=1 → 0), M21 (U=1 → 0), M22 (U=0) víctima 1 → M22 = 7
```
Tabla de B: p5 → M20 (P=1, U=0), p2 → M21 (P=1, U=0), p7 → M22 (P=1, U=1); p1, p3, p4 → P=0
```

### [Final 2025-09-25] Paginación bajo demanda, asignación fija de 2 frames de 1 MiB por proceso, paginación jerárquica de 2 niveles (mismos bits por nivel), DL de 32 bits, LRU. TLB solo con (página, frame). P1 ejecutando; luego se carga P2 (frames 50 y 60). TLB: 223h → 11, 001h → 30
Referencias: P1 223A0B34h (E), P1 00111001h (L), P1 01022222h (E), P2 010AA221h (E)

a) Offset 20 bits (1 MiB); quedan 12 bits de página → **6 bits 1er nivel | 6 bits 2do nivel | 20 bits offset** (página = 3 dígitos hex)

b)
| Ref | Página | TLB | i) Lee TP | ii) Modifica TP | iii) Escribe a disco | iv) DF |
|---|---|---|---|---|---|---|
| P1 223A0B34h (E) | 223h | hit (11) | No | Sí (bit M, la TLB no lo guarda) | No | 0BA0B34h |
| P1 00111001h (L) | 001h | hit (30) | No | No (salvo registrar el acceso para LRU) | No | 1E11001h |
| P1 01022222h (E) | 010h | miss → PF | Sí (2 niveles) | Sí (223h P=0; 010h P=1 en marco 11) | **Sí**: víctima LRU = 223h, modificada | 0B22222h |
| P2 010AA221h (E) | 010h | miss (TLB vaciada) → PF | Sí | Sí (010h P=1 en marco 50) | No (marco libre) | 32AA221h |
- DF = marco × 2^20 + offset (11 = Bh, 30 = 1Eh, 50 = 32h)

c) Al cambiar a P2 hay que vaciar la TLB (sus entradas no tienen PID: la página 010h existe en ambos procesos). TLB final: **010h → 50**

### [Final 2025-12-16] PC de 32 bits, páginas de 4 KiB, asignación fija de 3 frames, sustitución local. Proceso de 6 páginas con la mayor fragmentación interna posible. Clock (oficial)
| Puntero | Página | Frame | U | M |
|---|---|---|---|---|
| → | 1 | 11 | 1 | 1 |
| | 4 | 12 | 0 | 1 |
| | 3 | 13 | 1 | 0 |

Referencias: 3000 → p0; 4321 → p1; 18123 → p4; 20495 → p5 offset 10 → **inválida** (la pág 5 tiene 4095 B de fragmentación: solo es válido su byte 0) → se finaliza el proceso
| Frame | Inicial | p0 | p1 | p4 | p5 |
|---|---|---|---|---|---|
| 11 | →1* | 1 | 1* | →1 | dirección inválida |
| 12 | 4 | 0* | 0* | 0 | |
| 13 | 3* | →3* | →3* | 4* | |
| PF | | PF | | PF | 2 |
| Escritura | | Sí (p4, M=1) | | No | 1 |
(* = U en 1; → = puntero)

### [Final 2026-02-24] SO planifica KLTs con FIFO. Proceso 1: KLT1.1 y KLT1.2; proceso 2: 1 KLT con ULTs (biblioteca FIFO). Listos en orden de tabla. Paginación bajo demanda, 2 frames por proceso, LRU, páginas de 1 KiB, sin TLB. Leer la tabla = 1 ut de CPU, acceder a la DF = 1 ut de CPU; mover una página disco ↔ memoria = 2 ut de disco (oficial)
| KLT 1.1 | KLT 1.2 | ULT 2.1 | ULT 2.2 |
|---|---|---|---|
| E(28999) → p28 | L(21999) → p21 | E(9333) → p9 | E(9222) → p9 |
| L(21601) → p21 | L(15621) → p15 | L(9444) → p9 | |

| | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| KLT 1.1 | TP | E/S | E/S | TP | Acc | TP | Acc | X | | | | | | | | | | | |
| KLT 1.2 | | TP | | E/S | E/S | | | TP | Acc | TP | E/S | E/S | E/S | E/S | | | TP | Acc | X |
| ULT 2.1 | | | TP | | | E/S | E/S | | | | TP | Acc | TP | Acc | X | | | | |
| ULT 2.2 | | | | | | | | | | | | | | | TP | Acc | X | | |
- Un solo disco (FIFO): p28 en 1-3, p21 en 3-5, p9 en 5-7
- t5: KLT1.1 encuentra p21 ya cargada por KLT1.2 → no hay PF
- t9: KLT1.2 pide p15 → PF; víctima LRU del proceso 1 = p28 (modificada) → 4 ut de disco (escribir p28 + leer p15) en 10-14
- Proceso 2: todos sus accesos son a p9 → un solo PF

### [Final 2026-05-19] Memoria física de 256 KiB, carga de segmentos bajo demanda, ubicación Worst Fit. Un único proceso (oficial)
| Segmento | Base | Tamaño | Presencia | Modificado |
|---|---|---|---|---|
| 0 | 140 KiB | 32 KiB | 1 | 0 |
| 1 | 50 KiB | 4 KiB | 1 | 1 |
| 2 | 0 KiB | 20 KiB | 0 | 1 |
| 3 | 200 KiB | 6 KiB | 1 | 1 |

- Ocupado: 50-54 (seg 1), 140-172 (seg 0), 200-206 (seg 3). Huecos: 0-50 (50 KiB), **54-140 (86 KiB)**, 172-200 (28 KiB), 206-256 (50 KiB)
- a) Traducciones:
  - (1 ; 3000): 3000 < 4 KiB → DF = 50 × 1024 + 3000 = **54.200**
  - (2 ; 1225): segmento no presente → se carga; Worst Fit elige el hueco más grande (54-140) → base 54 KiB → DF = 54 × 1024 + 1225 = **56.521**
  - (3 ; 8192): 8192 ≥ 6 KiB (6144) → **dirección inválida** (supera el tamaño del segmento)
- b) i) Estado final: seg 2 → base 54 KiB, presencia 1 (modificado: 0 si la referencia fue lectura; la resolución lo deja como "?"); el resto igual
- ii) La traducción (3 ; 8192) no se pudo hacer por exceder el tamaño del segmento
- iii) El segmento 0 (nunca modificado) podría ser el **código**
