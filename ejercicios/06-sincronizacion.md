# 06 - Concurrencia y sincronización (ejercicios)

## Teóricos

### [Resumen] Defina sección crítica, condiciones que debe cumplir y un ejemplo de solución de HW dentro de wait/signal
- SC: código que accede/modifica recursos compartidos entre procesos/hilos donde puede haber condición de carrera
- Condiciones: mutua exclusión, progreso, espera limitada, no suponer velocidad relativa
```c
wait()   { deshabilitar_interrupciones(); sem--; if (sem < 0) bloquear();   habilitar_interrupciones(); }
signal() { deshabilitar_interrupciones(); sem++; if (sem <= 0) despertar(); habilitar_interrupciones(); }
```

### [Resumen] Tipos de semáforos y qué resuelve cada uno. Productor/consumidor con buffer infinito
- Contador (init n): acceso a n instancias. Binario (0/1): orden/sincronización. Mutex (init 1): mutua exclusión
- Buffer infinito: mutex para el buffer + contador `cant` (init 0) para que el consumidor sepa si hay elementos. No hace falta `lugar`

### [Resumen] V/F: Los semáforos, aún bien usados, pueden traer problemas con planificadores por prioridades
- **V**. Inversión de prioridades: uno de mayor prioridad bloqueado esperando algo que libera uno de menor prioridad

### [Resumen] ¿Qué resuelve un mutex? ¿De qué otra forma? Si su valor es negativo, ¿qué implica?
- Mutua exclusión → evita condiciones de carrera. Alternativas: soluciones de software (Dekker, Peterson), de HW (test_and_set, deshabilitar interrupciones), monitores
- Negativo: hay |valor| procesos bloqueados esperando entrar a la SC

### [Resumen] Requisitos mínimos y deseables para la mutua exclusión. Dos ventajas de semáforos sobre soluciones de software
- Mínimos: mutua exclusión, progreso, espera limitada. Deseables: no depender de la velocidad relativa ni de la cantidad de procesos, SC chica, sin espera activa
- Ventajas: sin espera activa (bloquean), los gestiona el SO (sirven para N procesos, portables)

### [Resumen] ¿Qué problema busca solucionar la mutua exclusión? Ejemplo y dos formas de sincronizarlo
- Evitar la condición de carrera: el resultado depende del orden porque `a = a + 1` son varias instrucciones
```
int a = 0;          P1: c = a; c++; a = c;      P2: b = a; b--; a = b;
```
- Si se intercalan, `a` puede terminar en 1 o −1 en vez de 0
- Con mutex: `wait(mutex); ...SC...; signal(mutex)` en ambos
- Deshabilitando interrupciones alrededor de la SC (solo en monoprocesador y en modo kernel)

### [Resumen] ¿Es eficiente detectar un bug de condición de carrera debuggeando?
- No: depende de la velocidad relativa, que el debugger altera → el bug puede no aparecer

### [Resumen] V/F: Si wait y signal no fueran atómicas se generaría condición de carrera
- **V**. Leen y escriben el semáforo compartido (no se cumplen las condiciones de Bernstein)

### [Resumen] V/F: Tanto los semáforos como deshabilitar/habilitar interrupciones son técnicas que los usuarios pueden usar para la mutua exclusión sin espera activa
- **F** (en el resumen figuraba "V (?)"). Deshabilitar interrupciones es una instrucción privilegiada: un proceso de usuario no puede usarla. Los semáforos con bloqueo sí evitan la espera activa

### [Resumen] ¿Por qué los semáforos son buena solución? ¿Cómo logra el SO que wait/signal sean atómicas?
- Garantizan mutua exclusión, progreso y espera limitada, sin espera activa
- Deshabilitando interrupciones durante wait/signal (se ejecutan en modo kernel) en monoprocesador; en multiprocesador con test_and_set/spinlock

