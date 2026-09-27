# 13 - Integradores / diseño (ejercicios)

## [Final 2026-07-14] Diseñar los aspectos esenciales de un SO según los requerimientos (justificar cada decisión) — ✅ solución oficial
Requerimientos (resumidos):
- Aplicaciones concurrentes que compartan recursos (archivos y memoria) **sin instalar bibliotecas externas**
- Programas de duración variada: que la espera de los **breves** no sea grande
- Aplicaciones de madurez desconocida: que ninguna haga uso abusivo/exclusivo de la CPU; **limitar de forma fija** el máximo de RAM por programa; limitar la carga total si se inician muchos programas en poco tiempo
- Poca RAM: que no limite el tamaño de los programas; evitar a toda costa **compactar** la memoria
- Discos poco fiables (sectores que se corrompen): que afecte lo menos posible al recupero de archivos; **poca metainformación** para direccionar los datos
- Operar con varios FS a la vez, con **referencias entre ellos**
- Evitar a toda costa el **deadlock** sin uso intensivo del SO

Preguntas:
1. El sistema dispondrá de procesos como unidad central de trabajo. ¿Debería disponer de algún mecanismo de hilos (KLTs, ULTs)? ¿Por qué?
2. ¿Cuál debería ser el algoritmo de planificación de corto plazo?
3. Proponga el diagrama de estados, con la menor cantidad de estados posibles
4. Defina una técnica para lograr que el sistema sea predecible cuando recibe una carga de trabajo elevada
5. Indique qué estrategia de organización de la memoria utilizaría
6. ¿Qué estrategia elegiría para administrar el conjunto residente de los procesos?
7. Seleccione la estrategia de asignación de bloques del filesystem
8. ¿Debería el FS soportar accesos directos? En caso afirmativo, ¿de algún tipo en particular?
9. ¿Sería necesario disponer de alguna estrategia para lidiar con posibles deadlocks?
10. Realice al menos dos preguntas cuya respuesta le permita elegir la arquitectura del kernel más adecuada

<details>
<summary>Ver respuesta</summary>

Resolución (oficial, hay muchas respuestas correctas):
1. ¿Hilos? **KLTs**: concurrencia compartiendo recursos sin instalar nada (los ULTs requieren una biblioteca)
2. Planificación de corto plazo: **SJF con desalojo** (los breves esperan poco) o algún **multinivel con desalojo** entre colas (el desalojo evita que una aplicación monopolice la CPU)
3. Diagrama de estados mínimo: **New** (para limitar la multiprogramación), **Ready, Running, Blocked** (+ Exit). El resto opcional
4. Predecibilidad con carga alta: establecer un **grado máximo de multiprogramación**; los que llegan después esperan en New
5. Organización de memoria: **paginación o segmentación paginada con memoria virtual** (procesos más grandes que la RAM y sin fragmentación externa → sin compactación)
6. Conjunto residente: **asignación fija con reemplazo local** (límite fijo de RAM por proceso)
7. Asignación de bloques: **contigua** (la enlazada se descarta por los discos que se corrompen y la indexada por pedir poca metainformación)
8. Accesos directos: **sí, symbolic links**, porque tienen que poder referenciar entre FS distintos
9. Deadlock: **prevención** (evasión tiene mayor consumo de recursos y overhead; detección permite que ocurra)
10. Preguntas para elegir la arquitectura del kernel:
    - ¿Qué tan importante es la eficiencia, en particular en las syscalls? Mucha → **monolítico**
    - ¿Qué tan importante es la estabilidad y la seguridad (fallas de módulos, accesos a memoria)? Mucha → **microkernel**

</details>

## [Final 2025-12-09] Parte B: aspectos prácticos / relación con la industria / plan de estudios — ✍️ respuesta propia (el PDF del final no trae solución)
Preguntas abiertas (criterio: 6 V/F bien + 1 de estas con coherencia argumentativa). Ideas para armar la respuesta:

### 1) Describir el TP de la materia y al menos tres unidades del programa relacionadas (máx. 1 carilla)

<details>
<summary>Ver respuesta</summary>

- Describir los módulos y cómo se comunican (sockets), qué simulaba cada uno
- Unidades típicas relacionadas: planificación (algoritmos de corto/largo plazo, estados de los procesos), hilos y sincronización (hilos por conexión, semáforos/mutex para estructuras compartidas, condiciones de carrera), memoria (paginación, tablas de páginas, swap), file system / E/S

</details>

### 2) Un uso de IA generativa para asistir o mejorar funcionalidades de un kernel (máx. 1 carilla)

<details>
<summary>Ver respuesta</summary>

- Planificación predictiva: estimar la próxima ráfaga de CPU con un modelo entrenado en vez del promedio exponencial de SJF
- Prefetching/paginación adelantada: predecir qué páginas se van a referenciar para reducir page faults
- Ajuste de parámetros (quantum, grado de multiprogramación, tamaño de caché) según la carga
- Detección de anomalías (malware, thrashing, procesos que monopolizan)
- Límites a mencionar: el overhead (el planificador de corto plazo debe ser liviano), falta de determinismo, latencia → correrlo fuera del camino crítico (en modo usuario o de forma offline) y que el kernel solo consuma sus sugerencias

</details>

### 3) Un concepto de Arquitectura de Computadores y otro de Paradigmas de Programación relevantes en SO (máx. 1 carilla)

<details>
<summary>Ver respuesta</summary>

- Arquitectura: ciclo de instrucción e interrupciones (se atienden al final de la instrucción), modos de ejecución y PSW, jerarquía de memoria/caché (principio de localidad, TLB), DMA y buses, MMU
- Paradigmas: concurrencia; el paradigma funcional (inmutabilidad → sin condiciones de carrera, condiciones de Bernstein); polimorfismo/interfaces (VFS, drivers con una interfaz común); encapsulamiento (monitores); manejo de memoria dinámica y punteros en C (heap, memory leaks)

</details>

## [Final 2026-07-14] Pregunta bonus: relacionar tres asignaturas cursadas con Sistemas Operativos (máx. 1 carilla) — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- Criterios posibles: conceptos de otra materia usados en el TP; conceptos similares a los teóricos; conocimientos de SO útiles en otra materia
- Ejemplos: Arquitectura de Computadores (interrupciones, MMU, DMA), Algoritmos y Estructuras de Datos (colas, listas, árboles B y hash en directorios), Paradigmas (concurrencia, funcional), Redes/Comunicaciones (sockets del TP), Bases de Datos (transacciones ↔ journaling, locks), Sintaxis y Semántica (compilación/enlazado, address binding)

</details>
