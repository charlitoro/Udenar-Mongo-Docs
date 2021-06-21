# Índices únicos

Un índice único asegura que los campos indexados no almacenen valores duplicados; es decir, impone la unicidad de los campos indexados. De forma predeterminada, MongoDB crea un índice único en el campo [\_id](https://docs.mongodb.com/manual/core/document/#std-label-document-id-field) durante la creación de una colección.NOTANuevo formato interno

A partir de MongoDB 4.2, para [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) \(fCV\) de 4.2 \(o superior\), MongoDB usa un nuevo formato interno para índices únicos que es incompatible con versiones anteriores de MongoDB. El nuevo formato se aplica tanto a índices únicos existentes como a índices únicos recién creados / reconstruidos.

### Crear un índice único  <a id="create-a-unique-index"></a>

Para crear un índice único, use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) método con la `unique`opción establecida en `true`.

```text
db.collection.createIndex( <key and index type specification>, { unique: true } )
```

#### Índice único en un solo campo  <a id="unique-index-on-a-single-field"></a>

Por ejemplo, para crear un índice único en el `user_id`campo de la `members`colección, use la siguiente operación en el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell:

```text
db.members.createIndex( { "user_id": 1 }, { unique: true } )
```

#### Índice compuesto único  <a id="unique-compound-index"></a>

También puede aplicar una restricción única a los [índices compuestos](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) . Si usa la restricción de unicidad en un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) , entonces MongoDB aplicará la unicidad en la _combinación_ de los valores clave del índice.

Por ejemplo, para crear un índice único en `groupNumber`, `lastname`y `firstname`campos de la `members`colección, utilice la siguiente operación en el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell:

```text
db.members.createIndex( { groupNumber: 1, lastname: 1, firstname: 1 }, { unique: true } )
```

Los hace cumplir de índice creados unicidad para la _combinación_ de `groupNumber`, `lastname`y `firstname`valores.

Para otro ejemplo, considere una colección con el siguiente documento:

```text
{ _id: 1, a: [ { loc: "A", qty: 5 }, { qty: 10 } ] }
```

Cree un índice compuesto de [varias cuñas](https://docs.mongodb.com/manual/core/index-multikey/) único en `a.loc`y `a.qty`:

```text
db.collection.createIndex( { "a.loc": 1, "a.qty": 1 }, { unique: true } )
```

El índice único permite la inserción de los siguientes documentos en la colección, ya que el índice impone unicidad para la _combinación_ de valores `a.loc`y `a.qty`:

```text
db.collection.insert( { _id: 2, a: [ { loc: "A" }, { qty: 5 } ] } )db.collection.insert( { _id: 3, a: [ { loc: "A", qty: 10 } ] } )
```

CONSEJOVer también:

* [Restricción única en documentos separados](https://docs.mongodb.com/manual/core/index-unique/#std-label-unique-separate-documents)
* [Índice único y campo faltante](https://docs.mongodb.com/manual/core/index-unique/#std-label-unique-index-and-missing-field)

### Comportamiento  <a id="behavior"></a>

#### Restricciones  <a id="restrictions"></a>

MongoDB no puede crear un [índice único](https://docs.mongodb.com/manual/core/index-unique/#std-label-index-type-unique) en los campos de índice especificados si la colección ya contiene datos que violarían la restricción única del índice.

No puede especificar una restricción única en un [índice hash](https://docs.mongodb.com/manual/core/index-hashed/#std-label-index-type-hashed) .

#### Creación de índices únicos en conjuntos de réplicas y clústeres fragmentados  <a id="building-unique-index-on-replica-sets-and-sharded-clusters"></a>

Para conjuntos de réplicas y clústeres fragmentados, el uso de un [procedimiento continuo](https://docs.mongodb.com/manual/tutorial/build-indexes-on-replica-sets/) para crear un índice único requiere que detenga todas las escrituras en la colección durante el procedimiento. Si no puede detener todas las escrituras en la colección durante el procedimiento, no utilice el procedimiento de sucesión. En su lugar, cree su índice único en la colección de la siguiente manera:

* emitir [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)en el primario para un conjunto de réplicas, o
* emitir [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)en el [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)para un clúster fragmentado.

#### Restricción única en documentos separados  <a id="unique-constraint-across-separate-documents"></a>

La restricción única se aplica a documentos separados de la colección. Es decir, el índice único evita que documentos _separados_ tengan el mismo valor para la clave indexada.

Debido a que la restricción se aplica a documentos separados, para un índice de [varias claves](https://docs.mongodb.com/manual/core/index-multikey/) único , un documento puede tener elementos de matriz que dan como resultado valores de clave de índice repetidos siempre que los valores de clave de índice para ese documento no dupliquen los de otro documento. En este caso, la entrada de índice repetida se inserta en el índice solo una vez.

Por ejemplo, considere una colección con los siguientes documentos:

```text
{ _id: 1, a: [ { loc: "A", qty: 5 }, { qty: 10 } ] }{ _id: 2, a: [ { loc: "A" }, { qty: 5 } ] }{ _id: 3, a: [ { loc: "A", qty: 10 } ] }
```

Cree un índice compuesto de varias cuñas único en `a.loc`y `a.qty`:

```text
db.collection.createIndex( { "a.loc": 1, "a.qty": 1 }, { unique: true } )
```

El índice único permite la inserción del siguiente documento en la colección si ningún otro documento de la colección tiene un valor de clave de índice de `{ "a.loc": "B", "a.qty": null }`.

```text
db.collection.insert( { _id: 4, a: [ { loc: "B" }, { loc: "B" } ] } )
```

#### Índice único y campo faltante  <a id="unique-index-and-missing-field"></a>

Si un documento no tiene un valor para el campo indexado en un índice único, el índice almacenará un valor nulo para este documento. Debido a la restricción única, MongoDB solo permitirá un documento que carece del campo indexado. Si hay más de un documento sin un valor para el campo indexado o falta el campo indexado, la generación del índice fallará con un error de clave duplicada.

Por ejemplo, una colección tiene un índice único en `x`:

```text
db.collection.createIndex( { "x": 1 }, { unique: true } )
```

El índice único permite la inserción de un documento sin el campo `x`si la colección aún no contiene un documento al que le falta el campo `x`:

```text
db.collection.insert( { y: 1 } )
```

Sin embargo, los errores de índice únicos en la inserción de un documento sin el campo `x`si la colección ya contiene un documento al que le falta el campo `x`:

```text
db.collection.insert( { z: 1 } )
```

La operación no puede insertar el documento debido a la violación de la restricción única sobre el valor del campo `x`:

```text
WriteResult({   "nInserted" : 0,   "writeError" : {      "code" : 11000,      "errmsg" : "E11000 duplicate key error index: test.collection.$a.b_1 dup key: { : null }"   }})
```