### [Resumen] Productor/consumidor: problemas si no se sincroniza y semáforos a usar
- Sin sincronizar: condición de carrera sobre el buffer, consumir de un buffer vacío, producir en uno lleno
```
s_lugar = 5 (contador); m_buffer = 1 (mutex); s_elementos = 0 (contador)
Productor: x = producir(); wait(s_lugar); wait(m_buffer); agregar(x); signal(m_buffer); signal(s_elementos)
Consumidor: wait(s_elementos); wait(m_buffer); y = extraer(); signal(m_buffer); signal(s_lugar); consumir(y)
```

### [Resumen] Semáforos con espera activa: ventajas y desventajas
- + Garantizan mutua exclusión y progreso; sirven si la SC es muy corta (evitan el costo de bloquear)
- − Consumen CPU esperando

### [Final 2022-08-02] Para sincronizar el orden entre procesos siempre se debe utilizar semáforos mutex o contadores
- **F**. Para ordenar se usan semáforos binarios (el mutex es para mutua exclusión); además existen otras técnicas (monitores, mensajes, soluciones de software)

### [Final 2022-09-07] En dispositivos con un solo procesador no es necesario implementar sincronización
- **F**. Ej: variable compartida entre hilos del mismo proceso; el planificador puede cortar en medio (oficial)

### [Final 2022-12-06] No es posible que haya condición de carrera sobre estructuras de datos que no pueden ser modificadas
- **V**. Si solo se leen se cumplen las condiciones de Bernstein (no hay escrituras)

### [Final 2023-02-14] En un monoprocesador, dos procesos que comparten una variable global no tienen problemas de concurrencia si la sentencia es `variable = variable - 1`
- **F**. La sentencia son varias instrucciones (MOV, SUB, MOV): una interrupción/cambio de proceso en el medio produce condición de carrera

### [Final 2023-08-01] Una solución de software resuelve la mutua exclusión, siempre ocasiona espera activa y puede provocar deadlocks (pero no livelocks)
- **F**. Siempre tienen espera activa, pero según cómo se usen pueden provocar deadlocks y también livelocks (oficial)

### [Final 2023-12-19 / 2024-05-10] La espera activa para proteger secciones críticas implica que se usan soluciones de software
- **F**. Soluciones de HW (test_and_set) y semáforos con espera activa también la tienen (oficial)

### [Final 2024-02-20] En un monoprocesador, el planificador de corto plazo podría generar una condición de carrera. No si los procesos están bien sincronizados
- **V**. El planificador puede interrumpir una operación que debía ser atómica; bien sincronizados el resultado es correcto sin importar el orden (oficial)

### [Final 2024-07-23] La existencia de una SC garantiza que el resultado será incoherente siempre que varios procesos escriban recursos críticos
- **F**. Solo existe la posibilidad (depende del orden de ejecución); sincronizando correctamente el resultado es coherente

### [Final 2024-07-30] La alternancia entre procesos puede solucionar la exclusión mutua, pero puede no cumplir los requisitos de la SC
- **V**. La variable turno asegura mutua exclusión pero no progreso (uno que no quiere entrar bloquea al otro) y tiene espera activa

### [Final 2025-02-18 / 2025-12-02] Algunos semáforos permiten asegurar la exclusión mutua pero pueden generar un bloqueo permanente
- **V**. Mal usados (ej: waits en distinto orden, falta de signal) llevan a deadlock

### [Final 2025-02-25] Dos ULTs del mismo proceso (sin KLTs) podrían sufrir una condición de carrera al modificar una variable compartida
- **V**. Si la biblioteca cambia de ULT en medio de la modificación (llamada a la biblioteca, desalojo), el otro puede leer un valor intermedio

### [Final 2025-07-15] En un sistema monoprocesador las condiciones de carrera no pueden ocurrir
- **F**. Con multiprogramación el planificador puede intercalar la ejecución

### [Final 2026-05-19] En productor–consumidor, mientras haya elementos en el buffer el consumidor podrá extraerlos inmediatamente
- **F**. El buffer es compartido: debe obtener la mutua exclusión primero (oficial)

