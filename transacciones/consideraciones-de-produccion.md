# Consideraciones de producción

La siguiente página enumera algunas consideraciones de producción para ejecutar transacciones. Estos se aplican tanto si ejecuta transacciones en conjuntos de réplicas como en clústeres fragmentados. Para ejecutar transacciones en clústeres fragmentados, consulte también [Consideraciones de producción \(clústeres fragmentados\)](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/) para obtener consideraciones adicionales que son específicas de los clústeres fragmentados.

### Disponibilidad  <a id="availability"></a>

* **En la versión 4.0** , MongoDB admite transacciones de múltiples documentos en conjuntos de réplicas.
* **En la versión 4.2** , MongoDB introduce transacciones distribuidas, que agrega soporte para transacciones de múltiples documentos en clústeres fragmentados e incorpora el soporte existente para transacciones de múltiples documentos en conjuntos de réplicas.

  Para usar transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\), los clientes **deben** usar controladores de MongoDB actualizados para MongoDB 4.2.

_**NOTA**_

_**Transacciones distribuidas y transacciones de varios documentos**_

_A partir de MongoDB 4.2, los dos términos son sinónimos. Las transacciones distribuidas se refieren a transacciones de varios documentos en grupos fragmentados y conjuntos de réplicas. Las transacciones de múltiples documentos \(ya sea en clústeres fragmentados o conjuntos de réplicas\) también se conocen como transacciones distribuidas a partir de MongoDB 4.2._

### Compatibilidad de  <a id="feature-compatibility"></a>

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

### Límite de tiempo de ejecución  <a id="runtime-limit"></a>

