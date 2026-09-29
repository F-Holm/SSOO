# 06 - Concurrencia y sincronización (ejercicios)

## Teóricos

### [Resumen] Defina sección crítica, condiciones que debe cumplir y un ejemplo de solución de HW dentro de wait/signal

<details>
<summary>Ver respuesta</summary>

- SC: código que accede/modifica recursos compartidos entre procesos/hilos donde puede haber condición de carrera
- Condiciones: mutua exclusión, progreso, espera limitada, no suponer velocidad relativa

| wait() | signal() |
|---|---|
| `deshabilitar_interrupciones();` | `deshabilitar_interrupciones();` |
| `sem--;` | `sem++;` |
| `if (sem < 0) bloquear();` | `if (sem <= 0) despertar();` |
| `habilitar_interrupciones();` | `habilitar_interrupciones();` |

</details>

### [Resumen] Tipos de semáforos y qué resuelve cada uno. Productor/consumidor con buffer infinito

<details>
<summary>Ver respuesta</summary>

- Contador (init n): acceso a n instancias. Binario (0/1): orden/sincronización. Mutex (init 1): mutua exclusión
- Buffer infinito: mutex para el buffer + contador `cant` (init 0) para que el consumidor sepa si hay elementos. No hace falta `lugar`

</details>

### [Resumen] V/F: Los semáforos, aún bien usados, pueden traer problemas con planificadores por prioridades

<details>
<summary>Ver respuesta</summary>

- **V**. Inversión de prioridades: uno de mayor prioridad bloqueado esperando algo que libera uno de menor prioridad

</details>

### [Resumen] ¿Qué resuelve un mutex? ¿De qué otra forma? Si su valor es negativo, ¿qué implica?

<details>
<summary>Ver respuesta</summary>

- Mutua exclusión → evita condiciones de carrera. Alternativas: soluciones de software (Dekker, Peterson), de HW (test_and_set, deshabilitar interrupciones), monitores
- Negativo: hay |valor| procesos bloqueados esperando entrar a la SC

</details>

### [Resumen] Requisitos mínimos y deseables para la mutua exclusión. Dos ventajas de semáforos sobre soluciones de software

<details>
<summary>Ver respuesta</summary>

- Mínimos: mutua exclusión, progreso, espera limitada. Deseables: no depender de la velocidad relativa ni de la cantidad de procesos, SC chica, sin espera activa
- Ventajas: sin espera activa (bloquean), los gestiona el SO (sirven para N procesos, portables)

</details>

### [Resumen] ¿Qué problema busca solucionar la mutua exclusión? Ejemplo y dos formas de sincronizarlo

<details>
<summary>Ver respuesta</summary>

- Evitar la condición de carrera: el resultado depende del orden porque `a = a + 1` son varias instrucciones

**Semáforos:** `int a = 0   // variable global`

| P1 | P2 |
|---|---|
| `c = a;` | `b = a;` |
| `c++;` | `b--;` |
| `a = c;` | `a = b;` |

- Si se intercalan, `a` puede terminar en 1 o −1 en vez de 0
- Con mutex: `wait(mutex); ...SC...; signal(mutex)` en ambos
- Deshabilitando interrupciones alrededor de la SC (solo en monoprocesador y en modo kernel)

</details>

### [Resumen] ¿Es eficiente detectar un bug de condición de carrera debuggeando?

<details>
<summary>Ver respuesta</summary>

- No: depende de la velocidad relativa, que el debugger altera → el bug puede no aparecer

</details>

### [Resumen] V/F: Si wait y signal no fueran atómicas se generaría condición de carrera

<details>
<summary>Ver respuesta</summary>

- **V**. Leen y escriben el semáforo compartido (no se cumplen las condiciones de Bernstein)

</details>

### [Resumen] V/F: Tanto los semáforos como deshabilitar/habilitar interrupciones son técnicas que los usuarios pueden usar para la mutua exclusión sin espera activa

<details>
<summary>Ver respuesta</summary>

> ⚠️ **Error en la respuesta del Resumen:** figuraba "Verdadero (?)".

- **F**. Deshabilitar interrupciones es una instrucción privilegiada: un proceso de usuario no puede usarla. Los semáforos con bloqueo sí evitan la espera activa

</details>

### [Resumen] ¿Por qué los semáforos son buena solución? ¿Cómo logra el SO que wait/signal sean atómicas?

