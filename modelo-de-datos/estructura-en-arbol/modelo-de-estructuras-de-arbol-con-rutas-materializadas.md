# Modelo de estructuras de árbol con rutas materializadas

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que describe una estructura en forma de árbol en los documentos de MongoDB almacenando rutas de relación completas entre documentos.

### Patrón  <a id="pattern"></a>

El patrón de _rutas materializadas_ almacena cada nodo del árbol en un documento; Además del nodo del árbol, el documento almacena como una cadena la \(s\) identificación \(es\) de los antepasados ​​o la ruta del nodo. Aunque el patrón de _rutas materializadas_ requiere pasos adicionales para trabajar con cadenas y expresiones regulares, el patrón también proporciona más flexibilidad para trabajar con la ruta, como buscar nodos por rutas parciales.

Considere la siguiente jerarquía de categorías:

![](../../.gitbook/assets/image%20%285%29.png)

El siguiente ejemplo modela el árbol usando _rutas materializadas_ , almacenando la ruta en el campo `path`; la cadena de ruta usa la coma `,`como delimitador:

```text
db.categories.insertMany( [
   { _id: "Books", path: null },
   { _id: "Programming", path: ",Books," },
   { _id: "Databases", path: ",Books,Programming," },
   { _id: "Languages", path: ",Books,Programming," },
   { _id: "MongoDB", path: ",Books,Programming,Databases," },
   { _id: "dbm", path: ",Books,Programming,Databases," }
] )
```

* Puede consultar para recuperar el árbol completo, ordenando por campo `path`:

  ```text
  db.categories.find().sort( { path: 1 } )
  ```

* Puede usar expresiones regulares en el `path`campo para encontrar los descendientes de `Programming`:

  ```text
  db.categories.find( { path: /,Programming,/ } )
  ```

* También puede recuperar los descendientes de `Books`donde `Books`también se encuentra en el nivel más alto de la jerarquía:

  ```text
  db.categories.find( { path: /^,Books,/ } )
  ```

* Para crear un índice en el campo, `path`use la siguiente invocación:

  ```text
  db.categories.createIndex( { path: 1 } )
  ```

  Este índice puede mejorar el rendimiento según la consulta:

  * Para consultas del subárbol raíz `Books`\(por ejemplo, `/^,Books,/` o `/^,Books,Programming,/`\), un índice en el `path`campo mejora significativamente el rendimiento de la consulta.
  * Para consultas de subárboles donde la ruta desde la raíz no se proporciona en la consulta \(por ejemplo `/,Databases,/`\), o consultas similares de subárboles, donde el nodo puede estar en el medio de la cadena indexada, la consulta debe inspeccionar el índice completo .

    Para estas consultas, un índice _puede_ proporcionar alguna mejora en el rendimiento _si_ el índice es significativamente más pequeño que la colección completa.

