# 01 - Arquitectura e interrupciones (ejercicios)

## Teóricos

### [Cuestionario 03-28] ¿En qué momento se atienden las interrupciones (si no están deshabilitadas)?
- a) En cuanto ocurren — b) Antes de que el planificador elija otro proceso — **c) Al finalizar la instrucción en curso** — d) Depende de si ejecutaba código de usuario o del SO
- No se puede interrumpir en medio del ciclo de instrucción. Se atienden aunque esté ejecutando el SO. Salvedad: si ya se está atendiendo otra, puede tener que esperar (atención secuencial o si la nueva no es más prioritaria)

### [Cuestionario 03-28] ¿Cuál de estos eventos podría interrumpir la ejecución y terminar de ejecutarse antes que el resto?
- **Aviso de finalización de una operación de E/S** (es una interrupción, se atiende al fin de la instrucción)
- Solicitar una E/S y crear un proceso son syscalls: la ejecución se detiene por esperar la respuesta de la syscall, no por una interrupción

### [Cuestionario 03-28 / 04-04] ¿Atender una interrupción involucra siempre un cambio de modo?
- **Falso**. Si ya se estaba atendiendo otra interrupción o ejecutando una rutina del SO, ya está en modo kernel (interrupciones anidadas)

### [Cuestionario 03-28 / 04-04] CLI deshabilita las interrupciones. ¿Qué ocurre si se ejecuta?
- **Depende del modo**. Es privilegiada: en modo kernel cambia el bit IF del PSW; en modo usuario lanza una excepción

### [Cuestionario 03-28] ¿Cuáles son interrupciones sincrónicas?
- **Sí**: acceder a una dirección de memoria no permitida, división por cero, llamado explícito a lanzar una interrupción
- **No**: fin de quantum, fin de E/S, error de un dispositivo (son asincrónicas, externas a la CPU)

### [Cuestionario 04-04] Otras respuestas del cuestionario
- La CPU sabe qué lugar de memoria está permitido según el modo de ejecución
- Fin de quantum no es sincrónica
- Todas las interrupciones no enmascarables son de máxima prioridad
- Interrupciones sincrónicas == excepciones (en esta materia)

### [Resumen] ¿En qué consiste el ciclo de ejecución de una instrucción? ¿Podría surgir una interrupción como consecuencia del ciclo?
- Fetch (busca según PC), decode (IR, operandos), execute; al final se chequean interrupciones y PC++ (o salto)
- Sí, excepciones (sincrónicas): división por cero, acceso inválido, page fault

### [Final 2022-12-06] Todas las interrupciones implican algún tipo de error producido en el sistema, implicando una finalización del proceso afectado
- **F**. Hay interrupciones que no son errores (fin de E/S, clock, syscalls) e incluso excepciones que no terminan al proceso (page fault: se trae la página y se reejecuta)

### [Final 2023-05-23] Dentro del ciclo de instrucción se incrementa el PC, por lo tanto la ejecución siempre se realiza en forma secuencial
- **F**. Las instrucciones de salto (JMP, CALL) y las interrupciones modifican el PC

### [Final 2024-02-27] El PSW es un conjunto de registros que no pueden ser manipulados explícita y arbitrariamente por los procesos
- **V**. Escribirlos requiere modo kernel; los manipula el procesador como resultado de ejecutar instrucciones (resolución oficial)

### [Final 2025-02-25] La rutina de ejecución de una llamada al sistema podría ser interrumpida al presionar una tecla
- **V**. Al final de cada instrucción se chequean interrupciones aunque se esté ejecutando código del SO. Se atiende la interrupción del teclado y luego se vuelve a la syscall (salvo que estén deshabilitadas)

### [Final 2025-07-29] Cuando un proceso es interrumpido, puede reanudar su ejecución después de la interrupción sólo si es el de prioridad más alta
- **F**. Depende del algoritmo: si es sin desalojo o la interrupción no genera replanificación, vuelve al mismo proceso sin importar prioridades; en RR vuelve cuando le toque

### [Final 2025-02-18] Al interrumpir un proceso siempre se cambia su estado y se ejecuta otro proceso
- **F**. Tras atender la interrupción (ej: fin de E/S de otro proceso) se puede volver al mismo proceso, que sigue en running

### [Final 2025-12-16] Las únicas interrupciones que se podrían atender en el medio del ciclo de instrucción son las no enmascarables
- **F**. Toda interrupción se atiende luego del ciclo de instrucción (oficial)

### [Final 2026-02-24] Evaluar la respuesta del LLM
- Afirmación: "Las instrucciones que ejecuta la CPU son atómicas en todos los casos"
- Respuesta del LLM: "No es del todo cierto. Solo son atómicas al deshabilitar las interrupciones; si no, podrían ser interrumpidas en el medio"
- **Incorrecta**. La atomicidad de una instrucción no depende de las interrupciones: solo se atienden al finalizar la instrucción en curso (oficial)
