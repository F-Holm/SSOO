# 03 - Procesos (teoría)

## Definiciones
- Programa = entidad pasiva (instrucciones en disco). Proceso = entidad activa (programa en ejecución con recursos asignados)
- Proceso = Tarea (término antiguo). Es la unidad de trabajo del SO
- Concurrencia: compartir recursos, varios en el mismo intervalo de tiempo (una CPU, varias colas). Simula paralelismo
- Paralelismo: varios en el mismo instante (cada uno con su procesador)
- Multiprogramación: varios procesos activos en memoria turnándose la CPU
- Multiprocesamiento: varios procesadores ejecutando a la vez
- Aunque sea monoprocesador, varios procesos monohilo **sí** son concurrentes
- Entorno del proceso = variables que se le pasan al ejecutarse

## Imagen del proceso (memoria usada por el proceso)
- Código: instrucciones (solo lectura)
- Datos: variables globales/estáticas y constantes
- Heap: memoria dinámica (malloc/free, responsabilidad del programador → memory leaks)
- Stack: variables locales, parámetros, direcciones de retorno, retornos de funciones
- PCB

## PCB (Process Control Block)
- Una por proceso, la crea y gestiona el SO. **Siempre está en RAM**. El proceso no la puede modificar
- Guarda el contexto para poder hacer cambios de estado/contexto sin problemas
- Contiene:
  - ID: PID, PPID, UID
  - Estado del proceso
  - PC / IP
  - Registros de la CPU (PSW)
  - Info de planificación (prioridad)
  - Info de gestión de memoria
  - Info contable (tiempo de CPU)
  - Info de estado de E/S (archivos abiertos)
  - Punteros
- Al finalizar se liberan todas las estructuras menos la PCB (queda para el valor de retorno / estadística → zombie)

## Duración de los datos (C)
- Estática: vive todo el proceso (globales o `static`)
- Automática: locales, viven mientras dure el bloque
- Asignada: heap, hasta el free

## Estados
### 5 estados
```
new -> ready <-> running -> exit
         ^--- blocked <---|
```
- new → ready: admitted (planificador de largo plazo)
- ready → running: scheduler dispatch (corto plazo)
- running → ready: interrupt (fin de quantum, desalojo)
- running → blocked: E/S o espera de evento (syscall bloqueante)
- blocked → ready: fin de E/S / evento (se avisa por interrupción; lo hace el SO al atender la interrupción)
- running → exit: fin (última instrucción, error, kill). De cualquier estado se puede ir a exit
- blocked → running directo: **no** (siempre pasa por ready)
- ready → blocked: **no** existe
- New: se crean las estructuras, se inicializa la PCB, espera la admisión

### 7 estados
- Agrega Suspendido listo y Suspendido bloqueado (imagen en swap/disco, la PCB sigue en RAM)
- bloqueado → susp. bloqueado: se manda a swap
- susp. bloqueado → susp. listo: terminó el evento (sigue en disco)
- susp. listo → listo: vuelve a MP
- listo → susp. listo: admitido pero sin RAM
- new → susp. listo: demasiados procesos en ready
- Con memoria virtual prácticamente no se usan
- Los SO suelen tener más estados, estos son los mínimos

### Estados en Linux
- R (running + ready), S (sleep interrumpible), D (uninterruptible sleep), T (stopped), Z (zombie: terminó pero la PCB sigue hasta que el padre la reclama), X (dead), + (foreground)

## Creación
- La crea el SO o otro proceso (padre → hijo), siempre por syscall
- Pasos: asignar PID → reservar espacio para estructuras → inicializar PCB → ubicar PCB en colas de planificación
- `fork()` crea una copia del proceso padre (registros, pila, heap, hasta el PC)
  - Distinto PID; el hijo tiene PPID = PID del padre
  - `fork()` retorna 0 en el hijo y el PID del hijo en el padre
  - Por defecto **no comparten memoria**, ni siquiera con el padre (copia). Pueden compartir memoria explícitamente o por paso de mensajes (ej: sockets del TP0)
  - Heredan archivos abiertos
- `exec()` reemplaza la imagen por otro programa
- Padre e hijo pueden ejecutar concurrentemente o el padre esperar (`wait()`)
- Tabla de procesos: el grado de multiprogramación sale de los procesos activos

## Finalización
- Normal exit (`exit()`), abnormal exit, lo mata el SO u otro proceso (`kill`), el padre (`abort()`)
- Al finalizar liberan sus recursos
- Si muere el padre: el hijo puede seguir (en Linux lo adopta init). En algunos sistemas se matan en cascada

## Procesos vs hilos
- Procesos: más estables y confiables (aislados); la muerte de uno no afecta a los demás
- Arquitectura multiproceso > multihilo si importa seguridad/estabilidad; multihilo si importa rendimiento (creación y comunicación más rápidas)
- La unidad mínima de planificación son los hilos

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
