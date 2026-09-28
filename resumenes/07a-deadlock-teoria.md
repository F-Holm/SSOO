# 07 - Deadlock / Interbloqueo (teoría)

## Definición
- Bloqueo permanente de un conjunto de procesos donde cada uno espera un evento que solo puede generar otro del conjunto
- Consecuencia de mala sincronización / mal uso de recursos compartidos
- No alcanza con "pedir recursos en común": tienen que darse las 4 condiciones
- Los procesos en deadlock están bloqueados: no pueden romper la espera circular por sí mismos
- Livelock: parecido, pero cambian de estado constantemente sin progresar (consumen CPU → sí afectan a otros procesos). Es una forma de inanición
- Un deadlock no afecta a procesos que no comparten recursos con los involucrados (no consume CPU); sí afecta a los que necesiten recursos retenidos
- Inanición (starvation) ≠ deadlock: al proceso se le niega el recurso indefinidamente pero podría obtenerlo

## Recursos
- Reutilizables (se piden, usan y liberan) → acá se da el deadlock
- Consumibles (se usan y desaparecen)
- Gestionados por el SO / no gestionados por el SO

## Grafo de asignación de recursos
- Círculos = procesos, cuadrados = recursos, puntos = instancias
- Flecha proceso → recurso = solicitud; recurso → proceso = asignación
- Sin ciclos → no hay deadlock
- Con ciclo → puede o no haber deadlock
- Ciclo y todos los recursos del ciclo con una sola instancia → hay deadlock
- En general no se justifica solo con el grafo, hay que usar el algoritmo de detección

## Condiciones
- Necesarias:
  - Mutua exclusión (recurso no compartible)
  - Retención y espera (retiene y pide más)
  - Sin desalojo (no se le puede quitar el recurso)
- Necesaria y suficiente: las 3 + **espera circular**
- Sin las 4 no hay deadlock (ni siquiera con semáforos con espera activa)
- Si el recurso se puede acceder en paralelo (compartido) no hay mutua exclusión → ni deadlock ni livelock
- Sin concurrencia (un proceso a la vez hasta que termina) no puede haber deadlock
- Hilos con condición de carrera (sin mutua exclusión) son menos propensos a deadlock
- Los que no pueden acceder a un recurso de 1 instancia no quedan necesariamente en deadlock: falta la espera circular

## Estrategias

| | Garantiza no deadlock | Overhead | Flexibilidad | Cuándo |
|---|---|---|---|---|
| Prevención | Sí | Bajo | Baja (restrictiva, subutiliza recursos) | Sistemas críticos, HW limitado, minimizar overhead |
| Evasión | Sí | Muy alto (en cada solicitud) | Media | Críticos, mejor uso de recursos que prevención |
| Detección y recuperación | No (ocurre y se arregla) | Bajo (periódico) | Alta (asigna libremente) | Uso general |
| No tratarlo | No | Nulo | Total | La mayoría de los SO, criticidad baja |

### Prevención: romper una de las 4 condiciones
- Mutua exclusión: no siempre se puede (recursos no compartibles)
- Retención y espera: pedir todos los recursos juntos al inicio (si falta uno se deniega todo) o no pedir si ya tiene uno → ineficiencia, puede generar inanición
- Sin desalojo: si A pide un recurso que tiene B (que está esperando), se le quita a B → hay que poder retrotraer y notificar
- Espera circular: numerar los recursos y pedir solo en orden creciente
- Si el SO expropia para romper la espera circular, podría volver a formarse

### Evasión (predicción)
- Denegar el inicio de un proceso si pide más que los totales
- Denegar la asignación: **algoritmo del banquero**. Requiere que cada proceso declare sus necesidades máximas
  - Simula la asignación; si el estado queda seguro, asigna; si no, el proceso espera (aunque el recurso esté disponible)
  - Estado seguro: existe una secuencia en la que todos pueden terminar → nunca deadlock
  - Estado inseguro: **podría** haber deadlock (no es seguro que ocurra)
  - Bien implementado el sistema nunca queda inseguro ni en deadlock → no "reacciona" ante deadlocks (no ocurren), no desaloja
  - El SO puede rechazar un pedido aunque el recurso esté disponible

### Detección y recuperación
- No hay restricciones al asignar. Se ejecuta periódicamente el algoritmo de detección (no necesita necesidades máximas, usa peticiones actuales)
- Recuperación:
  - Matar a todos los involucrados (no es buena solución)
  - Volver a un estado anterior (checkpoint, complejo)
  - Matar de a uno hasta que no haya deadlock
  - Expropiar recursos de a uno
- Criterio de víctima: menor tiempo de CPU consumido, menor salida producida, mayor tiempo restante, menos recursos asignados, menor prioridad
- Elegir siempre a la misma víctima (ej: baja prioridad) → starvation
- No sirve donde el deadlock es inaceptable; matar procesos puede tener impacto negativo

### No tratarlo
- La mayoría de los SO (criticidad baja)

## Prevención y detección pueden generar starvation
- Prevención: retención y espera (esperar a tener todo)
- Detección/recuperación: si siempre se elige la misma víctima

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
