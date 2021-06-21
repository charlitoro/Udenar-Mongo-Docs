# Caducar datos de colecciones configurando TTL

Este documento proporciona una introducción a la función de recopilación [TTL](https://docs.mongodb.com/manual/reference/glossary/#std-term-TTL) o " _tiempo de vida_ " de MongoDB . Las colecciones TTL hacen posible almacenar datos en MongoDB y hacer que eliminen automáticamente los datos después de un número específico de segundos o en una hora de reloj específica.[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)

La caducidad de los datos es útil para algunas clases de información, incluidos los datos de eventos generados por la máquina, los registros y la información de la sesión que solo deben persistir durante un período de tiempo limitado.

Una [propiedad de índice TTL](https://docs.mongodb.com/manual/core/index-ttl/) especial admite la implementación de colecciones TTL. La función TTL se basa en un hilo de fondo [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)que lee los valores de fecha en el índice y elimina los [documentos](https://docs.mongodb.com/manual/reference/glossary/#std-term-document) caducados de la colección.

### Procedimientos  <a id="procedures"></a>

Para crear un [índice TTL](https://docs.mongodb.com/manual/core/index-ttl/) , use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método con la `expireAfterSeconds`opción en un campo cuyo valor sea una [fecha](https://docs.mongodb.com/manual/reference/bson-types/#std-label-document-bson-type-date) o una matriz que contenga [valores de fecha](https://docs.mongodb.com/manual/reference/bson-types/#std-label-document-bson-type-date) .NOTA

El índice TTL es un índice de campo único. Los índices compuestos no admiten la propiedad TTL. Para obtener más información sobre los índices TTL, consulte [Índices TTL](https://docs.mongodb.com/manual/core/index-ttl/) .

Puede modificar el `expireAfterSeconds`de un índice TTL existente usando el [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)comando.

#### Caducar documentos después de un número específico de segundos  <a id="expire-documents-after-a-specified-number-of-seconds"></a>

Para caducar los datos después de que haya pasado un número específico de segundos desde el campo indexado, cree un índice TTL en un campo que contenga valores de tipo de fecha BSON o una matriz de objetos con tipo de fecha BSON _y_ especifique un valor positivo distinto de cero en el `expireAfterSeconds`campo . Un documento caducará cuando `expireAfterSeconds` haya transcurrido el número de segundos en el campo desde el tiempo especificado en su campo indexado. [\[ 1 \]](https://docs.mongodb.com/manual/tutorial/expire-data/#footnote-field-is-array-of-dates)

Por ejemplo, la siguiente operación crea un índice en el campo de la `log_events`colección `createdAt`y especifica el `expireAfterSeconds`valor de `3600`para establecer el tiempo de vencimiento en una hora después del tiempo especificado por `createdAt`.

```text
db.log_events.createIndex( { "createdAt": 1 }, { expireAfterSeconds: 3600 } )
```

Al agregar documentos a la `log_events`colección, configure el `createdAt`campo a la hora actual:

```text
db.log_events.insert( {   "createdAt": new Date(),   "logEvent": 2,   "logMessage": "Success!"} )
```

MongoDB eliminará automáticamente los documentos de la `log_events` colección cuando el `createdAt`valor del documento [\[ 1 \]](https://docs.mongodb.com/manual/tutorial/expire-data/#footnote-field-is-array-of-dates) sea ​​más antiguo que el número de segundos especificado en `expireAfterSeconds`.

| \[ 1 \] | _\(_ [_1_](https://docs.mongodb.com/manual/tutorial/expire-data/#ref-field-is-array-of-dates-id1) _,_ [_2_](https://docs.mongodb.com/manual/tutorial/expire-data/#ref-field-is-array-of-dates-id2) _\)_ Si el campo contiene una matriz de objetos con tipo de fecha BSON, los datos caducan si al menos uno de los objetos con tipo de fecha BSON es más antiguo que el número de segundos especificado en `expireAfterSeconds`. |
| :--- | :--- |


#### Caducar documentos a una hora específica  <a id="expire-documents-at-a-specific-clock-time"></a>

Para caducar documentos a una hora específica, comience creando un índice TTL en un campo que contenga valores de tipo de fecha BSON o una matriz de objetos con tipo de fecha BSON _y_ especifique un `expireAfterSeconds`valor de `0`. Para cada documento de la colección, establezca el campo de fecha indexada en un valor correspondiente a la hora en que el documento debe caducar. Si el campo de fecha indexada contiene una fecha en el pasado, MongoDB considera que el documento ha caducado.

Por ejemplo, la siguiente operación crea un índice en el campo de la `log_events`colección `expireAt`y especifica el `expireAfterSeconds`valor de `0`:

```text
db.log_events.createIndex( { "expireAt": 1 }, { expireAfterSeconds: 0 } )
```

Para cada documento, establezca el valor de `expireAt`para que corresponda a la hora en que el documento debe caducar. Por ejemplo, la siguiente [`insert()`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#mongodb-method-db.collection.insert)operación agrega un documento que debe caducar en `July 22, 2013 14:00:00`.

```text
db.log_events.insert( {   "expireAt": new Date('July 22, 2013 14:00:00'),   "logEvent": 2,   "logMessage": "Success!"} )
```

MongoDB eliminará automáticamente los documentos de la `log_events` colección cuando el `expireAt`valor de los documentos sea ​​más antiguo que el número de segundos especificado en `expireAfterSeconds`, es decir, `0` segundos más antiguos en este caso. Como tal, los datos caducan en el `expireAt`valor especificado .

