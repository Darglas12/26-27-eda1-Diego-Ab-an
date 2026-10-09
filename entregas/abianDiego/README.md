# Reto 003 - Simulación: un array utilizando una lista

**Equipo 1:** Diego Abián, Sergio del Rio y Victor Arenas.

Caso elegido: [simular un array utilizando una lista](https://github.com/mmasias/26-27-eda1/blob/main/evaluaciones/retos/arrayConLista/README.md).

La idea es partir de una lista simplemente enlazada de `int` (con referencia a la cabeza) y **quitarle** todo lo que un array no permite: solo puede crearse con un tamaño, leer una posición y cambiar una posición.

<div align=center>

|Fichero|Contenido|
|-|-|
|[Nodo.java](src/arrayConLista/Nodo.java)|Nodo con un `int` y la referencia al siguiente.|
|[ArrayConLista.java](src/arrayConLista/ArrayConLista.java)|La simulación: `ArrayConLista(longitud)`, `longitud()`, `obtener(posicion)` y `asignar(posicion, valor)`.|
|[Ejemplo.java](src/arrayConLista/Ejemplo.java)|Hace las mismas operaciones con un `int[]` y con un `ArrayConLista` y muestra que el resultado es el mismo.|

</div>

## Ejecución

Desde `src/` (con `-ea` para que se comprueben los `assert`):

```bash
javac arrayConLista/*.java
java -ea arrayConLista.Ejemplo
```

## Parte 1: Identificación de diferencias

<div align=center>

|Característica|Array (comportamiento esperado)|Lista (comportamiento real)|¿Qué habría que imponer o restringir?|
|-|:-:|:-:|-|
|**Tamaño**|Fijo|Variable|El constructor recibe la longitud y crea ya todos los nodos (a `0`, como un `int[]`). No hay ningún método que añada o quite nodos.|
|**Tipo de dato**|Homogéneo|Posiblemente heterogéneo|El `dato` del nodo se declara de un tipo concreto (`int`), no `Object`. El compilador impide guardar otra cosa.|
|**Inserción y eliminación**|No permitidas|Permitidas|No se ofrecen. `asignar` cambia el **valor** de un nodo que ya existe, nunca los enlaces.|
|**Acceso**|Por índice directo|Por índice o recorrido|Solo se accede por posición (`obtener` / `asignar`), de `0` a `longitud - 1`, comprobado con `assert`.|
|**Posiciones en memoria**|Contiguas|No necesariamente contiguas|No se puede imponer, solo ocultar: el usuario solo ve posiciones, pero internamente hay que recorrer los nodos hasta llegar a la pedida.|

</div>

```
asignar(1, 9)

antes:    cabeza → [4] → [7] → [2] → null
después:  cabeza → [4] → [9] → [2] → null     mismos nodos y mismos enlaces; solo cambia el dato
```

## Parte 2: Cuestiones para el análisis

**1. Si una lista puede crecer y reducirse, ¿cómo podría fijarse un tamaño constante?**

Creando todos los nodos en el constructor y guardando la longitud. Como la clase no tiene ningún método que añada o quite nodos, ese número ya no puede cambiar. Además, `cabeza` y `longitud` son `final`.

**2. ¿Qué mecanismos conceptuales permitirían "bloquear" la inserción o eliminación de elementos?**

La encapsulación: `cabeza` es `private` y `Nodo` no es pública, así que desde fuera solo se puede usar lo que la clase ofrece (`longitud`, `obtener` y `asignar`). La inserción y la eliminación no se bloquean con un `if`: simplemente no existen.

**3. ¿Cómo se garantizaría la homogeneidad de tipos dentro de la lista?**

Declarando el `dato` con un tipo concreto. Con `int dato`, intentar guardar un `String` no compila: lo garantiza el compilador, no el programa en ejecución.

**4. ¿Qué implicaciones tendría acceder siempre por índice, incluso si la lista internamente no está organizada de forma contigua?**

Que cada `obtener(i)` o `asignar(i, valor)` tiene que empezar en la cabeza y avanzar `i` nodos. En un array el acceso es **O(1)**; aquí es **O(n)**. Recorrer todo el "array" con un `for` y `obtener(i)` pasa a ser **O(n²)**, porque cada vuelta vuelve a empezar desde la cabeza. Por fuera se usa igual que un array, pero es más lento.

**5. ¿Qué se pierde y qué se gana al imponer a una lista el comportamiento de un array?**

<div align=center>

|Se gana|Se pierde|
|-|-|
|Reglas claras: nadie puede cambiar el tamaño ni el tipo.|El acceso directo: de O(1) a O(n).|
|Poder trabajar "como con un array" en un lenguaje que no los tiene.|La flexibilidad de la lista: ya no puede crecer ni reducirse.|
|No hace falta un bloque de memoria contiguo.|Memoria: cada nodo guarda además la referencia `siguiente`.|

</div>

## Parte 3: Reflexión

**1. ¿Hasta qué punto una estructura flexible puede comportarse de manera disciplinada como una estructura rígida?**

Por fuera, del todo: ofrece las mismas operaciones y cumple las mismas reglas que un array. Por dentro sigue siendo una lista, y eso se nota en el coste: imita el comportamiento, pero no el rendimiento.

**2. ¿Qué dice este ejercicio sobre la relación entre naturaleza de una estructura y modo de uso?**

Que quien usa una estructura solo ve lo que se le deja hacer con ella. `ArrayConLista` no *es* un array, pero *actúa como* uno porque cumple sus reglas. La naturaleza interna no decide qué se puede hacer, sino cuánto cuesta hacerlo.

**3. ¿Cuándo podría ser útil una simulación de este tipo en un contexto real de programación o diseño de sistemas?**

- En lenguajes o entornos que solo ofrecen listas.
- Cuando la memoria está fragmentada y no hay un bloque contiguo lo bastante grande.
- Cuando se quiere entregar a otra parte del programa una colección que no pueda cambiar de tamaño. Java lo hace con `Arrays.asList(...)`: devuelve una lista de tamaño fijo, que permite `set` pero lanza una excepción con `add` o `remove`.
