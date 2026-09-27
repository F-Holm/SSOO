# 11 - FAT y EXT2/UFS (ejercicios)

## Teóricos

### [Cuestionario 06-13] FAT y EXT
- FAT: asignación enlazada "modificada" con la tabla FAT (si se pierde un bloque en enlazada pura se pierde el resto, y no hay acceso directo)
- En FAT bloque = cluster; entradas de directorio de tamaño fijo; no tiene FCB; FAT12, 16 y 32
- FAT32: entradas de 32 bits pero direcciona 2^28 (4 bits reservados que nunca se usaron; en las otras no existen). Archivos de hasta 4 GiB
- La FAT está siempre entera en RAM y 2 veces en disco
- Softlink: archivo con su propio inodo que contiene la ruta (puede estar en otro FS). Hardlink: no puede apuntar a otro FS, no tiene inodo propio, más eficiente

### [Final 2022-08-02] En UFS, si todos los bloques de datos están ocupados es imposible crear un archivo nuevo
- **F**. Crear un archivo requiere un inodo libre y una entrada de directorio (en un bloque de directorio con lugar); un archivo vacío no necesita bloques de datos

### [Final 2022-09-07] En la mayoría de los casos es posible crear un hard link aún sin bloques libres
- **V**. Solo se agrega una entrada en un directorio, en un bloque que en general no está lleno (oficial)

### [Final 2022-12-06] Los hard links permiten accesos directos sin generar inodos extra (a diferencia de los symbolic links)
- **V**. El hard link es otra entrada de directorio al mismo inodo (sube el contador); el soft link crea un inodo nuevo

### [Final 2023-02-14] Un soft link es mejor que un hard link en versatilidad de almacenamiento en diversos volúmenes y espacio ocupado en disco
- **F**. Es más versátil (puede apuntar a otros volúmenes), pero ocupa **más** espacio (inodo + bloque con la ruta)

### [Final 2023-02-28] En EXT, un acceso directo a un byte puede requerir 4 accesos a bloques diferentes
- **V**. Si el bloque está en la indirección triple: 3 bloques de punteros + el de datos (oficial)

### [Final 2023-03-07] Es posible crear softlinks en FAT32 solo si las referencias están en distintos directorios
- **F**. En FAT32 no se pueden crear softlinks ni hardlinks (oficial)

### [Final 2023-04-25 / 2024-05-10] En FAT, agregar un bitmap de bloques libres (solo en memoria) mejoraría la performance
- **V**. Buscar bloques libres en el bitmap es mucho más rápido que recorrer la FAT secuencialmente (después igual hay que actualizar la FAT) (oficial)

### [Final 2023-05-23] Al copiar archivos de UFS a otra PC con UFS a través de un pendrive FAT32, algunos datos se perderían
- **V**. FAT no guarda permisos, propietario, grupo ni links (los hardlinks se vuelven copias separadas); tampoco admite archivos de más de 4 GiB

### [Final 2023-08-01] Ventajas de FAT sobre UFS: accesos directos más rápidos (en promedio) trayendo menos bloques de disco, y menos estructuras en memoria
- **F**. Lo primero es cierto, pero no requiere menos estructuras en memoria: la tabla FAT entera puede ser muy grande y debe estar en RAM (oficial)

### [Final 2023-12-12] Si un archivo no puede crecer en FAT32 es porque el FS se llenó (sin clusters dañados). Aplica también para ext2
- **F**. En FAT32 puede haber llegado al tamaño máximo de archivo (4 GiB, campo de 32 bits). En ext2 puede haber llegado al máximo que permiten los punteros del inodo, o no haber bloques para punteros / inodos

### [Final 2024-02-27] Los archivos de un FS basado en inodos no pueden ser accedidos de manera secuencial
- **F**. Recorriendo los punteros del inodo en orden se accede secuencialmente (oficial)

### [Final 2024-03-05] En EXT2 y en FAT, al corromperse un bloque de un archivo, siempre se podrá seguir accediendo al resto
- **F**. En EXT2, si se corrompe un bloque de punteros se pierden los N bloques que referencia (oficial). En FAT, si se corrompe la entrada de la FAT (y su copia) se pierde la cadena

### [Final 2025-02-11 / 2025-12-09] Un symbolic link ocupa siempre más espacio en disco que un hard link al mismo archivo
- **V**. El symlink es un archivo nuevo (inodo + bloque con la ruta o la ruta dentro del inodo) además de la entrada de directorio; el hardlink es solo la entrada

