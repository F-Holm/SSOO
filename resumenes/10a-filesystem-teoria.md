# 10 - File System (teoría)

## Objetivos
- Almacenar datos y operar con ellos
- Garantizar (en lo posible) la integridad de los datos
- Optimizar desempeño: usuario → tiempo de respuesta; sistema → aprovechamiento de recursos
- Soporte para distintos tipos de almacenamiento
- Minimizar o eliminar la pérdida o destrucción de datos
- Interfaz estandarizada para procesos de usuario (así cada proceso no implementa su propio acceso)
- Soporte para múltiples usuarios (protección)

## Archivo
- Conjunto de datos relacionados, etiquetados con un nombre (solo para humanos) y almacenados en un medio secundario
- Existencia de larga duración, compartible entre procesos, tiene estructura
- Atributos: nombre, identificador, tipo, ubicación, tamaño, permisos, fechas, propietario
- Tipos: regulares, directorios, dispositivos (/dev/sda), sockets, terminales, links, ejecutables

## FCB (File Control Block)
- Al archivo lo que la PCB al proceso. Uno por archivo
- Contiene: permisos, fechas (creación, acceso, modificación), dueño/grupo/ACL, tamaño, bloques de datos (o punteros a punteros)
- **No** contiene el nombre (está en la entrada de directorio)
- **No** tiene los procesos que lo tienen abierto (eso está en las tablas de archivos abiertos)
- Siempre está en disco y puede estar en RAM (se cachea mientras se usa)
- Se guarda en la metadata del FS (tabla de inodos en UFS). FAT no tiene FCB (atributos en la entrada de directorio)

## Directorios
- Archivo que contiene la lista de nombres de archivos y sus atributos/FCB asociados. Mapea nombre → archivo
- Estructura: un nivel, dos niveles, árbol, grafo acíclico (lo más normal, permite links), grafo general
- Implementación de las entradas: lista lineal, lista enlazada ordenada, árbol B, tabla de hash
- Operaciones: buscar, crear, renombrar, borrar, listar, recorrer
- Ruta absoluta (desde el punto de montaje, única) / relativa (desde el working directory, `./`)
- Punto de montaje: directorio donde se hace accesible otro FS (Linux: un directorio; Windows: letra C:, E:)

## Operaciones
- Básicas (syscalls): crear, eliminar, abrir, cerrar, leer, escribir, posicionar puntero (seek), truncar (cambiar tamaño)
- Crear y borrar no requieren abrir. El resto sí (para validar permisos y tener los atributos en memoria)
- `write` recibe el **descriptor** (no el path; el path es solo para abrir)
- Compuestas: copiar (abrir, leer, crear, escribir, cerrar), mover/renombrar
- Mover dentro del mismo FS = cambiar la entrada de directorio (rápido). A otro FS = copiar + borrar (tarda: se pasa al nuevo FS)
- Copiar muchos archivos chicos tarda más que uno grande del mismo tamaño total (crear entradas, FCBs, abrir/cerrar, metadata)

## Tablas de archivos abiertos (en memoria)
- Global: FCB (en memoria), cantidad de procesos que lo tienen abierto (contador de aperturas), permisos
- Por proceso: modo de apertura, puntero de posición, puntero a la entrada global
- Los permisos sobre archivos abiertos son a nivel proceso → iguales para todos sus hilos; el puntero de posición también se comparte entre hilos

## Locks
- Regulan el acceso a un archivo (o a un rango de bytes). Evitan leer información desactualizada, mantienen la integridad
- Exclusivo (escritura): uno a la vez
- Compartido (lectura): muchos a la vez, pero no junto a uno exclusivo
- Obligatorio (Windows): el SO garantiza el lock → archivos más tiempo bloqueados, más overhead
- Sugerido (Linux): el SO informa el estado pero es responsabilidad del programador → menos overhead
- Frente a un mutex: más granularidad (rango de bytes), compartidos para lectores, funcionan entre procesos sin compartir semáforos → más eficiente
- Más granularidad → más costo de mantenimiento (granularidad = qué tan específico)

