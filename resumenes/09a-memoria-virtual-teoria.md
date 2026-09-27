# 09 - Memoria virtual (teoría)

## Idea
- Antes: overlays (se cargaban/descargaban partes del programa a mano)
- Memoria virtual = almacenamiento secundario (disco/swap) direccionable como si fuera RAM
- Solo están en RAM las partes que se usan; el resto en disco
- Requisitos: traducción en tiempo de ejecución + proceso dividido en partes (páginas o segmentos)
- Se puede hacer con paginación o con segmentación pura
- Las DL referencian la memoria virtual (más grandes), las DF la real. La MMU traduce
- Sin DL no hay VM: el proceso usa siempre la misma DL aunque la página esté en disco o en cualquier marco
- Con VM no se suspenden procesos (siempre activos, al menos una página en RAM)
- Área de swap: Windows usa un archivo de paginación (tamaño configurable); Linux una partición swap (tamaño fijo)

### Ventajas
- Procesos más grandes que la RAM
- Más grado de multiprogramación (aún si los procesos entran en RAM)
- Carga más rápida (solo lo necesario), menos restricciones para el programador
### Desventajas
- Más accesos a memoria y a disco, no mejora el tiempo de ejecución (búsqueda + PF bloquean al proceso)
- Los procesos pueden estar más tiempo bloqueados (PF)
- Funciona bien gracias al **principio de localidad**: en un intervalo solo se usa activamente un conjunto de páginas (localidad). Es indispensable

## Paginación bajo demanda
- El proceso está en swap y solo se pasa a RAM lo necesario. Swapping lazy: mueve páginas, no procesos
- Tabla de páginas: marco, bit de **presencia** (P), bit de **modificado** (M), bit de **uso** (U), bits de protección
- P = 0 → **Page Fault** (interrupción sincrónica/excepción de la MMU)
- Siempre hay un bitmap/array de marcos libres
- Si no hay marcos libres → elegir **víctima**; si M = 1 hay que escribirla a disco

### Proceso de traducción (DL → DF)
1. La CPU ejecuta una instrucción que referencia una página
2. MMU busca en la TLB → hit → DF
3. Miss → MMU busca en la tabla de páginas → P = 1 → agrega a la TLB → DF
4. P = 0 → MMU lanza PF → el SO atiende la interrupción:
   - Página fuera del espacio de direcciones → fin del proceso / error
   - Página válida (en disco) → buscar frame libre
     - Hay frame libre → operación de lectura de disco + frame ocupado
     - No hay → algoritmo + política de sustitución elige víctima
       - M = 0 → se marca frame libre + página ausente
       - M = 1 → operación de escritura (esperar interrupción fin de E/S) y después liberar
   - El proceso queda bloqueado mientras tanto (otro puede usar la CPU)
   - Interrupción fin de E/S (DMA) → el SO marca la página presente, actualiza la tabla → proceso a Ready
5. Se vuelve a ejecutar la instrucción que causó el PF
- Un PF siempre accede a disco al menos una vez (traer la página)
- Con paginación jerárquica una referencia puede generar más de un PF (tabla + página)

## Políticas del SO para VM (objetivo: reducir PF)
- **Recuperación**: bajo demanda (cuando se necesita) / paginación adelantada (prepaging: trae páginas vecinas). Se combinan
- **Ubicación**: solo importa con segmentación (first/next/best/worst fit). En paginación da igual un marco u otro
- **Reemplazo/sustitución**: bloqueo de marcos (bit de lock, no se pueden reemplazar) + algoritmo de reemplazo
- **Conjunto residente**: asignación fija o variable, reemplazo local o global
- **Limpieza**: bajo demanda (se escribe al reemplazar) / adelantada (el SO libera páginas no usadas)

## Asignación y sustitución de frames
- Asignación fija: cantidad de frames del proceso constante
- Asignación dinámica: puede variar (busca la cantidad óptima para menos PF)
- Sustitución local: la víctima es un frame del mismo proceso
- Sustitución global: cualquier frame de cualquier proceso
- Fija + global no tiene sentido (el proceso pasaría a tener más frames que los permitidos)
- Dinámica + local: en cada PF se decide si darle un frame más o sustituir; si siempre se le da, se puede llevar más frames de los que debería

## Algoritmos de reemplazo (solo se ejecutan si hubo PF y no hay frame libre)
- **Óptimo**: reemplaza la que se va a usar más lejos en el futuro. No implementable, solo para comparar. Nadie tiene menos PF
- **FIFO**: la que está hace más tiempo (por instante de carga). Solo necesita un puntero. Sufre **anomalía de Belady** (+ frames ⇒ + PF)
- **LRU**: la usada hace más tiempo. Suele ser el mejor. No sufre Belady. Mucho overhead: registrar el instante de cada referencia → necesita soporte de HW
- **Clock (segunda oportunidad)**: cola circular con puntero y bit de uso (U = 1 al cargar o referenciar)
  - U = 0 → víctima
  - U = 1 → se pone U = 0 y avanza
  - La estructura circular no es la tabla de páginas
- **Clock mejorado / modificado**: usa (U, M), busca no tener que escribir a disco (no reduce PF necesariamente)
  1. Busca (0,0) avanzando **sin** cambiar U
  2. Si no hay, busca (0,1) avanzando **y poniendo U = 0**
  3. Si no hay, vuelve al paso 1 (y si hace falta al 2). Máximo 4 pasadas
  - Preferencia: (0,0) > (0,1) > (1,0) > (1,1)

## Thrashing / sobrepaginación
- Demasiados PF: no se puede ejecutar porque no alcanzan los frames (el proceso pasa más tiempo paginando que ejecutando)
- Ej: instrucción con op1 y op2 en páginas distintas y pocos frames
- Síntoma: **baja** utilización de CPU (procesos bloqueados esperando PF) y alta actividad de disco
- Causas: mal algoritmo de reemplazo, pocos frames por proceso, aumento del grado de multiprogramación
- Local + fija: ciclo infinito de PF de ese proceso (le puede pasar aunque sea el único proceso)
- Global + dinámica: el SO ve CPU baja → sube la multiprogramación → más robos de frames → más PF (círculo vicioso)
- Aumentar la multiprogramación ante CPU baja no siempre ayuda (si es thrashing, empeora)
- Cómo evitarlo:
  - Frecuencia de PF: si hay muchos → bajar el grado de multiprogramación / darle más frames
  - Conjunto de trabajo (lo que necesita ahora, según la localidad) debe entrar en los frames asignados
- Algoritmos de reemplazo y thrashing están relacionados

## Otras técnicas
- **Lockeo de páginas**: bloquear páginas necesarias para una E/S en curso para que no se reemplacen
- **Page buffering**: pool de frames libres; la víctima va a un buffer → se escriben en grupo y si se vuelve a pedir antes de escribirse no hay acceso a disco. Evita el bit de bloqueo
- **Compartir páginas**: procesos del mismo programa comparten páginas de código (read-only) → mismo marco en ambas tablas
- **Copy-on-write** (fork): el hijo comparte los marcos del padre en solo lectura; se copia una página recién cuando alguien la escribe
- **Archivos mapeados en memoria** (`mmap`): ver FS
