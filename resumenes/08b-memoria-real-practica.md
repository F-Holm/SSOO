# 08 - Memoria (práctica: traducción de direcciones y tamaños)

## Potencias útiles
- 2^10 = 1 Ki, 2^20 = 1 Mi, 2^30 = 1 Gi, 2^40 = 1 Ti
- 1 hex = 4 bits
- KiB/MiB (binario) vs KB/MB (decimal). En los enunciados suelen usarse como iguales

## Paginación
- Offset = log2(tam página) bits
- Bits de página = bits de la DL − bits de offset
- Cant. máx de páginas por proceso = 2^(bits de página)
- Tamaño máx de proceso = 2^(bits DL) (real: limitado por la RAM si no hay VM)
- DL en hex/binario: separar bits → (página, offset)
- DL en decimal: página = DL div tam_página, offset = DL mod tam_página
- DF = marco × tam_página + offset (en hex: marco concatenado con offset)
- Frames = tam RAM / tam página
- Bitmap de frames libres = cant. frames bits (/8 para bytes)
- Frag. interna máx = tam página − 1 (por proceso, en la última página). Si dan la frag. máx → tam página = frag + 1
- Tabla de páginas: cant. entradas × tam entrada
- Dirección fuera del espacio del proceso (página > última) → dirección inválida → se finaliza el proceso (no es PF)

### Deducir el formato
- Si dicen "A000h no produce PF" y la tabla tiene cierta página presente → probar cuántos bits de página hacen que A000h caiga en una página presente
- Si la TLB o tabla muestra la página de una DL dada → de ahí sale el tamaño del offset

## Segmentación
- DL = (segmento, offset). Bits de segmento = log2(cant. máx de segmentos)
- Validar offset < tamaño (límite) → si no, segmentation fault
- DF = base + offset
- Validar permisos (escribir en un segmento R-- → error de protección)
- Segmento inexistente (seg ≥ cantidad de segmentos) → error

## Segmentación paginada
- DL = segmento | página | offset
- Bits: segmento = log2(cant. segmentos), offset = log2(tam página), página = el resto
- Tamaño máx de segmento = 2^(bits pag + bits offset)
- Tamaño máx de proceso = cant. segmentos × tam máx segmento (real: RAM si no hay VM)
- Frag. interna máx = cant. segmentos × (tam página − 1)
- Traducción: tabla de segmentos → tabla de páginas del segmento → marco → DF = marco × tam + offset
- Validar: segmento existe, página existe/asignada (si no, dirección inválida), permisos

## Paginación multinivel
- DL = nivel 1 | nivel 2 | ... | offset. "Igual cantidad de bits por nivel" → dividir (bits DL − offset) por la cantidad de niveles
- Accesos a memoria = niveles + 1 (sin TLB)

## TLB
- Hit: 1 acceso (dato). Miss: tabla(s) + dato
- EAT = %hit × (t_TLB + t_mem) + %miss × (t_TLB + (niveles+1) × t_mem)
- Sin ASID → vaciar la TLB en cada cambio de proceso
- Reemplazo en la TLB: suele ser FIFO o LRU (independiente del de páginas)
- Si la TLB no guarda bits de modificado/uso, una escritura con TLB hit igual tiene que actualizar la tabla de páginas

## Tabla invertida
- Entradas = cantidad de marcos (fija)
- Cada entrada: PID + página (+ puntero de colisión si hay hash)

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
