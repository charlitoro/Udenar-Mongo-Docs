# Acerca de las transacciones

 En MongoDB, una operación en un solo documento es atómica. Debido a que puede utilizar matrices y documentos incrustados para capturar relaciones entre datos en una estructura de documento único en lugar de normalizar en varios documentos y colecciones, esta atomicidad de documento único evita la necesidad de transacciones de documentos múltiples para muchos casos prácticos de uso.

Para situaciones que requieren atomicidad de lecturas y escrituras en múltiples documentos \(en una o varias colecciones\), MongoDB admite transacciones de múltiples documentos. Con transacciones distribuidas, las transacciones se pueden utilizar en múltiples operaciones, colecciones, bases de datos, documentos y fragmentos.

### API de transacciones  <a id="transactions-api"></a>

➤ Utilice el menú desplegable **Seleccione su idioma** en la parte superior derecha para configurar el idioma del siguiente ejemplo.

Este ejemplo destaca los componentes clave de la API de transacciones.

El ejemplo utiliza la nueva API de devolución de llamada para trabajar con transacciones, que inicia una transacción, ejecuta las operaciones especificadas y confirma \(o aborta en caso de error\). La nueva API de devolución de llamada también incorpora lógica de reintento para `TransientTransactionError`o `UnknownTransactionCommitResult`cometer errores.IMPORTANTE

* _Recomendadas_ . Utilice el controlador de MongoDB actualizado para la versión de su implementación de MongoDB. Para transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\), los clientes **deben** usar controladores MongoDB actualizados para MongoDB 4.2.
* Al utilizar los controladores, cada operación de la transacción **debe** estar asociada con la sesión \(es decir, pasar la sesión a cada operación\).
* Las operaciones en el uso de transacciones [de lectura preocupación a nivel de transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern) , [la preocupación de escritura a nivel de transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) , y [el nivel de transacción leen preferencia](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference) .
* En MongoDB 4.2 y versiones anteriores, no puede crear colecciones en transacciones. Las operaciones de escritura que dan como resultado inserciones de documentos \(por ejemplo, `insert`operaciones de actualización con `upsert: true`\) deben estar en colecciones **existentes** si se ejecutan dentro de transacciones.
* A partir de MongoDB 4.4, puede crear colecciones en transacciones de forma implícita o explícita. Sin embargo, debe utilizar los controladores MongoDB actualizados a la versión 4.4. Consulte [Crear colecciones e índices en una transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) para obtener más detalles.

