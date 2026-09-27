# 11 - FAT y EXT2 (práctica: cálculos)

## FAT
- Bits útiles: FAT12 = 12, FAT16 = 16, FAT32 = **28** (entrada de 32 bits = 4 bytes)
- Tamaño de entrada: FAT12 = 1,5 B, FAT16 = 2 B, FAT32 = 4 B
- Cant. clusters del volumen = tam partición / tam cluster (≤ 2^bits útiles)
- Tamaño FAT = cant_entradas × tam_entrada (cant_entradas = clusters del volumen; si no, 2^bits)
- % disco ocupado por la FAT = (cant_copias × tam FAT) / tam disco
- Tam máx teórico FS = 2^bits × tam_cluster
- Tam máx real FS = min(teórico, partición)
- Tam máx archivo: cátedra = tam FS; real = 4 GiB (campo tamaño de 32 bits)
- Cluster mínimo para direccionar un disco = tam disco / 2^bits
- Elegir la FAT que "menos espacio ocupa" y direcciona todo: la de menos bits que alcance
- Bits sin usar en la entrada: 32 − bits necesarios − 4 reservados (FAT32)
- Espacio que ocupa un archivo = ⌈tamaño / cluster⌉ × cluster
- Leer un archivo entero: 1 acceso por cluster (la FAT está en memoria) + leer la entrada de directorio si no está cacheada
- Truncar a un tamaño mayor: entradas extra = ⌈nuevo/cluster⌉ − ⌈viejo/cluster⌉
- Buscar un cluster libre (peor caso): recorrer toda la FAT → cant_entradas accesos a memoria
- Graficar: entradas de directorio (nombre, tamaño, primer cluster) + extracto de la FAT (cadenas terminadas en EOF) + bloques de datos

## EXT2 / UFS
- Punteros por bloque = tam_bloque / tam_puntero (p)
- Tam máx teórico archivo = (PD + IS·p + ID·p² + IT·p³) × tam_bloque
  - Contar cada puntero indirecto: 2 dobles → 2·p²
- Tam máx teórico FS = 2^(bits puntero) × tam_bloque
- Tam máx real FS = min(teórico FS, disco/partición)
- Tam máx real archivo = min(teórico archivo, real FS)
- Bloques de datos de un archivo = ⌈tamaño / tam_bloque⌉
- Bloques de punteros:
  1. Restar los PD
  2. Si quedan: +1 bloque (IS) y restar p
  3. Si quedan: +1 (bloque del ID) + ⌈restantes / p⌉ bloques de 2do nivel (hasta p)
  4. Si quedan: +1 (IT) + bloques de 2do nivel + bloques de 3er nivel
- Total bloques = datos + punteros
- Bloque lógico de un byte = byte div tam_bloque (empieza en 0)
- Leer un rango de bytes: bloques de datos del rango + bloques de punteros que hay que atravesar (cada uno una vez si quedan cacheados)
- Acceso directo a un byte (peor caso, IT): 4 accesos a bloques (3 de punteros + dato)
- Para deducir el tamaño de un archivo por la cantidad de bloques de punteros: ver qué niveles están llenos (ej: 1026 bloques de punteros con p = 1024 → IS lleno (1) + ID lleno (1 + 1024))

## Copiar entre FS
- ¿Entra? comparar tamaño del archivo con el tam máx de archivo del FS destino (y espacio libre)
- Hardlinks copiados a FAT → se vuelven archivos separados (cada uno ocupa sus clusters). En UFS se leen una sola vez los bloques si quedan cacheados
- Para referenciar entre FS distintos → **softlink** (el hardlink necesita el nro de inodo del mismo volumen)

## Graficar links (inodos + entradas de directorio)
- Directorio: lista (nombre → nro inodo)
- Inodo del archivo: tipo regular, permisos, **contador de hardlinks = cant. de entradas que lo apuntan**, tamaño, punteros → bloque con el contenido
- Hardlink: otra entrada de directorio con el **mismo** nro de inodo → contador +1
- Softlink: entrada nueva → **inodo nuevo** tipo symlink (contador 1, permisos rwxrwxrwx), su bloque contiene la ruta ("/home/user/file1.doc")
- Los directorios también son archivos: tienen su inodo (tipo directorio) y bloque con las entradas

## Ejercicios de clase (06-13, solo resultados)
- 2) a: 2^12 × 8 KiB = 2^25 B = 32 MiB (FAT12 con clusters de 8 KiB) — b: 32 KiB
- 3) a: 8 GiB / 4 KiB = 2 Mi entradas — b: % FAT = 2 × T_FAT / T_disco = 2 × 2^23 / 2^33 = 1/2^9 = 0,1953 % — c: bits sin usar = 32 − log2(2 Mi) − 4 (reservados FAT32) = 32 − 21 − 4 = 7 bits
- 4) a: 4 GiB / 2^16 = 2^16 B = 64 KiB — b: 64 KiB, 64 KiB y 16 bloques
- 7) c: Σ (cant. punteros × tam bloque) con bloques de 1 KiB y 256 punteros por bloque = 12 KiB + 256 KiB + 64 MiB + 16 GiB 

> ⚠️ **Error en el apunte (06-13):** figuraba "54 MiB"; es 256² KiB = 64 MiB.
