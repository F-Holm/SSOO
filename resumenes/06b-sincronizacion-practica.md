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
```
mutex = 1
wait(mutex); SC; signal(mutex)
```

### Orden A → B
```
s = 0
A: hacerA(); signal(s)
B: wait(s); hacerB()
```

### Alternancia A, B, A, B...
```
sA = 1; sB = 0
A: wait(sA); A(); signal(sB)
B: wait(sB); B(); signal(sA)
```

### Máximo N en simultáneo (ej: "aprendizaje acepta 2 procesos")
```
cant = N
wait(cant); usar(); signal(cant)
```

### Máximo N pedidos pendientes (productor limitado)
```
limite = N; pedidos = 0
Productor: wait(limite); pedir(); signal(pedidos)
Consumidor: wait(pedidos); atender(); signal(limite)
```

### Productor / consumidor con buffer de N
```
mutex = 1; lugar = N; cant = 0
Productor: x = producir(); wait(lugar); wait(mutex); agregar(x); signal(mutex); signal(cant)
Consumidor: wait(cant); wait(mutex); x = sacar(); signal(mutex); signal(lugar); consumir(x)
```
- Buffer infinito: sacar `lugar`

### Un proceso que aparece varias veces en la secuencia (ej: A B A C A D ...)
- Opción 1: "desenrollar" el while del proceso repetido (cada vuelta hace varias impresiones con distintos signals). Es válido, el while sigue siendo infinito
- Opción 2: semáforo compartido + "tokens" de turno para los otros:
```
sA = 1; sOtro = 0; tB = 1; tC = 0; tD = 0
A: wait(sA); A(); signal(sOtro)
B: wait(tB); wait(sOtro); B(); signal(tC); signal(sA)
C: wait(tC); wait(sOtro); C(); signal(tD); signal(sA)
D: wait(tD); wait(sOtro); D(); signal(tB); signal(sA)
```

### Esperar a dos eventos (ej: dos "tun" antes de "we")
```
wait(s); wait(s)   // dos signals previos
```

## Ejemplo de clase: BACA BACA (B A C A B A C A ...)
Solución 1

> ⚠️ **Error en el apunte (04-18):** `mutexBoC` arrancaba en 0 y así se bloquean todos al inicio. Corregido: arranca en 1.

```
mutexA = 0; mutexB = 1; mutexC = 0; mutexBoC = 1

A: wait(mutexA); printf("A"); signal(mutexBoC)
B: wait(mutexB); wait(mutexBoC); printf("B"); signal(mutexA); signal(mutexC)
C: wait(mutexC); wait(mutexBoC); printf("C"); signal(mutexA); signal(mutexB)
```

Solución 2 (A avisa a B y a C; cada uno necesita 2 avisos)

> ⚠️ **Error en el apunte (04-18):** con B = 1 y C = 0 se bloquean todos al inicio. Corregido: B arranca en 2 y C en 1.

```
mutexA = 0; mutexB = 2; mutexC = 1

A: wait(mutexA); printf("A"); signal(mutexB); signal(mutexC)
B: wait(mutexB); wait(mutexB); printf("B"); signal(mutexA)
C: wait(mutexC); wait(mutexC); printf("C"); signal(mutexA)
```

## Semáforos en un Gantt
- Si el enunciado da duraciones de wait/signal, la syscall se ejecuta entera aunque el semáforo quede negativo; el proceso se bloquea al terminar el wait
- Anotar en cada instante: valor del semáforo y lista de bloqueados
- signal despierta al primero de la cola (FIFO) → pasa a ready (al final de la cola de listos)

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
