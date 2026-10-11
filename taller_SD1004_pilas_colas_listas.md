# Taller — Pilas, colas y listas enlazadas
**SD1004 · Estructura de Datos — Institución Universitaria Pascual Bravo · Semestre 2026-II**

---

## Parte 1 — Reconocer la estructura adecuada

| # | Situación | Estructura | Por qué es adecuada |
|---|-----------|-----------|---------------------|
| 1 | Deshacer la última acción realizada | **Pila** | Es LIFO: la última acción guardada es la primera que se retira (`pop`), justo lo que necesita "deshacer". |
| 2 | Fila de atención, primero el que llegó primero | **Cola** | Es FIFO: se agrega al final y se atiende desde el frente, respetando el orden de llegada. |
| 3 | Recorrer canciones y volver a la primera al terminar la última | **Lista circular** | El último nodo apunta al primero, así el recorrido continúa sin tener que "reiniciar" manualmente. |
| 4 | Recorrer en un solo sentido y agregar al comienzo con frecuencia | **Lista simple** | Cada nodo solo necesita `siguiente`, y insertar al comienzo es O(1): basta actualizar la cabeza. |
| 5 | Pasar a la canción siguiente y a la anterior desde cualquiera | **Lista doble** | Cada nodo tiene `anterior` y `siguiente`, por lo que se puede avanzar y retroceder sin recorrer desde el inicio. |

---

## Parte 2 — Pilas (LIFO)

### 2.1 Sigue las operaciones

Los dibujos son verticales: **la parte de arriba es el tope**.

```
push(4)      push(9)      push(2)      pop()→2      push(7)
                          | 2 | <-tope              | 7 | <-tope
            | 9 | <-tope | 9 |        | 9 | <-tope  | 9 |
| 4 | <-tope| 4 |        | 4 |        | 4 |         | 4 |
+---+       +---+        +---+        +---+         +---+
```

```
peek()→7     pop()→7      pop()→9
| 7 | <-tope
| 9 |        | 9 | <-tope
| 4 |        | 4 |        | 4 | <-tope
+---+        +---+        +---+
(no cambia)
```

| Operación | Resultado | Pila (de base a tope) |
|-----------|-----------|-----------------------|
| push(4) | — | 4 |
| push(9) | — | 4, 9 |
| push(2) | — | 4, 9, 2 |
| pop() | **2** | 4, 9 |
| push(7) | — | 4, 9, 7 |
| peek() | **7** | 4, 9, 7 (no cambia) |
| pop() | **7** | 4, 9 |
| pop() | **9** | 4 |

**Al terminar:** queda únicamente el elemento **4**, y ese mismo elemento es el **tope**.

```
| 4 | <-- tope
+---+
```

### 2.2 Casos límite

1. **`pop()` sobre pila vacía:** no hay nada que retirar (situación llamada *desbordamiento negativo* o *underflow*). Una forma de manejarlo es verificar `esVacia()` antes de retirar y, si está vacía, lanzar un error/excepción o devolver un valor especial (`NULO`) junto con un mensaje, sin modificar la pila.
2. **`peek()` sobre pila vacía:** tampoco existe un tope, así que debería devolver `NULO` (o lanzar el mismo error) y no cambiar la pila. Se maneja igual: comprobar primero si está vacía.
3. **Pseudocódigo de `deshacer`:**

```
función deshacer(pila):
    si pila.esVacia() entonces
        imprimir "No hay acciones para deshacer"
        retornar NULO
    fin si
    accion ← pila.pop()
    retornar accion
fin función
```

### 2.3 Aplicación: deshacer acciones

El usuario hace: escribir, borrar, pegar (en ese orden), así que se apilan en ese orden.

```
Antes del 1.er deshacer      Después del 1.er deshacer     Después del 2.º deshacer
| pegar   | <-- tope         | borrar   | <-- tope         | escribir | <-- tope
| borrar  |                  | escribir |                  +----------+
| escribir|                  +----------+
+---------+
```

- **1.er deshacer:** se retira **pegar** (devuelve `pegar`).
- **2.º deshacer:** se retira **borrar** (devuelve `borrar`).
- Queda solo `escribir`.

**¿Qué se deshace primero?** La acción **pegar**, porque fue la última en realizarse. Una pila representa este comportamiento porque solo permite acceder al tope, es decir, al elemento más reciente: la última acción en entrar es la primera en salir (LIFO).

---

## Parte 3 — Colas (FIFO)

### 3.1 Simula la fila de atención

| Operación | Devuelve | Cola (frente → final) |
|-----------|----------|-----------------------|
| encolar("Ana") | — | [Ana] |
| encolar("Luis") | — | [Ana, Luis] |
| encolar("Sol") | — | [Ana, Luis, Sol] |
| desencolar() | **Ana** | [Luis, Sol] |
| encolar("Mia") | — | [Luis, Sol, Mia] |
| desencolar() | **Luis** | [Sol, Mia] |
| encolar("Teo") | — | [Sol, Mia, Teo] |

