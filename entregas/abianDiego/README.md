# Reto 004 - Refactoring de la propuesta

Parto de mi propuesta de [Elecciones](https://github.com/mmasias/26-27-eda1/pull/110) (array para los votos, cola para los votantes y la lista descartada) y del enunciado de [pyElecciones](https://github.com/puntoReflex/pyElecciones).

## Supuestos

- Un alumno que se marcha deja de estar presente: no puede proponerse ni votar, pero si ya había votado su papeleta sigue en el montón.
- Al cerrar el plazo, los candidatos quedan fijados aunque alguno se marche después.
- La papeleta es secreta: lleva solo el nombre votado, no el de quien la entrega.
- Una papeleta sin nombre se guarda como texto vacío.

## Propuesta revisada

|Dato|Decisión|Justificación|Alternativa descartada|Limitación asumida|
|-|-|-|-|-|
|Alumnos presentes|Lista (`presentes`)|<sub>"Llegan alumnos tarde y otros se marchan": no se sabe cuántos habrá y se sale desde cualquier posición. La operación que más se repite es preguntar si alguien está, antes de proponerse o de votar.|<sub>Array de matriculados con una marca de presente: el enunciado no da la lista de matriculados ni su tamaño, y el array arrastraría a los ausentes.|<sub>Saber si alguien está obliga a recorrer la lista (coste lineal): la simulación lo muestra al rechazar un voto ("recorridos 7 presentes"). Con el tamaño de un aula es aceptable.|
|Candidaturas con el plazo abierto|Lista (`candidaturas`)|<sub>"Un candidato puede retirarse y pueden proponerse otros": mientras el plazo está abierto el número cambia y el que se retira puede estar en cualquier sitio.|<sub>Un array desde el principio: hasta cerrar el plazo no se sabe cuántos candidatos habrá.|<sub>Para comprobar que no se propone dos veces hay que recorrerla. Al cerrar el plazo hay que copiarla a un array (`cerrarCandidaturas`).|
|Opciones de la pizarra: candidatos fijados, en blanco y nulo|Array de `String` (`opciones`, en `Pizarra`)|<sub>"Al cerrar el plazo, los candidatos quedan fijados": desde ese momento el tamaño es fijo (candidatos + 2) y no hay altas ni bajas.|<sub>Seguir con la lista: su flexibilidad ya no hace falta y el recuento necesita llegar a cada opción por posición.|<sub>El secretario canta un nombre, no un número: para saber qué casilla es hay que recorrer el array (`posicionDe`, lineal). Si un candidato se marcha después del cierre, sigue en la pizarra.|
|Palitos de cada opción|Array de `int` paralelo (`palitos`): la posición `i` cuenta los votos de la opción `i`|<sub>"Añade un palito en su columna por cada voto cantado": sumar en una casilla ya conocida cuesta lo mismo con 2 que con 30 candidatos. "Si se equivoca al anotar, borra el palito": corregir es restar uno en esa casilla.|<sub>Guardar el contador dentro de cada nodo de la lista de candidatos: para cada voto habría que recorrerla.|<sub>Los dos arrays tienen que ir alineados. Borrar un palito solo sabe de qué columna es, no de qué papeleta salió; y nunca puede bajar de 0 (`assert` en `borrarPalito`).|
|Papeletas recogidas sin leer|Pila (`papeletas`)|<sub>"Deja las papeletas recogidas una sobre otra, formando un montón" y "el secretario las toma una a una": solo se usa la de encima, la última que se dejó.|<sub>Cola (solución A de los ejemplos): para leerlas en el orden de recogida habría que sacar la de abajo del montón, y el orden de lectura no cambia el recuento. Array o lista: dan acceso a cualquier posición, algo que nadie usa.|<sub>Se leen al revés de como se recogieron (la de Rosa, la última, es la primera en cantarse). Una papeleta leída sale de la pila: para volver a contarlas habría que haberlas guardado aparte. La simulación lo muestra al proclamar ("montón vacío").|
|Alumnos que ya han votado|Lista (`yaHanVotado`)|<sub>"Cada alumno vota una vez": antes de aceptar una papeleta hay que saber si esa persona ya entregó una. Crece con cada voto y no se sabe cuántos votarán.|<sub>Cola de votantes (mi propuesta): una cola solo deja ver al primero, así que para saber si alguien ya votó habría que vaciarla. Marcar "ha votado" en la papeleta: rompería el voto secreto.|<sub>Comprobarlo es recorrer la lista (lineal): la simulación lo muestra al rechazar el segundo voto de Pepito ("recorridos 5 votantes"). Sabe quién votó, pero no qué votó.|

## Cambios respecto a mi propuesta

- **Mantengo el array para contar los votos**, pero lo amplío. Mi motivo sigue valiendo: los candidatos ya no cambian y llegar a una casilla por su posición es inmediato. Mi propuesta decía que "los candidatos se conocen antes de votar y no cambian", y eso solo es cierto **después de cerrar el plazo**. Ahora la pizarra tiene dos arrays paralelos (nombres y palitos) con dos opciones más, en blanco y nulo, porque el enunciado las pide. El array de nombres lo tomo de los ejemplos: sin él, la pizarra no sabría qué columna corresponde al nombre que canta el secretario.
- **Cambio la cola de votantes por una lista de quién ha votado.** Yo decía "los alumnos votan por orden de llegada", pero el enunciado no habla de ninguna fila ni de que el orden importe. Lo que sí pide es que cada uno vote una vez, y para eso hay que preguntar "¿ya votó este?", cosa que una cola no permite sin vaciarla.
- **Ya no descarto la lista: la uso para presentes y candidaturas.** Yo decía que en una lista "se mete o se saca gente por el medio, y aquí eso no pasa", y en el enunciado sí pasa: llegan alumnos tarde, otros se marchan y los candidatos se retiran. Esta decisión es la de los ejemplos; me convenció que el tamaño de esos datos cambia durante el proceso.
- **Añado la pila para las papeletas**, que en mi propuesta ni aparecían. De las dos soluciones de los ejemplos me quedo con la B: el enunciado describe un montón y el secretario coge siempre la de encima.

## Simulación

`src/elecciones/`: cada estructura del código es una fila de la tabla.

|Fichero|Qué es|
|-|-|
|[Nodo.java](src/elecciones/Nodo.java)|Nodo de la lista y de la pila|
|[Lista.java](src/elecciones/Lista.java)|Presentes, candidaturas y quién ha votado (insertar y eliminar con nodo dummy)|
|[Pila.java](src/elecciones/Pila.java)|Montón de papeletas|
|[Pizarra.java](src/elecciones/Pizarra.java)|Arrays paralelos de opciones y palitos; anota, borra y proclama detectando el empate|
|[Eleccion.java](src/elecciones/Eleccion.java)|El proceso y sus fases (candidaturas → secretario → votación → escrutinio → proclamación): decide qué se acepta y qué se rechaza|
|[Ejemplo.java](src/elecciones/Ejemplo.java)|Ejecuta la simulación evento a evento|

Eventos que muestra:

|Evento|En la simulación|
|-|-|
|Un alumno llega tarde y otro se marcha|Llega Carlos; se marcha Luisito|
|Candidatura retirada y candidatura rechazada|Marta se retira; Luisito se propone sin estar presente|
|Papeleta en blanco y papeleta nula|Marta vota en blanco; Carlos vota a "Batman"|
|Voto de alguien que no está|Luisito intenta votar después de marcharse|
|Votar dos veces|Pepito intenta votar otra vez|
|Corrección de una anotación|Se canta "Pepito", el presidente anota a Anita y borra el palito|
|Proclamación con empate|Anita 2 - Pepito 2: hay que repetir la votación|

```
cd src
javac -encoding UTF-8 -d out elecciones/*.java
java -ea -cp out elecciones.Ejemplo
```

## Retos extendidos

### Abstención

Al cierre, `informarAbstencion` recorre `presentes` y cuenta los que no están en `yaHanVotado`: en la simulación, 1 de 7 (Lucas). Cuesta presentes × votantes comparaciones, porque por cada presente se recorre la otra lista.

No puede contarse leyendo papeletas por dos motivos: quien no vota no deja papeleta (no hay nada que leer) y las papeletas no llevan el nombre de quien las entrega. Una papeleta en blanco es un voto, no una abstención. Además, para leerlas habría que vaciar la pila.

### Lo que rebato de los ejemplos

En las soluciones A y B, `votar` solo comprueba que el alumno esté en `presentes`. **No atiende un hecho del enunciado: "cada alumno vota una vez".** Si Pepito entrega dos papeletas, las dos entran en el montón y se cuentan, y la tabla no lo declara como limitación. Les falta un dato, quién ha votado ya, con su estructura (aquí, la lista `yaHanVotado`), y separado de la papeleta para que el voto siga siendo secreto.

También rebato un argumento de la solución A: dice que la cola "facilita reconstruir el proceso si hay que revisarlo". Como la papeleta no lleva el nombre de quien vota, leerlas en orden de recogida no permite reconstruir nada que no dé también la pila.
