# 11 - FAT y EXT2 / UFS (teoría)

## FAT (File Allocation Table)
- 1977, diskettes; hoy memorias flash. Versiones FAT12, FAT16, FAT32 (VFAT, exFAT)
- Asignación **enlazada** "modificada": los punteros no están en los bloques sino en la tabla FAT
- Bloque = **cluster**
- Tabla FAT: una entrada por cluster (indexada por nro de cluster), cada entrada = puntero al siguiente cluster del archivo (o EOF / libre / dañado)
- La FAT se guarda al inicio del FS y está **2 veces** en disco (FAT + copia, para que no se corrompa). **Siempre entera en RAM**
- Leer un archivo: 1 acceso a disco por cluster + n accesos a memoria (seguir la cadena en la FAT). Tenerla en RAM evita accesos a disco innecesarios
- Sin la FAT en memoria habría un acceso a disco por cada eslabón
- Sigue siendo mala para acceso directo (recorrer la cadena, pero en memoria) y si se pierde una entrada de la FAT (y su copia) se pierde el resto del archivo

### Volumen FAT
```
Sector de arranque (boot) | FAT 1 | FAT 2 (copia) | Directorio raíz | Datos (archivos + subdirectorios)
```

### Entradas de directorio
- Tamaño fijo (32 bytes): nombre (8+3), extensión, atributos, fechas, **primer cluster**, **tamaño (32 bits)**
- **No tiene FCB**: los atributos están en la entrada de directorio
- No hay ID de archivo

### Versiones
- FAT12: entradas de 12 bits → 2^12 clusters
- FAT16: 16 bits → 2^16 clusters
- FAT32: entradas de 32 bits pero solo 28 útiles (4 bits reservados que nunca se usaron) → 2^28 clusters. "Es como FAT28"
- Tamaño FAT = cant. entradas × tam entrada
- Tam. máx teórico FS = 2^(bits útiles) × tam cluster
- Tam. máx real FS = min(teórico, tamaño de la partición/disco)
- Tam. máx archivo:
  - Cátedra: tan grande como el FS
  - Real: el campo tamaño es de 4 bytes → **4 GiB** (en FAT12/16/32)
- + tamaño de cluster → + fragmentación interna (un archivo de pocos bytes ocupa un cluster entero)

### Limitaciones de FAT
- Sin seguridad, sin permisos, sin propietarios
- Sin estructura de espacio libre (no hay bitmap): para un bloque libre se recorre la FAT (un bitmap en memoria mejoraría la performance)
- Sin journaling
- Sin FCB
- **Sin links** (ni soft ni hard): los atributos están en la entrada de directorio, dos entradas al mismo archivo quedarían inconsistentes
- Tolerancia a fallos: copias de la FAT
- Al copiar de UFS a FAT se pierden permisos, dueños, links, etc.

## UFS / EXT2
- Unix FS. Versiones EXT, EXT2, EXT3, EXT4. Base: esquema **indexado** (multinivel)
- FCB = **inodo**

### Volumen EXT2
```
Sector de arranque | Grupo de bloques 0 | Grupo de bloques 1 | ... | Grupo de bloques n-1
```
- Grupo de bloques:
  - **Superbloque** (igual en todos los grupos, copia): total inodos, total bloques, bloques libres, inodos libres, tamaño de bloque, tamaño de inodo
  - **Descriptores de grupo** (iguales en todos): ubicación de bitmaps y tabla de inodos, bloques e inodos libres del grupo
  - Bitmap de bloques de datos
  - Bitmap de inodos
  - Tabla de inodos (cantidad fija creada al formatear → cantidad de archivos limitada)
  - Bloques de datos
- Es importantísimo saber bien la estructura

### Inodo
- mode (tipo + permisos), owners (uid, gid), timestamps, size, cantidad de bloques, **contador de hardlinks**
- Punteros:
  - Directos: a bloques de datos (suelen ser 12)
  - Indirecto simple: a un bloque de punteros a datos
  - Indirecto doble: a un bloque de punteros a bloques de punteros
  - Indirecto triple: un nivel más
- Tamaño fijo (128 bytes en versiones viejas) → ~15 punteros
- Configuración estándar si no dicen nada: 12 PD, 1 IS, 1 ID, 1 IT
- Los bloques de punteros se crean a demanda usando bloques de datos
- Si se corrompe un bloque de punteros, se pierden los N bloques que referencia (no siempre se puede seguir leyendo el resto)
- Si se corrompe un bloque de datos solo se pierde ese bloque
- Se puede recorrer secuencialmente (siguiendo los punteros en orden) y acceder directo

### Directorios en UFS
- Entrada: nro de inodo, tamaño de la entrada, tamaño del nombre, nombre (y tipo)
- Mucha menos info que en FAT (el resto está en el inodo)

### Tipos de archivo
- Regular, directorio, links, dispositivos, etc.

### Links
- **Hard link**: nueva entrada de directorio que apunta al **mismo inodo** (no crea inodo, no crea archivo)
  - Incrementa el contador de hardlinks del inodo. Al borrar uno, se decrementa; el inodo y sus bloques se liberan cuando llega a 0
  - No se sabe cuál es el "original" (todas las entradas son equivalentes)
  - Mismos permisos (es el mismo inodo)
  - No puede apuntar a otro FS/partición (los nros de inodo son únicos por volumen)
  - No puede apuntar a directorios (por diseño)
  - Más eficiente (no lee un bloque extra). Casi nunca necesita bloques libres (salvo que el bloque del directorio esté lleno)
- **Soft / symbolic link** (acceso directo, como en Windows): **archivo nuevo** con su propio inodo (tipo symlink) que contiene la **ruta** del original (en un bloque de datos o en el propio inodo si es corta)
  - Puede apuntar a otro FS / partición / directorio
  - Ocupa más espacio (inodo + bloque)
  - Se sabe cuál es el original. Si se borra el original, queda roto
  - Tiene sus propios permisos (normalmente rwxrwxrwx, no importan: se validan los del archivo destino)
- Ambos requieren una nueva entrada de directorio

### Crear un archivo en UFS
- Buscar un inodo libre (bitmap de inodos) en algún grupo → completar el inodo → entrada de directorio → buscar bloques libres (bitmap de bloques) → escribir. Si es grande, usar bloques de punteros
- Más pasos que en FAT → más complejo y más overhead
- Un archivo vacío no necesita bloques de datos: con todos los bloques ocupados igual se puede crear (si hay inodo libre y lugar en el directorio)

## FAT vs UFS
| | FAT | UFS/EXT2 |
|---|---|---|
| Asignación | Enlazada (tabla FAT) | Indexada multinivel (inodos) |
| FCB | No (entrada de directorio) | Inodo |
| Espacio libre | Recorrer la FAT | Bitmaps |
| Permisos / dueños | No | Sí |
| Links | No | Hard y soft |
| Acceso directo | Recorrer cadena en memoria | Punteros (más rápido en promedio con FAT en RAM para archivos chicos) |
| Estructuras en memoria | La FAT entera (puede ser muy grande) | Inodos de archivos abiertos |
| Overhead | Menor | Mayor |

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