```
Después de cada paso (F = frente, Fin = final):

encolar Ana  :  F,Fin→[Ana]
encolar Luis :  F→[Ana][Luis]←Fin
encolar Sol  :  F→[Ana][Luis][Sol]←Fin
desencolar   :  (sale Ana)   F→[Luis][Sol]←Fin
encolar Mia  :  F→[Luis][Sol][Mia]←Fin
desencolar   :  (sale Luis)  F→[Sol][Mia]←Fin
encolar Teo  :  F→[Sol][Mia][Teo]←Fin
```

**Quedan esperando, en orden:** **Sol, Mia y Teo** (Sol es la siguiente en ser atendida).

### 3.2 Operaciones y casos límite

1. **Pseudocódigo de `atender`:**

```
función atender(cola):
    si cola.esVacia() entonces
        imprimir "No hay personas en la fila"
        retornar NULO
    fin si
    persona ← cola.desencolar()    // retira el nodo del frente
    retornar persona
fin función
```

2. **Desencolar una cola vacía:** no hay frente que retirar. Se maneja verificando `esVacia()` antes: si está vacía se devuelve `NULO` (o se lanza un error) y la cola queda igual. En una implementación enlazada, además, hay que evitar acceder a `frente.siguiente` cuando `frente` es nulo.

3. **Agregar al frente y retirar desde el final:** hay que ser precisos aquí. En cuanto al *orden*, si se usa de forma consistente (siempre agregar por un extremo y siempre retirar por el extremo opuesto) el comportamiento sigue siendo FIFO: el primero en entrar queda en el extremo opuesto y es el primero en salir. Lo que cambia es que se está invirtiendo el significado de "frente" y "final", contradiciendo la definición de la actividad (`encolar` agrega al final y `desencolar` retira del frente). Además hay un problema real de eficiencia: en una lista enlazada simple, retirar el último nodo exige recorrer la lista hasta el penúltimo para actualizar su referencia (O(n)), mientras que retirar del frente es O(1). El orden FIFO se conserva; la estructura se vuelve ineficiente y confusa con esas operaciones.

---

## Parte 4 — Listas enlazadas simples

### 4.1 Construye y modifica una lista

```
Inicio:                 cabeza → nulo

1. Insertar 18 al final:
   cabeza → [18|•] → nulo

2. Insertar 7 al comienzo:
   cabeza → [7|•] → [18|•] → nulo

3. Insertar 24 al final:
   cabeza → [7|•] → [18|•] → [24|•] → nulo

4. Insertar 13 después del 7:
   cabeza → [7|•] → [13|•] → [18|•] → [24|•] → nulo

5. Eliminar el 18:
   cabeza → [7|•] → [13|•] → [24|•] → nulo
                                 ^ el nodo final apunta a nulo
```

En el paso 5, el nodo 13 pasa a apuntar a 24 (`13.siguiente ← 24`), saltándose el 18. El nodo 24 sigue siendo el último y su `siguiente` es **nulo**.

### 4.2 Diseña operaciones

```
// Recorrer e imprimir
procedimiento imprimir(cabeza):
    actual ← cabeza
    mientras actual ≠ NULO hacer
        imprimir actual.dato
        actual ← actual.siguiente
    fin mientras
fin procedimiento

// Buscar un valor
función buscar(cabeza, valor):
    actual ← cabeza
    mientras actual ≠ NULO hacer
        si actual.dato = valor entonces
            retornar VERDADERO
        fin si
        actual ← actual.siguiente
    fin mientras
    retornar FALSO
fin función

// Insertar al comienzo
función insertarAlComienzo(cabeza, valor):
    nuevo ← crearNodo(valor)
    nuevo.siguiente ← cabeza
    cabeza ← nuevo
    retornar cabeza
fin función
```

### 4.3 Comprueba los bordes

**Insertar al comienzo:**
- **Lista vacía:** `cabeza` es nulo, así que `nuevo.siguiente ← NULO` y luego `cabeza ← nuevo`. El nuevo nodo queda como único nodo y su siguiente es nulo.
- **Lista con uno o más nodos:** `nuevo.siguiente` apunta a la antigua cabeza y luego `cabeza` apunta a `nuevo`. El mismo código funciona en ambos casos. El orden importa: primero se enlaza el nuevo nodo y después se mueve la cabeza; si se hace al revés, se pierde el resto de la lista.

**Eliminar el primer nodo:** solo hay que actualizar la referencia `cabeza` para que apunte al segundo nodo (`cabeza ← cabeza.siguiente`).