<details>
<summary>Ver respuesta</summary>

- Garantizan mutua exclusión, progreso y espera limitada, sin espera activa
- Deshabilitando interrupciones durante wait/signal (se ejecutan en modo kernel) en monoprocesador; en multiprocesador con test_and_set/spinlock

</details>

### [Resumen] Productor/consumidor: problemas si no se sincroniza y semáforos a usar

<details>
<summary>Ver respuesta</summary>

- Sin sincronizar: condición de carrera sobre el buffer, consumir de un buffer vacío, producir en uno lleno

**Semáforos:** `s_lugar = 5 (contador)` · `m_buffer = 1 (mutex)` · `s_elementos = 0 (contador)`

| Productor | Consumidor |
|---|---|
| `x = producir();` | `wait(s_elementos);` |
| `wait(s_lugar);` | `wait(m_buffer);` |
| `wait(m_buffer);` | `y = extraer();` |
| `agregar(x);` | `signal(m_buffer);` |
| `signal(m_buffer);` | `signal(s_lugar);` |
| `signal(s_elementos);` | `consumir(y);` |

</details>

### [Resumen] Semáforos con espera activa: ventajas y desventajas

<details>
<summary>Ver respuesta</summary>

- + Garantizan mutua exclusión y progreso; sirven si la SC es muy corta (evitan el costo de bloquear)
- − Consumen CPU esperando

</details>

### [Final 2022-08-02] Para sincronizar el orden entre procesos siempre se debe utilizar semáforos mutex o contadores — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Para ordenar se usan semáforos binarios (el mutex es para mutua exclusión); además existen otras técnicas (monitores, mensajes, soluciones de software)

</details>

### [Final 2022-09-07] En dispositivos con un solo procesador no es necesario implementar sincronización — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Ej: variable compartida entre hilos del mismo proceso; el planificador puede cortar en medio (oficial)

</details>

### [Final 2022-12-06] No es posible que haya condición de carrera sobre estructuras de datos que no pueden ser modificadas — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Si solo se leen se cumplen las condiciones de Bernstein (no hay escrituras)

</details>

### [Final 2023-02-14] En un monoprocesador, dos procesos que comparten una variable global no tienen problemas de concurrencia si la sentencia es `variable = variable - 1` — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. La sentencia son varias instrucciones (MOV, SUB, MOV): una interrupción/cambio de proceso en el medio produce condición de carrera

</details>

### [Final 2023-08-01] Una solución de software resuelve la mutua exclusión, siempre ocasiona espera activa y puede provocar deadlocks (pero no livelocks) — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Siempre tienen espera activa, pero según cómo se usen pueden provocar deadlocks y también livelocks (oficial)

</details>

### [Final 2023-12-19 / 2024-05-10] La espera activa para proteger secciones críticas implica que se usan soluciones de software — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Soluciones de HW (test_and_set) y semáforos con espera activa también la tienen (oficial)

</details>

### [Final 2024-02-20] En un monoprocesador, el planificador de corto plazo podría generar una condición de carrera. No si los procesos están bien sincronizados — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. El planificador puede interrumpir una operación que debía ser atómica; bien sincronizados el resultado es correcto sin importar el orden (oficial)

</details>

### [Final 2024-07-23] La existencia de una SC garantiza que el resultado será incoherente siempre que varios procesos escriban recursos críticos — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Solo existe la posibilidad (depende del orden de ejecución); sincronizando correctamente el resultado es coherente

</details>

### [Final 2024-07-30] La alternancia entre procesos puede solucionar la exclusión mutua, pero puede no cumplir los requisitos de la SC — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. La variable turno asegura mutua exclusión pero no progreso (uno que no quiere entrar bloquea al otro) y tiene espera activa

</details>

### [Final 2025-02-18 / 2025-12-02] Algunos semáforos permiten asegurar la exclusión mutua pero pueden generar un bloqueo permanente — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Mal usados (ej: waits en distinto orden, falta de signal) llevan a deadlock

</details>

### [Final 2025-02-25] Dos ULTs del mismo proceso (sin KLTs) podrían sufrir una condición de carrera al modificar una variable compartida — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Si la biblioteca cambia de ULT en medio de la modificación (llamada a la biblioteca, desalojo), el otro puede leer un valor intermedio

</details>

### [Final 2025-07-15] En un sistema monoprocesador las condiciones de carrera no pueden ocurrir — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Con multiprogramación el planificador puede intercalar la ejecución

