# 10 - File System (ejercicios)

Los cálculos de FAT/EXT2 y los gráficos de links están en [11-fat-ext2.md](11-fat-ext2.md).

## Teóricos

### [Cuestionario 06-13] ¿Qué se podría guardar en la FCB?
- **No** el nombre (está en la entrada de directorio)
- Sí: ubicación (bloques/punteros), timestamps, propietario, ACL/permisos, tamaño
- **No** los ID de los procesos que lo tienen abierto (eso está en la tabla de archivos abiertos de cada proceso y en la global)

### [Cuestionario 06-13] Otras respuestas
- En la syscall `write` se le pasa el path del archivo → **F**: el path es solo para abrir; `write` recibe el descriptor
- La FCB siempre está en disco y puede estar en RAM
- En una condición de carrera sobre archivos se pueden usar mutex, pero puede ser más eficiente usar locks
- + granularidad → + costo de mantenimiento
- El tamaño de bloque determina la cantidad de fragmentación interna
- Indexada: permite acceso casi directo, sin frag. externa, fácil encontrar huecos libres; si se corrompe el índice se pierde todo
- Enlazada pura: si se pierde un bloque se pierde el resto, sin acceso directo → FAT lo mejora con la tabla

### [Final 2022-12-21] De los mecanismos de asignación (contigua, enlazada, indexada), el indexado es el que más espacio ocupa en el FCB — ✅ solución oficial
- **F**. En los tres el FCB solo necesita un puntero a bloque (en indexada, al bloque índice) (oficial)

### [Final 2023-09-27] Cuanto más grandes sean las lecturas (bytes por syscall), menos accesos a disco se necesitarán — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Se reducen las syscalls, no los accesos a disco: se leen los mismos bloques (con buffer cache, leer de a poco un bloque ya cargado no vuelve a acceder al disco)

### [Final 2023-12-19] Journaling permite mantener un registro de todas las tareas realizadas por el FS — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Registra las operaciones a realizar (antes de hacerlas, con commits) para poder completarlas o revertirlas ante un fallo y mantener la consistencia; no es un historial permanente de todo lo hecho

### [Final 2024-02-20] La compactación es útil en todos los esquemas de asignación de bloques en disco — ✅ solución oficial
- **F** (oficial): en enlazada e indexada se puede asignar cualquier bloque aunque no sean contiguos. También se aceptó **V** justificando que tener los bloques contiguos mejora los tiempos de acceso en cualquier esquema

### [Final 2024-02-27] Con asignación enlazada, la mejor manera de lograr la menor fragmentación interna es usar next fit — ✅ solución oficial
- **F**. Para reducir la fragmentación interna hay que reducir el tamaño de bloque (oficial). Next fit es un algoritmo de ubicación de huecos (frag. externa)

### [Final 2024-07-23] Con asignación contigua, los archivos de un mismo directorio estarán contiguos entre ellos — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. La contigüidad es entre los bloques de un mismo archivo; cada archivo puede estar en cualquier lugar del disco

### [Final 2024-07-30] Tarda menos copiar un archivo grande a otro FS que muchos archivos chicos que ocupan lo mismo — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Cada archivo implica crear entrada de directorio y FCB, abrir/cerrar, buscar bloques libres, más metadata y más movimientos del cabezal

### [Final 2025-07-29] Todos los bloques de un mismo archivo, al ser de tamaño fijo, sufren fragmentación interna — ✍️ respuesta propia (el PDF del final no trae solución)
- **F**. Solo el último bloque puede tener fragmentación interna

### [Final 2025-12-02 / 2025-12-09] En UFS, el esquema ACL es más granular que el esquema tipo Unix — ✍️ respuesta propia (el PDF del final no trae solución)
- **V**. Unix solo distingue propietario/grupo/resto; ACL permite permisos por usuario/grupo específico

### [Final 2026-05-19] El SO puede impedir la apertura de un archivo aunque exista y no esté siendo usado — ✅ solución oficial
- **V**. Al abrir se validan los permisos: si pide escritura y solo tiene lectura, se impide (oficial)

### [Final 2023-07-25 N°4] ¿Por qué usar locks en lugar de mutex para operar concurrentemente un archivo? ChatGPT: "claridad del código". Dar al menos otro motivo; ¿cambia con locks sugeridos u obligatorios? — ✍️ respuesta propia (el PDF del final no trae solución)
- Granularidad: un lock puede bloquear un rango de bytes (varios procesos trabajando en partes distintas); un mutex bloquea todo
- Locks compartidos (lectura): muchos lectores a la vez; con mutex serían excluyentes → más eficiente
- Funcionan entre procesos no relacionados a través del FS (sin compartir un semáforo) y el SO los libera al cerrar/terminar
- Sugeridos: el SO no los hace cumplir, sirven solo si todos los procesos los usan (como un semáforo, responsabilidad del programador), menos overhead. Obligatorios: el SO los garantiza incluso contra procesos que no los usan, más overhead
