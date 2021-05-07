# Modelo de estructuras de árbol  con referencias

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que describe una estructura en forma de árbol en los documentos de MongoDB almacenando [referencias](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-referencing) en los nodos principales a los nodos secundarios.

### Patrón  <a id="pattern"></a>

El patrón _Child References_ almacena cada nodo del árbol en un documento; además del nodo del árbol, el documento almacena en una matriz la \(s\) identificación \(es\) de los hijos del nodo.

Considere la siguiente jerarquía de categorías:

![](../../.gitbook/assets/image%20%282%29.png)

El siguiente ejemplo modela el árbol usando _referencias_ secundarias, almacenando la referencia a los secundarios del nodo en el campo `children`:

```text
db.categories.insertMany( [
   { _id: "MongoDB", children: [] },
   { _id: "dbm", children: [] },
   { _id: "Databases", children: [ "MongoDB", "dbm" ] },
   { _id: "Languages", children: [] },
   { _id: "Programming", children: [ "Databases", "Languages" ] },
   { _id: "Books", children: [ "Programming" ] }
] )
```

* La consulta para recuperar los hijos inmediatos de un nodo es rápida y sencilla:

  ```text
  db.categories.findOne( { _id: "Databases" } ).children
  ```

* Puede crear un índice en el campo `children`para permitir una búsqueda rápida por los nodos secundarios:

  ```text
  db.categories.createIndex( { children: 1 } )
  ```

* Puede consultar un nodo en el `children`campo para encontrar su nodo principal y sus hermanos:

  ```text
  db.categories.find( { children: "MongoDB" } )
  ```

El patrón _Child References_ proporciona una solución adecuada para el almacenamiento de árboles siempre que no sea necesario realizar operaciones en los subárboles. Este patrón también puede proporcionar una solución adecuada para almacenar gráficos donde un nodo puede tener varios padres.