### [Final 2026-02-24] Evaluar la respuesta del LLM
- Afirmación: "El uso de test & set o instrucciones similares permite evitar las condiciones de carrera"
- LLM: "Correcto: son atómicas (leer y actualizar en una operación indivisible). No eliminan automáticamente todas las condiciones de carrera: proveen un mecanismo sobre el cual construir exclusión mutua"
- **Correcta** (oficial)

## Prácticos

### [Clase 04-18] BACA BACA
- Ver [resumen 06b](../resumenes/06b-sincronizacion-practica.md) (dos soluciones corregidas)

### [Final 2022-12-21] Proceso con 2 hilos, semáforos con bloqueo. a) Orden que produzca deadlock y qué algoritmo podría causarlo b) ¿Afecta a otros procesos? ¿Y sin bloqueo? c) Corregir
```
sem1 = 1; sem2 = 0; mutex = 1        // acumulador: global del proceso
Hilo 1: valor = cálculo(); wait(sem1); wait(mutex); acumulador += valor; signal(mutex); signal(sem2)
Hilo 2: valor = cálculo(); wait(mutex); wait(sem2); acumulador += valor; signal(mutex); signal(sem1)
```
- a) Si ejecuta primero el Hilo 2 (ej: FIFO): toma el mutex y se bloquea en sem2; el Hilo 1 pasa sem1 y se bloquea en el mutex → deadlock. También con RR si alternan instrucción a instrucción (oficial)
- b) No afecta a otros procesos (los semáforos son del proceso, los bloqueados no consumen CPU). Sin bloqueo (espera activa) sería un livelock que consume CPU y afecta al resto
- c) Invertir `wait(mutex)` y `wait(sem2)` en el Hilo 2 (que el mutex encierre solo la SC), o sacar el mutex (sem1/sem2 ya garantizan la alternancia)

### [Final 2023-02-14] Sincronizar para imprimir permanentemente "Qué mirás bobo, andá pa allá, bobo"
```
P1: while(true) printf("bobo")    P2: while(true) printf("Qué mirás")    P3: while(true) printf("andá pa allá")
```
a) Secuencia: QM, bobo, APA, bobo, QM, ...
```
sQM = 1; sAPA = 0; sBobo = 0; sFrase = 1   // sFrase: "se terminó el bobo anterior"

P2: while(true){ wait(sQM);  wait(sFrase); printf("Qué mirás");    signal(sBobo); signal(sAPA) }
P1: while(true){ wait(sBobo); printf("bobo"); signal(sFrase) }
P3: while(true){ wait(sAPA); wait(sFrase); printf("andá pa allá"); signal(sBobo); signal(sQM) }
```
- Después de "Qué mirás", P3 queda esperando sFrase hasta que P1 diga "bobo"; P2 no puede volver a entrar porque sQM está en 0 hasta que P3 termine

b) P4 habilita la frase con `iniciar()`, máximo 5 pedidos pendientes (la frase se inicia al decir "Qué mirás")
```
limite = 5; pedidos = 0
P4: while(true){ wait(limite); iniciar(); signal(pedidos) }
P2: while(true){ wait(pedidos); wait(sQM); wait(sFrase); printf("Qué mirás"); signal(limite); signal(sBobo); signal(sAPA) }
```

### [Final 2023-03-07] "¿Qué mirás? bobo. Andá pa allá, bobo" + "Tranquilo Leo". Aprendizaje: hasta 2 procesos en simultáneo (oficial)
```
cantAprendizaje = 2; semA = semC = 1; semB = semD = semE = 0

P1 (bobo):         wait(cantAprendizaje); aprendizaje(); signal(cantAprendizaje); wait(semB); printf("bobo"); signal(semC); signal(semE)
P2 (andá pa allá): wait(cantAprendizaje); aprendizaje(); signal(cantAprendizaje); wait(semD); wait(semC); printf(". Andá pa allá, "); signal(semB)
P3 (qué mirás):    wait(cantAprendizaje); aprendizaje(); signal(cantAprendizaje); wait(semA); wait(semC); printf("¿Qué mirás? "); signal(semB); signal(semD)
P4 (tranquilo):    wait(cantAprendizaje); aprendizaje(); signal(cantAprendizaje); wait(semE); wait(semE); printf("Tranquilo Leo"); signal(semA)
```
(todo dentro de `while(true)`; hay otras soluciones posibles, ej: con 3 semáforos para QM-B-APA-B)

