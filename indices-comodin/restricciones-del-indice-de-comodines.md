# Restricciones del índice de comodines

### Tipos o propiedades de índice incompatibles  <a id="incompatible-index-types-or-properties"></a>

Los índices comodín no admiten los siguientes tipos o propiedades de índice:

* [Compuesto](https://docs.mongodb.com/manual/core/index-compound/)
* [TTL](https://docs.mongodb.com/manual/core/index-ttl/)
* [Texto](https://docs.mongodb.com/manual/core/index-text/)
* [2d \(geoespacial\)](https://docs.mongodb.com/manual/core/geospatial-indexes/)
* [2dsphere \(geoespacial\)](https://docs.mongodb.com/manual/core/2dsphere/)
* [Troceado](https://docs.mongodb.com/manual/core/index-hashed/)
* [Único](https://docs.mongodb.com/manual/core/index-unique/)

NOTA

Los índices comodín son distintos e incompatibles con los [índices de texto comodín](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-wildcard) . Los índices comodín no pueden admitir consultas mediante el [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)operador.

### Patrones de consulta y agregación no admitidos  <a id="unsupported-query-and-aggregation-patterns"></a>

El campo **no** existe

Los índices comodín son [escasos](https://docs.mongodb.com/manual/core/index-sparse/) y no indexan campos vacíos. Por lo tanto, los índices comodín no pueden admitir consultas de documentos donde **no** existe un campo .

Por ejemplo, considere una colección `inventory`con un índice comodín activado `product_attributes`. El índice de comodines **no puede** admitir las siguientes consultas:

```text
db.inventory.find( {"product_attributes" : { $exists : false } } )db.inventory.aggregate([  { $match : { "product_attributes" : { $exists : false } } }])
```

El campo es igual a un documento o una matriz

Los índices comodín generan entradas para el **contenido** de un documento o matriz, y no el documento o matriz en sí. Por lo tanto, los índices comodín no pueden admitir coincidencias exactas de igualdad entre documentos y matrices. Los índices comodín _pueden_ admitir consultas en las que el campo equivale a un documento vacío `{}`.

Por ejemplo, considere una colección `inventory`con un índice comodín activado `product_attributes`. El índice de comodines **no puede** admitir las siguientes consultas:

```text
db.inventory.find({ "product_attributes" : { "price" : 29.99 } } )db.inventory.find({ "product_attributes.tags" : [ "waterproof", "fireproof" ] } )db.inventory.aggregate([{  $match : { "product_attributes" : { "price" : 29.99 } }}])db.inventory.aggregate([{  $match : { "product_attributes.tags" : ["waterproof", "fireproof" ] } }}])
```

El campo no es igual a un documento o matriz

Los índices comodín generan entradas para el **contenido** de un documento o matriz, y no el documento o matriz en sí. Por lo tanto, los índices comodín no pueden admitir coincidencias exactas de desigualdad entre documentos y matrices.

Por ejemplo, considere una colección `inventory`con un índice comodín activado `product_attributes`. El índice de comodines **no puede** admitir las siguientes consultas:

```text
db.inventory.find( { $ne : [ "product_attributes", { "price" : 29.99 } ] } )db.inventory.find( { $ne : [ "product_attributes.tags",  [ "waterproof", "fireproof" ] ] } )db.inventory.aggregate([{  $match : { $ne : [ "product_attributes", { "price" : 29.99 } ] }}])db.inventory.aggregate([{  $match : { $ne : [ "product_attributes.tags", [ "waterproof", "fireproof" ] ] }}])
```

El campo no es igual a `null`

Si un campo determinado es una matriz en cualquier documento de la colección, los índices comodín no pueden admitir consultas para documentos en los que ese campo no es igual a `null`.

Por ejemplo, considere una colección `inventory`con un índice comodín activado `product_attributes`. El índice comodín **no puede** admitir las siguientes consultas si `product_attributes.tags`es una matriz en cualquier documento de la colección:

```text
db.inventory.find( { $ne : [ "product_attributes.tags", null ] } )db.inventory.aggregate([{  $match : { $ne : [ "product_attributes.tags", null ] }}])
```

### Fragmentación  <a id="sharding"></a>

No puede fragmentar una colección mediante un índice comodín. Cree un índice que no sea comodín en el campo o los campos en los que desea fragmentar. Para obtener más información sobre la selección de claves de fragmentos, consulte [Claves de fragmentos](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-sharding-shard-key) .

