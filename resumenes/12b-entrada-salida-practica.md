# 12 - Entrada / Salida (práctica: disco)

## Sector lógico → CHS
- Sectores por cilindro = cabezas (caras × platos) × sectores por pista
- Cilindro = sector_lógico div sectores_por_cilindro
- resto = sector_lógico mod sectores_por_cilindro
- Cabeza = resto div sectores_por_pista
- Sector = resto mod sectores_por_pista
- Tamaño del disco = C × H × S × tam_sector

## Tiempos
- TA = TB + LR + TT
- Movimiento total = Σ |pista_i − pista_(i−1)| × tiempo entre pistas
- En SCAN/C-SCAN se cuenta hasta el tope (ej: pista máx = cant_pistas − 1). En C-SCAN/C-LOOK el salto de vuelta suele considerarse despreciable (aclararlo)
- Pedidos en la misma pista/cilindro: 0 movimiento

## Pasos
1. Pasar todos los pedidos a pista (cilindro)
2. Ubicar la posición inicial y la dirección (si leyó antes uno menor → va subiendo)
3. Aplicar el algoritmo, anotar el orden y sumar desplazamientos

## Ejemplo (100 pistas 0-99, 1 ms entre pistas, pedidos 10, 50, 20, 30, 85, 45, cabezal en 40 subiendo)
- SSTF: 45, 50, 30, 20, 10, 85 → 5+5+20+10+10+75 = **125 ms**
- SCAN: 45, 50, 85, (99), 30, 20, 10 → 5+5+35+14+69+10+10 = **148 ms**
- C-SCAN: 45, 50, 85, (99), salto a 0, 10, 20, 30 → 5+5+35+14+10+10+10 = **89 ms** (sin contar el salto)
- LOOK: 45, 50, 85, 30, 20, 10 → 5+5+35+55+10+10 = **120 ms**
- C-LOOK: 45, 50, 85, salto a 10, 20, 30 → 5+5+35+10+10 = **65 ms** (+75 si se cuenta el salto)
- FCFS (cabezal en 10, pedidos 50, 20, 30, 85, 45): 40+30+10+55+40 = **175 ms**

---
[⬆ Volver al índice de resúmenes](00-indice.md) · [Índice general](../README.md)
