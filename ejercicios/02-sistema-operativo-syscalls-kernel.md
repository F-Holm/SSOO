# 02 - SO, syscalls, modos y kernel (ejercicios)

## Teóricos

### [Resumen] Diferencia entre syscall y wrapper. ¿Cuándo usar cada una?
- Syscall: función del kernel para pedir servicios al SO; propia de cada SO (nombres y parámetros) → poco portable y poco simple, pero flexible/precisa
- Wrapper: función de biblioteca/API estándar que envuelve la syscall → portable y simple
- Wrapper por portabilidad/simplicidad; syscall directa si se necesita un parámetro/funcionalidad específica del SO

### [Resumen] Con modos de ejecución: a) forma correcta de operar sobre el HW, b) forma incorrecta
- a) El programa (modo usuario) hace una syscall y el SO ejecuta la operación en modo kernel
- b) Ejecutar la instrucción privilegiada en modo usuario: el HW la detecta y lanza una excepción; nunca se permite porque no sería seguro

### [Resumen] ¿Qué ocurre cuando se invoca una syscall desde un proceso en modo usuario?
- El proceso ejecuta la syscall → se guardan registros → cambio a modo kernel (si es bloqueante el proceso se bloquea) → el kernel busca la rutina en la syscall table y la ejecuta → deja el resultado en el stack → vuelve a modo usuario / el proceso vuelve a ready

### [Resumen] V/F: Los wrappers de las syscalls permiten que se realice el cambio de modo de usuario a kernel
- **F**. El wrapper es una función común que hace la syscall por detrás; el cambio de modo lo produce la syscall

### [Resumen] V/F: Nunca puede ocurrir un cambio de contexto sin realizar un cambio de proceso
- **F**. Ej: atendiendo una syscall llega una interrupción → contexto de la syscall → contexto del handler sin cambiar de proceso. También cambio entre KLTs del mismo proceso

### [Resumen] V/F: Si se ejecutaba una syscall y ocurre una interrupción, se espera a que finalice la syscall para atenderla, por ser código del SO
- **F**. Se atiende al finalizar la instrucción en curso aunque sea código del SO; después se vuelve a la syscall

### [Resumen] Compare wrapper y syscall en simplicidad, flexibilidad y portabilidad
| | Simplicidad | Flexibilidad | Portabilidad |
|---|---|---|---|
| Syscall | Baja (muchos parámetros específicos) | Alta | No (cada SO tiene las suyas) |
| Wrapper | Alta (menos parámetros) | Baja | Sí (solo requiere la biblioteca estándar) |

### [Resumen] Defina syscall. ¿Relación con modos e instrucciones privilegiadas?
- Función del kernel que cualquier proceso puede llamar para pedir un servicio. Implica cambio a modo kernel porque solo el SO puede ejecutar instrucciones privilegiadas

### [Cuestionario 04-04] Wrappers y kernels
- Los wrappers no brindan flexibilidad
- Microkernel vs monolítico: el microkernel tiene mayor complejidad, muchos cambios de modo y mejor compatibilidad (ej: distintos sistemas de archivos como módulos)

### [Cuestionario 03-28] ¿Cuáles son las ventajas de los microkernels?
- **Sí**: robustez, fiabilidad, tolerancia a fallas (menos corriendo en modo kernel); facilidad de intercambiar un módulo con otro
- **No**: eficiencia en la comunicación entre módulos (necesitan IPC → más cambios de modo y de contexto); no es el más adoptado (lo es el monolítico)

### [Resumen] V/F: Una de las mayores desventajas de los microkernels es su problema de performance
- **V**. Los módulos se comunican por mensajes a través del kernel → overhead extra

### [Final 2022-12-21] Las instrucciones privilegiadas sólo pueden ser ejecutadas por el sistema operativo
- **V**. Operan sobre el HW o serían disruptivas; los procesos acceden a ellas solo a través de syscalls (oficial)

### [Final 2023-04-25] Previo a cada ejecución de una instrucción privilegiada, el SO interviene para verificar que el proceso tenga los privilegios adecuados
- **F**. Quien valida el modo de ejecución es el hardware, no el SO (oficial)

### [Final 2023-12-12] Si un programa compilado contiene instrucciones privilegiadas, logrará ejecutar acciones críticas poniendo en peligro el sistema
- **F**. El proceso ejecuta en modo usuario: al intentar ejecutarla, el HW lanza una excepción y el SO toma el control (normalmente lo finaliza)

### [Final 2025-02-11 / 2025-12-09] Un proceso solamente puede ejecutar en modo kernel cuando ejecuta una syscall
- **F**. Los procesos siempre ejecutan en modo usuario; en modo kernel ejecuta el SO, y se entra no solo por syscalls sino también por interrupciones y excepciones

### [Final 2025-05-20] La ejecución de una llamada al sistema evita llamar al SO si no es bloqueante
- **F**. Toda syscall es atendida por el SO (cambio a modo kernel); no bloqueante solo significa que el proceso no queda bloqueado esperando

### [Final 2026-05-19] Todas las syscalls provocan que el proceso se bloquee en caso que el recurso no se encuentre disponible
- **F**. Hay syscalls no bloqueantes (o configurables así) que retornan inmediatamente aunque la operación no haya terminado (oficial)

### [Final 2025-12-09] El SO necesita estar permanentemente ejecutando para poder gestionar y controlar el comportamiento de los procesos
- **F**. El SO ejecuta solo cuando se lo invoca (syscalls, interrupciones, excepciones). Recupera el control periódicamente con la interrupción de clock

### [Final 2026-02-10] Un SO con arquitectura microkernel mejora el rendimiento en comparación con uno monolítico
- **F**. Al tener servicios en espacio de usuario requiere comunicación entre procesos → más overhead que un monolítico (oficial)

### [Final 2023-07-25 N°7] ChatGPT: "En Linux los procesos en modo usuario no pueden ejecutar instrucciones privilegiadas; cualquier intento resulta en interrupción". Evaluar. Si es correcta, ¿qué ocurre luego? ¿No sería más eficiente que el SO valide la instrucción?
- **Correcta**
- Luego de la interrupción (excepción): cambio a modo kernel, el handler del SO la atiende y normalmente le manda una señal al proceso (SIGILL/SIGSEGV) que lo finaliza
- Que el SO valide cada instrucción sería mucho menos eficiente: implicaría intervención del SO (cambio de modo) en cada instrucción. El HW lo chequea "gratis" mirando el modo en el PSW

### [Final 2023-07-25 N°3] Sistema de altas prestaciones y poca experiencia en desarrollo: ¿syscalls bloqueantes o no bloqueantes? ChatGPT: "bloqueantes, son más fáciles". Elaborar la justificación con pseudocódigo y cambios de estado
- Bloqueante: el código es secuencial, el resultado está disponible al retornar
  ```
  n = read(fd, buf, size)   // Running → Blocked → (fin E/S) Ready → Running
  procesar(buf, n)
  ```
  Mientras espera, el proceso está en Blocked y no consume CPU → la CPU la usan otros (bueno para altas prestaciones)
- No bloqueante: hay que manejar el "todavía no" (reintentar/EAGAIN) → más complejo y propenso a errores
  ```
  while ((n = read_nb(fd, buf, size)) == REINTENTAR)
      hacer_otra_cosa()      // el proceso sigue en Running/Ready
  procesar(buf, n)
  ```
  Si no hay otra cosa útil para hacer, queda en espera activa consumiendo CPU