</details>

### [Final 2026-05-19] En productor–consumidor, mientras haya elementos en el buffer el consumidor podrá extraerlos inmediatamente — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. El buffer es compartido: debe obtener la mutua exclusión primero (oficial)

</details>

### [Final 2026-02-24] Evaluar la respuesta del LLM — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- Afirmación: "El uso de test & set o instrucciones similares permite evitar las condiciones de carrera"
- LLM: "Correcto: son atómicas (leer y actualizar en una operación indivisible). No eliminan automáticamente todas las condiciones de carrera: proveen un mecanismo sobre el cual construir exclusión mutua"
- **Correcta** (oficial)

</details>

## Prácticos

### [Clase 04-18] BACA BACA

<details>
<summary>Ver respuesta</summary>

- Ver [resumen 06b](../resumenes/06b-sincronizacion-practica.md) (dos soluciones corregidas)

</details>

### [Final 2022-12-21] Proceso con 2 hilos, semáforos con bloqueo. a) Orden que produzca deadlock y qué algoritmo podría causarlo b) ¿Afecta a otros procesos? ¿Y sin bloqueo? c) Corregir — ✅ solución oficial

**Semáforos:** `sem1 = 1` · `sem2 = 0` · `mutex = 1`
`acumulador` es una variable global del proceso

| Hilo 1 | Hilo 2 |
|---|---|
| `valor = cálculo();` | `valor = cálculo();` |
| `wait(sem1);` | `wait(mutex);` |
| `wait(mutex);` | `wait(sem2);` |
| `acumulador += valor;` | `acumulador += valor;` |
| `signal(mutex);` | `signal(mutex);` |
| `signal(sem2);` | `signal(sem1);` |

<details>
<summary>Ver respuesta</summary>

- a) Si ejecuta primero el Hilo 2 (ej: FIFO): toma el mutex y se bloquea en sem2; el Hilo 1 pasa sem1 y se bloquea en el mutex → deadlock. También con RR si alternan instrucción a instrucción (oficial)
- b) No afecta a otros procesos (los semáforos son del proceso, los bloqueados no consumen CPU). Sin bloqueo (espera activa) sería un livelock que consume CPU y afecta al resto
- c) Invertir `wait(mutex)` y `wait(sem2)` en el Hilo 2 (que el mutex encierre solo la SC), o sacar el mutex (sem1/sem2 ya garantizan la alternancia)

</details>

### [Final 2023-02-14] Sincronizar para imprimir permanentemente "Qué mirás bobo, andá pa allá, bobo" — ✍️ respuesta propia (el PDF del final no trae solución)

| P1 | P2 | P3 |
|---|---|---|
| `while(true) {` | `while(true) {` | `while(true) {` |
| &emsp;`printf("bobo")` | &emsp;`printf("Qué mirás")` | &emsp;`printf("andá pa allá")` |
| `}` | `}` | `}` |

**Consigna:**
- a) Sincronice los procesos utilizando semáforos para que reproduzcan de manera permanente la famosa frase
- b) Se agrega un cuarto proceso, encargado de habilitar que se pronuncie la frase completa ejecutando `iniciar()`, con la restricción de que no puede haber más de 5 pedidos pendientes de iniciar la frase (se considera iniciada cuando se dice "Qué mirás"). Proponga el pseudocódigo de dicho proceso y agregue los semáforos y el pseudocódigo necesarios en los demás

<details>
<summary>Ver respuesta</summary>

a) Secuencia: QM, bobo, APA, bobo, QM, ...

**Semáforos:** `sQM = 1` · `sAPA = 0` · `sBobo = 0` · `sFrase = 1   // "se terminó el bobo anterior"`

| P1 (bobo) | P2 (qué mirás) | P3 (andá pa allá) |
|---|---|---|
| `while(true) {` | `while(true) {` | `while(true) {` |
| &emsp;`wait(sBobo);` | &emsp;`wait(sQM);` | &emsp;`wait(sAPA);` |
| &emsp;`printf("bobo");` | &emsp;`wait(sFrase);` | &emsp;`wait(sFrase);` |
| &emsp;`signal(sFrase);` | &emsp;`printf("Qué mirás");` | &emsp;`printf("andá pa allá");` |
| `}` | &emsp;`signal(sBobo);` | &emsp;`signal(sBobo);` |
|  | &emsp;`signal(sAPA);` | &emsp;`signal(sQM);` |
|  | `}` | `}` |