```text
static bool
with_transaction_example (bson_error_t *error)
{
   mongoc_client_t *client = NULL;
   mongoc_write_concern_t *wc = NULL;
   mongoc_read_concern_t *rc = NULL;
   mongoc_read_prefs_t *rp = NULL;
   mongoc_collection_t *coll = NULL;
   bool success = false;
   bool ret = false;
   bson_t *doc = NULL;
   bson_t *insert_opts = NULL;
   mongoc_client_session_t *session = NULL;
   mongoc_transaction_opt_t *txn_opts = NULL;

   /* For a replica set, include the replica set name and a seedlist of the
    * members in the URI string; e.g.
    * uri_repl = "mongodb://mongodb0.example.com:27017,mongodb1.example.com:" \
    *    "27017/?replicaSet=myRepl";
    * client = mongoc_client_new (uri_repl);
    * For a sharded cluster, connect to the mongos instances; e.g.
    * uri_sharded =
    * "mongodb://mongos0.example.com:27017,mongos1.example.com:27017/";
    * client = mongoc_client_new (uri_sharded);
    */

   client = get_client ();

   /* Prereq: Create collections. */
   wc = mongoc_write_concern_new ();
   mongoc_write_concern_set_wmajority (wc, 1000);
   insert_opts = bson_new ();
   mongoc_write_concern_append (wc, insert_opts);
   coll = mongoc_client_get_collection (client, "mydb1", "foo");
   doc = BCON_NEW ("abc", BCON_INT32 (0));
   ret = mongoc_collection_insert_one (
      coll, doc, insert_opts, NULL /* reply */, error);
   if (!ret) {
      goto fail;
   }
   bson_destroy (doc);
   mongoc_collection_destroy (coll);
   coll = mongoc_client_get_collection (client, "mydb2", "bar");
   doc = BCON_NEW ("xyz", BCON_INT32 (0));
   ret = mongoc_collection_insert_one (
      coll, doc, insert_opts, NULL /* reply */, error);
   if (!ret) {
      goto fail;
   }

   /* Step 1: Start a client session. */
   session = mongoc_client_start_session (client, NULL /* opts */, error);
   if (!session) {
      goto fail;
   }

   /* Step 2: Optional. Define options to use for the transaction. */
   txn_opts = mongoc_transaction_opts_new ();
   rp = mongoc_read_prefs_new (MONGOC_READ_PRIMARY);
   rc = mongoc_read_concern_new ();
   mongoc_read_concern_set_level (rc, MONGOC_READ_CONCERN_LEVEL_LOCAL);
   mongoc_transaction_opts_set_read_prefs (txn_opts, rp);
   mongoc_transaction_opts_set_read_concern (txn_opts, rc);
   mongoc_transaction_opts_set_write_concern (txn_opts, wc);

   /* Step 3: Use mongoc_client_session_with_transaction to start a transaction,
    * execute the callback, and commit (or abort on error). */
   ret = mongoc_client_session_with_transaction (
      session, callback, txn_opts, NULL /* ctx */, NULL /* reply */, error);
   if (!ret) {
      goto fail;
   }

   success = true;
fail:
   bson_destroy (doc);
   mongoc_collection_destroy (coll);
   bson_destroy (insert_opts);
   mongoc_read_concern_destroy (rc);
   mongoc_read_prefs_destroy (rp);
   mongoc_write_concern_destroy (wc);
   mongoc_transaction_opts_destroy (txn_opts);
   mongoc_client_session_destroy (session);
   mongoc_client_destroy (client);
   return success;
}

/* Define the callback that specifies the sequence of operations to perform
 * inside the transactions. */
static bool
callback (mongoc_client_session_t *session,
          void *ctx,
          bson_t **reply,
          bson_error_t *error)
{
   mongoc_client_t *client = NULL;
   mongoc_collection_t *coll = NULL;
   bson_t *doc = NULL;
   bool success = false;
   bool ret = false;

   client = mongoc_client_session_get_client (session);
   coll = mongoc_client_get_collection (client, "mydb1", "foo");
   doc = BCON_NEW ("abc", BCON_INT32 (1));
   ret =
      mongoc_collection_insert_one (coll, doc, NULL /* opts */, *reply, error);
   if (!ret) {
      goto fail;
   }
   bson_destroy (doc);
   mongoc_collection_destroy (coll);
   coll = mongoc_client_get_collection (client, "mydb2", "bar");
   doc = BCON_NEW ("xyz", BCON_INT32 (999));
   ret =
      mongoc_collection_insert_one (coll, doc, NULL /* opts */, *reply, error);
   if (!ret) {
      goto fail;
   }

   success = true;
fail:
   mongoc_collection_destroy (coll);
   bson_destroy (doc);
   return success;
}
```

**CONSEJO**:

Para ver un ejemplo en [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell, consulte [`mongo`Ejemplo de shell](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-txn-mongo-shell-example) .

### Transacciones y atomicidad  <a id="transactions-and-atomicity"></a>

_**NOTA**_

_Transacciones distribuidas y transacciones de varios documentos_

_A partir de MongoDB 4.2, los dos términos son sinónimos. Las transacciones distribuidas se refieren a transacciones de varios documentos en grupos fragmentados y conjuntos de réplicas. Las transacciones de múltiples documentos \(ya sea en clústeres fragmentados o conjuntos de réplicas\) también se conocen como transacciones distribuidas a partir de MongoDB 4.2._

Para situaciones que requieren atomicidad de lecturas y escrituras en múltiples documentos \(en una o varias colecciones\), MongoDB admite transacciones de múltiples documentos:

* **En la versión 4.0** , MongoDB admite transacciones de múltiples documentos en conjuntos de réplicas.
* **En la versión 4.2** , MongoDB introduce transacciones distribuidas, que agrega soporte para transacciones de múltiples documentos en clústeres fragmentados e incorpora el soporte existente para transacciones de múltiples documentos en conjuntos de réplicas.

  Para usar transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\), los clientes **deben** usar controladores de MongoDB actualizados para MongoDB 4.2.

Las transacciones de múltiples documentos son atómicas \(es decir, proporcionan una propuesta de "todo o nada"\):

* Cuando se confirma una transacción, todos los cambios de datos realizados en la transacción se guardan y son visibles fuera de la transacción. Es decir, una transacción no confirmará algunos de sus cambios mientras revierte otros.

  Hasta que se confirma una transacción, los cambios de datos realizados en la transacción no son visibles fuera de la transacción.

  Sin embargo, cuando una transacción escribe en varios fragmentos, no todas las operaciones de lectura externas deben esperar a que el resultado de la transacción confirmada sea visible en los fragmentos. Por ejemplo, si se confirma una transacción y la escritura 1 es visible en el fragmento A pero la escritura 2 aún no es visible en el fragmento B, una lectura externa en caso de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)puede leer los resultados de la escritura 1 sin ver la escritura 2.