```
Antes:   cabeza → [7|•] → [13|•] → [24|•] → nulo

Después: cabeza ──────────→ [13|•] → [24|•] → nulo
         ([7] queda desconectado y puede liberarse)
```

Si la lista tenía un solo nodo, `cabeza.siguiente` es nulo, así que `cabeza` queda en nulo y la lista queda vacía.

---

## Parte 5 — Listas circulares

### 5.1 Reproduce un recorrido circular

**a) Lista inicial:**

```
        ┌──────────────────────────────┐
        ↓                              │
  [P1|•] → [P2|•] → [P3|•] ───────────┘
  (inicio)           (último apunta a P1)
```

**b) Los primeros siete turnos** (P2 termina en el segundo turno que recibe, que es el turno 5):

| Turno | Proceso que recibe el turno | Siguiente |
|-------|-----------------------------|-----------|
| 1 | P1 | P2 |
| 2 | P2 (1.er turno) | P3 |
| 3 | P3 | P1 |
| 4 | P1 | P2 |
| 5 | P2 (2.º turno, **termina**) | P3 |
| 6 | P3 | P1 |
| 7 | P1 | P3 |

(Desde el turno 6 la lista ya no contiene a P2, así que después de P1 sigue P3.)

**c) Lista después de retirar P2:**

```
        ┌─────────────────┐
        ↓                 │
  [P1|•] → [P3|•] ────────┘
```

Para retirarlo, `P1.siguiente ← P3`. El último nodo (P3) sigue apuntando al primero (P1): `P3.siguiente = P1`. ✔

**d) Condición para detener el recorrido:** como nunca se llega a nulo, se usa un punto de partida: guardar el nodo inicial y detenerse cuando, al avanzar, se vuelve a él (`actual ≠ inicio`). También sirve contar los nodos y recorrer exactamente `n` pasos.

```
actual ← inicio
hacer
    procesar(actual)
    actual ← actual.siguiente
mientras actual ≠ inicio
```

### 5.2 Casos especiales (eliminar un nodo)

Supongamos que se conoce `cabeza` (el primero) y `ultimo` (cuyo `siguiente` es `cabeza`).

1. **Lista vacía:** no hay nada que eliminar ni enlaces que cambiar; se informa que está vacía (y no se debe acceder a `siguiente`).
2. **Un solo nodo:** el nodo se apunta a sí mismo. Al eliminarlo, la lista queda vacía: `cabeza ← NULO` (y `ultimo ← NULO`).

```
Antes:  ┌──┐              Después:  cabeza → nulo
        ↓  │                        (lista vacía)
       [A|•]┘
```

3. **Eliminar el primero (hay 2 o más nodos):** `ultimo.siguiente ← cabeza.siguiente` y `cabeza ← cabeza.siguiente`. Así el último sigue cerrando el círculo.
4. **Eliminar el último (hay 2 o más nodos):** el penúltimo pasa a apuntar a la cabeza (`penultimo.siguiente ← cabeza`) y `ultimo ← penultimo`.

```
Dos o más nodos, eliminar el primero (A):

Antes:   ┌───────────────────┐         Después:  ┌─────────────┐
         ↓                   │                   ↓             │
        [A|•] → [B|•] → [C|•]┘                  [B|•] → [C|•]──┘
        (cabeza)           (último)             (cabeza)  (último)
```

---

## Parte 6 — Listas doblemente enlazadas

### 6.1 Recorrido en ambos sentidos

```
 cabeza                                                          cola
   ↓                                                              ↓
┌────────┬────────┬────────┐   ┌────────┬─────────┬────────┐   ┌────────┬───────────┬────────┐   ┌────────┬────────┬────────┐
│ ant:   │ Lunes  │ sig: •─┼──→│ ant: • │ Martes  │ sig: •─┼──→│ ant: • │ Miércoles │ sig: •─┼──→│ ant: • │ Jueves │ sig:   │
│ nulo   │        │        │←──┼─•      │         │        │←──┼─•      │           │        │←──┼─•      │        │ nulo   │
└────────┴────────┴────────┘   └────────┴─────────┴────────┘   └────────┴───────────┴────────┘   └────────┴────────┴────────┘
```

Forma resumida:

```
nulo ⇄ [Lunes] ⇄ [Martes] ⇄ [Miércoles] ⇄ [Jueves] ⇄ nulo
```

| Nodo | anterior | dato | siguiente |
|------|----------|------|-----------|
| 1 | nulo | Lunes | Martes |
| 2 | Lunes | Martes | Miércoles |
| 3 | Martes | Miércoles | Jueves |
| 4 | Miércoles | Jueves | nulo |

- **Comienzo → final:** Lunes, Martes, Miércoles, Jueves.
- **Final → comienzo:** Jueves, Miércoles, Martes, Lunes.