## Protección
- Acceso total (sin protección) / Acceso prohibido o restringido (solo el propietario) / Acceso controlado (quién y cómo)
- Tipo Unix: propietario | grupo | universo, `tipo rwx rwx rwx` (9 bits). `chmod 777` (octal)
  - Directorios: r = listar (ls), w = modificar contenido, x = entrar (cd)
- Matriz de acceso (usuarios × recursos): máxima granularidad, desperdicia mucho espacio
- ACL: por archivo, lista de usuarios con permisos → más granular que Unix, menos espacio que la matriz
- Se puede combinar Unix + ACL (lista de excepciones)
- Contraseñas por archivo (casi no se usa)
- El SO puede impedir abrir un archivo existente y sin uso (permisos insuficientes para el modo pedido)

## Archivos mapeados en memoria (`mmap`)
- Tratar la E/S de archivo como accesos a memoria: se asocia parte del espacio virtual del proceso con el archivo
- Cada bloque se mapea a una página (no necesariamente 1:1)
- Una vez cargada la página, lecturas/escrituras = accesos a memoria. Eficiente para archivos grandes (`msync`, `munmap`)

## Organización en disco
```
Disco
  MBR
  Partición / Volumen
    Bloque de control de arranque del volumen (boot)
    Control de volumen (superbloque): info general del FS (cant. bloques, tamaño, libres). Siempre en memoria
    META DATA: FCBs (parcialmente en memoria), estructura de directorios
    Archivos (bloques de datos)
```
- Formatear = crear las estructuras del FS en una partición (instalar un volumen)
- En memoria: estructura de directorios, tablas de archivos abiertos, tabla de montaje
- Archivo → bloques lógicos → sectores. Bloque = múltiplo del tamaño de sector. Se lee/escribe de a bloque
- Un bloque no puede tener datos de dos archivos

## Métodos de acceso
- Secuencial, directo, indexado, hashed

## Asignación de bloques
- Todos tienen frag. interna (solo en el último bloque del archivo, máx tam_bloque − 1). + tamaño de bloque → + frag. interna
- **Contigua**: bloque inicial + cantidad. Rápida (secuencial y directo), mucha frag. externa, difícil crecer. Poca metadata. Los archivos de un directorio NO tienen por qué estar contiguos entre sí
- **Enlazada/encadenada**: cada bloque apunta al siguiente (se guarda inicio y fin). Sin frag. externa, crece fácil. Malo para acceso directo (hay que recorrer), espacio de punteros, un puntero corrupto = se pierde el resto
- **Indexada**: bloque índice con punteros a todos los bloques. Acceso casi directo (2 accesos), sin frag. externa, fácil crecer. Limita el tamaño máx del archivo, espacio de punteros, si se corrompe el índice se pierde todo. Se pueden enlazar índices o usar multinivel/combinado (inodos)
  - En el FCB solo se guarda un puntero (al índice) igual que en las otras → no ocupa más en el FCB
- Compactación: solo necesaria/útil en contigua (en enlazada/indexada no hace falta, aunque la contigüidad mejora tiempos)

## Gestión de espacio libre
- Bitmap / bit vector (rápido, tamaño fijo)
- Lista de bloques libres enlazados
- Lista de porciones contiguas
- Indexada
- Agrupamiento (bloques con listas de libres)

## Recuperación / consistencia
- La info en memoria suele estar más actualizada que en disco → un fallo genera inconsistencias
- Comprobación de coherencia (chkdsk / fsck), backups
- **Journaling**: antes de operar se registran (log) las operaciones a realizar; con commits entre operaciones. Si falla, se rehace o se revierte hasta el último commit → la operación se hace entera o no se hace. No es un historial permanente de todo lo hecho

## Ejemplo: crear y escribir un archivo nuevo
1. Crear: FCB libre + entrada de directorio
2. Abrir: agregar a tablas de archivos abiertos (global y del proceso)
3. Obtener bloques libres y asignarlos
4. Escribir bloques, actualizar atributos (tamaño, fechas)
5. Cerrar: actualizar tablas