### [Final 2025-02-18] Los inodos guardan qué procesos usan el archivo para saber cuándo borrarlo
- **F**. El inodo guarda el contador de hardlinks; los procesos que lo tienen abierto están en las tablas de archivos abiertos (global y por proceso)

### [Final 2025-02-25] Es posible implementar hardlinks en un FS FAT32
- **F**. No hay inodos: los atributos están en la entrada de directorio → dos entradas al mismo archivo quedarían inconsistentes (y no hay contador de links)

### [Final 2025-05-20] Es imposible que un hardlink apunte a un archivo fuera de la partición de origen
- **V**. Apunta a un nro de inodo, que solo es único dentro de su volumen

### [Final 2025-07-15] Un softlink debe tener los mismos permisos que el archivo original
- **F**. Es un inodo propio con sus permisos (normalmente rwxrwxrwx); al usarlo se validan los permisos del archivo destino

### [Final 2025-09-25] Que la FAT esté cargada en memoria es fundamental para evitar accesos consecutivos a disco innecesarios
- **V**. Seguir la cadena de clusters requiere leer una entrada de la FAT por cada cluster; si estuviera en disco sería un acceso por eslabón

### [Final 2025-12-16] Crear un softlink implica crear una nueva entrada de directorio, pero crear un hardlink no
- **F**. Ambos necesitan una nueva entrada de directorio (oficial)

### [Final 2026-02-10] En EXT, al eliminar un hardlink el sistema no elimina el inodo porque asume que puede haber otro hardlink
- **F**. No asume nada: decrementa el contador de hardlinks del inodo y, si llega a 0, libera el inodo y sus bloques (oficial)

### [Final 2026-02-24] Evaluar la respuesta del LLM
- Afirmación: "Un hardlink es análogo al acceso directo de Windows"
- LLM: "Verdadera. Un hardlink es un nuevo archivo que no contiene los datos sino la ruta al archivo real"
- **Incorrecta**: lo que describe es un softlink (oficial). El hardlink es otra entrada de directorio al mismo inodo

## Prácticos

### [Final 2022-09-07 / 2023-04-25 / 2024-05-10] UFS con 12 PD, 1 IS, 1 ID; punteros de 4 B, bloques de 2 KiB; los bloques leídos quedan cacheados (oficial)
a) Punteros por bloque = 2048 / 4 = 512
- i) Archivo máx = (12 + 512 + 512²) × 2 KiB ≈ **513 MiB**
- ii) FS máx = 2^32 × 2^11 = 2^43 = **8 TiB**

b) Copiar dos hardlinks de un archivo de 25 KiB a FAT32 con bloques de 2048 B
- i) 25 KiB / 2 KiB = 12,5 → **13 bloques de datos + 1 bloque de punteros** (IS) = 14. Los dos hardlinks son el mismo inodo: con caché se leen una sola vez
- ii) En FAT son dos archivos distintos: 13 × 2 = **26 clusters**

c) ¿Cualquier archivo del FAT32 se podría copiar al UFS? **No**: en FAT32 el máximo teórico de archivo (criterio cátedra) es 2^28 × 2 KiB = 512 GiB (o 4 GiB por el campo de tamaño) y en el UFS ≈ 513 MiB. Ej: un archivo de 600 MiB entra en FAT32 pero no en el UFS

### [Final 2022-12-21] Disco de 4 TiB formateado con FAT32 (oficial)
- a) Cluster mínimo: 2^42 = 2^28 × 2^x → x = 14 → **16 KiB**
- b) Archivo de 491.521 B: 491.521 / 16.384 = 30 clusters + 1 byte → 31 clusters → 31 × 16 KiB = **507.904 B**
- c) FAT para un pendrive de 1 TiB con el mismo cluster: 2^40 / 2^14 = 2^26 entradas × 4 B = **256 MiB**
- d) Dos archivos de 10 KiB y 20 KiB:
```
Entradas de directorio: archivo1.bin | 10240 B | primer cluster 0
                        archivo2.bin | 20480 B | primer cluster 1
FAT:     [0] EOF  [1] 2  [2] EOF  [3] libre ...
Bloques: [0] datos archivo1  [1] datos archivo2 (1ra parte)  [2] datos archivo2 (2da parte)
```

