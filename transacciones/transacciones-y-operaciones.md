# Transacciones y operaciones

Para transacciones:

* Puede especificar operaciones de lectura / escritura \(CRUD\) en colecciones **existentes** . Para obtener una lista de operaciones CRUD, consulte [Operaciones CRUD](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud) .
* Cuando usa la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` o superior, puede crear colecciones e índices en las transacciones. Para obtener más información, consulte [Crear colecciones e índices en una transacción.](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes)
* Las colecciones utilizadas en una transacción pueden estar en diferentes bases de datos.

_**NOTA**: No puede crear nuevas colecciones en transacciones de escritura entre particiones. Por ejemplo, si escribe en una colección existente en un fragmento e implícitamente crea una colección en un fragmento diferente, MongoDB no puede realizar ambas operaciones en la misma transacción._

* No puede escribir en colecciones [limitadas](https://docs.mongodb.com/manual/core/capped-collections/) . \(A partir de MongoDB 4.2\)
* No se puede leer / escribir en colecciones en las `config`, `admin`o `local`bases de datos.
* No puede escribir en `system.*`colecciones.
* No puede devolver el plan de consulta de la operación admitida \(es decir `explain`\).
* Para los cursores creados fuera de una transacción, no puede llamar [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)dentro de la transacción.
* Para los cursores creados en una transacción, no puede llamar [`getMore`](https://docs.mongodb.com/manual/reference/command/getMore/#mongodb-dbcommand-dbcmd.getMore)fuera de la transacción.
* A partir de MongoDB 4.2, no puede especificar [`killCursors`](https://docs.mongodb.com/manual/reference/command/killCursors/#mongodb-dbcommand-dbcmd.killCursors)como primera operación en una [transacción](https://docs.mongodb.com/manual/core/transactions/) .

Las operaciones que afectan el catálogo de la base de datos, como la creación o eliminación de una colección o un índice, no se permiten en transacciones de varios documentos. Por ejemplo, una transacción de varios documentos no puede incluir una operación de inserción que daría lugar a la creación de una nueva colección. Consulte [Operaciones restringidas](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ref-restricted) .

### Operaciones admitidas en transacciones de varios documentos  <a id="operations-supported-in-multi-document-transactions"></a>

#### Operaciones CRUD  <a id="crud-operations"></a>

Las siguientes operaciones de lectura / escritura están permitidas en las transacciones:

<table>
  <thead>
    <tr>
      <th style="text-align:left">M&#xE9;todo</th>
      <th style="text-align:left">Mando</th>
      <th style="text-align:left">Nota</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate"><code>db.collection.aggregate()</code></a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/aggregate/#mongodb-dbcommand-dbcmd.aggregate"><code>aggregate</code></a>
      </td>
      <td style="text-align:left">
        <p>Excluyendo las siguientes etapas:</p>
        <ul>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/collStats/#mongodb-pipeline-pipe.-collStats"><code>$collStats</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#mongodb-pipeline-pipe.-currentOp"><code>$currentOp</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/indexStats/#mongodb-pipeline-pipe.-indexStats"><code>$indexStats</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/listLocalSessions/#mongodb-pipeline-pipe.-listLocalSessions"><code>$listLocalSessions</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/listSessions/#mongodb-pipeline-pipe.-listSessions"><code>$listSessions</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge"><code>$merge</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out"><code>$out</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/planCacheStats/#mongodb-pipeline-pipe.-planCacheStats"><code>$planCacheStats</code></a>
          </li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#mongodb-method-db.collection.countDocuments"><code>db.collection.countDocuments()</code></a>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>Excluyendo las siguientes expresiones de operador de consulta:</p>
        <ul>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where"><code>$where</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near"><code>$near</code></a>
          </li>
          <li><a href="https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere"><code>$nearSphere</code></a>
          </li>
        </ul>
        <p>El m&#xE9;todo utiliza la <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/match/#mongodb-pipeline-pipe.-match"><code>$match</code></a>etapa
          de agregaci&#xF3;n para la consulta y <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group"><code>$group</code></a>la
          etapa de agregaci&#xF3;n con una <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum"><code>$sum</code></a>expresi&#xF3;n
          para realizar el recuento.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct"><code>db.collection.distinct()</code></a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct"><code>distinct</code></a>
      </td>
      <td style="text-align:left">Disponible en colecciones sin fragmentar.Para colecciones fragmentadas,
        use la canalizaci&#xF3;n de agregaci&#xF3;n con el <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group"><code>$group</code></a>escenario.
        Consulte <a href="https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-distinct">Operaci&#xF3;n distinta</a> .</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find"><code>db.collection.find()</code></a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/find/#mongodb-dbcommand-dbcmd.find"><code>find</code></a>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch"><code>geoSearch</code></a>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#mongodb-method-db.collection.deleteMany"><code>db.collection.deleteMany()</code></a>
        <a
        href="https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#mongodb-method-db.collection.deleteOne"><code>db.collection.deleteOne()</code>
          </a><a href="https://docs.mongodb.com/manual/reference/method/db.collection.remove/#mongodb-method-db.collection.remove"><code>db.collection.remove()</code></a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/delete/#mongodb-dbcommand-dbcmd.delete"><code>delete</code></a>
      </td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndDelete/#mongodb-method-db.collection.findOneAndDelete"><code>db.collection.findOneAndDelete()</code></a>
        <a
        href="https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#mongodb-method-db.collection.findOneAndReplace"><code>db.collection.findOneAndReplace()</code>
          </a><a href="https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#mongodb-method-db.collection.findOneAndUpdate"><code>db.collection.findOneAndUpdate()</code></a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/findAndModify/#mongodb-dbcommand-dbcmd.findAndModify"><code>findAndModify</code></a>
      </td>
      <td style="text-align:left">
        <p>Para la <a href="https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv">versi&#xF3;n de compatibilidad de funciones (fcv)</a>  <code>&quot;4.4&quot;</code> o
          superior, si la operaci&#xF3;n de actualizaci&#xF3;n / reemplazo se ejecuta
          con <code>upsert: true</code>una colecci&#xF3;n no existente, la colecci&#xF3;n
          se crea impl&#xED;citamente.</p>
        <p>Para fcv <code>&quot;4.2&quot;</code>o menos, si <code>upsert: true</code>,
          la operaci&#xF3;n debe ejecutarse en una colecci&#xF3;n existente.CONSEJOVer
          tambi&#xE9;n:</p>
        <p><a href="https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl">Operaciones DDL</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#mongodb-method-db.collection.insertMany"><code>db.collection.insertMany()</code></a>
        <a
        href="https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#mongodb-method-db.collection.insertOne"><code>db.collection.insertOne()</code>
          </a><a href="https://docs.mongodb.com/manual/reference/method/db.collection.insert/#mongodb-method-db.collection.insert"><code>db.collection.insert()</code></a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/insert/#mongodb-dbcommand-dbcmd.insert"><code>insert</code></a>
      </td>
      <td style="text-align:left">
        <p>Para la <a href="https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv">versi&#xF3;n de compatibilidad de funciones (fcv)</a>  <code>&quot;4.4&quot;</code> o
          superior, cuando se ejecuta en una colecci&#xF3;n no existente, la colecci&#xF3;n
          se crea impl&#xED;citamente.</p>
        <p>Para fcv <code>&quot;4.2&quot;</code>o menos, solo se puede ejecutar en
          una colecci&#xF3;n existente.CONSEJOVer tambi&#xE9;n:</p>
        <p><a href="https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl">Operaciones DDL</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.save/#mongodb-method-db.collection.save"><code>db.collection.save()</code></a>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>Para la <a href="https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv">versi&#xF3;n de compatibilidad de funciones (fcv)</a>  <code>&quot;4.4&quot;</code> o
          superior, si se inserta contra una colecci&#xF3;n no existente, la colecci&#xF3;n
          se crea impl&#xED;citamente.</p>
        <p>Con fcv <code>&quot;4.2&quot;</code>o menos, si es un inserto, solo se
          puede ejecutar en una colecci&#xF3;n existente.CONSEJOVer tambi&#xE9;n:</p>
        <p><a href="https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl">Operaciones DDL</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#mongodb-method-db.collection.updateOne"><code>db.collection.updateOne()</code></a>
        <a
        href="https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#mongodb-method-db.collection.updateMany"><code>db.collection.updateMany()</code>
          </a><a href="https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#mongodb-method-db.collection.replaceOne"><code>db.collection.replaceOne()</code></a>
          <a
          href="https://docs.mongodb.com/manual/reference/method/db.collection.update/#mongodb-method-db.collection.update"><code>db.collection.update()</code>
            </a>
      </td>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/command/update/#mongodb-dbcommand-dbcmd.update"><code>update</code></a>
      </td>
      <td style="text-align:left">
        <p>Para la <a href="https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv">versi&#xF3;n de compatibilidad de funciones (fcv)</a>  <code>&quot;4.4&quot;</code> o
          superior, si se ejecuta con <code>upsert: true</code>una colecci&#xF3;n
          no existente, la colecci&#xF3;n se crea impl&#xED;citamente.</p>
        <p>Para fcv <code>&quot;4.2&quot;</code>o menos, si <code>upsert: true</code>,
          la operaci&#xF3;n debe ejecutarse en una colecci&#xF3;n existente.CONSEJOVer
          tambi&#xE9;n:</p>
        <p><a href="https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl">Operaciones DDL</a>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#mongodb-method-db.collection.bulkWrite"><code>db.collection.bulkWrite()</code></a>Varios
        <a
        href="https://docs.mongodb.com/manual/reference/method/js-bulk/">m&#xE9;todos de operaci&#xF3;n a granel</a>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>Para la <a href="https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv">versi&#xF3;n de compatibilidad de funciones (fcv)</a>  <code>&quot;4.4&quot;</code> y
          superior, si <code>upsert: true</code>se ejecuta una operaci&#xF3;n de inserci&#xF3;n
          o actualizaci&#xF3;n con en una transacci&#xF3;n contra una colecci&#xF3;n
          no existente, la colecci&#xF3;n se crea impl&#xED;citamente.</p>
        <p>Para fcv <code>&quot;4.2&quot;</code>o menos, la colecci&#xF3;n ya debe
          existir para la inserci&#xF3;n y las <code>upsert: true</code>operaciones.CONSEJOVer
          tambi&#xE9;n:</p>
        <p><a href="https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl">Operaciones DDL</a>
        </p>
      </td>
    </tr>
  </tbody>
</table>

_**NOTA**:_     ****_**Actualizaciones de los valores clave de Shard:** A partir de MongoDB 4.2, puede actualizar el valor de la clave de fragmento de un documento \(a menos que el campo de clave de fragmento sea el `_id`campo inmutable \) mediante la emisión de operaciones de actualización / findAndModify de un solo documento en una transacción o como una_ [_escritura recuperable_](https://docs.mongodb.com/manual/core/retryable-writes/) _. Para obtener más información, consulte_ [_Cambiar el valor de clave de fragmento de un documento_](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-update-shard-key) _._

#### Operación de conteo  <a id="count-operation"></a>

Para realizar una operación de recuento dentro de una transacción, use la [`$count`](https://docs.mongodb.com/manual/reference/operator/aggregation/count/#mongodb-pipeline-pipe.-count)etapa de agregación o la etapa de agregación [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)\(con una [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)expresión\).

Los controladores MongoDB compatibles con las características 4.0 proporcionan una API de nivel de colección `countDocuments(filter, options)`como método auxiliar que usa la expresión [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)con una [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)para realizar un recuento. Los controladores 4.0 han desaprobado la `count()`API.

A partir de MongoDB 4.0.3, el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell proporciona el [`db.collection.countDocuments()`](https://docs.mongodb.com/manual/reference/method/db.collection.countDocuments/#mongodb-method-db.collection.countDocuments)método auxiliar que usa [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)con una [`$sum`](https://docs.mongodb.com/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum)expresión para realizar un recuento.

#### Operación distinta  <a id="distinct-operation"></a>

Para realizar una operación distinta dentro de una transacción:

* Para las colecciones no fragmentadas, puede usar el [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct)método / el [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)comando, así como la canalización de agregación con el [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)escenario.
* Para las colecciones fragmentadas, no puede utilizar el [`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#mongodb-method-db.collection.distinct)método ni el [`distinct`](https://docs.mongodb.com/manual/reference/command/distinct/#mongodb-dbcommand-dbcmd.distinct)comando.

  Para encontrar los valores distintos para una colección fragmentada, use la canalización de agregación con el [`$group`](https://docs.mongodb.com/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group)escenario en su lugar. Por ejemplo:

  * En lugar de `db.coll.distinct("x")`, usa

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

#### Operaciones DDL  <a id="ddl-operations"></a>

A partir de MongoDB 4.4 con la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.4"` , puede crear colecciones e índices dentro de una [transacción de varios documentos](https://docs.mongodb.com/manual/core/transactions/) si la transacción **no** es una transacción de escritura entre fragmentos.

**Operaciones DDL explícitas** 

| Mando | Método | Notas |
| :--- | :--- | :--- |
| [`create`](https://docs.mongodb.com/manual/reference/command/create/#mongodb-dbcommand-dbcmd.create) | [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection) | Consulte también las [operaciones DDL implícitas](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-implicit) . |
| [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) | [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)[`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes) | El índice a crear debe estar en una colección no existente, en cuyo caso, la colección se crea como parte de la operación, o en una nueva colección vacía creada anteriormente en la misma transacción. |

_**NOTA**: Para la creación explícita de una colección o un índice dentro de una transacción, el nivel de preocupación de lectura de la transacción debe ser_ [_`"local"`_](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-)_._

Para obtener más información sobre cómo crear colecciones e índices en una transacción, consulte [Crear colecciones e índices en una transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) .

_**CONSEJO Ver también:**_[_`shouldMultiDocTxnCreateCollectionAndIndexes`_](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.shouldMultiDocTxnCreateCollectionAndIndexes)\_\_

**Operaciones DDL implícitas** 

También puede crear implícitamente una colección a través de las siguientes operaciones de escritura contra una colección **no existente** :

| Método ejecutado contra colección no existente | Ejecutar comando contra colección no existente |
| :--- | :--- |
| [`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify) con `upsert: true`[`db.collection.findOneAndReplace()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndReplace/#mongodb-method-db.collection.findOneAndReplace) con `upsert: true`[`db.collection.findOneAndUpdate()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#mongodb-method-db.collection.findOneAndUpdate) con `upsert: true` | [`findAndModify`](https://docs.mongodb.com/manual/reference/command/findAndModify/#mongodb-dbcommand-dbcmd.findAndModify) con `upsert: true` |
| [`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#mongodb-method-db.collection.insertMany)[`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#mongodb-method-db.collection.insertOne)[`db.collection.insert()`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/#mongodb-method-db.collection.insert) | [`insert`](https://docs.mongodb.com/manual/reference/command/insert/#mongodb-dbcommand-dbcmd.insert) |
| [`db.collection.save()`](https://docs.mongodb.com/manual/reference/method/db.collection.save/#mongodb-method-db.collection.save) da como resultado un inserto |  |
| [`db.collection.updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#mongodb-method-db.collection.updateOne) con `upsert: true`[`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#mongodb-method-db.collection.updateMany) con `upsert: true`[`db.collection.replaceOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#mongodb-method-db.collection.replaceOne) con `upsert: true`[`db.collection.update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/#mongodb-method-db.collection.update) con `upsert: true` | [`update`](https://docs.mongodb.com/manual/reference/command/update/#mongodb-dbcommand-dbcmd.update) con `upsert: true` |
| [`db.collection.bulkWrite()`](https://docs.mongodb.com/manual/reference/method/db.collection.bulkWrite/#mongodb-method-db.collection.bulkWrite) con inserto u `upsert:true`operacionesVarios [métodos de operación masiva](https://docs.mongodb.com/manual/reference/method/js-bulk/) con inserción u `upsert:true`operaciones |  |

Para otras operaciones CRUD permitidas en transacciones, consulte [Operaciones CRUD](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-crud) .

Para obtener más información sobre cómo crear colecciones e índices en una transacción, consulte [Crear colecciones e índices en una transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) .

_**CONSEJO Ver también:**_[_`shouldMultiDocTxnCreateCollectionAndIndexes`_](https://docs.mongodb.com/manual/reference/parameters/#mongodb-parameter-param.shouldMultiDocTxnCreateCollectionAndIndexes)\_\_

#### Operaciones informativas  <a id="informational-operations"></a>

Comandos informativos, tales como [`hello`](https://docs.mongodb.com/manual/reference/command/hello/#mongodb-dbcommand-dbcmd.hello), [`buildInfo`](https://docs.mongodb.com/manual/reference/command/buildInfo/#mongodb-dbcommand-dbcmd.buildInfo), [`connectionStatus`](https://docs.mongodb.com/manual/reference/command/connectionStatus/#mongodb-dbcommand-dbcmd.connectionStatus)\(y sus métodos de ayuda\) se permiten en las transacciones; sin embargo, no pueden ser la primera operación de la transacción.

### Operaciones restringidas  <a id="restricted-operations"></a>

_Modificado en la versión 4.4_ .

Las siguientes operaciones no están permitidas en transacciones:

* Operaciones que afectan el catálogo de la base de datos, como crear o eliminar una colección o un índice cuando se usa la [versión de compatibilidad de funciones \(fcv\)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) `"4.2"` o una [versión anterior](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) . Con fcv `"4.4"`o superior, puede crear colecciones e índices en transacciones a menos que la transacción sea una transacción de escritura entre fragmentos. Para obtener más información, consulte [Crear colecciones e índices en una transacción](https://docs.mongodb.com/manual/core/transactions/#std-label-transactions-create-collections-indexes) .
* Creación de nuevas colecciones en transacciones de escritura entre fragmentos. Por ejemplo, si escribe en una colección existente en un fragmento e implícitamente crea una colección en un fragmento diferente, MongoDB no puede realizar ambas operaciones en la misma transacción.
* [Creación explícita de colecciones](https://docs.mongodb.com/manual/core/transactions-operations/#std-label-transactions-operations-ddl-explicit) , por ejemplo, [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection)método e índices, por ejemplo, [`db.collection.createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes)y [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)métodos, cuando se utiliza un nivel de preocupación de lectura diferente a [`"local"`](https://docs.mongodb.com/manual/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-).
* Las [`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections)y los [`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes) comandos y sus métodos de ayuda.
* Otras operaciones no CRUD y no informativos, como [`createUser`](https://docs.mongodb.com/manual/reference/command/createUser/#mongodb-dbcommand-dbcmd.createUser), [`getParameter`](https://docs.mongodb.com/manual/reference/command/getParameter/#mongodb-dbcommand-dbcmd.getParameter), [`count`](https://docs.mongodb.com/manual/reference/command/count/#mongodb-dbcommand-dbcmd.count), etc, y sus ayudantes.

_**CONSEJO Ver también:**_ [_Operaciones y transacciones de DDL pendientes_](https://docs.mongodb.com/manual/core/transactions-production-consideration/#std-label-txn-prod-considerations-ddl)\_\_