- Después de "Qué mirás", P3 queda esperando sFrase hasta que P1 diga "bobo"; P2 no puede volver a entrar porque sQM está en 0 hasta que P3 termine

b) P4 habilita la frase con `iniciar()`, máximo 5 pedidos pendientes (la frase se inicia al decir "Qué mirás")

**Semáforos:** `limite = 5` · `pedidos = 0` · `(más los del punto a)`

| P2 (qué mirás) | P4 (iniciar) |
|---|---|
| `while(true) {` | `while(true) {` |
| &emsp;`wait(pedidos);` | &emsp;`wait(limite);` |
| &emsp;`wait(sQM);` | &emsp;`iniciar();` |
| &emsp;`wait(sFrase);` | &emsp;`signal(pedidos);` |
| &emsp;`printf("Qué mirás");` | `}` |
| &emsp;`signal(limite);` |  |
| &emsp;`signal(sBobo);` |  |
| &emsp;`signal(sAPA);` |  |
| `}` |  |

</details>

### [Final 2023-03-07] "¿Qué mirás? bobo. Andá pa allá, bobo" + "Tranquilo Leo". Aprendizaje: hasta 2 procesos en simultáneo (oficial) — ✅ solución oficial
> ⚠️ **Error en la consigna:** dice "se arma una simulación con 3 procesos" pero hay 4 (el cuarto dice "Tranquilo Leo").

| P1 | P2 | P3 | P4 |
|---|---|---|---|
| `while(true) {` | `while(true) {` | `while(true) {` | `while(true) {` |
| &emsp;`aprendizaje();` | &emsp;`aprendizaje();` | &emsp;`aprendizaje();` | &emsp;`aprendizaje();` |
| &emsp;`printf("bobo");` | &emsp;`printf(". Andá pa allá, ");` | &emsp;`printf("¿Qué mirás? ");` | &emsp;`printf("Tranquilo Leo");` |
| `}` | `}` | `}` | `}` |

**Consigna:** sincronice los procesos con semáforos para que reproduzcan de manera permanente la frase "¿Qué mirás? bobo. Andá pa allá, bobo" seguida de "Tranquilo Leo", sabiendo que la etapa de aprendizaje solo acepta hasta dos procesos en simultáneo

<details>
<summary>Ver respuesta</summary>

**Semáforos:** `cantAprendizaje = 2` · `semA = 1` · `semC = 1` · `semB = 0` · `semD = 0` · `semE = 0`

| P1 (bobo) | P2 (andá pa allá) | P3 (qué mirás) | P4 (tranquilo) |
|---|---|---|---|
| `while(true) {` | `while(true) {` | `while(true) {` | `while(true) {` |
| &emsp;`wait(cantAprendizaje);` | &emsp;`wait(cantAprendizaje);` | &emsp;`wait(cantAprendizaje);` | &emsp;`wait(cantAprendizaje);` |
| &emsp;`aprendizaje();` | &emsp;`aprendizaje();` | &emsp;`aprendizaje();` | &emsp;`aprendizaje();` |
| &emsp;`signal(cantAprendizaje);` | &emsp;`signal(cantAprendizaje);` | &emsp;`signal(cantAprendizaje);` | &emsp;`signal(cantAprendizaje);` |
| &emsp;`wait(semB);` | &emsp;`wait(semD);` | &emsp;`wait(semA);` | &emsp;`wait(semE);` |
| &emsp;`printf("bobo");` | &emsp;`wait(semC);` | &emsp;`wait(semC);` | &emsp;`wait(semE);` |
| &emsp;`signal(semC);` | &emsp;`printf(". Andá pa allá, ");` | &emsp;`printf("¿Qué mirás? ");` | &emsp;`printf("Tranquilo Leo");` |
| &emsp;`signal(semE);` | &emsp;`signal(semB);` | &emsp;`signal(semB);` | &emsp;`signal(semA);` |
| `}` | `}` | &emsp;`signal(semD);` | `}` |
|  |  | `}` |  |

(todo dentro de `while(true)`; hay otras soluciones posibles, ej: con 3 semáforos para QM-B-APA-B)

</details>

### [Final 2023-07-25 N°8] Productor agrega libremente, consumidor retira si hay al menos uno, sin condiciones de carrera. ChatGPT propuso: — ✍️ respuesta propia (el PDF del final no trae solución)
**Consigna original:** dados estos pseudocódigos, cada uno de un hilo del mismo proceso:
`LISTA` es una variable global

