# 01 - Arquitectura e interrupciones (ejercicios)

## Teóricos

Las primeras 6 preguntas son las del cuestionario de repaso de la clase 03-28 (`apuntes/Cuestionario - Repaso arquitectura e intro a SO (03-28).pdf`), con la respuesta correcta y el comentario de la cátedra.

### [Cuestionario de repaso 03-28 (PDF)] ¿En qué momento se atienden las interrupciones (considerando que no están deshabilitadas)?
Opciones:

- a) En cuanto ocurren
- b) Antes de que el planificador elija a otro proceso a ejecutar
- c) Al finalizar de atender la instrucción en curso
- d) Depende de si lo que estaba ejecutando era código de un proceso de usuario o del SO

<details>
<summary>Ver respuesta</summary>

- **c) Al finalizar de atender la instrucción en curso**
- No se puede interrumpir en medio del ciclo de instrucción. Se atienden aunque esté ejecutando el SO. Salvedad: si ya se está atendiendo otra, puede tener que esperar (atención secuencial o si la nueva no es más prioritaria)

</details>

### [Cuestionario de repaso 03-28 (PDF)] ¿Cuál de estos eventos podría interrumpir la ejecución y por lo tanto terminar de ejecutar antes que el resto?
Opciones:

- a) Aviso de finalización de una operación de E/S
- b) Solicitud de realizar una operación de E/S
- c) Creación de un nuevo proceso

<details>
<summary>Ver respuesta</summary>

- **a) Aviso de finalización de una operación de E/S** (es una interrupción del DMA, se atiende al fin de la instrucción en curso)
- Solicitar una E/S y crear un proceso son syscalls: la ejecución se detiene por esperar la respuesta de la syscall, no por una interrupción

</details>

### [Cuestionario de repaso 03-28 (PDF) / Cuestionario 04-04] ¿Atender una interrupción involucrará siempre un cambio de modo?
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Falso**. Si ya se estaba atendiendo otra interrupción o ejecutando una rutina del SO, ya está en modo kernel (interrupciones anidadas)

</details>

### [Cuestionario de repaso 03-28 (PDF) / Cuestionario 04-04] CLI es una instrucción que deshabilita las interrupciones. ¿Qué debería ocurrir si se ejecuta?
Opciones:

- a) Lanzar una excepción
- b) Cambiar el bit de IF (interrupt flag) en el PSW
- c) Depende de en qué modo se ejecute

<details>
<summary>Ver respuesta</summary>

- **c) Depende del modo**. Es privilegiada: en modo kernel cambia el bit IF del PSW; en modo usuario lanza una excepción

</details>

### [Cuestionario de repaso 03-28 (PDF)] ¿Cuál/es de las siguientes son interrupciones sincrónicas? (varias opciones)
Opciones:

- acceder a una dirección de memoria no permitida
- fin de quantum
- fin de E/S
- división por cero
- error de un dispositivo
- llamado explícito a lanzar una interrupción

<details>
<summary>Ver respuesta</summary>

- **Sí**: acceder a una dirección de memoria no permitida, división por cero, llamado explícito a lanzar una interrupción
- **No**: fin de quantum, fin de E/S, error de un dispositivo (son asincrónicas, externas a la CPU)

</details>

### [Cuestionario de repaso 03-28 (PDF)] ¿Cuáles son las ventajas de los microkernels? (varias opciones)
Opciones:

- robustez, fiabilidad, tolerancia a fallas (menos corriendo en kernel mode)
- eficiencia en la comunicación entre módulos
- facilidad de intercambiar un módulo con otro
- es el formato de kernel más adoptado en los SOs actuales

<details>
<summary>Ver respuesta</summary>

- **Sí**: robustez, fiabilidad, tolerancia a fallas; facilidad de intercambiar un módulo con otro (ej: el file system)
- **No**: eficiencia en la comunicación entre módulos (necesitan IPC → más cambios de modo y de contexto); no es el más adoptado (lo es el monolítico, justamente por la velocidad)

</details>

### [Cuestionario 04-04] Otras respuestas del cuestionario

<details>
<summary>Ver respuesta</summary>

- La CPU sabe qué lugar de memoria está permitido según el modo de ejecución
- Fin de quantum no es sincrónica
- Todas las interrupciones no enmascarables son de máxima prioridad
- Interrupciones sincrónicas == excepciones (en esta materia)

</details>

### [Resumen] ¿En qué consiste el ciclo de ejecución de una instrucción? ¿Podría surgir una interrupción como consecuencia del ciclo?

<details>
<summary>Ver respuesta</summary>

- Fetch (busca según PC), decode (IR, operandos), execute; al final se chequean interrupciones y PC++ (o salto)
- Sí, excepciones (sincrónicas): división por cero, acceso inválido, page fault

</details>

### [Final 2022-12-06] Todas las interrupciones implican algún tipo de error producido en el sistema, implicando una finalización del proceso afectado — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Hay interrupciones que no son errores (fin de E/S, clock, syscalls) e incluso excepciones que no terminan al proceso (page fault: se trae la página y se reejecuta)

</details>

### [Final 2023-05-23] Dentro del ciclo de instrucción se incrementa el PC, por lo tanto la ejecución siempre se realiza en forma secuencial — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Las instrucciones de salto (JMP, CALL) y las interrupciones modifican el PC

</details>

### [Final 2024-02-27] El PSW es un conjunto de registros que no pueden ser manipulados explícita y arbitrariamente por los procesos — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Escribirlos requiere modo kernel; los manipula el procesador como resultado de ejecutar instrucciones (resolución oficial)

</details>

### [Final 2025-02-25] La rutina de ejecución de una llamada al sistema podría ser interrumpida al presionar una tecla — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Al final de cada instrucción se chequean interrupciones aunque se esté ejecutando código del SO. Se atiende la interrupción del teclado y luego se vuelve a la syscall (salvo que estén deshabilitadas)

</details>

### [Final 2025-07-29] Cuando un proceso es interrumpido, puede reanudar su ejecución después de la interrupción sólo si es el de prioridad más alta — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Depende del algoritmo: si es sin desalojo o la interrupción no genera replanificación, vuelve al mismo proceso sin importar prioridades; en RR vuelve cuando le toque

</details>

### [Final 2025-02-18] Al interrumpir un proceso siempre se cambia su estado y se ejecuta otro proceso — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Tras atender la interrupción (ej: fin de E/S de otro proceso) se puede volver al mismo proceso, que sigue en running

</details>

### [Final 2025-12-16] Las únicas interrupciones que se podrían atender en el medio del ciclo de instrucción son las no enmascarables — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Toda interrupción se atiende luego del ciclo de instrucción (oficial)

</details>

### [Final 2026-02-24] Evaluar la respuesta del LLM — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- Afirmación: "Las instrucciones que ejecuta la CPU son atómicas en todos los casos"
- Respuesta del LLM: "No es del todo cierto. Solo son atómicas al deshabilitar las interrupciones; si no, podrían ser interrumpidas en el medio"
- **Incorrecta**. La atomicidad de una instrucción no depende de las interrupciones: solo se atienden al finalizar la instrucción en curso (oficial)

</details>

---
[⬆ Volver al índice de ejercicios](00-indice.md) · [Índice general](../README.md)
