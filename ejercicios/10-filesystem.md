# 10 - File System (ejercicios)

Los cálculos de FAT/EXT2 y los gráficos de links están en [11-fat-ext2.md](11-fat-ext2.md).

## Teóricos

### [Cuestionario File Systems (PDF)] ¿Qué cosas se podrían guardar en el FCB? (varias opciones)
Opciones:

- Nombre del archivo
- Información de ubicación en disco
- Timestamps de modificación, creación, etc.
- Propietario
- ACL
- IDs de los procesos que tienen abierto a dicho archivo
- Tamaño del archivo

<details>
<summary>Ver respuesta</summary>

- **Correctas**: ubicación en disco, timestamps, propietario, ACL, tamaño
- Comentario de la cátedra: el FCB guarda la info inherente al archivo (owner, fechas, info de protección como una ACL, tamaño y cómo encontrar su contenido). El nombre, aunque parece un atributo, es parte del contenido del directorio (el mismo archivo puede tener distintos nombres). Los procesos que lo usan están en las tablas de archivos abiertos.

</details>

### [Cuestionario File Systems (PDF)] Para escribir un archivo hay que hacerlo con una syscall. A dicha syscall, write, se le pasa el path del archivo a escribir
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Falso**
- Comentario de la cátedra: antes de operar hay que abrir el archivo (open()), que recupera el FCB de disco y lo deja en memoria; write recibe el descriptor.

</details>

### [Cuestionario File Systems (PDF)] ¿En dónde podría estar almacenado un FCB?
Opciones:

- En RAM
- En disco
- Ambas son correctas
- Ninguna es correcta

<details>
<summary>Ver respuesta</summary>

- **Ambas son correctas**
- Comentario de la cátedra: tiene que estar en disco para ser durable, y cuando el archivo está abierto su FCB está cargado en memoria.

</details>

### [Cuestionario File Systems (PDF)] Ante una situación de condición de carrera sobre el acceso a un archivo se podría usar mutex, pero podría ser más eficiente usar locks
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Verdadero**
- Comentario de la cátedra: el mutex resuelve la condición de carrera, pero varios procesos podrían querer solo leer: con locks compartidos no se bloquea a quienes no generan problemas de consistencia.

</details>

### [Cuestionario File Systems (PDF)] Compare las distintas estrategias de protección (Propietario/Grupo/Resto, Matriz de accesos, ACL) según granularidad y costo de mantenimiento

<details>
<summary>Ver respuesta</summary>

- **Propietario/Grupo/Resto**: granularidad baja, costo de mantenimiento bajo
- **Matriz de accesos**: granularidad alta, costo de mantenimiento alto
- **ACL**: granularidad media, costo de mantenimiento medio

</details>

### [Cuestionario File Systems (PDF)] En un FS, tener más o menos fragmentación interna dependerá más que nada de la estrategia de asignación de bloques que se utilice
Opciones:

- Verdadero
- Falso

<details>
<summary>Ver respuesta</summary>

- **Falso** (según la cátedra podía justificarse V o F)
- Comentario de la cátedra: todas las estrategias tienen fragmentación interna porque la unidad de asignación es el bloque; la indexada además la tiene en los bloques de punteros. Pero el factor que más la hace variar es el tamaño del bloque.

</details>

### [Cuestionario File Systems (PDF)] ¿Qué ventajas tiene la asignación indexada frente a las otras estrategias? (varias opciones)
Opciones:

- Permite hacer un acceso directo bastante eficiente (aunque podría generar accesos extras)
- No desperdicia espacio en punteros
- No tiene fragmentación externa
- Optimiza el tiempo de acceso en disco por minimizar los movimientos mecánicos del disco
- Es fácil encontrar un hueco libre

<details>
<summary>Ver respuesta</summary>

- **Correctas**: acceso directo bastante eficiente; no tiene fragmentación externa; es fácil encontrar un hueco libre
- Comentario de la cátedra: frente a contigua: no tiene fragmentación externa y cualquier bloque es asignable. Frente a enlazada: buen acceso directo (no lee todos los bloques anteriores, aunque puede leer bloques de punteros). Desperdicia aún más espacio en punteros que la enlazada, y minimizar movimientos del disco solo vale para contigua.

