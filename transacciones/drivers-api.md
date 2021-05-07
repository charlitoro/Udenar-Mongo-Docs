# Drivers API

### API de devolución de llamada frente a API central  <a id="callback-api-vs-core-api"></a>

La [API de devolución de llamada](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-txn-callback-api) :

* Inicia una transacción, ejecuta las operaciones especificadas y confirma \(o aborta en caso de error\).
* Incorpora automáticamente la lógica de manejo de errores para [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error)y [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result).

La [API principal](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-txn-core-api) :

* Requiere una llamada explícita para iniciar la transacción y confirmar la transacción.
* No incorpora la lógica de manejo de errores para [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error)y [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result), y en su lugar proporciona la flexibilidad para incorporar el manejo de errores personalizado para estos errores.

### API de devolución de llamada  <a id="callback-api"></a>

La API de devolución de llamada incorpora lógica:

* Para reintentar la transacción como un todo si la transacción encuentra un [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error).
* Para reintentar la operación de confirmación si la confirmación encuentra un [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result).

#### Ejemplo  <a id="example"></a>

➤ Utilice el menú desplegable **Seleccione su idioma** en la parte superior derecha para establecer el idioma de los ejemplos en esta página.

_**IMPORTANTE**_

* _Recomendadas . Utilice el controlador de MongoDB actualizado para la versión de su implementación de MongoDB. Para transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\), los clientes **deben** usar controladores MongoDB actualizados para MongoDB 4.2._
* _Al utilizar los controladores, cada operación de la transacción **debe** estar asociada con la sesión \(es decir, pasar la sesión a cada operación\)._
* _Las operaciones en el uso de transacciones_ [_de lectura preocupación a nivel de transacción_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern) _,_ [_la preocupación de escritura a nivel de transacción_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) _, y_ [_el nivel de transacción leen preferencia_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-preference) _._
* _En MongoDB 4.2 y versiones anteriores, no puede crear colecciones en transacciones. Las operaciones de escritura que dan como resultado inserciones de documentos \(por ejemplo, `insert`operaciones de actualización con `upsert: true`\) deben estar en colecciones **existentes** si se ejecutan dentro de transacciones._
* _A partir de MongoDB 4.4, puede crear colecciones en transacciones de forma implícita o explícita. Sin embargo, debe utilizar los controladores MongoDB actualizados a la versión 4.4. Consulte_ [_Crear colecciones e índices en una transacción_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) _para obtener más detalles._

El ejemplo utiliza la nueva API de devolución de llamada para trabajar con transacciones, que inicia una transacción, ejecuta las operaciones especificadas y confirma \(o aborta en caso de error\). La nueva API de devolución de llamada incorpora lógica de reintento para [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error)o [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result)cometer errores.

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

### API principal  <a id="core-api"></a>

La API de transacciones principal no incorpora la lógica de reintento para los errores etiquetados:

