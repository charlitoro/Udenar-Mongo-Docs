# Modelo de estructuras de árbol con referencias principales

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que describe una estructura en forma de árbol en los documentos de MongoDB almacenando [referencias](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-referencing) a los nodos "principales" en los nodos secundarios.

### Patrón  <a id="pattern"></a>

El patrón de _referencias principales_ almacena cada nodo del árbol en un documento; además del nodo del árbol, el documento almacena la identificación del padre del nodo.

Considere la siguiente jerarquía de categorías:

![](../../.gitbook/assets/image%20%283%29.png)

El siguiente ejemplo modela el árbol usando _referencias principales_ , almacenando la referencia a la categoría principal en el campo `parent`:

```text
db.categories.insertMany( [
   { _id: "MongoDB", parent: "Databases" },
   { _id: "dbm", parent: "Databases" },
   { _id: "Databases", parent: "Programming" },
   { _id: "Languages", parent: "Programming" },
   { _id: "Programming", parent: "Books" },
   { _id: "Books", parent: null }
] )
```

*   La consulta para recuperar el padre de un nodo es rápida y sencilla:

  ```text
  db.categories.findOne( { _id: "MongoDB" } ).parent
  ```

*   Puede crear un índice en el campo `parent`para permitir una búsqueda rápida por el nodo principal:

  ```text
  db.categories.find( { parent: "Databases" } )
  ```

*   Puede consultar por el `parent`campo para encontrar sus nodos secundarios inmediatos:

  ```text
  db.categories.find( { parent: "Databases" } )
  ```

* Para recuperar subárboles, consulte [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup).