</details>

### [Cuestionario 06-13] Otras respuestas de la clase

<details>
<summary>Ver respuesta</summary>

- + granularidad → + costo de mantenimiento
- El tamaño de bloque determina la cantidad de fragmentación interna
- Indexada: permite acceso casi directo, sin frag. externa, fácil encontrar huecos libres; si se corrompe el índice se pierde todo
- Enlazada pura: si se pierde un bloque se pierde el resto, sin acceso directo → FAT lo mejora con la tabla

</details>

### [Final 2022-12-21] De los mecanismos de asignación (contigua, enlazada, indexada), el indexado es el que más espacio ocupa en el FCB — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. En los tres el FCB solo necesita un puntero a bloque (en indexada, al bloque índice) (oficial)

</details>

### [Final 2023-09-27] Cuanto más grandes sean las lecturas (bytes por syscall), menos accesos a disco se necesitarán — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Se reducen las syscalls, no los accesos a disco: se leen los mismos bloques (con buffer cache, leer de a poco un bloque ya cargado no vuelve a acceder al disco)

</details>

### [Final 2023-12-19] Journaling permite mantener un registro de todas las tareas realizadas por el FS — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Registra las operaciones a realizar (antes de hacerlas, con commits) para poder completarlas o revertirlas ante un fallo y mantener la consistencia; no es un historial permanente de todo lo hecho

</details>

### [Final 2024-02-20] La compactación es útil en todos los esquemas de asignación de bloques en disco — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F** (oficial): en enlazada e indexada se puede asignar cualquier bloque aunque no sean contiguos. También se aceptó **V** justificando que tener los bloques contiguos mejora los tiempos de acceso en cualquier esquema

</details>

### [Final 2024-02-27] Con asignación enlazada, la mejor manera de lograr la menor fragmentación interna es usar next fit — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Para reducir la fragmentación interna hay que reducir el tamaño de bloque (oficial). Next fit es un algoritmo de ubicación de huecos (frag. externa)

</details>

### [Final 2024-07-23] Con asignación contigua, los archivos de un mismo directorio estarán contiguos entre ellos — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. La contigüidad es entre los bloques de un mismo archivo; cada archivo puede estar en cualquier lugar del disco

</details>

### [Final 2024-07-30] Tarda menos copiar un archivo grande a otro FS que muchos archivos chicos que ocupan lo mismo — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Cada archivo implica crear entrada de directorio y FCB, abrir/cerrar, buscar bloques libres, más metadata y más movimientos del cabezal

</details>

### [Final 2025-07-29] Todos los bloques de un mismo archivo, al ser de tamaño fijo, sufren fragmentación interna — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Solo el último bloque puede tener fragmentación interna

</details>

### [Final 2025-12-02 / 2025-12-09] En UFS, el esquema ACL es más granular que el esquema tipo Unix — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Unix solo distingue propietario/grupo/resto; ACL permite permisos por usuario/grupo específico

</details>

### [Final 2026-05-19] El SO puede impedir la apertura de un archivo aunque exista y no esté siendo usado — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Al abrir se validan los permisos: si pide escritura y solo tiene lectura, se impide (oficial)

</details>

### [Final 2023-07-25 N°4] ¿Por qué usar locks en lugar de mutex para operar concurrentemente un archivo? ChatGPT: "claridad del código". Dar al menos otro motivo; ¿cambia con locks sugeridos u obligatorios? — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- Granularidad: un lock puede bloquear un rango de bytes (varios procesos trabajando en partes distintas); un mutex bloquea todo
- Locks compartidos (lectura): muchos lectores a la vez; con mutex serían excluyentes → más eficiente
- Funcionan entre procesos no relacionados a través del FS (sin compartir un semáforo) y el SO los libera al cerrar/terminar
- Sugeridos: el SO no los hace cumplir, sirven solo si todos los procesos los usan (como un semáforo, responsabilidad del programador), menos overhead. Obligatorios: el SO los garantiza incluso contra procesos que no los usan, más overhead

</details>