### [Final 2023-07-25 N°8] Productor agrega libremente, consumidor retira si hay al menos uno, sin condiciones de carrera. ChatGPT propuso:
```
lleno = 0; vacio = 1
P: while(TRUE){ wait(vacio); agregar(LISTA, new_value()); signal(lleno) }
C: while(TRUE){ wait(lleno); e = sacar(LISTA); signal(vacio) }
```
- No cumple: con `vacio = 1` el productor solo puede agregar un elemento y debe esperar a que el consumidor lo saque (alternancia estricta, buffer de 1). No hay condición de carrera por esa alternancia, pero no se pide eso
- Corrección:
```
mutex = 1; lleno = 0
P: while(TRUE){ v = new_value(); wait(mutex); agregar(LISTA, v); signal(mutex); signal(lleno) }
C: while(TRUE){ wait(lleno); wait(mutex); e = sacar(LISTA); signal(mutex) }
```

### [Final 2023-12-12] Semáforos para el patrón PA, PB, PA, PC, PA, PD, PA, PB, ...
Opción 1 (desenrollando PA):
```
sA = 1; sB = sC = sD = 0
PA: while(1){ wait(sA); A(); signal(sB); wait(sA); A(); signal(sC); wait(sA); A(); signal(sD) }
PB: while(1){ wait(sB); B(); signal(sA) }
PC: while(1){ wait(sC); C(); signal(sA) }
PD: while(1){ wait(sD); D(); signal(sA) }
```
Opción 2 (turnos entre B, C, D):
```
sA = 1; sOtro = 0; tB = 1; tC = 0; tD = 0
PA: while(1){ wait(sA); A(); signal(sOtro) }
PB: while(1){ wait(tB); wait(sOtro); B(); signal(tC); signal(sA) }
PC: while(1){ wait(tC); wait(sOtro); C(); signal(tD); signal(sA) }
PD: while(1){ wait(tD); wait(sOtro); D(); signal(tB); signal(sA) }
```

### [Final 2024-02-20] Red social "Z": N usuarios y un analizador. El sistema es lento y deja de funcionar. Encontrar al menos 3 errores/mejoras (sin resincronizar)
```
Usuario (N): wait(mutexPosts); post = generarPost(); postear(post, postsNuevos); signal(hayPosts); mostrarEnPantalla(post); signal(mutexPosts)
Analizador (1): wait(mutexPosts); wait(hayPosts); post = obtenerPost(postsNuevos); resultado = procesar(post); guardarEnDisco(resultado); signal(mutexPosts)
hayPosts = 0 (contador); mutexPosts = mutex
```
(oficial)
- Usuario: el `wait(mutexPosts)` debería ir después de `generarPost()` (no bloquearse antes de generar)
- Usuario: el `signal(mutexPosts)` debería ir justo después de `postear` (SC lo más chica posible; `mostrarEnPantalla` no la necesita)
- Analizador: los waits están al revés → si la cola está vacía se bloquea en `hayPosts` con el mutex tomado y ningún usuario puede postear → **deadlock** (por eso "deja de funcionar")
- Analizador: hacer el signal del mutex justo después de `obtenerPost` (no retener el mutex mientras procesa y escribe en disco)

### [Final 2024-07-23] "CrowdStrike": 4 instancias de Calculador (llegan en t0, 1, 2, 3). wait/signal = 4 ut, atómicas deshabilitando interrupciones; el resto de sentencias 2 ut. RR Q=3, 1 CPU, S = 2. Gantt y lista de bloqueados
```
wait(S); total1++; total2++; signal(S)
```
Supuestos: el wait se ejecuta entero (4 ut) aunque el semáforo quede negativo y el proceso se bloquea al terminarlo; como wait/signal deshabilitan interrupciones, el fin de quantum se atiende recién al terminar la syscall; las sentencias comunes sí se pueden cortar a la mitad.