| P (productor) | C (consumidor) |
|---|---|
| `while(TRUE) {` | `while(TRUE) {` |
| &emsp;`agregar(LISTA, new_value());` | &emsp;`e = sacar(LISTA);` |
| `}` | `}` |

modifíquelos de manera que el productor pueda agregar elementos libremente y el consumidor retire cuando exista al menos uno, evitando condiciones de carrera, usando solo semáforos. Esta fue la propuesta de ChatGPT:

**Semáforos:** `lleno = 0` · `vacio = 1`

| P (productor) | C (consumidor) |
|---|---|
| `while(TRUE) {` | `while(TRUE) {` |
| &emsp;`wait(vacio);` | &emsp;`wait(lleno);` |
| &emsp;`agregar(LISTA, new_value());` | &emsp;`e = sacar(LISTA);` |
| &emsp;`signal(lleno);` | &emsp;`signal(vacio);` |
| `}` | `}` |

**Se pide:** analice la solución. ¿Se cumplieron los requisitos? ¿Existe algún problema de concurrencia? En caso de ser necesario, provea un nuevo pseudocódigo corregido

<details>
<summary>Ver respuesta</summary>

- No cumple: con `vacio = 1` el productor solo puede agregar un elemento y debe esperar a que el consumidor lo saque (alternancia estricta, buffer de 1). No hay condición de carrera por esa alternancia, pero no se pide eso
- Corrección:

**Semáforos:** `mutex = 1` · `lleno = 0`

| P (productor) | C (consumidor) |
|---|---|
| `while(TRUE) {` | `while(TRUE) {` |
| &emsp;`v = new_value();` | &emsp;`wait(lleno);` |
| &emsp;`wait(mutex);` | &emsp;`wait(mutex);` |
| &emsp;`agregar(LISTA, v);` | &emsp;`e = sacar(LISTA);` |
| &emsp;`signal(mutex);` | &emsp;`signal(mutex);` |
| &emsp;`signal(lleno);` | `}` |
| `}` |  |

</details>

### [Final 2023-12-12] Semáforos para el patrón PA, PB, PA, PC, PA, PD, PA, PB, ... — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

Opción 1 (desenrollando PA):

**Semáforos:** `sA = 1` · `sB = 0` · `sC = 0` · `sD = 0`

| PA | PB | PC | PD |
|---|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`wait(sA);` | &emsp;`wait(sB);` | &emsp;`wait(sC);` | &emsp;`wait(sD);` |
| &emsp;`A();` | &emsp;`B();` | &emsp;`C();` | &emsp;`D();` |
| &emsp;`signal(sB);` | &emsp;`signal(sA);` | &emsp;`signal(sA);` | &emsp;`signal(sA);` |
| &emsp;`wait(sA);` | `}` | `}` | `}` |
| &emsp;`A();` |  |  |  |
| &emsp;`signal(sC);` |  |  |  |
| &emsp;`wait(sA);` |  |  |  |
| &emsp;`A();` |  |  |  |
| &emsp;`signal(sD);` |  |  |  |
| `}` |  |  |  |

Opción 2 (turnos entre B, C, D):

**Semáforos:** `sA = 1` · `sOtro = 0` · `tB = 1` · `tC = 0` · `tD = 0`

| PA | PB | PC | PD |
|---|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`wait(sA);` | &emsp;`wait(tB);` | &emsp;`wait(tC);` | &emsp;`wait(tD);` |
| &emsp;`A();` | &emsp;`wait(sOtro);` | &emsp;`wait(sOtro);` | &emsp;`wait(sOtro);` |
| &emsp;`signal(sOtro);` | &emsp;`B();` | &emsp;`C();` | &emsp;`D();` |
| `}` | &emsp;`signal(tC);` | &emsp;`signal(tD);` | &emsp;`signal(tB);` |
|  | &emsp;`signal(sA);` | &emsp;`signal(sA);` | &emsp;`signal(sA);` |
|  | `}` | `}` | `}` |

</details>

### [Final 2024-02-20] Red social "Z": N usuarios y un analizador. El sistema es lento y deja de funcionar. Encontrar al menos 3 errores/mejoras (sin resincronizar) — ✅ solución oficial