### [Final 2023-03-07] Copiar un archivo de 16 MiB de Ext2 a Ext3 con 10 PD, 1 IS, 1 ID; punteros de 64 bits; bloques de 1 KiB (oficial)
- Punteros por bloque = 1024 / 8 = 128. Máx archivo = (10 + 128 + 128²) KiB = 16.522 KiB > 16.384 KiB → **se puede**
- a) 16.384 bloques de datos + 1 (IS) + 1 (ID) + 127 (bloques de punteros de 2do nivel: (16.384 − 10 − 128) / 128 = 126,9) = **16.513 bloques**
- b) Leer del byte 10.000 al 400.000: bloques 9 (10.000 div 1024) a 390 → 382 bloques de datos + 1 IS + 1 ID + 2 de 2do nivel (bloques 138-265 y 266-393) = **386 bloques**
- c) **Softlink**: referencia por ruta y puede cruzar FS. Un hardlink no, porque apunta a un nro de inodo que Ext2 no conoce en el otro FS

### [Final 2025-12-02] Copiar un archivo de 16 MiB de Ext2 a otro Ext2 con 10 PD, 1 IS, 1 ID; punteros de 64 bits; bloques de 1 KiB
- a) Igual que el anterior: se puede (máx 16.522 KiB); hay que escribir **16.513 bloques** (16.384 de datos + 129 de punteros)
- b) **Softlink** (desde el FS original a la copia en otro FS). Hardlink no: los nros de inodo son locales a cada volumen

### [Final 2023-07-25 N°9] Copiar un archivo de 11 KiB de FAT32 (clusters de 4 KB) a EXT2 con 10 PD, 1 IS, 1 ID, punteros de 64 bits, bloques de 1 KiB. ChatGPT: 10 bloques directos + 1 IS que apunta a 10 más + 1 ID + 10 bloques de IS = 22 bloques. Evaluar
- **Incorrecta**: 11 KiB = 11 bloques de datos. 10 por punteros directos + 1 por la IS (que tiene 128 punteros, no 10) → **11 + 1 = 12 bloques**. No hace falta la indirección doble
- Se puede copiar: máx = (10 + 128 + 128²) KiB ≈ 16,1 MiB

### [Final 2023-09-27] FAT32 con bloques de 4 KiB dañado; hay que recuperar un archivo de 82 MiB. Solo hay dos bloques dañados en todo el FS y los bloques de datos del archivo son legibles, pero solo se pudieron leer los primeros 13 MiB
- a) Los datos del archivo se ubican siguiendo la cadena en la FAT. 13 MiB = 3.328 clusters: la entrada de la FAT que indica el cluster siguiente está en un bloque de la FAT dañado, y el mismo bloque de la copia (FAT2) es el otro dañado → no hay forma de saber cuál es el siguiente cluster, aunque los datos estén sanos
- b) Sí: si un bloque dañado es el del directorio que contiene la entrada del archivo, se pierde el primer cluster (y el tamaño) → no se puede recuperar nada (también si se daña el sector de arranque, que dice dónde están la FAT y el raíz). Otro caso: las dos copias dañadas justo en la entrada del primer cluster → solo se recuperan los primeros 4 KiB
- c) En EXT2 (inodo legible, bloques de 4 KiB, punteros de 4 B → 1024 por bloque; IS = 4 MiB, ID = 4 GiB)
  1. Más de 13 MiB: los dos bloques dañados son de datos → se recupera todo menos 8 KiB. O dos bloques de punteros de 2do nivel del ID → se pierden 8 MiB y se recuperan ~74 MiB
  2. Menos de 13 MiB: se daña el bloque del ID (primer nivel) → solo se recuperan los 12 directos + el IS = 48 KiB + 4 MiB. Si además se daña el IS → solo 48 KiB

### [Final 2023-12-12] Graficar las estructuras de /home/user/file1.doc (contenido "hola") con un symlink /etc/s-file1.doc y un hardlink /etc/h-file1.doc
```
Inodo 2  (/)          tipo dir, links 4  → bloque 100: [ . → 2 | .. → 2 | home → 10 | etc → 20 ]
Inodo 10 (/home)      tipo dir, links 3  → bloque 101: [ . → 10 | .. → 2 | user → 11 ]
Inodo 11 (/home/user) tipo dir, links 2  → bloque 102: [ . → 11 | .. → 10 | file1.doc → 12 ]
Inodo 20 (/etc)       tipo dir, links 2  → bloque 103: [ . → 20 | .. → 2 | s-file1.doc → 30 | h-file1.doc → 12 ]

Inodo 12 (file1.doc / h-file1.doc)  tipo regular, links 2, tamaño 4   → bloque 200: "hola"
Inodo 30 (s-file1.doc)              tipo symlink, links 1, tamaño 20  → bloque 201: "/home/user/file1.doc"
```
- El hardlink no crea inodo: es una entrada más que apunta al inodo 12 (contador = 2)
- El symlink es un inodo nuevo (30) cuyo contenido es la ruta del original

