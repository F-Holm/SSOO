# 09 - Memoria virtual (práctica)

## Pasos para un ejercicio de referencias
1. Sacar el formato de la DL (tam página → bits de offset) y pasar cada referencia a (página, offset). Marcar lectura/escritura
2. Chequear que la página sea válida (dentro del proceso). Si no → dirección inválida, se termina el proceso (y no se siguen procesando referencias)
3. Para cada referencia: ¿hit o PF?
4. Si PF: ¿hay frame libre (y la asignación lo permite)? → cargar. Si no → víctima según algoritmo y **alcance** (local: solo sus frames; global: cualquiera)
5. Si la víctima tiene M = 1 → escritura a disco
6. Actualizar bits (U = 1 al cargar/referenciar, M = 1 al escribir), instantes, puntero
7. Contar PF y escrituras

## Conteo de accesos a disco (convención de clase)
- PF sin reemplazo o con víctima no modificada: 1 acceso (leer la página)
- PF con víctima modificada: 2 accesos (escribirla + leer la nueva)

## Tiempo de atención
- Referencia sin PF (sin TLB): 2 accesos a memoria (tabla + dato)
- Con PF: acceso a la tabla + tiempo de PF (con o sin remoción/escritura) + reejecución (2 accesos)
- Con TLB: hit → 1 acceso (+ tiempo TLB)

## Algoritmos
- FIFO: víctima = menor instante de carga (el puntero FIFO no se mueve en los hits)
- LRU: víctima = menor instante de última referencia (en los hits se actualiza)
- Óptimo: víctima = la que se usa más lejos en el futuro (o nunca más)
- Clock: puntero al próximo frame. U = 1 → U = 0 y avanza; U = 0 → víctima; el puntero queda en el siguiente al reemplazado. En un hit NO se mueve el puntero, solo U = 1. Si hay frame libre apuntado, se usa y avanza
- Clock modificado: (0,0) sin tocar U → (0,1) poniendo U = 0 → repetir

## Ejemplo de clase: cadena 2 3' 2 1 5' 2 4' 5 3 2' 5 2 (' = escritura), 3 frames

### LRU → 7 PF, 9 accesos a disco

| Ref | 2 | 3' | 2 | 1 | 5' | 2 | 4' | 5 | 3 | 2' | 5 | 2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Fr1 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 3 | 3 | 3 | 3 |
| Fr2 | - | 3 M | 3 M | 3 M | 5 M | 5 M | 5 M | 5 M | 5 M | 5 M | 5 M | 5 M |
| Fr3 | - | - | - | 1 | 1 | 1 | 4 M | 4 M | 4 M | 2 M | 2 M | 2 M |
| PF | PF | PF | - | PF | PF | - | PF | - | PF | PF | - | - |
| Acc disco | 1 | 1 | 0 | 1 | 2 | 0 | 1 | 0 | 1 | 2 | 0 | 0 |

### CLOCK → 8 PF, 11 accesos

| Ref | 2 | 3' | 2 | 1 | 5' | 2 | 4' | 5 | 3 | 2' | 5 | 2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Fr1 | → 2 U | → 2 U | → 2 U | → 2 U | 5 U M | 5 U M | → 5 U M | → 5 U M | 3 U | 3 U | → 3 U | → 3 U |
| Fr2 | - | 3 U M | 3 U M | 3 U M | → 3 M | 2 U | 2 U | 2 U | → 2 | → 2 U M | 2 M | 2 U M |
| Fr3 | - | - | - | 1 U | 1 | → 1 | 4 U M | 4 U M | 4 M | 4 M | 5 U | 5 U |
| Puntero | Fr1 | Fr1 | Fr1 | Fr1 | Fr2 | Fr3 | Fr1 | Fr1 | Fr2 | Fr2 | Fr1 | Fr1 |
| PF | PF | PF | - | PF | PF | PF | PF | - | PF | - | PF | - |
| Acc disco | 1 | 1 | 0 | 1 | 1 | 2 | 1 | 0 | 2 | 0 | 2 | 0 |

(→ y fila Puntero: frame al que apunta el puntero después de cada referencia; mientras se llenan los frames libres queda en Fr1. U = bit de uso en 1, M = bit de modificado en 1; si la letra no está, el bit está en 0)

### CLOCK MODIFICADO → 9 PF, 12 accesos

| Ref | 2 | 3' | 2 | 1 | 5' | 2 | 4' | 5 | 3 | 2' | 5 | 2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Fr1 | → 2 U | → 2 U | → 2 U | → 2 U | 5 U M | → 5 U M | 5 M | 5 U M | → 5 M | 2 U M | 2 U M | 2 U M |
| Fr2 | - | 3 U M | 3 U M | 3 U M | → 3 M | 3 M | 4 U M | 4 U M | 4 M | → 4 M | 5 U | 5 U |
| Fr3 | - | - | - | 1 U | 1 | 2 U | → 2 U | → 2 U | 3 U | 3 U | → 3 U | → 3 U |
| Puntero | Fr1 | Fr1 | Fr1 | Fr1 | Fr2 | Fr1 | Fr3 | Fr3 | Fr1 | Fr2 | Fr3 | Fr3 |
| PF | PF | PF | - | PF | PF | PF | PF | - | PF | PF | PF | - |
| Acc disco | 1 | 1 | 0 | 1 | 1 | 1 | 2 | 0 | 1 | 2 | 2 | 0 |

## Thrashing en ejercicios
- Comparar la **localidad** (páginas distintas que se usan en el ciclo) con los frames asignados
- Recorrido cíclico de N páginas con < N frames y LRU/FIFO → PF en **todas** las referencias
- Solución: más frames (asignación ≥ localidad), bajar multiprogramación, compartir páginas de código entre instancias, asignación variable según necesidad
- TLB más grande ayuda si el proceso repite pocas páginas (su localidad entra en la TLB); no ayuda si recorre muchas páginas distintas

## Fork / copy-on-write
- El hijo apunta a los mismos marcos que el padre (marcados solo lectura). No necesita frames nuevos hasta que alguien escriba

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