**Semáforos:** `hayPosts = 0 (contador)` · `mutexPosts = 1 (mutex)`
`postsNuevos`: cola compartida con los nuevos posts a analizar

| Usuario (N instancias) | Analizador (1 instancia) |
|---|---|
| `while(1) {` | `while(1) {` |
| &emsp;`wait(mutexPosts);` | &emsp;`wait(mutexPosts);` |
| &emsp;`post = generarPost();` | &emsp;`wait(hayPosts);` |
| &emsp;`postear(post, postsNuevos);` | &emsp;`post = obtenerPost(postsNuevos);` |
| &emsp;`signal(hayPosts);` | &emsp;`resultado = procesar(post);` |
| &emsp;`mostrarEnPantalla(post);` | &emsp;`guardarEnDisco(resultado);` |
| &emsp;`signal(mutexPosts);` | &emsp;`signal(mutexPosts);` |
| `}` | `}` |

<details>
<summary>Ver respuesta</summary>

(oficial)
- Usuario: el `wait(mutexPosts)` debería ir después de `generarPost()` (no bloquearse antes de generar)
- Usuario: el `signal(mutexPosts)` debería ir justo después de `postear` (SC lo más chica posible; `mostrarEnPantalla` no la necesita)
- Analizador: los waits están al revés → si la cola está vacía se bloquea en `hayPosts` con el mutex tomado y ningún usuario puede postear → **deadlock** (por eso "deja de funcionar")
- Analizador: hacer el signal del mutex justo después de `obtenerPost` (no retener el mutex mientras procesa y escribe en disco)

</details>

### [Final 2024-07-23] "CrowdStrike": 4 instancias de Calculador (llegan en t0, 1, 2, 3). wait/signal = 4 ut, atómicas deshabilitando interrupciones; el resto de sentencias 2 ut. RR Q=3, 1 CPU, S = 2. Gantt y lista de bloqueados — ✍️ respuesta propia (el PDF del final no trae solución)

**Semáforos:** `S = 2`

| Calculador (4 instancias) |
|---|
| `wait(S);` |
| `total1++;` |
| `total2++;` |
| `signal(S);` |

<details>
<summary>Ver respuesta</summary>

Supuestos: el wait se ejecuta entero (4 ut) aunque el semáforo quede negativo y el proceso se bloquea al terminarlo; como wait/signal deshabilitan interrupciones, el fin de quantum se atiende recién al terminar la syscall; las sentencias comunes sí se pueden cortar a la mitad.

| | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 | 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 | 48 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| C1 | wait | wait | wait | wait | – | – | – | – | – | – | – | – | – | – | – | – | t1++ | t1++ | t2++ | – | – | – | t2++ | signal | signal | signal | signal | F |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| C2 |  | – | – | – | wait | wait | wait | wait | – | – | – | – | – | – | – | – | – | – | – | t1++ | t1++ | t2++ | – | – | – | – | – | t2++ | signal | signal | signal | signal | F |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| C3 |  |  | – | – | – | – | – | – | wait | wait | wait | wait | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | – | – | – | – | – | t1++ | t1++ | t2++ | – | – | – | t2++ | signal | signal | signal | signal | F |  |  |  |  |  |
| C4 |  |  |  | – | – | – | – | – | – | – | – | – | wait | wait | wait | wait | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | bloq | – | – | – | t1++ | t1++ | t2++ | – | – | – | – | – | t2++ | signal | signal | signal | signal | F |

(wait / t1++ / t2++ / signal = qué sentencia ejecuta; – = listo; bloq = bloqueado en el semáforo)

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

</details>

### [Final 2025-02-25] Imprimir "Tun, tun, We will we will rock you!" permanentemente. "tun" lo emite el baterista o el percusionista (cualquiera); ambos toman los palillos del mismo lugar (de a uno) — ✍️ respuesta propia (el PDF del final no trae solución)
> ⚠️ **Aclaración de la consigna:** en el PDF la frase aparece como "We will we ~~will rock~~ you!" (con "will rock" tachado, parece un error de formato). Como el Vocalista 3 imprime "rock you!", se toma la frase completa: tun, tun, we, will, we, will, rock you!

| Vocalista 1 | Vocalista 2 | Vocalista 3 | Baterista | Percusionista |
|---|---|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`print("we");` | &emsp;`print("will");` | &emsp;`print("rock you!");` | &emsp;`tomar_palillo();` | &emsp;`tomar_palillo();` |
| `}` | `}` | `}` | &emsp;`print("tun");` | &emsp;`print("tun");` |
|  |  |  | `}` | `}` |

