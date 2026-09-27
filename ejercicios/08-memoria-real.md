# 08 - Memoria real (ejercicios)

## Teóricos

### [Cuestionario 05-30] Tabla de páginas invertida

<details>
<summary>Ver respuesta</summary>

- Ventaja sobre la convencional: al ser una única tabla en el sistema ocupa menos espacio
- Desventajas: búsqueda secuencial (se soluciona con hash), difícil compartir; no es compatible (fácilmente) con memoria virtual

</details>

### [Final 2022-12-06] En una referencia a memoria, un acierto en la TLB tiene como ventaja evitar un fallo de página — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. La ventaja es evitar el acceso a la tabla de páginas en memoria (1 acceso en vez de 2). Que no haya PF no se debe a la TLB

</details>

### [Final 2022-12-21] Sin memoria virtual no es necesaria la TLB porque no se producen page faults — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. La TLB reduce el tiempo de traducción con o sin memoria virtual (oficial)

</details>

### [Final 2023-02-14] El tamaño de una tabla de páginas invertida no varía nunca, independientemente de la cantidad de procesos o su tamaño — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Tiene una entrada por marco de la memoria física

</details>

### [Final 2024-03-05] La tabla de páginas invertida nunca cambia su tamaño durante la ejecución de un proceso — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Siempre tiene una entrada por marco (oficial)

</details>

### [Final 2023-03-07] Segmentación paginada podría causar fragmentación externa con muchos procesos chicos — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. La información se guarda en páginas de tamaño fijo: solo hay fragmentación interna (oficial)

</details>

### [Final 2023-05-23] Segmentación paginada tiene más fragmentación interna que segmentación y paginación, pero usa la memoria más eficientemente que segmentación — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Frag. interna en la última página de cada segmento (segmentación no tiene; paginación solo en la última página del proceso), pero elimina la externa

</details>

### [Final 2023-12-12] La segmentación paginada no sufre de fragmentación externa — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Los segmentos se reparten en marcos de tamaño fijo

</details>

### [Final 2023-12-19] Comparada con paginación, la segmentación es más eficaz para asignar permisos a las distintas porciones de un proceso — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **V**. Cada segmento es una parte lógica (código, datos, pila) → permisos naturales por segmento; las páginas cortan el proceso arbitrariamente

</details>

### [Final 2024-07-23] La fragmentación externa e interna son igual de perjudiciales para segmentación y segmentación paginada — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Segmentación: solo externa. Segmentación paginada: solo interna

</details>

### [Final 2025-07-29] Comparada con paginación simple, la paginación jerárquica optimiza el tiempo de traducción — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Agrega un acceso a memoria por nivel; optimiza el espacio de las tablas, no el tiempo

</details>

### [Final 2025-09-25 / 2025-12-02] Paginación multinivel podría implicar muchos accesos a memoria por traducción solo si no hay TLB — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **F**. Con TLB también: en cada TLB miss hay que recorrer todos los niveles (niveles + 1 accesos)

</details>

### [Final 2025-12-16] Es posible combinar segmentación con paginación multinivel para obtener los beneficios de ambas — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **V**. Es común en sistemas modernos (oficial)

</details>

### [Final 2026-05-19] En paginación simple, usar una tabla de páginas por proceso no permite compartir memoria — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- **F**. Entradas de distintas tablas pueden apuntar al mismo marco (oficial)

</details>

### [Final 2026-02-24] Evaluar la respuesta del LLM — ✅ solución oficial

<details>
<summary>Ver respuesta</summary>

- Afirmación: "La segmentación paginada combina beneficios de paginación y segmentación mitigando sus desventajas"
- LLM: "Correcta: mantiene la división lógica en segmentos pero evita la fragmentación externa al paginar cada segmento"
- **Correcta**: permite permisos y gestión por segmento y evita la fragmentación externa (oficial)

</details>

### [Final 2023-07-25 N°2] ChatGPT ordenó de menor a mayor overhead: Buddy System, Segmentación paginada, Particionamiento fijo. Evaluar; ubicar Particionamiento dinámico — ✍️ respuesta propia (el PDF del final no trae solución)

<details>
<summary>Ver respuesta</summary>

- **Incorrecta**: el particionamiento fijo es el de **menor** overhead (una tabla simple, base + límite); su problema es la fragmentación interna (desperdicio), no el overhead. La respuesta confunde desperdicio con overhead
- Orden: Particionamiento fijo < Buddy System < Particionamiento dinámico (búsqueda de huecos + compactación) < Segmentación paginada (tablas de segmentos y páginas, traducción en dos niveles). Sin compactación, dinámico y buddy quedan parecidos

</details>

## Prácticos

### [Final 2024-07-30] Sistema de 16 bits, sin memoria virtual, 32 KiB de RAM, segmentación paginada. 4 segmentos siempre (Code, Data, Stack, Heap), páginas de 256 B — ✍️ respuesta propia (el PDF del final no trae solución)
| Code S0 | Data S1 | Stack S2 | Heap S3 |
|---|---|---|---|
| p0 → 3 (RX) | p0 → 6 (RW) | p0 → 9 (RW) | p0 → 10 (RW) |
| p1 → 18 (RX) | p1 → - | p1 → 11 (RW) | p1 → 21 (RW) |
| p2 → 7 (RX) | p2 → - | p2 → 17 (RW) | p2 → 24 (RW) |

**Consigna:**
- a) ¿A qué números de marco hacen referencia las direcciones lógicas 41ACh, 3283h, 015Ah?
- b) ¿Cuál es el tamaño máximo teórico y real de un proceso?
- c) Asuma que el marco 6 solo contiene variables de tipo entero y que el proceso reduce su tamaño de tal manera que se liberan los marcos 9, 11 y 17. ¿Detecta algún problema en el uso de la memoria?
- d) ¿Cuál es la fragmentación interna máxima posible para cualquier proceso?

<details>
<summary>Ver respuesta</summary>

Formato DL (16 bits): segmento 2 bits (4 segmentos) | página 6 bits | offset 8 bits (256 B)

a) ¿A qué marcos hacen referencia?
- 41ACh = 01 000001 10101100 → seg 1 (Data), pág 1, offset ACh → la página 1 de Data no está asignada → **dirección inválida** (sin VM no hay PF: error/segmentation fault)
- 3283h = 00 110010 10000011 → seg 0 (Code), pág 50 → Code tiene 3 páginas → **dirección inválida**
- 015Ah = 00 000001 01011010 → seg 0, pág 1, offset 5Ah → **marco 18** → DF = 18 × 256 + 90 = 4698 = 125Ah

b) Tamaño máximo de un proceso: teórico = 2^16 = **64 KiB** (4 seg × 64 pág × 256 B); real = **32 KiB** (sin VM el proceso tiene que entrar en la RAM, menos lo que ocupe el SO)

c) Liberar los marcos 9, 11 y 17 es liberar **toda la pila**: el proceso siempre necesita stack (variables locales, parámetros y direcciones de retorno de main y las funciones) → cualquier acceso a la pila sería inválido. (Además, que Data tenga una sola página con enteros muestra la fragmentación interna de la última página)

d) Fragmentación interna máxima = 4 segmentos × (256 − 1) = **1020 bytes**

</details>
