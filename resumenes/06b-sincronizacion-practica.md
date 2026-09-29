# 06 - Sincronización (práctica: semáforos)

## Cómo encarar un ejercicio
1. Identificar recursos compartidos → un **mutex** (init 1) por recurso, rodeando SOLO la sección crítica
2. Identificar órdenes → semáforos **binarios** (init 0 o 1 según quién arranca)
3. Identificar límites/cantidades → semáforos **contadores** (init N)
4. Escribir la traza a mano y probar arrancar por cada proceso (¿alguien se bloquea para siempre?)
5. Chequear: ¿puede un proceso "adelantarse" y hacer dos vueltas? ¿quedan bien los valores al final del ciclo (igual que al inicio)?

## Errores típicos (ejercicios de "encontrá los errores")
- Mutex que encierra más de lo necesario (ej: `generarPost()` o `guardarEnDisco()` dentro) → lentitud
- `wait(mutex)` antes de `wait(contador)` → se bloquea con el mutex tomado → **deadlock**. Siempre primero el contador/orden y después el mutex
- Faltan signals o sobran → bloqueo permanente o se rompe el orden
- Semáforo inicializado mal (ej: binario en 0 para el que debe arrancar) → deadlock al inicio
- Productor/consumidor con `vacio = 1` → alternancia estricta (buffer de 1)
- Mutex innecesario sobre un recurso que usa un solo proceso

## Patrones
### Mutua exclusión

**Semáforos:** `mutex = 1`

| Cada proceso |
|---|
| `wait(mutex);` |
| `SC` |
| `signal(mutex);` |

### Orden A → B

**Semáforos:** `s = 0`

| A | B |
|---|---|
| `hacerA();` | `wait(s);` |
| `signal(s);` | `hacerB();` |

### Alternancia A, B, A, B...

**Semáforos:** `sA = 1` · `sB = 0`

| A | B |
|---|---|
| `while(1) {` | `while(1) {` |
| &emsp;`wait(sA);` | &emsp;`wait(sB);` |
| &emsp;`A();` | &emsp;`B();` |
| &emsp;`signal(sB);` | &emsp;`signal(sA);` |
| `}` | `}` |

### Máximo N en simultáneo (ej: "aprendizaje acepta 2 procesos")

**Semáforos:** `cant = N`

| Cada proceso |
|---|
| `wait(cant);` |
| `usar();` |
| `signal(cant);` |

### Máximo N pedidos pendientes (productor limitado)

**Semáforos:** `limite = N` · `pedidos = 0`

| Productor | Consumidor |
|---|---|
| `while(1) {` | `while(1) {` |
| &emsp;`wait(limite);` | &emsp;`wait(pedidos);` |
| &emsp;`pedir();` | &emsp;`atender();` |
| &emsp;`signal(pedidos);` | &emsp;`signal(limite);` |
| `}` | `}` |

### Productor / consumidor con buffer de N

**Semáforos:** `mutex = 1` · `lugar = N` · `cant = 0`

| Productor | Consumidor |
|---|---|
| `while(1) {` | `while(1) {` |
| &emsp;`x = producir();` | &emsp;`wait(cant);` |
| &emsp;`wait(lugar);` | &emsp;`wait(mutex);` |
| &emsp;`wait(mutex);` | &emsp;`x = sacar();` |
| &emsp;`agregar(x);` | &emsp;`signal(mutex);` |
| &emsp;`signal(mutex);` | &emsp;`signal(lugar);` |
| &emsp;`signal(cant);` | &emsp;`consumir(x);` |
| `}` | `}` |

- Buffer infinito: sacar `lugar`

### Un proceso que aparece varias veces en la secuencia (ej: A B A C A D ...)
- Opción 1: "desenrollar" el while del proceso repetido (cada vuelta hace varias impresiones con distintos signals). Es válido, el while sigue siendo infinito
- Opción 2: semáforo compartido + "tokens" de turno para los otros:

**Semáforos:** `sA = 1` · `sOtro = 0` · `tB = 1` · `tC = 0` · `tD = 0`

| A | B | C | D |
|---|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`wait(sA);` | &emsp;`wait(tB);` | &emsp;`wait(tC);` | &emsp;`wait(tD);` |
| &emsp;`A();` | &emsp;`wait(sOtro);` | &emsp;`wait(sOtro);` | &emsp;`wait(sOtro);` |
| &emsp;`signal(sOtro);` | &emsp;`B();` | &emsp;`C();` | &emsp;`D();` |
| `}` | &emsp;`signal(tC);` | &emsp;`signal(tD);` | &emsp;`signal(tB);` |
|  | &emsp;`signal(sA);` | &emsp;`signal(sA);` | &emsp;`signal(sA);` |
|  | `}` | `}` | `}` |

### Esperar a dos eventos (ej: dos "tun" antes de "we")
```
wait(s); wait(s)   // dos signals previos
```

## Ejemplo de clase: BACA BACA (B A C A B A C A ...)
Solución 1

> ⚠️ **Error en el apunte (04-18):** `mutexBoC` arrancaba en 0 y así se bloquean todos al inicio. Corregido: arranca en 1.

**Semáforos:** `mutexA = 0` · `mutexB = 1` · `mutexC = 0` · `mutexBoC = 1`

| A | B | C |
|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`wait(mutexA);` | &emsp;`wait(mutexB);` | &emsp;`wait(mutexC);` |
| &emsp;`printf("A");` | &emsp;`wait(mutexBoC);` | &emsp;`wait(mutexBoC);` |
| &emsp;`signal(mutexBoC);` | &emsp;`printf("B");` | &emsp;`printf("C");` |
| `}` | &emsp;`signal(mutexA);` | &emsp;`signal(mutexA);` |
|  | &emsp;`signal(mutexC);` | &emsp;`signal(mutexB);` |
|  | `}` | `}` |

Solución 2 (A avisa a B y a C; cada uno necesita 2 avisos)

> ⚠️ **Error en el apunte (04-18):** con B = 1 y C = 0 se bloquean todos al inicio. Corregido: B arranca en 2 y C en 1.

**Semáforos:** `mutexA = 0` · `mutexB = 2` · `mutexC = 1`

| A | B | C |
|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`wait(mutexA);` | &emsp;`wait(mutexB);` | &emsp;`wait(mutexC);` |
| &emsp;`printf("A");` | &emsp;`wait(mutexB);` | &emsp;`wait(mutexC);` |
| &emsp;`signal(mutexB);` | &emsp;`printf("B");` | &emsp;`printf("C");` |
| &emsp;`signal(mutexC);` | &emsp;`signal(mutexA);` | &emsp;`signal(mutexA);` |
| `}` | `}` | `}` |

## Semáforos en un Gantt
- Si el enunciado da duraciones de wait/signal, la syscall se ejecuta entera aunque el semáforo quede negativo; el proceso se bloquea al terminar el wait
- Anotar en cada instante: valor del semáforo y lista de bloqueados
- signal despierta al primero de la cola (FIFO) → pasa a ready (al final de la cola de listos)

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