```
t     0         1         2         3         4
      0123456789012345678901234567890123456789012345678
C1    WWWW------------112---2SSSSF
C2     ---WWWW-----------112-----2SSSSF
C3      ------WWWWbbbbbbbbbbbbbbb-----112---2SSSSF
C4       ---------WWWWbbbbbbbbbbbbbbbb---112-----2SSSSF
```
(W = wait, 1 = total1++, 2 = total2++, S = signal, - = listo, b = bloqueado)

| t | Evento | S | Bloqueados |
|---|---|---|---|
| 4 | C1 termina wait (el quantum vencía en 3, se atiende en 4) | 1 | - |
| 8 | C2 termina wait | 0 | - |
| 12 | C3 termina wait → se bloquea | −1 | C3 |
| 16 | C4 termina wait → se bloquea | −2 | C3, C4 |
| 19 | C1 cortado por quantum en medio de total2++ | | |
| 27 | C1 termina signal → despierta a C3; C1 termina | −1 | C4 |
| 32 | C2 termina signal → despierta a C4; C2 termina | 0 | - |
| 43 | C3 termina signal | 1 | - |
| 48 | C4 termina signal | 2 | - |

### [Final 2025-02-25] Imprimir "Tun, tun, We will we will rock you!" permanentemente. "tun" lo emite el baterista o el percusionista (cualquiera); ambos toman los palillos del mismo lugar (de a uno)
```
palillos = 1   // mutex
tun = 2        // tuns habilitados en la vuelta
tunListo = 0; will = 0; we2 = 0; rock = 0

Baterista y Percusionista (igual):
  while(1){ wait(tun); wait(palillos); tomar_palillo(); signal(palillos); print("tun"); signal(tunListo) }
Vocalista 1:
  while(1){ wait(tunListo); wait(tunListo); print("we"); signal(will); wait(we2); print("we"); signal(will) }
Vocalista 2:
  while(1){ wait(will); print("will"); signal(we2); wait(will); print("will"); signal(rock) }
Vocalista 3:
  while(1){ wait(rock); print("rock you!"); signal(tun); signal(tun) }
```
- Traza: tun, tun → we → will → we → will → rock you! → se habilitan 2 tuns de nuevo
- Los vocalistas 1 y 2 aparecen dos veces por vuelta → se "desenrolla" su loop

### [Final 2025-07-29] Un productor, 6 consumidores y un notificador
```
Productor (1):   a = producir(); wait(B); depositar(lista, a); signal(B); signal(C)
Consumidor (6):  wait(C); wait(B); e = retirar(lista); signal(B); procesar(e); signal(D)
Notificador (1): wait(D); wait(B); notificar(lista_mensajes); signal(B)
```
- a) Ninguno limita la cantidad de elementos de la lista (no hay semáforo de "lugares libres": la lista es ilimitada). **C** indica cuántos elementos hay disponibles
- b) **D**, inicializado en **0** (contador de elementos procesados pendientes de notificar)
- c) B es un mutex → **1**. Es innecesario en el Notificador: `lista_mensajes` solo la usa él (no es la lista compartida) y además bloquea de más al productor y consumidores
- d) Si los 6 consumidores están en `wait(B)`, los 6 pasaron `wait(C)` → se produjeron **al menos 6** elementos
- e) Sí. Si el consumidor hace `wait(B)` antes que `wait(C)` con la lista vacía: toma B (B=0), hace wait(C) (C=−1) y se bloquea con el mutex tomado. El productor produce, hace wait(B) (B=−1) y se bloquea → nadie hará signal(C) ni signal(B) → **deadlock** (y los demás consumidores y el notificador quedan bloqueados en B)