* Cuando una transacción se anula, todos los cambios de datos realizados en la transacción se descartan sin que se hagan visibles. Por ejemplo, si falla alguna operación en la transacción, la transacción se cancela y todos los cambios de datos realizados en la transacción se descartan sin que nunca sean visibles.

_**IMPORTANTE**_

_En la mayoría de los casos, la transacción de múltiples documentos genera un mayor costo de rendimiento que la escritura de un solo documento, y la disponibilidad de transacciones de múltiples documentos no debe reemplazar el diseño de esquemas efectivo. Para muchos escenarios, el_ [_modelo de datos desnormalizados \(documentos y matrices incrustados\)_](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) _seguirá siendo óptimo para sus datos y casos de uso. Es decir, para muchos escenarios, modelar sus datos de manera adecuada minimizará la necesidad de transacciones de múltiples documentos._

_Para consideraciones adicionales sobre el uso de transacciones \(como el límite de tiempo de ejecución y el límite de tamaño del registro de operaciones\), consulte también_ [_Consideraciones de producción_](https://docs.mongodb.com/manual/core/transactions-production-consideration/) _._

_**CONSEJO**:_

\_\_[_Lecturas externas durante la confirmación_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-prod-consideration-outside-reads)\_\_

### Transacciones y operaciones  <a id="transactions-and-operations"></a>

Las transacciones distribuidas se pueden usar en múltiples operaciones, colecciones, bases de datos, documentos y, a partir de MongoDB 4.2, fragmentos.

Para transacciones:

* Puede especificar operaciones de lectura / escritura \(CRUD\) en colecciones **existentes** . Para obtener una lista de operaciones CRUD, consulte [Operaciones CRUD](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud) .
* Cuando usa la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` o superior, puede crear colecciones e índices en las transacciones. Para obtener más información, consulte [Crear colecciones e índices en una transacción.](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)
* Las colecciones utilizadas en una transacción pueden estar en diferentes bases de datos.

_**NOTA**_

_No puede crear nuevas colecciones en transacciones de escritura entre particiones. Por ejemplo, si escribe en una colección existente en un fragmento e implícitamente crea una colección en un fragmento diferente, MongoDB no puede realizar ambas operaciones en la misma transacción._

* No puede escribir en colecciones [limitadas](https://docs.mongodb.com/manual/core/capped-collections/) . \(A partir de MongoDB 4.2\)
* No se puede leer / escribir en colecciones en las `config`, `admin`o `local`bases de datos.
* No puede escribir en `system.*`colecciones.
* No puede devolver el plan de consulta de la operación admitida \(es decir `explain`\).
* Para los cursores creados fuera de una transacción, no puede llamar [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)dentro de la transacción.
* Para los cursores creados en una transacción, no puede llamar [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)fuera de la transacción.
* A partir de MongoDB 4.2, no puede especificar [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors)como primera operación en una [transacción](https://docs.mongodb.com/manual/core/transactions/) .

Para obtener una lista de operaciones no admitidas en transacciones, consulte [Operaciones restringidas](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted) .

_**CONSEJO:**_

_Al crear o eliminar una colección inmediatamente antes de iniciar una transacción, si se accede a la colección dentro de la transacción, emita la operación de creación o eliminación con preocupación de escritura_ [_`"majority"`_](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)_para garantizar que la transacción pueda adquirir los bloqueos necesarios._

_**CONSEJO**:_

\_\_[_Referencia de transacciones y operaciones_](https://docs.mongodb.com/manual/core/transactions-operations/)\_\_

#### Crear colecciones e índices en una transacción  <a id="create-collections-and-indexes-in-a-transaction"></a>

A partir de MongoDB 4.4 con la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` , puede crear colecciones e índices dentro de una [transacción de varios documentos, a](https://docs.mongodb.com/manual/core/transactions/) menos que la transacción sea una transacción de escritura entre fragmentos. Con `"4.2"`o menos, las operaciones que afectan el catálogo de la base de datos, como crear o eliminar una colección o un índice, **no** se **permiten** en las transacciones.

Al crear una colección dentro de una transacción:

* Puede [crear implícitamente una colección](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) , como con:
  * una [operación de inserción](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) contra una colección no existente, o
  * una [operación update / findAndModify](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) con `upsert: true` contra una colección no existente.
* Puede [crear explícitamente una colección](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) utilizando el [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) comando o su ayudante [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection).

Al [crear un índice dentro de una transacción ](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit)[\[ 1 \]](https://docs.mongodb.com/manual/core/transactions/#footnote-create-existing-index) , el índice a crear debe estar en:

* una colección inexistente. La colección se crea como parte de la operación.
* una nueva colección vacía creada anteriormente en la misma transacción.

| \[ [1](https://docs.mongodb.com/manual/core/transactions/#ref-create-existing-index-id1) \] | También puede ejecutar [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)y [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes)en índices existentes para verificar su existencia. Estas operaciones se devuelven correctamente sin crear el índice. |
| :--- | :--- |


**Restricciones** 

* No puede crear nuevas colecciones en transacciones de escritura entre particiones. Por ejemplo, si escribe en una colección existente en un fragmento e implícitamente crea una colección en un fragmento diferente, MongoDB no puede realizar ambas operaciones en la misma transacción.
* Para la creación explícita de una colección o un índice dentro de una transacción, el nivel de preocupación de lectura de la transacción debe ser [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-). La creación explícita se realiza a través de:

  | Mando | Método |
  | :--- | :--- |
  | [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) | [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) |
  | [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) | [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) |

* El parámetro [`shouldMultiDocTxnCreateCollectionAndIndexes`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.shouldMultiDocTxnCreateCollectionAndIndexes)debe ser `true`\(el predeterminado\). Cuando establezca el parámetro para un clúster fragmentado, establezca el parámetro en todos los fragmentos.

_**CONSEJO**:_

\_\_[_Operaciones restringidas_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-ops-restricted)\_\_

#### Operación de conteo  <a id="count-operation"></a>

Para realizar una operación de recuento dentro de una transacción, use la [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#mongodb-pipeline-pipe.-count)etapa de agregación o la etapa de agregación [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)\(con una [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)expresión\).

Los controladores MongoDB compatibles con las características 4.0 proporcionan una API de nivel de colección `countDocuments(filter, options)`como método auxiliar que usa la expresión [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)con una [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)para realizar un recuento. Los controladores 4.0 han desaprobado la `count()`API.

A partir de MongoDB 4.0.3, el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell proporciona el [`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#mongodb-method-db.collection.countDocuments)método auxiliar que usa [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)con una [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)expresión para realizar un recuento.

#### Operación distinta  <a id="distinct-operation"></a>

Para realizar una operación distinta dentro de una transacción:

* Para las colecciones no fragmentadas, puede usar el [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct)método / el [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)comando, así como la canalización de agregación con el [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)escenario.
* Para las colecciones fragmentadas, no puede utilizar el [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct)método ni el [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)comando.

  Para encontrar los valores distintos para una colección fragmentada, use la canalización de agregación con el [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)escenario en su lugar. Por ejemplo:

  * En lugar de `db.coll.distinct("x")`, usa:

```text
db.coll.aggregate([
   { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
   { $project: { _id: 0 } }
])
```

* En lugar de `db.coll.distinct("x", { status: "A" })`, use:

  ```text
  db.coll.aggregate([
     { $match: { status: "A" } },
     { $group: { _id: null, distinctValues: { $addToSet: "$x" } } },
     { $project: { _id: 0 } }
  ])
  ```

La canalización devuelve un cursor a un documento:

```text
{ "distinctValues" : [ 2, 3, 1 ] }
```

Itere el cursor para acceder al documento de resultados.

#### Operaciones informativas  <a id="informational-operations"></a>

Comandos informativos, tales como [`hello`](https://docs.mongodb.com/manual/reference/command/hello/#mongodb-dbcommand-dbcmd.hello), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#mongodb-dbcommand-dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#mongodb-dbcommand-dbcmd.connectionStatus)\(y sus métodos de ayuda\) se permiten en las transacciones; sin embargo, no pueden ser la primera operación de la transacción.

#### Operaciones restringidas  <a id="restricted-operations"></a>

_Modificado en la versión 4.4_ .

Las siguientes operaciones no están permitidas en transacciones:

* Operaciones que afectan el catálogo de la base de datos, como crear o eliminar una colección o un índice cuando se usa la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.2"` o una [versión anterior](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) . Con fcv `"4.4"`o superior, puede crear colecciones e índices en transacciones a menos que la transacción sea una transacción de escritura entre fragmentos. Para obtener más información, consulte [Crear colecciones e índices en una transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) .
* Creación de nuevas colecciones en transacciones de escritura entre fragmentos. Por ejemplo, si escribe en una colección existente en un fragmento e implícitamente crea una colección en un fragmento diferente, MongoDB no puede realizar ambas operaciones en la misma transacción.
* [Creación explícita de colecciones](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) , por ejemplo, [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection)método e índices, por ejemplo, [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes)y [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)métodos, cuando se utiliza un nivel de preocupación de lectura diferente a [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-).
* Las [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections)y los [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes) comandos y sus métodos de ayuda.
* Otras operaciones no CRUD y no informativos, como [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count), etc, y sus ayudantes.

_**CONSEJO**_

_Ver también:_

* \_\_[_Operaciones y transacciones de DDL pendientes_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)\_\_
* \_\_[_Referencia de transacciones y operaciones_](https://docs.mongodb.com/manual/core/transactions-operations/)\_\_

### Transacciones y sesiones  <a id="transactions-and-sessions"></a>

* Las transacciones están asociadas con una sesión; es decir, inicia una transacción para una sesión.
* En cualquier momento, puede tener como máximo una transacción abierta para una sesión.
* Al utilizar los controladores, cada operación de la transacción debe estar asociada con la sesión. Consulte la documentación específica de su controlador para obtener más detalles.
* Si una sesión finaliza y tiene una transacción abierta, la transacción se cancela.

### Leer preocupación / Escribir preocupación / Leer preferencia  <a id="read-concern-write-concern-read-preference"></a>

#### Transacciones y preferencia de lectura  <a id="transactions-and-read-preference"></a>

Las operaciones en una transacción utilizan la [preferencia de lectura a](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference) nivel de transacción .

Con los controladores, puede establecer la [preferencia de lectura a](https://docs.mongodb.com/manual/core/read-preference/#std-label-replica-set-read-preference) nivel de transacción al inicio de la transacción:

* Si la preferencia de lectura a nivel de transacción no está establecida, la transacción utiliza la preferencia de lectura a nivel de sesión.
* Si la preferencia de lectura a nivel de transacción y a nivel de sesión no está establecida, la transacción utiliza la preferencia de lectura a nivel de cliente. De forma predeterminada, la preferencia de lectura a nivel de cliente es [`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary).

[Las transacciones de varios documentos](https://docs.mongodb.com/manual/core/transactions/) que contienen operaciones de lectura deben utilizar la preferencia de lectura [`primary`](https://docs.mongodb.com/manual/core/read-preference/#mongodb-readmode-primary). Todas las operaciones en una transacción determinada deben enrutarse al mismo miembro.

#### Transacciones y preocupación por la lectura  <a id="transactions-and-read-concern"></a>

Las operaciones en una transacción utilizan la [preocupación de lectura a](https://docs.mongodb.com/manual/reference/read-concern/) nivel de transacción . Es decir, cualquier preocupación de lectura establecida en el nivel de la colección y la base de datos se ignora dentro de la transacción.

Puede establecer la [preocupación de lectura a](https://docs.mongodb.com/manual/reference/read-concern/) nivel de transacción al inicio de la transacción.

* Si la preocupación de lectura a nivel de transacción no está configurada, la preocupación de lectura a nivel de transacción toma como valor predeterminado la preocupación de lectura a nivel de sesión.
* Si la preocupación de lectura a nivel de transacción y de sesión no está configurada, la preocupación de lectura a nivel de transacción se establece de forma predeterminada en la preocupación de lectura a nivel de cliente. De forma predeterminada, la preocupación de lectura a nivel de cliente es [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)para las lecturas del primario. Ver también:
  * [Transacciones y preferencia de lectura](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference)
  * [Preocupaciones de lectura / escritura de MongoDB predeterminadas](https://docs.mongodb.com/manual/reference/mongodb-defaults/)

Las transacciones admiten los siguientes niveles de preocupación de lectura:

**"local"**

* La preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)devuelve los datos más recientes disponibles del nodo, pero se puede revertir.
* Para transacciones en clústeres fragmentados, la [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)preocupación por la lectura no puede garantizar que los datos provengan de la misma vista de instantánea en todos los fragmentos. Si se requiere el aislamiento de instantáneas, utilice la [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot)preocupación de lectura.
* A partir de MongoDB 4.4, con la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` o superior, puede [crear colecciones e índices](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) dentro de una transacción. Si crea [explícitamente](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) una colección o un índice, la transacción debe utilizar la preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-). [La](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) creación [implícita](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) de una colección puede utilizar cualquiera de las preocupaciones de lectura disponibles para las transacciones.

**"majority"**

* El problema de lectura [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)devuelve datos que han sido reconocidos por la mayoría de los miembros del conjunto de réplicas \(es decir, los datos no se pueden deshacer\) **si** la transacción se confirma con el [problema de escritura "mayoría"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) .
* Si la transacción no usa la [preocupación de escritura "mayoría"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) para la confirmación, la [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)preocupación de lectura **no** proporciona garantías de que las operaciones de lectura lean los datos confirmados por mayoría.
* Para transacciones en clústeres fragmentados, la [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)preocupación por la lectura no puede garantizar que los datos provengan de la misma vista de instantánea en todos los fragmentos. Si se requiere el aislamiento de instantáneas, utilice la [`"snapshot"`](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern-snapshot)preocupación de lectura.

**"snapshot"**

* La preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)devuelve datos de una instantánea de los datos comprometidos mayoritarios **si** la transacción se confirma con la [preocupación de escritura "mayoría"](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) .
* Si la transacción no usa ["mayoría" de preocupación de escritura](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) para la confirmación, la [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)preocupación de lectura **no** proporciona **ninguna** garantía de que las operaciones de lectura usen una instantánea de los datos confirmados por mayoría.
* Para transacciones en clústeres fragmentados, la [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)vista de los datos **se** sincroniza entre fragmentos.

#### Transacciones y preocupación de escritura  <a id="transactions-and-write-concern"></a>

Las transacciones utilizan la [preocupación de escritura a](https://docs.mongodb.com/manual/reference/write-concern/) nivel de transacción para confirmar las operaciones de escritura. Las operaciones de escritura dentro de las transacciones deben emitirse sin una especificación explícita de preocupación de escritura y deben utilizar la preocupación de escritura predeterminada. En el momento de la confirmación, las escrituras se confirman utilizando la preocupación de escritura a nivel de transacción.

_**CONSEJO**_

_No establezca explícitamente la preocupación de escritura para las operaciones de escritura individuales dentro de una transacción. Establecer preocupaciones de escritura para las operaciones de escritura individuales dentro de una transacción da como resultado un error._

Puede establecer la [preocupación de escritura a](https://docs.mongodb.com/manual/reference/write-concern/) nivel de transacción al inicio de la transacción:

* Si la preocupación de escritura a nivel de transacción no está configurada, la preocupación de escritura a nivel de transacción toma como valor predeterminado la preocupación de escritura a nivel de sesión para la confirmación.
* Si la preocupación de escritura a nivel de transacción y la preocupación de escritura a nivel de sesión no están configuradas, la preocupación de escritura a nivel de transacción se establece por defecto en la preocupación de escritura a nivel de cliente. De forma predeterminada, la preocupación de escritura a nivel de cliente es [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-). Consulte también [Preocupaciones de lectura / escritura de MongoDB predeterminadas](https://docs.mongodb.com/manual/reference/mongodb-defaults/) .

Las transacciones admiten todos los valores de [w de](https://docs.mongodb.com/manual/reference/write-concern/#std-label-wc-w) preocupación de escritura , que incluyen:

**w: 1**

* Escribir [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)reconocimiento devuelve inquietud después de que la confirmación se haya aplicado al primario.

_**IMPORTANTE**_

_Cuando se compromete con_ [_`w: 1`_](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)_, su transacción se puede_ [_revertir si hay una conmutación por error_](https://docs.mongodb.com/manual/core/replica-set-rollbacks/) _._

* Cuando se compromete con [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)preocupación de escritura, la preocupación de [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)lectura a nivel de transacción **no** proporciona garantías de que las operaciones de lectura en la transacción lean datos confirmados por mayoría.
* Cuando se compromete con [`w: 1`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-number-)preocupación de escritura, la preocupación de [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)lectura a nivel de transacción **no** proporciona **ninguna** garantía de que las operaciones de lectura en la transacción usen una instantánea de los datos comprometidos por mayoría.

**w: "majority"**

* Escribir [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)reconocimiento de declaraciones de inquietud después de que el compromiso se haya aplicado a una mayoría \(M\) de miembros votantes; es decir, el compromiso se ha aplicado a las primarias y a las secundarias de votación \(M-1\).
* Cuando se compromete con [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) preocupación de escritura, la preocupación de [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)lectura a nivel de transacción garantiza que las operaciones hayan leído datos confirmados por mayoría. Para transacciones en clústeres fragmentados, esta vista de los datos comprometidos por la mayoría no se sincroniza entre fragmentos.
* Cuando se compromete con [`w: "majority"`](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-) preocupación de escritura, la preocupación de [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)lectura a nivel de transacción garantiza que las operaciones tengan una instantánea sincronizada de datos comprometidos por mayoría.

_**NOTA**_

_Independientemente del_ [_problema de escritura especificado para la transacción_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) _, la operación de confirmación para una transacción de clúster fragmentado incluye algunas partes que utilizan el `{w: "majority", j: true}`problema de escritura._

### Información general  <a id="general-information"></a>

#### Consideraciones de producción  <a id="production-considerations"></a>

Para conocer varias consideraciones de producción con el uso de transacciones, consulte [Consideraciones de producción](https://docs.mongodb.com/manual/core/transactions-production-consideration/) . Además, o clústeres fragmentados, consulte también [Consideraciones de producción \(clústeres fragmentados\)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/) .

#### Árbitros  <a id="arbiters"></a>

Las transacciones cuyas operaciones de escritura abarcan varios fragmentos generarán errores y se cancelarán si alguna operación de transacción lee o escribe en un fragmento que contiene un árbitro.

Consulte también [Mayoría de preocupación de lectura deshabilitada](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-disabled-rc-majority) para conocer las restricciones de transacción en fragmentos que tienen la mayoría de preocupación de lectura deshabilitada.

#### Mayoría de preocupación de lectura deshabilitada  <a id="disabled-read-concern-majority"></a>

Un conjunto de réplicas de PSA \(árbitro principal-secundario-árbitro\) de 3 miembros o un clúster fragmentado con fragmentos de PSA de 3 miembros puede haber deshabilitado la mayoría de interés de lectura \( [`--enableMajorityReadConcern false`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern)o [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern)\)

**En clústeres fragmentados,**

* Si una transacción involucra un fragmento que ha [inhabilitado la preocupación de lectura "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) , no puede utilizar la preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)para la transacción. Solo puede usar la preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)o [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)para la transacción. Si usa la preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-), la transacción se produce un error y se cancela.
* ```text
  readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false.
  ```
* Las transacciones cuyas operaciones de escritura abarcan varios fragmentos generarán errores y se abortarán si alguna de las operaciones de lectura o escritura de la transacción involucra un fragmento que ha deshabilitado el problema de lectura `"majority"`.

**En el juego de réplicas,**

Puede especificar la preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)o [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-), o [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)incluso en el conjunto de réplicas tiene [desactivado lectura preocupación "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) .

Sin embargo, si planea realizar la transición a un clúster fragmentado con fragmentos mayoritarios de preocupación de lectura deshabilitados, es posible que desee evitar el uso de la preocupación de lectura `"snapshot"`.

_**CONSEJO**_

_Para comprobar si la preocupación de lectura "mayoría" está deshabilitada, puede ejecutar_ [_`db.serverStatus()`_](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus)_las_ [_`mongod`_](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)_instancias y marcar el_ [_`storageEngine.supportsCommittedReads`_](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads) _campo. Si `false`, lea la preocupación "mayoría" está deshabilitada._

Para obtener más información, ver [3-Miembro Primario-Secundario-Árbitro Arquitectura](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-psa) y [tres miembros Primario-Secundario-árbitro Fragmentos](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-psa) .  


#### Restricción de configuración de fragmentos  <a id="shard-configuration-restriction"></a>

No puede ejecutar transacciones en un clúster fragmentado que tiene un fragmento con [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)establecido en `false` \(como un fragmento con un miembro votante que usa el [motor de almacenamiento en memoria](https://docs.mongodb.com/manual/core/inmemory/) \).

_**NOTA**_

_Independientemente del_ [_problema de escritura especificado para la transacción_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) _, la operación de confirmación para una transacción de clúster fragmentado incluye algunas partes que utilizan el `{w: "majority", j: true}`problema de escritura._

#### Diagnóstico  <a id="diagnostics"></a>

MongoDB proporciona varias métricas de transacciones:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Fuente</th>
      <th style="text-align:left">Devoluciones</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus"><code>db.serverStatus()</code></a> m&#xE9;todo
        <a
        href="https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-dbcommand-dbcmd.serverStatus"><code>serverStatus</code>
          </a>mando</td>
      <td style="text-align:left">Devuelve m&#xE9;tricas de <a href="https://docs.mongodb.com/manual/reference/command/serverStatus/#std-label-server-status-transactions">transacciones</a> .</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-pipeline-pipe.-currentOp"><code>$currentOp</code></a> canalizaci&#xF3;n
        de agregaci&#xF3;n</td>
      <td style="text-align:left">
        <p>Devoluciones:</p>
        <ul>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.transaction"><code>$currentOp.transaction</code></a> si
            una operaci&#xF3;n es parte de una transacci&#xF3;n.</li>
          <li>Informaci&#xF3;n sobre <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#std-label-currentOp-stage-idleSessions">sesiones inactivas</a> que
            mantienen bloqueos como parte de una transacci&#xF3;n.</li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.twoPhaseCommitCoordinator"><code>$currentOp.twoPhaseCommitCoordinator</code></a> m&#xE9;tricas
            para transacciones fragmentadas que invocan escrituras en m&#xFA;ltiples
            fragmentos.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.currentOp/#mongodb-method-db.currentOp"><code>db.currentOp()</code></a> m&#xE9;todo
        <a
        href="https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-dbcommand-dbcmd.currentOp"><code>currentOp</code>
          </a>mando</td>
      <td style="text-align:left">
        <p>Devoluciones:</p>
        <ul>
          <li><a href="https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-data-currentOp.transaction"><code>currentOp.transaction</code></a> si
            una operaci&#xF3;n es parte de una transacci&#xF3;n.</li>
          <li><a href="https://docs.mongodb.com/manual/reference/command/currentOp/#mongodb-data-currentOp.twoPhaseCommitCoordinator"><code>currentOp.twoPhaseCommitCoordinator</code></a> m&#xE9;tricas
            para transacciones fragmentadas que invocan escrituras en m&#xFA;ltiples
            fragmentos.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod"><code>mongod</code></a>y
        <a
        href="https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos"><code>mongos</code>
          </a>registrar mensajes</td>
      <td style="text-align:left">Incluye informaci&#xF3;n sobre transacciones lentas (es decir, transacciones
        que superan el <a href="https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs"><code>operationProfiling.slowOpThresholdMs</code></a> umbral)
        en el <a href="https://docs.mongodb.com/manual/reference/log-messages/#mongodb-data-TXN"><code>TXN</code></a>componente
        de registro.</td>
    </tr>
  </tbody>
</table>

#### Versión de compatibilidad de funciones \(FCV\)  <a id="feature-compatibility-version--fcv-"></a>

Para usar transacciones, [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) para todos los miembros de la implementación debe ser al menos:

| Despliegue | Mínimo `featureCompatibilityVersion` |
| :--- | :--- |
| Conjunto de réplicas | `4.0` |
| Clúster fragmentado | `4.2` |

Para verificar el fCV de un miembro, conéctese al miembro y ejecute el siguiente comando:

```text
db.adminCommand( { getParameter: 1, featureCompatibilityVersion: 1 } )
```

Para obtener más información, consulte la [`setFeatureCompatibilityVersion`](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#mongodb-dbcommand-dbcmd.setFeatureCompatibilityVersion)página de referencia.

#### Motores de almacenamiento  <a id="storage-engines"></a>

A partir de MongoDB 4.2, [las transacciones de varios documentos](https://docs.mongodb.com/manual/core/transactions/) se admiten en conjuntos de réplicas y clústeres fragmentados donde:

* el primario usa el motor de almacenamiento WiredTiger, y
* los miembros secundarios utilizan el motor de almacenamiento WiredTiger o los motores de almacenamiento [en memoria](https://docs.mongodb.com/manual/core/inmemory/) .

En MongoDB 4.0, solo los conjuntos de réplicas que utilizan el motor de almacenamiento WiredTiger admitían transacciones.

_**NOTA**_

_No puede ejecutar transacciones en un clúster fragmentado que tiene un fragmento con_ [_`writeConcernMajorityJournalDefault`_](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)_establecido en `false`, como un fragmento con un miembro votante que usa el_ [_motor de almacenamiento en memoria_](https://docs.mongodb.com/manual/core/inmemory/) _._

### Temas de transacciones adicionales  <a id="additional-transactions-topics"></a>

* [API de controladores](https://docs.mongodb.com/manual/core/transactions-in-applications/)
* [Consideraciones de producción](https://docs.mongodb.com/manual/core/transactions-production-consideration/)
* [Consideraciones de producción \(clústeres fragmentados\)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)
* [Transacciones y operaciones](https://docs.mongodb.com/manual/core/transactions-operations/)
* Para obtener más información sobre cuándo usar transacciones y si son compatibles con su caso de uso, consulte [¿Son las transacciones adecuadas para usted? ](https://www.mongodb.com/presentations/are-transactions-right-for-you-) presentación de **MongoDB.live 2020** .