De forma predeterminada, una transacción debe tener un tiempo de ejecución de menos de un minuto. Puede modificar este límite utilizando [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)para las [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancias. Para los clústeres fragmentados, el parámetro debe modificarse para todos los miembros del conjunto de réplicas de fragmentos. Las transacciones que exceden este límite se consideran vencidas y serán canceladas por un proceso de limpieza periódico.

Para los clústeres fragmentados, también puede especificar un `maxTimeMS`límite en `commitTransaction`. Para obtener más información, consulte [Límite de tiempo de transacciones de clústeres fragmentados](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-time-limit) .

### Límite de tamaño de Oplog  <a id="oplog-size-limit"></a>

A partir de la versión 4.2,MongoDB crea tantas entradas de registro de operaciones como sea necesario para encapsular todas las operaciones de escritura en una transacción, en lugar de una sola entrada para todas las operaciones de escritura en la transacción. Esto elimina el límite de tamaño total de 16 MB para una transacción impuesto por la entrada única del registro de operaciones para todas sus operaciones de escritura. Aunque se elimina el límite de tamaño total, cada entrada de registro de operaciones aún debe estar dentro del límite de tamaño de documento BSON de 16 MB.En la versión 4.0,MongoDB crea una única [entrada de registro de operaciones \(registro de operaciones\)](https://docs.mongodb.com/manual/core/replica-set-oplog/) en el momento de la confirmación si la transacción contiene alguna operación de escritura. Es decir, las operaciones individuales en las transacciones no tienen una entrada de registro correspondiente. En cambio, una sola entrada de registro de operaciones contiene todas las operaciones de escritura dentro de una transacción. La entrada del registro de operaciones para la transacción debe estar dentro del límite de tamaño del documento BSON de 16 MB.

### Caché WiredTiger  <a id="wiredtiger-cache"></a>

Para evitar que la presión de la caché de almacenamiento afecte negativamente al rendimiento:

* Cuando abandone una transacción, anótela.
* Cuando encuentre un error durante una operación individual en la transacción, cancele y vuelva a intentar la transacción.

Las [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)asegura también que expiraron las transacciones son abortados periódicamente a la presión de almacenamiento caché aliviar.

### Transacciones y seguridad  <a id="transactions-and-security"></a>

* Si se ejecuta con [control de acceso](https://docs.mongodb.com/manual/core/authorization/) , debe tener [privilegios](https://docs.mongodb.com/manual/reference/built-in-roles/) para las [operaciones en la transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-operations) .
* Si se ejecuta con [auditoría](https://docs.mongodb.com/manual/core/auditing/) , las operaciones en una transacción abortada aún se auditan. Sin embargo, no hay ningún evento de auditoría que indique que la transacción se anuló.

### Restricción de configuración de fragmentos  <a id="shard-configuration-restriction"></a>

No puede ejecutar transacciones en un clúster fragmentado que tiene un fragmento con [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)establecido en `false` \(como un fragmento con un miembro votante que usa el [motor de almacenamiento en memoria](https://docs.mongodb.com/manual/core/inmemory/) \).

### Clústeres y árbitros fragmentados  <a id="sharded-clusters-and-arbiters"></a>

Las transacciones cuyas operaciones de escritura abarcan varios fragmentos generarán errores y se cancelarán si alguna operación de transacción lee o escribe en un fragmento que contiene un árbitro.

Consulte también [Arquitectura de árbitro primario-secundario de 3 miembros](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-psa) para conocer las restricciones de transacción en fragmentos que han deshabilitado la mayoría de interés de lectura.

### Arquitectura de árbitro primario-secundario de 3 miembros  <a id="3-member-primary-secondary-arbiter-architecture"></a>

Para un conjunto de réplicas de tres miembros con una arquitectura de árbitro primario-secundario \(PSA\) o un clúster fragmentado con fragmentos de PSA de tres miembros, es posible que haya [desactivado la preocupación de lectura "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) para evitar la presión de la caché.En clústeres fragmentados,

* Si una transacción involucra un fragmento que ha [inhabilitado la preocupación de lectura "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) , no puede utilizar la preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)para la transacción. Solo puede usar la preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)o [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)para la transacción. Si usa la preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-), la transacción se produce un error y se cancela.

  ```text
  readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false.
  ```

* Las transacciones cuyas operaciones de escritura abarcan varios fragmentos generarán errores y se abortarán si alguna de las operaciones de lectura o escritura de la transacción involucra un fragmento que ha deshabilitado el problema de lectura `"majority"`.

En el juego de réplicas,

Puede especificar la preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)o [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-), o [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)incluso en el conjunto de réplicas tiene [desactivado lectura preocupación "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) .

Sin embargo, si planea realizar la transición a un clúster fragmentado con fragmentos mayoritarios de preocupación de lectura deshabilitados, es posible que desee evitar el uso de la preocupación de lectura `"snapshot"`.

_**CONSEJO**_

_Para comprobar si la preocupación de lectura "mayoría" está deshabilitada, puede ejecutar_ [_`db.serverStatus()`_](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus)_las_ [_`mongod`_](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)_instancias y marcar el_ [_`storageEngine.supportsCommittedReads`_](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads) _campo. Si `false`, lea la preocupación "mayoría" está deshabilitada._

_**CONSEJO**_

_Ver también:_

* \_\_[_`--enableMajorityReadConcern false`_](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern)\_\_
* \_\_[_`replication.enableMajorityReadConcern: false`_](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern)_._

### Adquirir bloqueos  <a id="acquiring-locks"></a>

De forma predeterminada, las transacciones esperan hasta `5`milisegundos para adquirir los bloqueos requeridos por las operaciones en la transacción. Si la transacción no puede adquirir sus bloqueos requeridos dentro de los `5`milisegundos, la transacción se aborta.

Las transacciones liberan todos los bloqueos al abortar o confirmar.

_**CONSEJO**_

_Al crear o eliminar una colección inmediatamente antes de iniciar una transacción, si se accede a la colección dentro de la transacción, emita la operación de creación o eliminación con preocupación de escritura_ [_`"majority"`_](https://docs.mongodb.com/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.-majority-)_para garantizar que la transacción pueda adquirir los bloqueos necesarios._

#### Tiempo de espera de solicitud de bloqueo  <a id="lock-request-timeout"></a>

Puede utilizar el [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis) parámetro para ajustar cuánto tiempo esperan las transacciones para adquirir bloqueos. El aumento [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)permite que las operaciones en las transacciones esperen el tiempo especificado para adquirir los bloqueos requeridos. Esto puede ayudar a evitar abortos de transacciones en adquisiciones de bloqueos simultáneos momentáneos, como operaciones de metadatos de ejecución rápida. Sin embargo, esto posiblemente podría retrasar la interrupción de las operaciones de transacción bloqueadas.

También puede utilizar el tiempo de espera específico de la operación estableciendo [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)en `-1`.

### Operaciones y transacciones DDL pendientes  <a id="pending-ddl-operations-and-transactions"></a>

Si hay una transacción de varios documentos en curso, las nuevas operaciones de DDL que afectan a las mismas bases de datos o colecciones esperan detrás de la transacción. Si bien existen estas operaciones DDL pendientes, las nuevas transacciones que acceden a las mismas bases de datos o colecciones que las operaciones DDL pendientes no pueden obtener los bloqueos requeridos y se abortarán después de esperar [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis). Además, las operaciones nuevas que no sean transacciones y que accedan a las mismas bases de datos o colecciones se bloquearán hasta que alcancen su `maxTimeMS`límite.

Considere los siguientes escenarios:

**Operación DDL que requiere un bloqueo de colección**

Mientras una transacción en curso está realizando varias operaciones CRUD en la `employees`colección en la `hr`base de datos, un administrador emite la [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)operación DDL contra la `employees`colección. [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)requiere un candado de colección exclusivo en la colección.

Hasta que se complete la transacción en curso, la [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)operación debe esperar para obtener el bloqueo. Cualquier transacción nueva que afecte la `employees` colección y comience mientras [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) está pendiente debe esperar hasta que se [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)complete.

La [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)operación DDL pendiente no afecta las transacciones en otras colecciones de la `hr`base de datos. Por ejemplo, una nueva transacción en la `contractors`colección en la `hr`base de datos puede comenzar y completarse normalmente.

**Operación DDL que requiere un bloqueo de base de datos**

Mientras una transacción en curso está realizando varias operaciones CRUD en la `employees`colección en la `hr`base de datos, un administrador emite la [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)operación DDL contra la `contractors`colección en la misma base de datos. [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)requiere un bloqueo de base de datos en la `hr` base de datos principal .

Hasta que se complete la transacción en curso, la [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod) operación debe esperar para obtener el bloqueo. Cualquier nueva transacción que afecte la `hr`base de datos o _cualquiera_ de sus colecciones y comience mientras [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)está pendiente debe esperar hasta que se [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)complete.

En cualquier escenario, si la operación DDL permanece pendiente durante más de [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis), las transacciones pendientes que esperan detrás de esa operación se cancelan. Es decir, el valor de [`maxTransactionLockRequestTimeoutMillis`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.maxTransactionLockRequestTimeoutMillis)debe cubrir al menos el tiempo necesario para que se complete la transacción en curso _y_ la operación DDL pendiente.

_**CONSEJO**_

_Ver también:_

* \_\_[_Transacciones en curso y conflictos de escritura_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-write-conflicts)\_\_
* \_\_[_Transacciones en curso y lecturas obsoletas_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-transactions-stale-reads)\_\_
* \_\_[_¿Qué comandos administrativos bloquean una base de datos?_](https://docs.mongodb.com/manual/faq/concurrency/#std-label-faq-concurrency-database-lock)\_\_
* \_\_[_¿Qué comandos administrativos bloquean una colección?_](https://docs.mongodb.com/manual/faq/concurrency/#std-label-faq-concurrency-collection-lock)\_\_

### Transacciones en curso y conflictos de escritura  <a id="in-progress-transactions-and-write-conflicts"></a>

Si una transacción está en progreso y una escritura fuera de la transacción modifica un documento que luego una operación en la transacción intenta modificar, la transacción se aborta debido a un conflicto de escritura.

Si una transacción está en curso y se ha bloqueado para modificar un documento, cuando una escritura fuera de la transacción intenta modificar el mismo documento, la escritura espera hasta que finaliza la transacción.

_**CONSEJO**_

_Ver también:_

* \_\_[_Adquirir cerraduras_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txns-locks)\_\_
* \_\_[_Operaciones y transacciones de DDL pendientes_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)\_\_
* \_\_[_`$currentOp output`_](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-data--currentOp.prepareReadConflicts)\_\_

### Transacciones en curso y lecturas obsoletas  <a id="in-progress-transactions-and-stale-reads"></a>

Las operaciones de lectura dentro de una transacción pueden devolver datos obsoletos. Es decir, no se garantiza que las operaciones de lectura dentro de una transacción vean escrituras realizadas por otras transacciones comprometidas o escrituras no transaccionales. Por ejemplo, considere la siguiente secuencia: 1\) una transacción está en curso 2\) una escritura fuera de la transacción elimina un documento 3\) una operación de lectura dentro de la transacción puede leer el documento ahora eliminado ya que la operación está usando una instantánea desde antes de la escritura.

Para evitar lecturas obsoletas dentro de las transacciones de un solo documento, puede utilizar el [`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#mongodb-method-db.collection.findOneAndUpdate)método. Por ejemplo:

```text
session.startTransaction( { readConcern: { level: "snapshot" }, writeConcern: { w: "majority" } } );

employeesCollection = session.getDatabase("hr").employees;

employeeDoc = employeesCollection.findOneAndUpdate(
   { _id: 1, employee: 1, status: "Active" },
   { $set: { employee: 1 } },
   { returnNewDocument: true }
);
```



* Si el documento del empleado ha cambiado fuera de la transacción, la transacción se cancela.
* Si el documento del empleado no ha cambiado, la transacción devuelve el documento y lo bloquea.

### Transacciones en curso y migración de fragmentos  <a id="in-progress-transactions-and-chunk-migration"></a>

[La migración de fragmentos](https://docs.mongodb.com/manual/core/sharding-balancer-administration/#std-label-chunk-migration-procedure) adquiere bloqueos de colección exclusivos durante determinadas etapas.

Si una transacción en curso tiene un bloqueo en una colección y se inicia una migración de fragmentos que implica esa recopilación, estas etapas de migración deben esperar a que la transacción libere los bloqueos de la colección, lo que afectará el rendimiento de las migraciones de fragmentos.

Si una migración de fragmentos se entrelaza con una transacción \(por ejemplo, si una transacción comienza mientras una migración de fragmentos ya está en curso y la migración se completa antes de que la transacción bloquee la colección\), la transacción se produce un error durante la confirmación y se cancela.

Dependiendo de cómo se intercalen las dos operaciones, algunos errores de muestra incluyen \(los mensajes de error se han abreviado\):

* `an error from cluster data placement change ... migration commit in progress for <namespace>`
* `Cannot find shardId the chunk belonged to at cluster time ...`

_**CONSEJO Ver también**_[_`shardingStatistics.countDonorMoveChunkLockTimeout`_](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.shardingStatistics.countDonorMoveChunkLockTimeout)\_\_

### Lecturas externas durante la confirmación  <a id="outside-reads-during-commit"></a>

Durante la confirmación de una transacción, las operaciones de lectura externas pueden intentar leer los mismos documentos que serán modificados por la transacción. Si la transacción escribe en varios fragmentos, durante el intento de confirmación en los fragmentos

* Exterior lee que el uso leyó preocupación [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)o [`"linearizable"`](https://docs.mongodb.com/manual/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-), o son parte de las sesiones causalmente consistentes \(es decir, incluyen [afterClusterTime](https://docs.mongodb.com/manual/reference/read-concern/#std-label-afterClusterTime) \) espera para todas las escrituras de una transacción para que sea visible.
* Las lecturas externas que utilizan otras preocupaciones de lectura no esperan a que todas las escrituras de una transacción sean visibles, sino que leen la versión anterior a la transacción de los documentos disponibles.

### Errores  <a id="errors"></a>

#### Uso de controladores MongoDB 4.0  <a id="use-of-mongodb-4.0-drivers"></a>

Para usar transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\), los clientes **deben** usar controladores de MongoDB actualizados para MongoDB 4.2.

En clústeres fragmentados con varias [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)instancias, la realización de transacciones con controladores actualizados para MongoDB 4.0 \(en lugar de MongoDB 4.2\) fallará y puede generar errores, que incluyen:

_**NOTA:** Su controlador puede devolver un error diferente. Consulte la documentación de su controlador para obtener más detalles._

| Código de error | Mensaje de error |
| :--- | :--- |
| 251 | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940 | `cannot commit with no participants` |

### Información adicional  <a id="additional-information"></a>

_**CONSEJO  Ver también:**_

\_\_[_Consideraciones de producción \(clústeres fragmentados\)_](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/)\_\_

