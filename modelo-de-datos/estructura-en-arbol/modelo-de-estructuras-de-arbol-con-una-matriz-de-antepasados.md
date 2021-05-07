# Modelo de estructuras de árbol  con una matriz de antepasados

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que describe una estructura en forma de árbol en los documentos de MongoDB utilizando [referencias](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-referencing) a los nodos principales y una matriz que almacena todos los antepasados.

### Patrón  <a id="pattern"></a>

El patrón _Array of Ancestors_ almacena cada nodo del árbol en un documento; Además del nodo del árbol, el documento almacena en una matriz la \(s\) identificación \(es\) de los antepasados ​​o la ruta del nodo.

Considere la siguiente jerarquía de categorías:

![](../../.gitbook/assets/image%20%287%29.png)

El siguiente ejemplo modela el árbol usando _Array of Ancestors_ . Además del `ancestors`campo, estos documentos también almacenan la referencia a la categoría principal inmediata en el `parent`campo:

```text
db.categories.insertMany( [
  { _id: "MongoDB", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" },
  { _id: "dbm", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" },
  { _id: "Databases", ancestors: [ "Books", "Programming" ], parent: "Programming" },
  { _id: "Languages", ancestors: [ "Books", "Programming" ], parent: "Programming" },
  { _id: "Programming", ancestors: [ "Books" ], parent: "Books" },
  { _id: "Books", ancestors: [ ], parent: null }
] )
```

* La consulta para recuperar los ancestros o la ruta de un nodo es rápida y sencilla:

  ```text
  db.categories.findOne( { _id: "MongoDB" } ).ancestors
  ```

* Puede crear un índice en el campo `ancestors`para permitir una búsqueda rápida por los nodos ancestros:

  ```text
  db.categories.createIndex( { ancestors: 1 } )
  ```

* Puede consultar por campo `ancestors`para encontrar todos sus descendientes:

  ```text
  db.categories.find( { ancestors: "Programming" } )
  ```

El patrón _Array of Ancestors_ proporciona una solución rápida y eficiente para encontrar los descendientes y los antepasados ​​de un nodo mediante la creación de un índice sobre los elementos del campo de antepasados. Esto hace que _Array of Ancestors sea_ una buena opción para trabajar con subárboles.

El patrón _Array of Ancestors_ es un poco más lento que el patrón [Materialized Paths](https://docs.mongodb.com/manual/tutorial/model-tree-structures-with-materialized-paths/) , pero es más sencillo de usar.

