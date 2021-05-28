# Índices Multikey

Para indexar un campo que contiene un valor de matriz, MongoDB crea una clave de índice para cada elemento de la matriz. Estos índices de _varias claves_ admiten consultas eficientes contra campos de matriz. Los índices de varias claves se pueden construir sobre matrices que contienen tanto valores escalares [\[ 1 \]](https://docs.mongodb.com/manual/core/index-multikey/#footnote-scalar) \(por ejemplo, cadenas, números\) _como_ documentos anidados.

![](../../.gitbook/assets/image%20%2814%29.png)

| \[ [1](https://docs.mongodb.com/manual/core/index-multikey/#ref-scalar-id1) \] | Un valor escalar se refiere a un valor que no es ni un documento incrustado ni una matriz. |
| :--- | :--- |


### Crear índice de varias ciudades  <a id="create-multikey-index"></a>

Para crear un índice de varias claves, use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método:

```text
db.coll.createIndex( { <field>: < 1 or -1 > } )
```

MongoDB crea automáticamente un índice de varias claves si algún campo indexado es una matriz; no es necesario especificar explícitamente el tipo de múltiples claves.

_Modificado en la versión 3.4_ : solo _para los motores de almacenamiento WiredTiger e In-Memory_ ,

A partir de MongoDB 3.4, para índices de múltiples claves creados con MongoDB 3.4 o posterior, MongoDB realiza un seguimiento de qué campo o campos indexados hacen que un índice sea un índice de múltiples claves. El seguimiento de esta información permite que el motor de consultas de MongoDB utilice límites de índice más estrictos.

### Límites de índice  <a id="index-bounds"></a>

Si un índice es de varias claves, el cálculo de los límites del índice sigue reglas especiales. Para obtener detalles sobre los límites de índice de múltiples claves, consulte [Límites de índice de múltiples claves](https://docs.mongodb.com/manual/core/multikey-index-bounds/) .

### Índice único de múltiples vías  <a id="unique-multikey-index"></a>

Para índices [únicos](https://docs.mongodb.com/manual/core/index-unique/) , la restricción de unicidad se aplica a documentos separados de la colección en lugar de a un solo documento.

Debido a que la restricción de [unicidad se](https://docs.mongodb.com/manual/core/index-unique/#std-label-unique-separate-documents) aplica a documentos separados, para un [índice de múltiples claves único](https://docs.mongodb.com/manual/core/index-unique/#std-label-unique-separate-documents) , un documento puede tener elementos de matriz que dan como resultado valores de clave de índice repetidos siempre que los valores de clave de índice para ese documento no dupliquen los de otro documento.

Para obtener más información, consulte [Restricción única en documentos separados](https://docs.mongodb.com/manual/core/index-unique/#std-label-unique-separate-documents) .

### Limitaciones  <a id="limitations"></a>

#### Índices compuestos de múltiples ciclos  <a id="compound-multikey-indexes"></a>

Para un índice [compuesto de](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) varias claves, cada documento indexado puede tener _como máximo_ un campo indexado cuyo valor es una matriz. Es decir:

* No puede crear un índice compuesto de varias claves si más de un campo a indexar de un documento es una matriz. Por ejemplo, considere una colección que contiene el siguiente documento:

  ```text
  { _id: 1, a: [ 1, 2 ], b: [ 1, 2 ], category: "AB - both arrays" }
  ```

  No puede crear un índice compuesto de varias claves `{ a: 1, b: 1 }`en la colección, ya que los campos `a`y `b`son matrices.

* O, si ya existe un índice compuesto de varias claves, no puede insertar un documento que infrinja esta restricción.

  Considere una colección que contiene los siguientes documentos:

```text
{ _id: 1, a: [1, 2], b: 1, category: "A array" }
{ _id: 2, a: 1, b: [1, 2], category: "B array" }
```

Se permite un índice compuesto de múltiples claves `{ a: 1, b: 1 }`ya que para cada documento, solo un campo indexado por el índice compuesto de múltiples claves es una matriz; es decir, ningún documento contiene valores de matriz para los campos `a` y `b`.

Sin embargo, después de crear el índice compuesto de varias claves, si intenta insertar un documento donde ambos campos `a`y `b`son matrices, MongoDB fallará la inserción.

Si un campo es una matriz de documentos, puede indexar los campos incrustados para crear un índice compuesto. Por ejemplo, considere una colección que contiene los siguientes documentos:

```text
{ _id: 1, a: [ { x: 5, z: [ 1, 2 ] }, { z: [ 1, 2 ] } ] }
{ _id: 2, a: [ { x: 5 }, { z: 4 } ] }
```

Puede crear un índice compuesto en `{ "a.x": 1, "a.z": 1 }`. También se aplica la restricción en la que _como máximo_ un campo indexado puede ser una matriz.

Para obtener un ejemplo, consulte [Matrices de índice con documentos incrustados](https://docs.mongodb.com/manual/core/index-multikey/#std-label-multikey-embedded-documents) .

**CONSEJO:** _**Ver también:**_

* _\*\*\*\*_[_Restricción única en documentos separados_](https://docs.mongodb.com/manual/core/index-unique/#std-label-unique-separate-documents)\_\_
* \_\_[_Índice único en un solo campo_](https://docs.mongodb.com/manual/core/index-unique/#std-label-index-unique-index)\_\_

#### Ordenar  <a id="sorting"></a>

Como resultado de los cambios en el comportamiento de clasificación en los campos de la matriz en MongoDB 3.6, al ordenar en una matriz indexada con un [índice de varias claves,](https://docs.mongodb.com/manual/core/index-multikey/) el plan de consulta incluye una etapa SORT de bloqueo. El nuevo comportamiento de clasificación puede afectar negativamente al rendimiento.

En una CLASIFICACIÓN de bloqueo, toda la entrada debe ser consumida por el paso de clasificación antes de que pueda producir una salida. En una clasificación _indexada_ o sin bloqueo, el paso de clasificación escanea el índice para producir resultados en el orden solicitado.

#### Claves de fragmentos  <a id="shard-keys"></a>

No **puede** especificar un índice de varias claves como índice de clave de fragmento.

Sin embargo, si el índice de clave de fragmento es un [prefijo](https://docs.mongodb.com/manual/core/index-compound/#std-label-compound-index-prefix) de un índice compuesto, el índice compuesto puede convertirse en un índice compuesto de _varias claves_ si una de las otras claves \(es decir, claves que no forman parte de la clave de fragmento\) indexa una matriz. Los índices compuestos de múltiples ciclos pueden tener un impacto en el rendimiento.

#### Índices hash  <a id="hashed-indexes"></a>

[Los](https://docs.mongodb.com/manual/core/index-hashed/) índices [hash ](https://docs.mongodb.com/manual/core/index-hashed/)**no pueden** ser de varias claves.

#### Consultas cubiertas  <a id="covered-queries"></a>

[Los índices de varias claves](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multikey) no pueden cubrir consultas sobre campos de matriz.

Sin embargo, a partir de 3.6, los índices de múltiples claves pueden cubrir consultas sobre los campos que no son de matriz si el índice rastrea qué campo o campos hacen que el índice sea de múltiples claves. Los índices de múltiples claves creados en MongoDB 3.4 o posterior en motores de almacenamiento distintos de MMAPv1 [\[ 2 \]](https://docs.mongodb.com/manual/core/index-multikey/#footnote-2) rastrean estos datos.

| \[ [2](https://docs.mongodb.com/manual/core/index-multikey/#ref-id2) \] | A partir de la versión 4.2, MongoDB elimina el motor de almacenamiento MMAPv1 obsoleto. |
| :--- | :--- |


#### Consulta sobre el campo de matriz como un todo  <a id="query-on-the-array-field-as-a-whole"></a>

Cuando un filtro de consulta especifica una [coincidencia exacta para una matriz como un todo](https://docs.mongodb.com/manual/tutorial/query-arrays/#std-label-array-match-exact) , MongoDB puede usar el índice de múltiples claves para buscar el primer elemento de la matriz de consulta, pero no puede usar el escaneo de índice de múltiples claves para encontrar la matriz completa. En su lugar, después de usar el índice de varias claves para buscar el primer elemento de la matriz de consulta, MongoDB recupera los documentos asociados y filtra los documentos cuya matriz coincide con la matriz de la consulta.

Por ejemplo, considere una `inventory`colección que contiene los siguientes documentos:

```text
{ _id: 5, type: "food", item: "aaa", ratings: [ 5, 8, 9 ] }
{ _id: 6, type: "food", item: "bbb", ratings: [ 5, 9 ] }
{ _id: 7, type: "food", item: "ccc", ratings: [ 9, 5, 8 ] }
{ _id: 8, type: "food", item: "ddd", ratings: [ 9, 5 ] }
{ _id: 9, type: "food", item: "eee", ratings: [ 5, 9, 5 ] }
```

La colección tiene un índice de varias claves en el `ratings`campo:

```text
db.inventory.createIndex( { ratings: 1 } )
```

La siguiente consulta busca documentos donde el `ratings`campo es la matriz `[ 5, 9 ]`:

```text
db.inventory.find( { ratings: [ 5, 9 ] } )
```

MongoDB puede usar el índice de múltiples claves para encontrar documentos que tengan `5`en cualquier posición en la `ratings`matriz. Luego, MongoDB recupera estos documentos y filtra los documentos cuya `ratings`matriz es igual a la matriz de consulta `[ 5, 9 ]`.

#### $ expr  <a id="-expr"></a>

[`$expr`](https://docs.mongodb.com/manual/reference/operator/query/expr/#mongodb-query-op.-expr) no admite índices de varias claves.

#### Índices creados en MongoDB 3.2 o anterior  <a id="indexes-built-on-mongodb-3.2-or-earlier"></a>

Los índices creados en MongoDB 3.2 o anteriores no contienen los indicadores necesarios para admitir el uso optimizado de índices de múltiples claves. Para beneficiarse de las mejoras de rendimiento de los índices de varias claves, debe:

* Reconstruya los índices de formato anterior en MongoDB 3.4 o posterior. Ver [`db.collection.reIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.reIndex/#mongodb-method-db.collection.reIndex).
* En un conjunto de réplicas, [vuelva a sincronizar los miembros del conjunto de réplicas que](https://docs.mongodb.com/manual/tutorial/resync-replica-set-member/) contienen índices de formato más antiguo.

### Ejemplos  <a id="examples"></a>

#### Indexar matrices básicas  <a id="index-basic-arrays"></a>

Considere una `survey`colección con el siguiente documento:

```text
{ _id: 1, item: "ABC", ratings: [ 2, 5, 9 ] }
```

Cree un índice en el campo `ratings`:

```text
db.survey.createIndex( { ratings: 1 } )
```

Dado que el `ratings`campo contiene una matriz, el índice de `ratings` es de varias claves. El índice de varias claves contiene las siguientes tres claves de índice, cada una de las cuales apunta al mismo documento:

* `2`,
* `5`, y
* `9`.

#### Matrices de índice con documentos incrustados  <a id="index-arrays-with-embedded-documents"></a>

Puede crear índices de varias claves en campos de matriz que contienen objetos anidados.

Considere una `inventory`colección con documentos de la siguiente forma:

```text
{
  _id: 1,
  item: "abc",
  stock: [
    { size: "S", color: "red", quantity: 25 },
    { size: "S", color: "blue", quantity: 10 },
    { size: "M", color: "blue", quantity: 50 }
  ]
}
{
  _id: 2,
  item: "def",
  stock: [
    { size: "S", color: "blue", quantity: 20 },
    { size: "M", color: "blue", quantity: 5 },
    { size: "M", color: "black", quantity: 10 },
    { size: "L", color: "red", quantity: 2 }
  ]
}
{
  _id: 3,
  item: "ijk",
  stock: [
    { size: "M", color: "blue", quantity: 15 },
    { size: "L", color: "blue", quantity: 100 },
    { size: "L", color: "red", quantity: 25 }
  ]
}

...
```

La siguiente operación crea un índice de varias claves en los campos `stock.size` y `stock.quantity`:

```text
db.inventory.createIndex( { "stock.size": 1, "stock.quantity": 1 } )
```

El índice compuesto de varias claves puede admitir consultas con predicados que incluyen tanto campos indexados como predicados que solo incluyen el prefijo del índice `"stock.size"`, como en los siguientes ejemplos:

```text
db.inventory.find( { "stock.size": "M" } )
db.inventory.find( { "stock.size": "S", "stock.quantity": { $gt: 20 } } )
```

Para obtener detalles sobre cómo MongoDB puede combinar límites de índice de múltiples claves, consulte [Límites de índice de múltiples claves](https://docs.mongodb.com/manual/core/multikey-index-bounds/) . Para obtener más información sobre el comportamiento de índices compuestos y prefijos, consulte [índices compuestos y prefijos](https://docs.mongodb.com/manual/core/index-compound/#std-label-compound-index-prefix) .

El índice compuesto de varias claves también puede admitir operaciones de ordenación, como los siguientes ejemplos:

```text
db.inventory.find( ).sort( { "stock.size": 1, "stock.quantity": 1 } )
db.inventory.find( { "stock.size": "M" } ).sort( { "stock.quantity": 1 } )
```

Para obtener más información sobre el comportamiento de los índices compuestos y las operaciones de ordenación, consulte [Usar índices para ordenar los resultados de la consulta](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/) .