**Consigna:** sincronice solamente con semáforos para que se impriman los mensajes en ese orden de forma permanente

<details>
<summary>Ver respuesta</summary>

**Semáforos:** `palillos = 1 (mutex)` · `tun = 2 (tuns habilitados en la vuelta)` · `tunListo = 0` · `will = 0` · `we2 = 0` · `rock = 0`

| Baterista y Percusionista (igual) | Vocalista 1 | Vocalista 2 | Vocalista 3 |
|---|---|---|---|
| `while(1) {` | `while(1) {` | `while(1) {` | `while(1) {` |
| &emsp;`wait(tun);` | &emsp;`wait(tunListo);` | &emsp;`wait(will);` | &emsp;`wait(rock);` |
| &emsp;`wait(palillos);` | &emsp;`wait(tunListo);` | &emsp;`print("will");` | &emsp;`print("rock you!");` |
| &emsp;`tomar_palillo();` | &emsp;`print("we");` | &emsp;`signal(we2);` | &emsp;`signal(tun);` |
| &emsp;`signal(palillos);` | &emsp;`signal(will);` | &emsp;`wait(will);` | &emsp;`signal(tun);` |
| &emsp;`print("tun");` | &emsp;`wait(we2);` | &emsp;`print("will");` | `}` |
| &emsp;`signal(tunListo);` | &emsp;`print("we");` | &emsp;`signal(rock);` |  |
| `}` | &emsp;`signal(will);` | `}` |  |
|  | `}` |  |  |

- Traza: tun, tun → we → will → we → will → rock you! → se habilitan 2 tuns de nuevo
- Los vocalistas 1 y 2 aparecen dos veces por vuelta → se "desenrolla" su loop

</details>

### [Final 2025-07-29] Un productor, 6 consumidores y un notificador — ✍️ respuesta propia (el PDF del final no trae solución)

**Semáforos:** `(valores iniciales: los pide el ejercicio)`

| Productor (1) | Consumidor (6) | Notificador (1) |
|---|---|---|
| `a = producir();` | `wait(C);` | `wait(D);` |
| `wait(B);` | `wait(B);` | `wait(B);` |
| `depositar(lista, a);` | `e = retirar(lista);` | `notificar(lista_mensajes);` |
| `signal(B);` | `signal(B);` | `signal(B);` |
| `signal(C);` | `procesar(e);` |  |
|  | `signal(D);` |  |

**Consigna:** (justificando cada respuesta)
- a) ¿Qué semáforo limita la cantidad de elementos de la lista y cuál indica cuántos hay en un momento?
- b) ¿Cuál es y en qué valor se debe inicializar el semáforo utilizado para disparar notificaciones?
- c) ¿En qué valor debería inicializarse el semáforo B? ¿Es innecesario su uso en algún proceso?
- d) Si todos los consumidores estuvieran ejecutando `wait(B)` concurrentemente, ¿cuántos elementos (como mínimo) fueron producidos?
- e) ¿Podría ocurrir un problema si en el Consumidor se intercambian `wait(C)` y `wait(B)`? En caso afirmativo explíquelo paso a paso; si no, detalle una traza de ejecución exitosa

<details>
<summary>Ver respuesta</summary>

- a) Ninguno limita la cantidad de elementos de la lista (no hay semáforo de "lugares libres": la lista es ilimitada). **C** indica cuántos elementos hay disponibles
- b) **D**, inicializado en **0** (contador de elementos procesados pendientes de notificar)
- c) B es un mutex → **1**. Es innecesario en el Notificador: `lista_mensajes` solo la usa él (no es la lista compartida) y además bloquea de más al productor y consumidores
- d) Si los 6 consumidores están en `wait(B)`, los 6 pasaron `wait(C)` → se produjeron **al menos 6** elementos
- e) Sí. Si el consumidor hace `wait(B)` antes que `wait(C)` con la lista vacía: toma B (B=0), hace wait(C) (C=−1) y se bloquea con el mutex tomado. El productor produce, hace wait(B) (B=−1) y se bloquea → nadie hará signal(C) ni signal(B) → **deadlock** (y los demás consumidores y el notificador quedan bloqueados en B)

</details>

---
[⬆ Volver al índice de ejercicios](00-indice.md) · [Índice general](../README.md)