### [Final 2023-12-19] Mismo ejercicio con file1.doc de solo lectura, indicando también permisos
- Igual al anterior, agregando permisos:
  - Inodo 12 (file1.doc y h-file1.doc): `-r--r--r--` (444), links 2 → el hardlink tiene exactamente los mismos permisos (es el mismo inodo)
  - Inodo 30 (s-file1.doc): `lrwxrwxrwx` (777), links 1 → sus permisos no importan, al accederlo se validan los del inodo 12 (solo lectura)
  - Directorios: `drwxr-xr-x` (755)

### [Final 2024-03-05] Discos de 0,5 TiB con EXT2, bloques de 1024 B, punteros de 32 bits; 12 PD, 1 IS y **dos** ID (oficial, con correcciones)
- Punteros por bloque = 1024 / 4 = 256
- a) Máx archivo = [12 + 256 + 2 × 256²] × 1024 = 134.492.160 B ≈ **128,26 MiB** → el archivo de 52 MiB entra. Puede crecer 128,26 − 52 = **76,26 MiB** (la información administrativa no cuenta dentro de lo que direcciona el inodo)
- b) Leer 3024 bloques desde el byte 13.312: 13.312 / 1024 = bloque 13 (en la oficial figura 133024, es un typo). Bloques 13 a 3036: 255 están en el IS (13-267) y 2769 en el primer ID → ⌈2769 / 256⌉ = 11 bloques de 2do nivel + 1 del ID → 3024 + 1 (IS) + 12 = **3037 accesos**
- c) FS teórico = 2^32 × 2^10 = 4 TiB; real = min(4 TiB, 0,5 TiB) = 0,5 TiB → diferencia = 4096 GiB − 512 GiB = **3584 GiB**

### [Final 2024-07-23] Archivo de 234.567 B en /etc/so, EXT2 con bloques de 1 KiB y punteros de 8 B; inodos con 5 PD, 1 IS, 1 ID. Se quiere un acceso directo en /home/desktop; /home y /etc están en particiones distintas
- Punteros por bloque = 1024 / 8 = 128
- a) Máx archivo = (5 + 128 + 128²) × 1 KiB = 16.517 KiB ≈ **16,13 MiB**
- b) 234.567 / 1024 = 229,07 → 230 bloques de datos. 5 directos, 128 por el IS, quedan 97 → ID + 1 bloque de 2do nivel. Punteros: 1 + 1 + 1 = 3 → **233 bloques**
- c) **Soft link** (el hard link no puede cruzar particiones: el nro de inodo es local al volumen). Necesita: un inodo nuevo (tipo symlink) en la partición de /home, una entrada en el directorio /home/desktop que apunte a ese inodo, y un bloque de datos (o el propio inodo si es corta) con la ruta "/etc/so/archivo"

### [Final 2024-07-30] Archivo de 98.765.432 B en el directorio raíz de una partición FAT que ocupa todo un disco de 256 GiB, clusters de 1 KiB, usando la FAT que menos espacio ocupe
- Clusters = 2^38 / 2^10 = 2^28 → FAT12 y FAT16 no alcanzan → **FAT32** (28 bits útiles, justo)
- a) 98.765.432 / 1024 = 96.450,6 → **96.451 accesos** a clusters de datos (la FAT está en memoria; +1 si hay que leer el bloque del directorio raíz para obtener el primer cluster)
- b) Máx real de archivo: **4 GiB** (campo de tamaño de 32 bits en la entrada de directorio). Criterio cátedra: el FS = 256 GiB
- c) Truncar a 126.049.321 B → ⌈126.049.321 / 1024⌉ = 123.096 clusters → 123.096 − 96.451 = **26.645 entradas** adicionales
- d) Con clusters de 4 KiB: máx teórico = 2^28 × 4 KiB = **1 TiB** (criterio cátedra; real: 4 GiB por el campo y 256 GiB por el disco)
- e) FAT = 2^28 entradas × 4 B = 1 GiB → 1 / 256 = **0,39 %** del disco

