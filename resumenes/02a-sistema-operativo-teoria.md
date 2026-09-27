# 02 - Sistema operativo, syscalls, modos y kernel (teoría)

## SO
- Programa/conjunto de programas que administra el HW, el SW y la interfaz de usuario. Se administra a sí mismo
- Funciones: ejecutar programas, interfaz usuario/programador, administrar recursos, E/S, archivos, comunicación entre programas, protección y seguridad
- Kernel = núcleo, funciones principales (gestión de recursos, sincronización y comunicación entre procesos)
- Distribución = kernel + paquetes (aplicaciones, utilidades: terminal, compilador, debugger)
- El SO **no ejecuta permanentemente**: ejecuta cuando se lo invoca (syscall, interrupción, excepción). Recupera el control con la interrupción de clock
- Si no hay procesos de usuario, la CPU ejecuta el proceso idle

### Evolución
- Monoprogramados: un programa a la vez
  - Procesamiento en serie
  - Lotes sencillos
- Multiprogramados: varios programas concurrentes
  - Lotes multiprogramados (mientras uno espera E/S ejecuta otro)
  - Tiempo compartido (varios usuarios)

## Syscalls
- Funciones del kernel para pedirle servicios al SO (acceso al HW, a recursos que solo maneja el SO)
- Único modo de que una aplicación hable con el HW
- Ej: archivos `open() read() write()`, procesos `fork() exit() kill()`, dispositivos, `time()`, `sem_wait()`
- Cada SO tiene sus propias syscalls (nombres y parámetros) → no portables
- Toda syscall implica ejecutar el SO (cambio a modo kernel), sea bloqueante o no
- Bloqueante: el proceso se bloquea hasta que termina (retorna OK/Error)
- No bloqueante: retorna enseguida aunque no haya terminado (OK/Error/Reintentar), el proceso sigue ejecutando

### Qué pasa al invocar una syscall
- El proceso ejecuta la syscall (instrucción de trap/INT)
- Se guardan los registros del procesador
- Cambio a modo kernel
- El kernel busca la rutina en la syscall table y la ejecuta
- Deja el resultado en el stack del programa
- Si es bloqueante: el proceso pasa a Blocked y se replanifica
- Vuelve a modo usuario

## Wrappers
- Funciones de biblioteca/API estándar (ej: libc) que envuelven a las syscalls
- El wrapper hace la syscall por detrás → el cambio de modo lo hace la syscall, no el wrapper
- Aportan: portabilidad, simplicidad, eficiencia
- No brindan flexibilidad (menos parámetros)

| | Simplicidad | Flexibilidad | Portabilidad |
|---|---|---|---|
| Syscall | Baja (muchos parámetros) | Alta | No |
| Wrapper | Alta | Baja | Sí |

## Modos de ejecución
- Anillos de protección (0 a 3), usamos 2:
  - Kernel (0): cualquier instrucción. Se entra por interrupciones o syscalls
  - Usuario (3): solo instrucciones no privilegiadas
- Las aplicaciones siempre en modo usuario, el SO siempre en modo kernel
- **Todos los procesos ejecutan en modo usuario** (con KLTs o ULTs). El que ejecuta en modo kernel es el SO
- El modo está en el PSW. La CPU sabe qué memoria está permitida según el modo
- Un usuario no puede cambiar el modo. Se cambia por syscall o interrupción
- Nunca se pasa de una aplicación a otra directamente: siempre hay cambio a modo kernel en el medio
- Instrucción privilegiada en modo usuario → excepción → el SO toma el control y normalmente finaliza al proceso (señal). No lo logra nunca
- Validar esto por SW en cada instrucción sería carísimo → lo hace el HW

## Cambio de modo vs cambio de contexto vs cambio de proceso
- Cambio de contexto = guardar el contexto de ejecución (registros) actual y cargar otro. Objetivos:
  - Ejecutar otro proceso
  - Atender una interrupción (interrupt handler)
  - Ejecutar una syscall
- Un cambio de modo siempre implica un cambio de contexto
- Un cambio de contexto no siempre implica un cambio de modo (si ya estaba en kernel, ej: interrupción durante otra interrupción)
- Un cambio de contexto no siempre implica cambio de proceso (interrupciones, syscalls, cambio entre KLTs)
- Un cambio de proceso implica al menos 2 cambios de contexto (proceso → SO → otro proceso) y más de un cambio de modo
- Pasar de una interrupción a otra: cambio de contexto pero no de modo

## Arquitectura de kernel
- **Monolítico**: todo en un módulo, todos acceden a todo
  - + Eficiencia, fluidez de la información, sin overhead
  - − Difícil de mantener, trackeo de errores complejo, un cambio afecta a todo
  - Es el más usado hoy
- **Capas**: cada capa un módulo con interfaz
  - + Más simple de mantener, cambios localizados
  - − Menos fluidez (hay que atravesar capas)
- **Microkernel**: en modo kernel solo lo mínimo (interrupciones, IPC, planificación básica), el resto (FS, drivers, etc.) como procesos en modo usuario
  - + Robustez, fiabilidad, tolerancia a fallos (si falla un módulo no cae todo), flexibilidad para agregar/sacar/intercambiar módulos, compatibilidad (ej: distintos FS)
  - − Performance: los módulos se comunican por mensajes a través del kernel → más cambios de modo y de contexto → más overhead. Mayor complejidad
  - No mejora el rendimiento respecto al monolítico
- Para elegir: ¿importa más la performance de las syscalls (→ monolítico) o la estabilidad/seguridad (→ microkernel)?
