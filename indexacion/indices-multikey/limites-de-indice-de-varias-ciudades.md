# Límites de índice de varias ciudades

Los límites de un escaneo de índice definen las partes de un índice para buscar durante una consulta. Cuando existen múltiples predicados sobre un índice, MongoDB intentará combinar los límites de estos predicados, ya sea por _intersección_ o _composición_ , para producir un escaneo con límites más pequeños.

### Límites de intersección para índice de múltiples vías  <a id="intersect-bounds-for-multikey-index"></a>

La intersección de límites se refiere a una conjunción lógica \(es decir `AND`\) de múltiples límites. Por ejemplo, dados dos límites `[ [ 3, Infinity ] ]` y `[ [ -Infinity, 6 ] ]`, la intersección de los límites da como resultado `[ [ 3, 6 ] ]`.

Dado un campo de matriz [indexado](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multikey) , considere una consulta que especifique múltiples predicados en la matriz y pueda usar un [índice de varias claves](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multikey) . MongoDB puede intersecar los [límites del índice de varias claves](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multikey) si una se [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)une a los predicados.

Por ejemplo, una colección `survey`contiene documentos con un campo `item`y un campo de matriz `ratings`:

```text
{ _id: 1, item: "ABC", ratings: [ 2, 9 ] }{ _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
```

Cree un [índice de varias claves](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multikey) en la `ratings` matriz:

```text
db.survey.createIndex( { ratings: 1 } )
```

La siguiente consulta suele [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)requerir que la matriz contenga al menos un elemento _único_ que coincida con ambas condiciones:

```text
db.survey.find( { ratings : { $elemMatch: { $gte: 3, $lte: 6 } } } )
```

Tomando los predicados por separado:

* los límites para el predicado mayor o igual a 3 \(es decir `$gte: 3`\) son `[ [ 3, Infinity ] ]`;
* los límites para el predicado menor o igual a 6 \(es decir `$lte: 6`\) son `[ [ -Infinity, 6 ] ]`.

Debido a que la consulta utiliza [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)para unir estos predicados, MongoDB puede cruzar los límites para:

```text
ratings: [ [ 3, 6 ] ]
```

Si la consulta _no_ une las condiciones en el campo de matriz con [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch), MongoDB no puede cruzar los límites del índice de múltiples claves. Considere la siguiente consulta:

```text
db.survey.find( { ratings : { $gte: 3, $lte: 6 } } )
```

La consulta busca en la `ratings`matriz al menos un elemento mayor o igual a 3 y al menos un elemento menor o igual a 6. Debido a que un solo elemento no necesita cumplir ambos criterios, MongoDB _no_ cruza los límites y usa `[ [ 3, Infinity ] ]`o `[ [ -Infinity, 6 ] ]`. MongoDB no garantiza cuál de estos dos límites elige.

### Límites compuestos para índice de múltiples ciclos  <a id="compound-bounds-for-multikey-index"></a>

Los límites compuestos se refieren al uso de límites para varias claves de [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) . Por ejemplo, dado un índice compuesto `{ a: 1, b: 1 }`con límites en el campo `a`de `[ [ 3, Infinity ] ]`y límites en el campo `b`de `[ [ -Infinity, 6 ] ]`, la combinación de los límites da como resultado el uso de ambos límites:

```text
{ a: [ [ 3, Infinity ] ], b: [ [ -Infinity, 6 ] ] }
```

Si MongoDB no puede componer los dos límites, MongoDB siempre restringe el escaneo de índice por el límite en su campo inicial, en este caso `a: [ [ 3, Infinity ] ]`,.

#### Índice compuesto en un campo de matriz  <a id="compound-index-on-an-array-field"></a>

Considere un índice compuesto de varias cuñas; es decir, un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) donde uno de los campos indexados es una matriz. Por ejemplo, una colección `survey`contiene documentos con un campo `item`y un campo de matriz `ratings`:

```text
{ _id: 1, item: "ABC", ratings: [ 2, 9 ] }{ _id: 2, item: "XYZ", ratings: [ 4, 3 ] }
```

Cree un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) en el `item` campo y el `ratings`campo:

```text
db.survey.createIndex( { item: 1, ratings: 1 } )
```

La siguiente consulta especifica una condición en ambas claves del índice:

```text
db.survey.find( { item: "XYZ", ratings: { $gte: 3 } } )
```

Tomando los predicados por separado:

* los límites del `item: "XYZ"`predicado son `[ [ "XYZ", "XYZ" ] ]`;
* los límites del `ratings: { $gte: 3 }`predicado son `[ [ 3, Infinity ] ]`.

MongoDB puede componer los dos límites para usar los límites combinados de:

```text
{ item: [ [ "XYZ", "XYZ" ] ], ratings: [ [ 3, Infinity ] ] }
```

**Consultas de rango en un campo indexado escalar \(WiredTiger\)** 

_Modificado en la versión 3.4_ : solo _para los motores de almacenamiento WiredTiger e In-Memory_ ,

A partir de MongoDB 3.4, para índices de múltiples claves creados con MongoDB 3.4 o posterior, MongoDB realiza un seguimiento de qué campo o campos indexados hacen que un índice sea un índice de múltiples claves. El seguimiento de esta información permite que el motor de consultas de MongoDB utilice límites de índice más estrictos.

