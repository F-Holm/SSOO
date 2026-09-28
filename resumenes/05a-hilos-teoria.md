# 05 - Hilos (teoría)

## Hilo
- Unidad básica de utilización de la CPU (línea de ejecución de un proceso)
- Un proceso está formado por uno o más hilos
- Comparte con sus hilos pares: código, datos, heap, recursos (archivos abiertos, permisos), PCB
- Propio de cada hilo: **stack** y **TCB** (Thread Control Block)
- TCB: TID, estado, prioridad, contexto (PC, registros, flags), puntero al stack
- Permiten paralelismo dentro de un proceso
- Al compartir memoria se comunican sin mecanismos de IPC del SO (sin syscalls)
- No hay protección entre hilos: uno puede escribir en la pila de otro (a nivel memoria es posible, lógicamente no deberían)
- Las variables locales (stack) no se comparten, las globales sí
- Cambio de hilo = cambio de contexto

## Hilos vs procesos
- + Comunicación privada entre hilos sin intervención del SO
- + Más eficiente cambiar de hilo que de proceso
- + Más eficiente crear un hilo que un proceso hijo (no se copia la imagen, solo nueva TCB + stack)
- − Si muere el proceso, mueren todos sus hilos (el SO libera los recursos del proceso)
- − Un hilo puede afectar a todo el proceso (comparten recursos); un proceso multihilo podría recuperarse de la muerte de un hilo
- − Memory leaks: los hilos comparten el heap → lo que no libera un hilo queda ocupado hasta que termina el proceso (también en KLTs aunque el TCB no apunte al heap)
- Procesos: más estables y confiables, la muerte de uno no afecta a los pares
- Server con biblioteca con memory leaks → conviene un **proceso** por solicitud (al terminar se libera todo), no un hilo

## Estado del proceso según sus hilos
- Si algún hilo está Ejecutando → proceso Ejecutando
- Si ninguno ejecuta y alguno está Listo → proceso Listo
- Bloqueado solo si todos sus hilos están bloqueados

## pthreads
```c
#include <pthread.h>
pthread_t hilo;
int iret;
pthread_create(&hilo, NULL, funcion, parametro);
pthread_join(hilo, &iret);
pthread_detach(hilo);
```

## KLT (Kernel Level Threads)
- El SO los conoce, crea, destruye y planifica (planifica hilos, no procesos)
- + Syscall bloqueante solo bloquea a ese hilo
- + Multiprocesamiento: hilos del mismo proceso en distintas CPUs (paralelismo real)
- + Menos overhead que usar procesos
- − Más overhead que ULTs: toda operación (crear, switch) implica syscall/cambio de modo
- − Menos estables que procesos (memory leaks)

## ULT (User Level Threads)
- Gestionados por una biblioteca en modo usuario; el SO no sabe que existen (ve un proceso/KLT)
- Doble planificación: el SO planifica el proceso, la biblioteca los hilos (algoritmo propio, puede ser uno que el SO no soporta)
- Se crean sin syscalls
- + Bajo overhead (sin syscalls ni cambios de modo), conmutación más rápida
- + Planificación personalizada
- + Portabilidad (biblioteca estándar)
- + Creación y switch entre ULTs del mismo KLT más livianos que entre KLTs
- − No permiten paralelismo entre hilos pares (no se pueden repartir en CPUs)
- − Una operación bloqueante (E/S) bloquea a todos los hilos del proceso
- Solución: **jacketing** → la biblioteca hace la E/S de forma no bloqueante y "bloquea internamente" al hilo; pregunta cada tanto si terminó
- Jacketing **simula** paralelismo: evita el bloqueo del proceso pero NO permite ejecutar ULTs en paralelo
- ULTs con jacketing ≠ KLTs: siguen invisibles para el SO, sin paralelismo, con otra planificación
- Por default se bloquea el KLT (sin jacketing); podría bloquearse el KLT y el proceso asociado
- Al terminar la operación bloqueante podría seguir el mismo ULT o otro (según si hubo wrapper/replanificación)
- Si se bloquean todos los ULTs de un KLT, suele devolver la CPU
- Se pueden invocar syscalls desde un ULT (ej: `fork()`): el KLT hace el pedido al SO
- Se pueden combinar KLTs y ULTs en un mismo proceso (beneficios de ambos)

### Syscalls y wrappers en ULTs
- Syscall bloqueante directa: no hay replanificación, cuando vuelve sigue el mismo ULT (el PC quedó ahí)
- Wrapper de la biblioteca de ULTs: guarda el contexto antes del bloqueo y replanifica al finalizar → se respeta la planificación de la biblioteca
- Casi siempre se usan los wrappers en vez de las syscalls con ULTs
- Si el enunciado dice "la biblioteca maneja sus E/S" → usa sus wrappers
- wait/signal de semáforos del SO son syscalls → implican cambio de modo también desde ULTs

## Cambio de KLT a otro del mismo proceso
- Se guarda PC y flags en el stack del sistema; el SO guarda el resto de registros
- El SO decide hacer thread switch; mueve el contexto a la TCB del hilo
- Elige el próximo hilo, copia su TCB al procesador y vuelve a modo usuario
- No se tocan punteros a código/datos/heap (son del proceso)

## Comparación
| | Overhead | Multiprocesamiento | Protección |
|---|---|---|---|
| Procesos | Alto | Sí | Alta (aislados) |
| KLT | Medio (todo pasa por el SO) | Sí | Hilos manejados por el SO |
| ULT | Bajo | No | Hilos manejados por biblioteca de usuario |

## Cuándo usar ULT en vez de KLT
- Planificación propia, bajo overhead, portabilidad, create/switch rápidos
- KLT si se necesita paralelismo o que la E/S no bloquee a todos
- Si se pide concurrencia compartiendo recursos **sin instalar bibliotecas** → KLTs
- Permisos sobre archivos: son del proceso (tabla de archivos abiertos por proceso) → iguales para todos sus hilos

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
