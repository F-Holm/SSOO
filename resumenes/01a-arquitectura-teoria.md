# 01 - Repaso de arquitectura (teoría)

## Componentes
- Procesador = registros + ALU + unidad de control
- Arquitectura de Von Neumann
- Memoria RAM = gran vector de bytes (direcciones lineales). No conoce su contenido
- Bus = transfiere datos entre CPU, E/S y memoria
  - Bus de datos
  - Bus de direcciones
  - Bus de control
- Pipelining
- HyperThreading (Intel)
- Vemos los drivers como si fueran parte del Kernel

## Registros
- Visibles por el usuario (uso general): AX, BX, CX, ... (MMX)
- De control y estado (no modificables por el programador, sí consultables):
  - PC / IP: dirección de la próxima instrucción
  - IR: instrucción que se está ejecutando (no la dirección)
  - MAR: dirección del próximo operando
  - MBR: dato/instrucción de la celda que apunta MAR
  - PSW: estado luego de una operación (overflow, carry, flags), **modo de ejecución** e **IF** (interrupt flag)
  - SP: stack pointer
- El PSW no puede ser manipulado arbitrariamente por los procesos (escribirlo requiere modo kernel)

## Instrucciones
- Sentencia (ej: `i = i + 1`) != instrucción → son 3 instrucciones (MOV, ADD, MOV)
- No privilegiadas: cualquier programa (MOV, ADD, SUB, JNZ, JZ, CALL)
- Privilegiadas: solo en modo kernel (CLI, STI, HLT, operaciones de E/S)
- CLI = deshabilita interrupciones
  - En modo kernel: cambia el bit IF del PSW
  - En modo usuario: lanza una excepción
- El HW (no el SO) valida el modo de ejecución antes de ejecutar una instrucción privilegiada

## Ciclo de instrucción
- Fetch (busca la instrucción según el PC) → Decode (a IR, busca operandos) → Execute
- Si no hubo salto: PC++. Con JMP/CALL el PC cambia → la ejecución NO siempre es secuencial
- Las etapas son inseparables: **las interrupciones se atienden al final de la instrucción en curso** (nunca en el medio, ni siquiera las no enmascarables)
- Ciclo con interrupciones: al final de cada instrucción →
  - ¿Hubo no enmascarable? → se atiende
  - ¿Están habilitadas las enmascarables? → ¿hubo alguna? → se atiende
  - Si no → siguiente instrucción

## Interrupciones
- Mecanismo de HW para avisar a la CPU que ocurrió un evento
- Todas deben ser atendidas en algún momento
- Atender una interrupción cambia a modo kernel, **salvo que ya esté en modo kernel** (interrupciones anidadas)
- Clasificación:
  - Hardware (externas a la CPU) / Software (generadas por la CPU)
  - Enmascarables (se pueden ignorar temporalmente, no son de máxima prioridad) / No enmascarables (críticas, fallas de HW, máxima prioridad)
  - Sincrónicas (resultado de ejecutar una instrucción, "predecibles") / Asincrónicas (externas, en cualquier momento)
  - Interrupciones sincrónicas == excepciones (en esta materia)
  - E/S: fin de una operación de E/S (asincrónica)
  - Clock: fin de quantum (asincrónica, **no** es sincrónica)
  - Excepciones: errores o condiciones anómalas (div por 0, PF, acceso inválido)
    - Aborts: error grave de HW
    - Fallos: se corrigen y se retoma (ej: page fault)
    - Traps: debugging / llamado explícito (INT)
- Ejemplos sincrónicas: acceso a memoria no permitida, división por cero, llamado explícito a una interrupción
- Ejemplos asincrónicas: fin de quantum, fin de E/S, error de un dispositivo
- Teclado tiene prioridad 1 (máxima prioridad)
- No todas las interrupciones son errores ni terminan al proceso (fin de E/S, clock, PF)

### Pasos al atender una interrupción
- HW:
  1. Se genera la interrupción (en cualquier momento)
  2. Finaliza la instrucción actual
  3. Se identifica la interrupción y el dispositivo
  4. Se guardan PC y PSW
  5. Se carga en el PC la dirección del manejador (interrupt handler) → ejecuta el SO
- SO:
  6. Guarda el resto de los registros
  7. Inhabilita interrupciones (si corresponde)
  8. Procesa la interrupción
  9. Restaura registros
  10. Restaura PSW y después PC (primero PSW porque al restaurar el PC ya ejecuta)
  11. Habilita interrupciones

### Múltiples interrupciones
- Secuenciales (una atrás de otra)
- Por prioridad: una más prioritaria interrumpe a la rutina de una menos prioritaria (se pausa la actual)
- Deshabilitar interrupciones mientras se atiende una

## Jerarquía de memoria
- Arriba: rápido, chico, caro (registros, caché). Abajo: lento, grande, barato (disco)
- RAM: ns (nanosegundos) | SSD: µs (microsegundos)
- Volátil (se borra al apagar) / No volátil
- Suspender la PC = mantener energía en la memoria

## Overhead
- Overhead = procesamiento extra "innecesario" (no útil para el usuario), ej: cambios de contexto, planificación

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