El [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) antes mencionado está en el campo escalar [\[ 1 \]](https://docs.mongodb.com/manual/core/multikey-index-bounds/#footnote-scalar) `item` y el campo de matriz `ratings`:

```text
db.survey.createIndex( { item: 1, ratings: 1 } )
```

Para los motores de almacenamiento WiredTiger y In-Memory, si una operación de consulta especifica múltiples predicados en los campos escalares indexados de un índice compuesto de múltiples claves creado en MongoDB 3.4 o posterior, MongoDB intersecará los límites del campo.

Por ejemplo, la siguiente operación especifica una consulta de rango en el campo escalar, así como una consulta de rango en el campo de matriz:

```text
db.survey.find( {   item: { $gte: "L", $lte: "Z"}, ratings : { $elemMatch: { $gte: 3, $lte: 6 } }} )
```

MongoDB se cruzará los límites de `item`a `[ [ "L", "Z" ] ]` y clasificaciones a `[[3.0, 6.0]]`utilizar los límites combinados de:

```text
"item" : [ [ "L", "Z" ] ], "ratings" : [ [3.0, 6.0] ]
```

Para otro ejemplo, considere dónde pertenecen los campos escalares a un documento anidado. Por ejemplo, una colección `survey`contiene los siguientes documentos:

```text
{ _id: 1, item: { name: "ABC", manufactured: 2016 }, ratings: [ 2, 9 ] }{ _id: 2, item: { name: "XYZ", manufactured: 2013 },  ratings: [ 4, 3 ] }
```

Crear un índice compuesto multicircuito en los campos escalares `"item.name"`, `"item.manufactured"`y el campo de matriz `ratings`:

```text
db.survey.createIndex( { "item.name": 1, "item.manufactured": 1, ratings: 1 } )
```

Considere la siguiente operación que especifica predicados de consulta en los campos escalares:

```text
db.survey.find( {   "item.name": "L" ,   "item.manufactured": 2012} )
```

Para esta consulta, MongoDB puede usar los límites combinados de:

```text
"item.name" : [ ["L", "L"] ], "item.manufactured" : [ [2012.0, 2012.0] ]
```

Las versiones anteriores de MongoDB no pueden combinar estos límites para los campos escalares.

| \[ [1](https://docs.mongodb.com/manual/core/multikey-index-bounds/#ref-scalar-id1) \] | Un campo escalar es un campo cuyo valor no es ni un documento ni una matriz; por ejemplo, un campo cuyo valor es una cadena o un número entero es un campo escalar. Un campo escalar puede ser un campo anidado en un documento, siempre que el campo en sí no sea una matriz o un documento. Por ejemplo, en el documento `{ a: { b: { c: 5, d: 5 } } }`, `c`y `d`son campos escalares donde como `a`y `b`no son. |
| :--- | :--- |


#### Índice compuesto en campos de una matriz de documentos incrustados  <a id="compound-index-on-fields-from-an-array-of-embedded-documents"></a>

Si una matriz contiene documentos incrustados, para indexar los campos contenidos en los documentos incrustados, utilice el [nombre del campo con puntos](https://docs.mongodb.com/manual/core/document/#std-label-document-dot-notation) en la especificación del índice. Por ejemplo, dada la siguiente matriz de documentos incrustados:

```text
ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]
```

El nombre del campo con puntos para el `score`campo es `"ratings.score"`.

**Límites compuestos de un campo que no es una matriz y un campo de una matriz** 

Considere que una colección `survey2`contiene documentos con un campo `item`y un campo de matriz `ratings`:

```text
{  _id: 1,  item: "ABC",  ratings: [ { score: 2, by: "mn" }, { score: 9, by: "anon" } ]}{  _id: 2,  item: "XYZ",  ratings: [ { score: 5, by: "anon" }, { score: 7, by: "wv" } ]}
```

Cree un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) en el campo que no sea de matriz `item`, así como dos campos de una matriz `ratings.score`y `ratings.by`:

```text
db.survey2.createIndex( { "item": 1, "ratings.score": 1, "ratings.by": 1 } )
```

La siguiente consulta especifica una condición en los tres campos:

```text
db.survey2.find( { item: "XYZ",  "ratings.score": { $lte: 5 }, "ratings.by": "anon" } )
```

Tomando los predicados por separado:

* los límites del `item: "XYZ"`predicado son `[ [ "XYZ", "XYZ" ] ]`;
* los límites del `score: { $lte: 5 }`predicado son `[ [ -Infinity, 5 ] ]`;
* los límites del `by: "anon"`predicado son `[ "anon", "anon" ]`.

MongoDB puede agravar los límites para la `item`llave con _cualquiera_ de los límites para `"ratings.score"`o de los límites para `"ratings.by"`, dependiendo de los predicados de consulta y los valores de clave de índice. MongoDB no garantiza los límites que componen con el `item` campo. Por ejemplo, MongoDB elegirá componer los `item`límites con los `"ratings.score"`límites:

```text
{  "item" : [ [ "XYZ", "XYZ" ] ],  "ratings.score" : [ [ -Infinity, 5 ] ],  "ratings.by" : [ [ MinKey, MaxKey ] ]}
```

O, MongoDB puede optar por componer los `item`límites con `"ratings.by"`límites:

```text
{  "item" : [ [ "XYZ", "XYZ" ] ],  "ratings.score" : [ [ MinKey, MaxKey ] ],  "ratings.by" : [ [ "anon", "anon" ] ]}
```

Sin embargo, para componer los límites para `"ratings.score"`con los límites para `"ratings.by"`, la consulta debe usar [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch). Consulte [Límites compuestos de campos de índice de una matriz](https://docs.mongodb.com/manual/core/multikey-index-bounds/#std-label-compound-fields-from-array) para obtener más información.

**Límites compuestos de campos de índice de una matriz** 

Para combinar los límites de las claves de índice de la misma matriz:

* las claves de índice deben compartir la misma ruta de campo hasta pero excluyendo los nombres de campo, y
* la consulta debe especificar predicados en los campos que se utilizan [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)en esa ruta.

Para un campo en un documento incrustado, el [nombre del campo con puntos](https://docs.mongodb.com/manual/core/document/#std-label-document-dot-notation) , como `"a.b.c.d"`, es la ruta del campo para `d`. Para aumentar los límites de las claves de índice de la misma matriz, [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)debe estar en la ruta hasta el nombre del campo en sí, _pero sin_ incluirlo; es decir `"a.b.c"`.

Por ejemplo, cree un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) en `ratings.score`los `ratings.by`campos y:

```text
db.survey2.createIndex( { "ratings.score": 1, "ratings.by": 1 } )
```

Los campos `"ratings.score"`y `"ratings.by"`comparten la ruta del campo `ratings`. La siguiente consulta se usa [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)en el campo `ratings`para requerir que la matriz contenga al menos un elemento _único_ que cumpla ambas condiciones:

```text
db.survey2.find( { ratings: { $elemMatch: { score: { $lte: 5 }, by: "anon" } } } )
```

Tomando los predicados por separado:

* los límites del `score: { $lte: 5 }`predicado son `[ -Infinity, 5 ]`;
* los límites del `by: "anon"`predicado son `[ "anon", "anon" ]`.

MongoDB puede componer los dos límites para usar los límites combinados de:

```text
{ "ratings.score" : [ [ -Infinity, 5 ] ], "ratings.by" : [ [ "anon", "anon" ] ] }
```

**Consulta sin $elemMatch**

Si la consulta _no_ une las condiciones de los campos de matriz indexados con [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch), MongoDB _no puede_ aumentar sus límites. Considere la siguiente consulta:

```text
db.survey2.find( { "ratings.score": { $lte: 5 }, "ratings.by": "anon" } )
```

Debido a que un solo documento incrustado en la matriz no necesita cumplir ambos criterios, MongoDB _no_ agrava los límites. Cuando se usa un índice compuesto, si MongoDB no puede restringir todos los campos del índice, MongoDB siempre restringe el campo inicial del índice, en este caso `"ratings.score"`:

```text
{  "ratings.score": [ [ -Infinity, 5 ] ],  "ratings.by": [ [ MinKey, MaxKey ] ]}
```

**$elemMatchen ruta incompleta** 

Si la consulta no especifica [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)en la ruta de los campos incrustados, hasta pero excluyendo los nombres de campo, MongoDB **no puede** combinar los límites de las claves de índice de la misma matriz.

Por ejemplo, una colección `survey3`contiene documentos con un campo `item`y un campo de matriz `ratings`:

```text
{  _id: 1,  item: "ABC",  ratings: [ { scores: [ { q1: 2, q2: 4 }, { q1: 3, q2: 8 } ], loc: "A" },             { scores: [ { q1: 2, q2: 5 } ], loc: "B" } ]}{  _id: 2,  item: "XYZ",  ratings: [ { scores: [ { q1: 7 }, { q1: 2, q2: 8 } ], loc: "B" } ]}
```

Cree un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) en `ratings.scores.q1`los `ratings.scores.q2`campos y:

```text
db.survey3.createIndex( { "ratings.scores.q1": 1, "ratings.scores.q2": 1 } )
```

Los campos `"ratings.scores.q1"`y `"ratings.scores.q2"`comparten la ruta del campo `"ratings.scores"`y [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)deben estar en esa ruta.

Sin embargo, la siguiente consulta usa un [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)pero no en la ruta requerida:

```text
db.survey3.find( { ratings: { $elemMatch: { 'scores.q1': 2, 'scores.q2': 8 } } } )
```

Como tal, MongoDB **no puede**`"ratings.scores.q2"` aumentar los límites y el campo no estará restringido durante el escaneo del índice. Para aumentar los límites, la consulta debe usar [`$elemMatch`](https://docs.mongodb.com/manual/reference/operator/query/elemMatch/#mongodb-query-op.-elemMatch)en la ruta `"ratings.scores"`:

```text
db.survey3.find( { 'ratings.scores': { $elemMatch: { 'q1': 2, 'q2': 8 } } } )
```

