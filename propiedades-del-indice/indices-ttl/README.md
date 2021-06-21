# Índices TTL

NOTA

Si está eliminando documentos para ahorrar en costos de almacenamiento, considere [Archivar](https://docs.atlas.mongodb.com/online-archive/manage-online-archive/) en [línea](https://docs.atlas.mongodb.com/online-archive/manage-online-archive/) en [MongoDB Atlas](https://www.mongodb.com/cloud?tck=docs_server) . Online Archive archiva automáticamente los datos a los que se accede con poca frecuencia en buckets S3 totalmente administrados para una distribución de datos rentable.

Los índices TTL son índices especiales de un solo campo que MongoDB puede usar para eliminar automáticamente documentos de una colección después de una cierta cantidad de tiempo o en una hora específica. La caducidad de los datos es útil para ciertos tipos de información, como datos de eventos generados por una máquina, registros e información de sesión, que solo necesitan persistir en una base de datos durante un período de tiempo finito.

Para crear un índice TTL, use el [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) método en un campo cuyo valor sea una [fecha](https://docs.mongodb.com/manual/reference/bson-types/#std-label-document-bson-type-date) o una matriz que contenga [valores de fecha](https://docs.mongodb.com/manual/reference/bson-types/#std-label-document-bson-type-date) , y especifique la `expireAfterSeconds` opción con el valor TTL deseado en segundos.

Por ejemplo, para crear un índice TTL en el `lastModifiedDate`campo de la `eventlog`colección, con un valor TTL de `3600`segundos, use la siguiente operación en el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell:

```text
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```

### Comportamiento  <a id="behavior"></a>

#### Caducidad de datos  <a id="expiration-of-data"></a>

Los índices TTL caducan los documentos después de que haya pasado el número especificado de segundos desde el valor del campo indexado; es decir, el umbral de caducidad es el valor del campo indexado más el número especificado de segundos.

Si el campo es una matriz y hay varios valores de fecha en el índice, MongoDB usa el valor de fecha más _bajo_ \(es decir, el más antiguo\) en la matriz para calcular el umbral de vencimiento.

Si el campo indexado en un documento no es una [fecha](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON-types) o una matriz que contiene un valor de fecha, el documento no caducará.

Si un documento no contiene el campo indexado, el documento no caducará.

#### Eliminar operaciones  <a id="delete-operations"></a>

Un hilo en segundo plano [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)lee los valores del índice y elimina los [documentos](https://docs.mongodb.com/manual/reference/glossary/#std-term-document) caducados de la colección.

Cuando el subproceso TTL está activo, verá operaciones de eliminación en la salida [`db.currentOp()`](https://docs.mongodb.com/manual/reference/method/db.currentOp/#mongodb-method-db.currentOp)o en los datos recopilados por el [generador de perfiles de](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#std-label-database-profiler) la [base de datos](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#std-label-database-profiler) .

**Momento de la operación de eliminación** 

MongoDB comienza a eliminar los documentos caducados tan pronto como el índice termina de crearse en el [principal](https://docs.mongodb.com/manual/reference/glossary/#std-term-primary) . Para obtener más información sobre el proceso de generación de índices, consulte [Generaciones de índices en colecciones pobladas](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations) .

El índice TTL no garantiza que los datos caducados se eliminarán inmediatamente después de su vencimiento. Puede haber un retraso entre el momento en que expira un documento y el momento en que MongoDB elimina el documento de la base de datos.

La tarea en segundo plano que elimina los documentos caducados se ejecuta _cada 60 segundos_ . Como resultado, los documentos pueden permanecer en una colección durante el período entre el vencimiento del documento y la ejecución de la tarea en segundo plano.

Debido a que la duración de la operación de eliminación depende de la carga de trabajo de su [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancia, los datos caducados pueden existir durante algún tiempo _más allá_ del período de 60 segundos entre ejecuciones de la tarea en segundo plano.

**Conjuntos de réplicas** 

En los miembros del [conjunto de réplicas](https://docs.mongodb.com/manual/reference/glossary/#std-term-replica-set) , el subproceso de fondo TTL _solo_ elimina documentos cuando un miembro está en estado [primario](https://docs.mongodb.com/manual/reference/glossary/#std-term-primary) . El subproceso de fondo TTL está inactivo cuando un miembro está en estado [secundario](https://docs.mongodb.com/manual/reference/glossary/#std-term-secondary) . [Los](https://docs.mongodb.com/manual/reference/glossary/#std-term-secondary) miembros [secundarios](https://docs.mongodb.com/manual/reference/glossary/#std-term-secondary) replican las operaciones de eliminación desde el primario.

#### Soporte para consultas  <a id="support-for-queries"></a>

Un índice TTL admite consultas de la misma manera que lo hacen los índices que no son TTL.

### Restricciones  <a id="restrictions"></a>

* Los índices TTL son índices de un solo campo. [Los índices compuestos](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) no admiten TTL e ignoran la `expireAfterSeconds`opción.
* El `_id`campo no admite índices TTL.
* No puede crear un índice TTL en una [colección limitada](https://docs.mongodb.com/manual/core/capped-collections/) porque MongoDB no puede eliminar documentos de una colección limitada.
* No se puede utilizar [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)para cambiar el valor `expireAfterSeconds`de un índice existente. En su lugar, utilice el [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)comando de la base de datos junto con el [`index`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-collflag-index)indicador de colección. De lo contrario, para cambiar el valor de la opción de un índice existente, primero debe eliminar el índice y volver a crearlo.
* Si ya existe un índice de campo único que no es TTL para un campo, no puede crear un índice TTL en el mismo campo, ya que no puede crear índices que tengan la misma especificación de clave y difieran solo por las opciones. Para cambiar un índice de campo único que no sea TTL a un índice TTL, primero debe eliminar el índice y volver a crearlo con la `expireAfterSeconds`opción.

