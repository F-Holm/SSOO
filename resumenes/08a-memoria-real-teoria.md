# 08 - Memoria real / principal (teoría)

## Básico
- La RAM es un gran vector de bytes. Para ejecutar, el proceso tiene que estar en RAM
- Memoria dividida en espacio del kernel (suele ser los primeros x bytes) y espacio de usuario
- Requerimientos que el SO debe satisfacer:
  - Reubicación (el proceso puede cargarse en otro lugar, ej: al volver de swap)
  - Protección (solo acceder a lo suyo; si no → interrupción y el SO suele matarlo)
  - Compartir memoria (con el SO como intermediario)
  - Organización lógica
  - Organización física

## Address binding (cuándo se traduce la variable a dirección)
- Compilación: DL == DF → sistemas monoprogramados, no permite reubicar ni 2 instancias
- Carga: DL == DF (dirección relativa, base fija durante toda la vida) → no permite reubicar al volver de swap
- Ejecución: DL != DF → la traduce el HW (MMU) en cada acceso. Lo normal hoy. Permite reubicar
- Etapas: programa.c → compilación → programa.o → enlazado → ejecutable → carga

## MMU
- HW que traduce dirección lógica (la que conoce el proceso) → dirección física (real)
- Dirección relativa = tipo de lógica expresada respecto de un punto (inicio + 34)
- La traducción solo tiene sentido con binding en ejecución

## Enlace de bibliotecas
- Estático: se copia la biblioteca en el ejecutable → stand-alone, fácil de distribuir, ejecutable más grande, 10 procesos = 10 copias
- Dinámico (bibliotecas compartidas): se carga en ejecución cuando se referencia → ejecutable chico, una sola copia en RAM para todos; la PC destino necesita tener la biblioteca

## Asignación contigua
- Registro base + límite: DF = base + DL, si DL < límite (si no, interrupción)
- Compartir memoria: se solapan las bases (solo lo necesario). Es complicado

### Particiones fijas
- N particiones de tamaño fijo (iguales o distintas), el proceso entra entero en una
- Un proceso no puede ser más grande que la partición
- **Fragmentación interna** (lo que sobra dentro de la partición)
- Limita el grado de multiprogramación (= cantidad de particiones)
- Simple, poco overhead (el de menor overhead)
- **Sin fragmentación externa**: cualquier partición libre se puede asignar

> ⚠️ **Error en el Resumen SO:** decía que las particiones fijas también pueden tener fragmentación externa (ej: libres 12 + 4 y un proceso de 16 que no entra). El cuestionario de Memoria Real de la cátedra dice que nunca hay fragmentación externa: ese caso es un proceso más grande que las particiones, no huecos inutilizables.


### Particiones dinámicas
- La partición se crea del tamaño exacto del proceso
- Un proceso podría ser tan grande como toda la memoria física. Se necesita base y límite de cada partición. Administración más compleja que fijas
- **Fragmentación externa** (huecos chicos no contiguos) → se soluciona con **compactación** (overhead: mover procesos, no pueden ejecutar)
- Sin fragmentación interna, grado de multiprogramación no limitado
- No se lleva bien con procesos que crecen
- Tabla de huecos
- Algoritmos de ubicación:
  - Primer ajuste (first fit): el primer hueco desde el inicio
  - Siguiente ajuste (next fit): el primero desde la última asignación (no se reinicia la posición)
  - Mejor ajuste (best fit): el hueco más chico donde entra → deja huecos muy chicos (peor frag. externa)
  - Peor ajuste (worst fit): el hueco más grande → menos frag. externa. Si los huecos están ordenados por tamaño es el más rápido

### Buddy system
- Bloques de tamaño potencia de 2; se divide a la mitad hasta el más chico donde entra; al liberar se fusionan los "compañeros"
- Frag. interna y externa. Overhead por dividir/combinar