### 6.2 Inserta y elimina

```
Inicio:
nulo ⇄ [Lunes] ⇄ [Martes] ⇄ [Miércoles] ⇄ [Jueves] ⇄ nulo

1. Insertar Domingo antes de Lunes:
nulo ⇄ [Domingo] ⇄ [Lunes] ⇄ [Martes] ⇄ [Miércoles] ⇄ [Jueves] ⇄ nulo
   (Domingo.siguiente = Lunes; Lunes.anterior = Domingo; cabeza = Domingo)

2. Insertar Viernes después de Jueves:
nulo ⇄ [Domingo] ⇄ [Lunes] ⇄ [Martes] ⇄ [Miércoles] ⇄ [Jueves] ⇄ [Viernes] ⇄ nulo
   (Jueves.siguiente = Viernes; Viernes.anterior = Jueves; cola = Viernes)

3. Eliminar Miércoles:
nulo ⇄ [Domingo] ⇄ [Lunes] ⇄ [Martes] ⇄ [Jueves] ⇄ [Viernes] ⇄ nulo
```

**Referencias que cambian al eliminar Miércoles:**
- `Martes.siguiente` pasa de Miércoles a **Jueves**.
- `Jueves.anterior` pasa de Miércoles a **Martes**.

Miércoles queda desconectado de la lista.

### 6.3 Compara los enlaces

**Ventaja:** en una lista doble se puede recorrer hacia atrás (de Jueves a Lunes, como en el ejercicio 6.1) sin volver a empezar desde la cabeza, y se puede desconectar un nodo conocido en O(1) porque el nodo ya conoce a su anterior. **Costo:** cada nodo guarda una referencia extra (`anterior`), lo que usa más memoria, y cada inserción o eliminación debe actualizar más enlaces (como las dos referencias de los vecinos de Miércoles), con mayor riesgo de errores.

---

## Parte 7 — Análisis y elección

| Estructura y representación | Operación | Complejidad | Justificación breve |
|-----------------------------|-----------|-------------|---------------------|
| Pila enlazada con referencia al tope | Insertar en el tope | **O(1)** | Se crea un nodo, apunta al tope actual y se actualiza el tope; sin recorrer. |
| Pila enlazada con referencia al tope | Retirar del tope | **O(1)** | Basta mover el tope al `siguiente` del nodo tope. |
| Cola enlazada con referencias al frente y al final | Encolar al final | **O(1)** | La referencia al final permite enlazar directamente después del último nodo. |
| Cola enlazada con referencias al frente y al final | Desencolar del frente | **O(1)** | Se mueve la referencia del frente al siguiente nodo. |
| Lista simple sin referencia al último nodo | Insertar al comienzo | **O(1)** | Solo se actualiza la cabeza, sin importar el tamaño. |
| Lista simple sin referencia al último nodo | Insertar al final | **O(n)** | Sin referencia al último nodo hay que recorrer toda la lista para encontrarlo. |
| Lista doble con referencia a un nodo conocido | Desconectar ese nodo | **O(1)** | El nodo conoce a su anterior y siguiente; solo se reconectan esos vecinos entre sí (se asume que no hay que buscarlo). |

**Preguntas adicionales:**

1. **Cola que solo guarda el frente:** para encolar al final tendría que recorrer todos los nodos desde el frente hasta llegar al último (O(n)). Si también guarda la referencia al último nodo, accede a él directamente y encola en O(1).
2. **Ventaja de la lista circular al volver al primer elemento:** como el último nodo ya apunta al primero, el recorrido continúa de forma natural y repetida sin necesidad de reiniciar ni buscar la cabeza; es ideal para turnos rotativos o listas de reproducción en bucle.
3. **Información adicional de la lista doble:** cada nodo guarda además una referencia al nodo **anterior**. Esto permite recorrer en ambos sentidos y eliminar o insertar alrededor de un nodo conocido sin tener que buscar su predecesor desde la cabeza.

---

## Parte 8 — Reflexión

La forma en que se conectan los nodos determina qué extremos son accesibles de forma rápida y, por tanto, en qué orden pueden insertarse y retirarse los datos. En una **pila** solo se trabaja con el tope, por eso el último dato en entrar es el primero en salir (LIFO). En una **cola** se mantienen referencias al frente y al final: se inserta por un extremo y se retira por el otro, lo que produce el orden de llegada (FIFO). La **lista simple** permite avanzar en un solo sentido, así que insertar al comienzo es muy barato, pero llegar al final o retroceder exige recorrer nodos. La **lista circular** conecta el último con el primero y favorece los recorridos repetidos, como los turnos. La **lista doble** añade el enlace anterior y permite moverse hacia ambos lados. En resumen, el diseño de los enlaces no es un detalle: define el orden natural de acceso y la eficiencia de cada operación.
