# Modelo de estructuras de árbol con conjuntos anidados

### Descripción general  <a id="overview"></a>

Este documento describe un modelo de datos que describe una estructura en forma de árbol que optimiza el descubrimiento de subárboles a expensas de la mutabilidad del árbol.

### Patrón  <a id="pattern"></a>

El patrón _Conjuntos anidados_ identifica cada nodo del árbol como paradas en un recorrido de ida y vuelta del árbol. La aplicación visita cada nodo del árbol dos veces; primero durante el viaje inicial y segundo durante el viaje de regreso. El patrón _Conjuntos anidados_ almacena cada nodo del árbol en un documento; Además del nodo del árbol, el documento almacena la identificación del padre del nodo, la parada inicial del nodo en el `left`campo y su parada de retorno en el `right`campo.

Considere la siguiente jerarquía de categorías:

![](../../.gitbook/assets/image%20%286%29.png)

El siguiente ejemplo modela el árbol usando _conjuntos anidados_ :

```text
db.categories.insertMany( [
   { _id: "Books", parent: 0, left: 1, right: 12 },
   { _id: "Programming", parent: "Books", left: 2, right: 11 },
   { _id: "Languages", parent: "Programming", left: 3, right: 4 },
   { _id: "Databases", parent: "Programming", left: 5, right: 10 },
   { _id: "MongoDB", parent: "Databases", left: 6, right: 7 },
   { _id: "dbm", parent: "Databases", left: 8, right: 9 }
] )
```

Puede consultar para recuperar los descendientes de un nodo:

```text
var databaseCategory = db.categories.findOne( { _id: "Databases" } );
db.categories.find( { left: { $gt: databaseCategory.left }, right: { $lt: databaseCategory.right } } );
```

El patrón de _conjuntos anidados_ proporciona una solución rápida y eficaz para encontrar subárboles, pero es ineficaz para modificar la estructura del árbol. Como tal, este patrón es mejor para árboles estáticos que no cambian.  