## Segmentación
- El proceso se divide en segmentos de tamaño variable según la visión del programador: código, datos, pila, heap, bibliotecas
- No necesita estar contiguo. Todos sus segmentos en RAM (sin memoria virtual)
- DL = (nro segmento, desplazamiento). DF = base del segmento + desplazamiento, validando desplazamiento < límite (tamaño) → si no, segmentation fault
- Tabla de segmentos por proceso (base, límite, permisos) cargada en la MMU
- **Sin frag. interna, con frag. externa** (menos que particiones dinámicas)
- Permisos por segmento (R, W, X) → más eficaz que paginación para asignar permisos
  - Código: X (RX) | Datos: RW | Pila: RW | Heap: RW | Bibliotecas: X
- Muy fácil compartir (ej: 2 instancias del mismo programa comparten el segmento de código read-only)

## Paginación simple
- Proceso dividido en páginas, memoria en marcos (frames). Tamaño página = tamaño marco
- **Sin frag. externa**, con frag. interna solo en la última página (máx = tam página − 1)
- Tabla de páginas por proceso: nro de marco + bit de validez (si la página pertenece al espacio del proceso) (+ bits de permisos RWX)
- La tabla tiene cantidad fija de entradas; las que no son del proceso con validez = 0
- Marcos libres: bitmap `[0,1,1,0,...]` (tamaño = cantidad de marcos / 8 bytes)
- Tabla de páginas siempre en memoria. PTBR = puntero a la tabla del proceso en ejecución
- Traducción: 2 accesos a memoria (tabla + dato)
- Compartir: dos tablas apuntan al mismo marco. Se pueden poner permisos
- Tablas en paginación simple: muchos bytes de overhead

## TLB (Translation Look-aside Buffer)
- Caché de HW asociativa (por contenido) de alta velocidad, administrada por la MMU
- Guarda (página → marco) para evitar el acceso a la tabla de páginas → con TLB hit: 1 acceso a memoria en vez de 2
- Útil con o sin memoria virtual
- La ventaja es ahorrar accesos a memoria, **no** evitar page faults
- Pocas entradas. Se vacía al cambiar de proceso, salvo que tenga **ASID** (identificador de proceso)
- Miss: buscar en la tabla; si está presente se agrega a la TLB; si no → PF

## Paginación jerárquica (multinivel)
- Paginar la tabla de páginas: tabla de 1er nivel → n tablas de 2do nivel → ...
- Solo se cargan las partes de la tabla que se usan → ocupa menos RAM en general
- Accesos a memoria por traducción = niveles + 1 (sin TLB o con TLB miss) → **no** optimiza el tiempo, optimiza espacio
- La tabla de 1er nivel siempre en RAM. Con VM puede haber PF por la tabla y otro por la página
- Procesadores modernos: 3-4 niveles

## Tabla de páginas invertida
- Una sola tabla para todo el sistema, **una entrada por marco** (indexada por marco) → tamaño fijo, nunca cambia
- Cada entrada: PID + nro de página
- Ocupa muy poco espacio, casi no se usa
- Búsqueda secuencial (lenta) → variante con **hash** (colisiones encadenadas con puntero)
- Difícil compartir memoria (hacen falta estructuras extra) y difícil usar memoria virtual (no tiene bit de presencia: PF cuando no se encuentra)

## Segmentación paginada
- Cada segmento se divide en páginas: tabla de segmentos → tabla de páginas por segmento
- DL = segmento | página | desplazamiento
- Ventajas de ambas: permisos por segmento + **sin frag. externa**
- Más frag. interna que segmentación y paginación (última página de cada segmento); uso más eficiente que segmentación
- Más overhead. Puede ser multinivel (se combina con paginación multinivel en sistemas modernos)

## Cuadro comparativo

| Técnica | Descripción | Ventajas | Desventajas |
|---|---|---|---|
| Particiones fijas | Particiones de tamaño fijo, el proceso ≤ partición | Simple, poco overhead | Frag. interna, limita multiprogramación |
| Particiones dinámicas | Partición del tamaño exacto | Mejor uso de memoria | Frag. externa (compactación) |
| Segmentación | Segmentos de distinto tamaño | Sin frag. interna, no contiguos | Frag. externa |
| Paginación | Frames fijos, páginas del mismo tamaño | Sin frag. externa, no contiguas | Frag. interna en la última página, más estructuras |

- Overhead de menor a mayor: particiones fijas < particiones dinámicas / buddy < segmentación paginada

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
