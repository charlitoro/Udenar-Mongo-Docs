# Consideraciones de producción \(clústeres fragmentados\)

A partir de la versión 4.2, MongoDB brinda la capacidad de realizar transacciones de múltiples documentos para clústeres fragmentados.

La siguiente página enumera preocupaciones específicas para ejecutar transacciones en un clúster fragmentado. Estas preocupaciones se suman a las enumeradas en [Consideraciones de producción](https://docs.mongodb.com/manual/core/transactions-production-consideration/) .

### Transacciones fragmentadas y controladores MongoDB  <a id="sharded-transactions-and-mongodb-drivers"></a>

_Para transacciones en implementaciones de MongoDB 4.2 \(conjuntos de réplicas y clústeres fragmentados\)_ , los clientes **deben** usar controladores MongoDB actualizados para MongoDB 4.2.

En clústeres fragmentados con varias [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)instancias, la realización de transacciones con controladores actualizados para MongoDB 4.0 \(en lugar de MongoDB 4.2\) fallará y puede generar errores, que incluyen:

_**NOTA:** Su controlador puede devolver un error diferente. Consulte la documentación de su controlador para obtener más detalles._

| Código de error | Mensaje de error |
| :--- | :--- |
| 251 | `cannot continue txnId -1 for session ... with txnId 1` |
| 50940 | `cannot commit with no participants` |

### Rendimiento  <a id="performance"></a>

#### Fragmento único  <a id="single-shard"></a>

Las transacciones que tienen como objetivo un solo fragmento deben tener el mismo rendimiento que las transacciones de conjuntos de réplicas.

#### Múltiples fragmentos  <a id="multiple-shards"></a>

Las transacciones que afectan a múltiples fragmentos incurren en un mayor costo de rendimiento.

_**NOTA**: En un clúster fragmentado, las transacciones que abarcan varios fragmentos generarán errores y se cancelarán si alguno de los fragmentos implicados contiene un árbitro._

#### Límite de tiempo  <a id="time-limit"></a>

Para especificar un límite de tiempo, especifique un `maxTimeMS`límite en `commitTransaction`.

Si `maxTimeMS`no se especifica, MongoDB utilizará la extensión [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds).

Si `maxTimeMS`se especifica pero daría como resultado una transacción que exceda [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds), MongoDB usará el [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds).

Para modificar [`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.transactionLifetimeLimitSeconds)para un clúster fragmentado, el parámetro debe modificarse para todos los miembros del conjunto de réplicas de fragmentos.

### Leer inquietudes  <a id="read-concerns"></a>

Transacciones multi-documento de apoyo [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-), [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)y [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)leen los niveles de preocupación.

Para las transacciones en un clúster fragmentado, solo la [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)preocupación de lectura proporciona una instantánea coherente en varios fragmentos.

Para obtener más información sobre problemas de lectura y transacciones, consulte [Transacciones y problemas de lectura](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-read-concern) .

### Escribir inquietudes  <a id="write-concerns"></a>

No puede ejecutar transacciones en un clúster fragmentado que tiene un fragmento con [`writeConcernMajorityJournalDefault`](https://docs.mongodb.com/manual/reference/replica-configuration/#mongodb-rsconf-rsconf.writeConcernMajorityJournalDefault)establecido en `false` \(como un fragmento con un miembro votante que usa el [motor de almacenamiento en memoria](https://docs.mongodb.com/manual/core/inmemory/) \).

_**NOTA: I**ndependientemente del_ [_problema de escritura especificado para la transacción_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-write-concern) _, la operación de confirmación para una transacción de clúster fragmentado incluye algunas partes que utilizan el `{w: "majority", j: true}`problema de escritura._

### Árbitros  <a id="arbiters"></a>

Las transacciones cuyas operaciones de escritura abarcan varios fragmentos generarán errores y se cancelarán si alguna operación de transacción lee o escribe en un fragmento que contiene un árbitro.

Consulte también [Fragmentos de árbitro primario-secundario-árbitro de tres miembros](https://docs.mongodb.com/manual/core/transactions-sharded-clusters/#std-label-transactions-sharded-clusters-psa) para conocer las restricciones de transacción de los fragmentos que han desactivado la mayoría de interés de lectura.

### Fragmentos de árbitro principal-secundario-de tres miembros  <a id="three-member-primary-secondary-arbiter-shards"></a>

Para un clúster fragmentado con fragmentos de PSA de tres miembros, es posible que haya [desactivado la preocupación de lectura "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) \(es decir, [`--enableMajorityReadConcern false`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--enableMajorityReadConcern)o [`replication.enableMajorityReadConcern: false`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern)\) para evitar la presión de la caché.En clústeres fragmentados,

* Si una transacción involucra un fragmento que ha [inhabilitado la preocupación de lectura "mayoría"](https://docs.mongodb.com/manual/reference/read-concern-majority/#std-label-disable-read-concern-majority) , no puede utilizar la preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-)para la transacción. Solo puede usar la preocupación de lectura [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)o [`"majority"`](https://docs.mongodb.com/manual/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-)para la transacción. Si usa la preocupación de lectura [`"snapshot"`](https://docs.mongodb.com/manual/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-), la transacción se produce un error y se cancela.

  ```text
  readConcern level 'snapshot' is not supported in sharded clusters when enableMajorityReadConcern=false.
  ```

* Las transacciones cuyas operaciones de escritura abarcan varios fragmentos generarán errores y se abortarán si alguna de las operaciones de lectura o escritura de la transacción involucra un fragmento que ha deshabilitado el problema de lectura `"majority"`.

Para comprobar si la preocupación de lectura "mayoría" está desactivada,

Puede ejecutar [`db.serverStatus()`](https://docs.mongodb.com/manual/reference/method/db.serverStatus/#mongodb-method-db.serverStatus)y comprobar el [`storageEngine.supportsCommittedReads`](https://docs.mongodb.com/manual/reference/command/serverStatus/#mongodb-serverstatus-serverstatus.storageEngine.supportsCommittedReads)campo. Si `false`, lea la preocupación "mayoría" está deshabilitada.

### Copias de seguridad y restauraciones  <a id="backups-and-restores"></a>

_**ADVERTENCIA:**_ [_`mongodump`_](https://docs.mongodb.com/database-tools/mongodump/#mongodb-binary-bin.mongodump)_y **no puede** ser parte de una estrategia de copia de seguridad para clústeres fragmentados 4.2+ que tienen transacciones fragmentadas en curso, ya que las copias de seguridad creadas con no mantienen las garantías de atomicidad de las transacciones entre fragmentos._[_`mongorestore`_](https://docs.mongodb.com/database-tools/mongorestore/#mongodb-binary-bin.mongorestore) __[_`mongodump`_](https://docs.mongodb.com/database-tools/mongodump/#mongodb-binary-bin.mongodump)\_\_

_Para clústeres fragmentados 4.2+ con transacciones fragmentadas en curso, utilice uno de los siguientes procesos coordinados de copia de seguridad y restauración que mantienen las garantías de atomicidad de las transacciones entre fragmentos:_

* \_\_[_Atlas de MongoDB_](https://www.mongodb.com/cloud/atlas?tck=docs_server) _,_
* \_\_[_MongoDB Cloud Manager_](https://www.mongodb.com/cloud/cloud-manager?tck=docs_server) _, o_
* \_\_[_Gerente de Operaciones de MongoDB_](https://www.mongodb.com/products/ops-manager?tck=docs_server) _._

### Migraciones de fragmentos  <a id="chunk-migrations"></a>

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

_**CONSEJO Ver también:**_ [_Transacciones y atomicidad_](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-atomicity)\_\_

### Información adicional  <a id="additional-information"></a>

Consulte también [Consideraciones de producción](https://docs.mongodb.com/manual/core/transactions-production-consideration/) .