### [Final 2025-02-18] EXT2: crear final.doc de 140 KiB. 10 PD, 1 IS, 1 ID, 1 IT; bloques de 1 KiB, punteros de 8 B
- a) 128 punteros por bloque. 140 bloques de datos: 10 directos + 128 por IS + 2 por ID (1 bloque ID + 1 de 2do nivel) → 140 + 3 = **143 bloques**
- b) Hardlink: una entrada nueva en el directorio con el **mismo** inodo → el contador del inodo pasa a 2. Softlink: un **inodo nuevo** tipo symlink + una entrada nueva + un bloque con la ruta
```
Directorio (inodo 50) → bloque 700: [ . → 50 | .. → 2 | final.doc → 100 | hl-final.doc → 100 | sl-final.doc → 101 ]
Inodo 100: regular, links 2, 140 KiB, 10 PD + IS + ID
Inodo 101: symlink, links 1, tamaño 9, PD[0] → bloque 900: "final.doc"
Modificados: bloque del directorio (700), tabla de inodos (100 y 101), bitmap de inodos (101), bitmap de bloques (900)
```

### [Final 2025-07-15] FAT16 con clusters de 64 KiB en la primera partición de un disco de 16 GiB. La segunda partición tiene EXT2 con FS real máximo de 8 GiB y otros 2 GiB que no puede direccionar → la segunda mide 10 GiB y la primera **6 GiB**
- a) Archivo: teórico = 2^16 × 64 KiB = **4 GiB**; real = **4 GiB** (el FS no pasa de 4 GiB; además el campo de tamaño es de 32 bits)
- b) FS: teórico = **4 GiB**; real = min(4 GiB, 6 GiB) = **4 GiB** (quedan 2 GiB de la partición sin direccionar)
- c) FAT = 2^16 entradas × 2 B = **128 KiB**
- d) Peor caso para buscar un cluster libre: recorrer toda la FAT (en memoria) → **65.536 accesos** a memoria

### [Final 2025-12-16] Mover un archivo de EXT2 (punteros de 32 bits, bloques de 4 KiB; 12 PD, 1 IS, 1 ID, 1 IT) a FAT32. El archivo usa 1026 bloques de punteros, todos llenos (oficial)
- a) 1024 punteros por bloque. 1026 bloques de punteros = IS (1) + ID (1 + 1024) → llenos el nivel directo, el simple y el doble → tamaño ≥ (12 + 1024 + 1024²) × 4 KiB > 4 GiB (solo el doble ya son 2^10 × 2^10 × 2^12 = 4 GiB) → **no se puede** mover a FAT32
- b) FAT32 de 1 GiB con bloques de 2 KiB: 2^30 / 4 B = 2^28 entradas → máx direccionable = 2^28 × 2^11 = **512 GiB**

### [Final 2026-02-10] Tamaño máximo teórico y real de un archivo (oficial, con observación)
- a) UFS con 12 PD, 1 IS, 1 ID y **dos** IT; bloques de 512 B, punteros de 2 B → 256 punteros por bloque; disco de 10 GB
  - Teórico = (12 + 256 + 256² + 2 × 256³) × 512 B ≈ **16 GiB**; real = **10 GB** (disco) según la resolución
  - Ojo: con punteros de 2 B el FS solo puede direccionar 2^16 bloques × 512 B = 32 MiB, así que en rigor el real sería 32 MiB
- b) FAT32 con clusters de 32 KB y disco de 1 TB: teórico = 2^28 × 2^15 = **8 TiB**; real = **1 TB** (disco) según la resolución (por el campo de tamaño, un archivo real no supera 4 GiB)

### [Final 2026-02-24] Disco de 2 GiB con FAT16; los archivos de configuración de pocos bytes ocupan mucho. Al pasar a FAT32 mejora (oficial)
- a) FAT16 direcciona 2^16 clusters → cluster = 2^31 / 2^16 = **32 KiB**. Cada archivo ocupa como mínimo un cluster → cada archivo de pocos bytes ocupaba 32 KiB (fragmentación interna)
- b) Ahora ocupan mínimo 4 KiB → cluster de 4 KiB → entradas = 2^31 / 2^12 = **2^19** (524.288)

### [Clase 06-13] Ejercicios de clase (solo resultados, sin enunciado)
- Ver [resumen 11b](../resumenes/11b-fat-ext2-practica.md)