* [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error). Si una operación en una transacción devuelve un error etiquetado [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error), se puede volver a intentar la transacción en su totalidad.

  Para manejarlo [`"TransientTransactionError"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transient-transaction-error), las aplicaciones deben incorporar explícitamente la lógica de reintento para el error.

* [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result). Si la confirmación devuelve un error etiquetado [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result), se puede reintentar la confirmación.

  Para manejarlo [`"UnknownTransactionCommitResult"`](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-unknown-transaction-commit-result), las aplicaciones deben incorporar explícitamente la lógica de reintento para el error.

#### Ejemplo  <a id="example-1"></a>

➤ Utilice el menú desplegable **Seleccione su idioma** en la parte superior derecha para establecer el idioma de los ejemplos en esta página.

El siguiente ejemplo incorpora lógica para reintentar la transacción en caso de errores transitorios y reintentar la confirmación para un error de confirmación desconocido:

```text
/* takes a session, an out-param for server reply, and out-param for error. */
typedef bool (*txn_func_t) (mongoc_client_session_t *,
                            bson_t *,
                            bson_error_t *);


/* runs transactions with retry logic */
bool
run_transaction_with_retry (txn_func_t txn_func,
                            mongoc_client_session_t *cs,
                            bson_error_t *error)
{
   bson_t reply;
   bool r;

   while (true) {
      /* perform transaction */
      r = txn_func (cs, &reply, error);
      if (r) {
         /* success */
         bson_destroy (&reply);
         return true;
      }

      MONGOC_WARNING ("Transaction aborted: %s", error->message);
      if (mongoc_error_has_label (&reply, "TransientTransactionError")) {
         /* on transient error, retry the whole transaction */
         MONGOC_WARNING ("TransientTransactionError, retrying transaction...");
         bson_destroy (&reply);
      } else {
         /* non-transient error */
         break;
      }
   }

   bson_destroy (&reply);
   return false;
}


/* commit transactions with retry logic */
bool
commit_with_retry (mongoc_client_session_t *cs, bson_error_t *error)
{
   bson_t reply;
   bool r;

   while (true) {
      /* commit uses write concern set at transaction start, see
       * mongoc_transaction_opts_set_write_concern */
      r = mongoc_client_session_commit_transaction (cs, &reply, error);
      if (r) {
         MONGOC_INFO ("Transaction committed");
         break;
      }

      if (mongoc_error_has_label (&reply, "UnknownTransactionCommitResult")) {
         MONGOC_WARNING ("UnknownTransactionCommitResult, retrying commit ...");
         bson_destroy (&reply);
      } else {
         /* commit failed, cannot retry */
         break;
      }
   }

   bson_destroy (&reply);

   return r;
}


/* updates two collections in a transaction and calls commit_with_retry */
bool
update_employee_info (mongoc_client_session_t *cs,
                      bson_t *reply,
                      bson_error_t *error)
{
   mongoc_client_t *client;
   mongoc_collection_t *employees;
   mongoc_collection_t *events;
   mongoc_read_concern_t *rc;
   mongoc_write_concern_t *wc;
   mongoc_transaction_opt_t *txn_opts;
   bson_t opts = BSON_INITIALIZER;
   bson_t *filter = NULL;
   bson_t *update = NULL;
   bson_t *event = NULL;
   bool r;

   bson_init (reply);

   client = mongoc_client_session_get_client (cs);
   employees = mongoc_client_get_collection (client, "hr", "employees");
   events = mongoc_client_get_collection (client, "reporting", "events");

   rc = mongoc_read_concern_new ();
   mongoc_read_concern_set_level (rc, MONGOC_READ_CONCERN_LEVEL_SNAPSHOT);
   wc = mongoc_write_concern_new ();
   mongoc_write_concern_set_w (wc, MONGOC_WRITE_CONCERN_W_MAJORITY);
   txn_opts = mongoc_transaction_opts_new ();
   mongoc_transaction_opts_set_read_concern (txn_opts, rc);
   mongoc_transaction_opts_set_write_concern (txn_opts, wc);

   r = mongoc_client_session_start_transaction (cs, txn_opts, error);
   if (!r) {
      goto done;
   }

   r = mongoc_client_session_append (cs, &opts, error);
   if (!r) {
      goto done;
   }

   filter = BCON_NEW ("employee", BCON_INT32 (3));
   update = BCON_NEW ("$set", "{", "status", "Inactive", "}");
   /* mongoc_collection_update_one will reinitialize reply */
   bson_destroy (reply);
   r = mongoc_collection_update_one (
      employees, filter, update, &opts, reply, error);

   if (!r) {
      goto abort;
   }

   event = BCON_NEW ("employee", BCON_INT32 (3));
   BCON_APPEND (event, "status", "{", "new", "Inactive", "old", "Active", "}");

   bson_destroy (reply);
   r = mongoc_collection_insert_one (events, event, &opts, reply, error);
   if (!r) {
      goto abort;
   }

   r = commit_with_retry (cs, error);

abort:
   if (!r) {
      MONGOC_ERROR ("Aborting due to error in transaction: %s", error->message);
      mongoc_client_session_abort_transaction (cs, NULL);
   }

done:
   mongoc_collection_destroy (employees);
   mongoc_collection_destroy (events);
   mongoc_read_concern_destroy (rc);
   mongoc_write_concern_destroy (wc);
   mongoc_transaction_opts_destroy (txn_opts);
   bson_destroy (&opts);
   bson_destroy (filter);
   bson_destroy (update);
   bson_destroy (event);

   return r;
}


void
example_func (mongoc_client_t *client)
{
   mongoc_client_session_t *cs;
   bson_error_t error;
   bool r;

   cs = mongoc_client_start_session (client, NULL, &error);
   if (!cs) {
      MONGOC_ERROR ("Could not start session: %s", error.message);
      return;
   }

   r = run_transaction_with_retry (update_employee_info, cs, &error);
   if (!r) {
      MONGOC_ERROR ("Could not update employee, permanent error: %s",
                    error.message);
   }

   mongoc_client_session_destroy (cs);
}
```

### Versiones del controlador  <a id="driver-versions"></a>

_Para transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\)_ , los clientes **deben** usar controladores MongoDB actualizados para MongoDB 4.2:

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <ul>
          <li><a href="http://mongoc.org/libmongoc/">C 1.15.0</a>
          </li>
          <li><a href="https://mongodb.github.io/mongo-csharp-driver/">C # 2.9.0</a>
          </li>
          <li><a href="https://godoc.org/go.mongodb.org/mongo-driver/mongo">Ir 1.1</a>
          </li>
        </ul>
      </th>
      <th style="text-align:left">
        <ul>
          <li><a href="https://mongodb.github.io/mongo-java-driver/">Java 3.11.0</a>
          </li>
          <li><a href="https://mongodb.github.io/node-mongodb-native/">Nodo 3.3.0</a>
          </li>
          <li><a href="https://metacpan.org/author/MONGODB">Perl 2.2.0</a>
          </li>
        </ul>
      </th>
      <th style="text-align:left">
        <ul>
          <li><a href="https://api.mongodb.com/pymongo">Python 3.9.0</a>
          </li>
          <li><a href="https://docs.mongodb.com/ruby-driver/current/">Rub&#xED; 2.10.0</a>
          </li>
          <li><a href="https://mongodb.github.io/mongo-scala-driver/">Scala 2.7.0</a>
          </li>
        </ul>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

_Para transacciones en conjuntos de réplicas de MongoDB 4.0_ , los clientes requieren controladores de MongoDB actualizados para MongoDB 4.0 o posterior.

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <ul>
          <li>Java 3.8.0</li>
          <li>Python 3.7.0</li>
          <li>C 1.11.0</li>
        </ul>
      </th>
      <th style="text-align:left">
        <ul>
          <li>C # 2.7</li>
          <li>Nodo 3.1.0</li>
          <li>Ruby 2.6.0</li>
        </ul>
      </th>
      <th style="text-align:left">
        <ul>
          <li>Perl 2.0.0</li>
          <li>PHP (PHPC) 1.5.0</li>
          <li>Scala 2.4.0</li>
        </ul>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

### Manejo de errores de transacción  <a id="transaction-error-handling"></a>

Independientemente del sistema de base de datos, ya sea MongoDB o bases de datos relacionales, las aplicaciones deben tomar medidas para manejar los errores durante las confirmaciones de transacciones e incorporar la lógica de reintento para las transacciones.

#### `"TransientTransactionError"` <a id="-transienttransactionerror-"></a>

Las operaciones de escritura _individuales_ dentro de la transacción no se pueden recuperar, independientemente del valor de [`retryWrites`](https://docs.mongodb.com/manual/reference/connection-string/#mongodb-urioption-urioption.retryWrites). Si una operación encuentra un error [asociado con la etiqueta](https://github.com/mongodb/specifications/blob/master/source/transactions/transactions.rst#error-labels) `"TransientTransactionError"` , como cuando el primario se retira, se puede reintentar la transacción en su totalidad.

* La API de devolución de llamada incorpora lógica de reintento para `"TransientTransactionError"`.
* La API de transacción principal no incorpora lógica de reintento para `"TransientTransactionError"`. Para manejarlo `"TransientTransactionError"`, las aplicaciones deben incorporar explícitamente la lógica de reintento para el error.

#### `"UnknownTransactionCommitResult"` <a id="-unknowntransactioncommitresult-"></a>

Las operaciones de confirmación son [operaciones de escritura recuperables](https://docs.mongodb.com/manual/core/retryable-writes/) . Si la operación de confirmación encuentra un error, los controladores de MongoDB vuelven a intentar la confirmación independientemente del valor de [`retryWrites`](https://docs.mongodb.com/manual/reference/connection-string/#mongodb-urioption-urioption.retryWrites).

Si la operación de confirmación encuentra un error etiquetado `"UnknownTransactionCommitResult"`, la confirmación se puede reintentar.

* La API de devolución de llamada incorpora lógica de reintento para `"UnknownTransactionCommitResult"`.
* La API de transacción principal no incorpora lógica de reintento para `"UnknownTransactionCommitResult"`. Para manejarlo `"UnknownTransactionCommitResult"`, las aplicaciones deben incorporar explícitamente la lógica de reintento para el error.

#### Errores de versión del controlador  <a id="driver-version-errors"></a>

En clústeres fragmentados con varias [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)instancias, la realización de transacciones con controladores actualizados para MongoDB 4.0 \(en lugar de MongoDB 4.2\) fallará y puede generar errores, que incluyen:

_**NOTA**_

_Su controlador puede devolver un error diferente. Consulte la documentación de su controlador para obtener más detalles._

| Código de error | Mensaje de error |
| :--- | :--- |
| 251 | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940 | `cannot commit with no participants` |

_Para transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\)_ , use los controladores de MongoDB actualizados para MongoDB 4.2

### Información adicional  <a id="additional-information"></a>

#### `mongo`Ejemplo de Shell  <a id="mongo-shell-example"></a>

Los siguientes [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)métodos de shell están disponibles para transacciones:

* [`Session.startTransaction()`](https://docs.mongodb.com/manual/reference/method/Session.startTransaction/#mongodb-method-Session.startTransaction)
* [`Session.commitTransaction()`](https://docs.mongodb.com/manual/reference/method/Session.commitTransaction/#mongodb-method-Session.commitTransaction)
* [`Session.abortTransaction()`](https://docs.mongodb.com/manual/reference/method/Session.abortTransaction/#mongodb-method-Session.abortTransaction)

_**NOTA**_

_El_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_ejemplo de shell omite la lógica de reintento y el manejo robusto de errores por simplicidad. Para obtener un ejemplo más práctico de incorporación de transacciones en aplicaciones, consulte_ [_Manejo de errores de transacciones_](https://docs.mongodb.com/manual/core/transactions-in-applications/#std-label-transactions-retry) _._

```text
// Create collections:
db.getSiblingDB("mydb1").foo.insert( {abc: 0}, { writeConcern: { w: "majority", wtimeout: 2000 } } );
db.getSiblingDB("mydb2").bar.insert( {xyz: 0}, { writeConcern: { w: "majority", wtimeout: 2000 } } );
// Start a session.
session = db.getMongo().startSession( { readPreference: { mode: "primary" } } );
coll1 = session.getDatabase("mydb1").foo;
coll2 = session.getDatabase("mydb2").bar;
// Start a transaction
session.startTransaction( { readConcern: { level: "local" }, writeConcern: { w: "majority" } } );
// Operations inside the transaction
try {
   coll1.insertOne( { abc: 1 } );
   coll2.insertOne( { xyz: 999 } );
} catch (error) {
   // Abort transaction on error
   session.abortTransaction();
   throw error;
}
// Commit the transaction using write concern set at transaction start
session.commitTransaction();
session.endSession();
```

