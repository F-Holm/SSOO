# 06 - Concurrencia y sincronización (teoría)

## Concurrencia
- Formas: multiprogramación, multiprocesamiento, procesamiento distribuido
- Se usa por performance (no dejar la CPU ociosa) y por diseño (aplicaciones estructuradas en procesos/hilos)
- Interacción: competencia por recursos (el SO administra, no se conocen), cooperación por compartición (se conocen indirectamente), cooperación por comunicación (se conocen)

## Condición de carrera
- Varios hilos/procesos modifican datos compartidos y el resultado depende del orden (velocidad relativa) de ejecución
- Pasa porque operaciones "atómicas" para nosotros no lo son para la CPU (`a = a - 1` → MOV, SUB, MOV)
- Puede pasar **aún en monoprocesador** (el planificador interrumpe en medio)
- Si solo leen, no hay condición de carrera (estructuras que no se modifican no necesitan sincronización)
- La existencia de una SC no garantiza un resultado incoherente, solo lo hace posible
- Debuggear no sirve para detectarla: cambia la velocidad relativa
- Dos ULTs del mismo proceso también pueden sufrirla (si la biblioteca cambia de hilo en medio)
- Hilos (ULT o KLT) que operan el mismo archivo comparten el puntero del archivo → inconsistencias si no usan locks

## Sección crítica
- Porción de código que accede a recursos compartidos. Debe ser lo más chica posible, atómica y de duración finita
- Un proceso puede tener muchas SC
- Requisitos:
  - Mutua exclusión: un solo proceso en la SC a la vez
  - Progreso: al salir avisa; uno que no está en la SC no impide la entrada de otros
  - Espera limitada: no esperar indefinidamente
  - Velocidad relativa: no se puede suponer cuánto tarda cada uno
- Condiciones de Bernstein (si se cumplen NO hay SC):
  - R(A) ∩ W(B) = ∅
  - R(B) ∩ W(A) = ∅
  - W(A) ∩ W(B) = ∅

## Soluciones
### Software (todas con espera activa)
- Espera activa = evaluar una condición en loop ocupando CPU (overhead)
- Sol 1 (variable turno): mutua exclusión sí, pero alternancia estricta, sin progreso, solo 2 procesos
- Sol 2 (array de interesados, pregunta y después se marca): progreso sí, **no** mutua exclusión
- Sol 3 (se marca y después pregunta): mutua exclusión sí, pero **deadlock**
- Sol 4 (ceden el paso con sleep): mutua exclusión, sin deadlock, pero **livelock** (cambian de estado sin progresar)
- Funcionan: Dekker y Peterson (secciones de entrada y salida, progreso) pero con espera activa
- Una solución de software puede provocar deadlocks y livelocks

### Hardware
- Deshabilitar interrupciones: solo enmascarables, solo en modo kernel (privilegiada) → no la pueden usar los procesos de usuario
  - − Si el proceso muere no se rehabilitan, da mucho poder, más lento, no sirve en multiprocesador
- Instrucciones atómicas (test_and_set, swap/exchange): leen y escriben en una sola instrucción indivisible
  - Mutua exclusión y progreso, sirve en multiprocesador, pero con **espera activa** (spinlock)
  - No evitan todas las condiciones de carrera: son la base para construir exclusión mutua
- La espera activa no implica solución de software (test_and_set y semáforos con espera activa también la tienen)

### SO: semáforos
```c
struct semaphore { int count; queue cola; }
wait(s):   s.count--; if (s.count < 0) bloquear(proceso)
signal(s): s.count++; if (s.count <= 0) desbloquear(uno)
```
- wait y signal son **syscalls** (cambio de modo, también desde ULTs). signal no es bloqueante
- Deben ser atómicas (si no, condición de carrera sobre el semáforo): se implementan deshabilitando interrupciones (monoprocesador) o con test_and_set (multiprocesador). No necesariamente deshabilitan interrupciones
- Valores:
  - Inicialización: ≥ 0 (nunca negativo)
  - > 0: instancias disponibles
  - < 0: |valor| = procesos bloqueados esperando
  - = 0: nada disponible y nadie esperando
- Cola de bloqueados: FIFO (la usamos en los ejercicios)
- Semáforos con bloqueo: sin espera activa. Semáforos con espera activa: posibles, desperdician CPU
- Tipos:
  - Mutex: mutua exclusión, init 1
  - Binario (0/1): garantiza orden de ejecución (sincronización)
  - Contador/general (init n): controla n instancias de un recurso
- Semáforos bien usados pueden igual traer problemas con prioridades (inversión de prioridades)
- Mal usados (orden de waits) → deadlock. Con espera activa → livelock (consume CPU y afecta a otros)
- Si la SC es muy chica se puede usar espera activa (`pthread_spinlock_t`). En el TP se usa mutex (con bloqueo)
- Ventajas sobre soluciones de software: sin espera activa, los maneja el SO, sirven para N procesos

### Lenguajes: monitores
- Clase que asegura que solo un proceso ejecute dentro a la vez (encapsulamiento). Mutua exclusión + sincronización

## Inversión de prioridades
- Uno de mayor prioridad espera un recurso (mutex) que tiene uno de menor prioridad, que puede ser desalojado con el recurso tomado
- Herencia de prioridades: al que tiene el semáforo se le asigna la prioridad más alta de la lista de espera; al terminar vuelve a la suya

## Productor / consumidor
- Buffer = recurso compartido → mutex (init 1)
- `lugar` = N (lugares libres, contador) → el productor espera si está lleno
- `cant` = 0 (elementos disponibles, contador) → el consumidor espera si está vacío
- Buffer infinito → no hace falta `lugar`
- Aún con elementos en el buffer, el consumidor no siempre extrae inmediatamente: debe obtener la mutua exclusión

## Locks de archivos (ver FS)
- Mejor que un mutex para archivos: granularidad por rango de bytes, compartidos (lectura) vs exclusivos (escritura), entre procesos no relacionados
