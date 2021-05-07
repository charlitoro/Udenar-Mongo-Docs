# Factores operativos y modelos de datos

El modelado de datos de la aplicación para MongoDB debe considerar varios factores operativos que afectan el rendimiento de MongoDB. Por ejemplo, diferentes modelos de datos pueden permitir consultas más eficientes, aumentar el rendimiento de las operaciones de inserción y actualización o distribuir la actividad a un clúster fragmentado de manera más eficaz.

Al desarrollar un modelo de datos, analice todas las [operaciones de lectura y escritura](https://docs.mongodb.com/manual/crud/) de su aplicación junto con las siguientes consideraciones.

### Atomicidad  <a id="atomicity"></a>

En MongoDB, una operación de escritura es atómica al nivel de un solo documento, incluso si la operación modifica varios documentos incrustados _dentro de_ un solo documento. Cuando una sola operación de escritura modifica varios documentos \(p [`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#mongodb-method-db.collection.updateMany). Ej. \), La modificación de cada documento es atómica, pero la operación en su conjunto no es atómica.

#### Modelo de datos integrado  <a id="embedded-data-model"></a>

El modelo de datos incrustado combina todos los datos relacionados en un solo documento en lugar de normalizarlos en múltiples documentos y colecciones. Este modelo de datos facilita las operaciones atómicas.

Consulte [Datos del modelo para operaciones atómicas](https://docs.mongodb.com/manual/tutorial/model-data-for-atomic-operations/#std-label-data-modeling-atomic-operation) para ver un modelo de datos de ejemplo que proporciona actualizaciones atómicas para un solo documento.

#### Transacción de múltiples documentos  <a id="multi-document-transaction"></a>

Para los modelos de datos que almacenan referencias entre datos relacionados, la aplicación debe emitir operaciones de lectura y escritura independientes para recuperar y modificar estos datos relacionados.

Para situaciones que requieren atomicidad de lecturas y escrituras en múltiples documentos \(en una o varias colecciones\), MongoDB admite transacciones de múltiples documentos:

* **En la versión 4.0** , MongoDB admite transacciones de múltiples documentos en conjuntos de réplicas.
* **En la versión 4.2** , MongoDB introduce transacciones distribuidas, que agrega soporte para transacciones de múltiples documentos en clústeres fragmentados e incorpora el soporte existente para transacciones de múltiples documentos en conjuntos de réplicas.

Para obtener detalles sobre las transacciones en MongoDB, consulte la página [Transacciones](https://docs.mongodb.com/manual/core/transactions/) .IMPORTANTE

En la mayoría de los casos, la transacción de múltiples documentos genera un mayor costo de rendimiento que la escritura de un solo documento, y la disponibilidad de transacciones de múltiples documentos no debe reemplazar el diseño de esquemas efectivo. Para muchos escenarios, el [modelo de datos desnormalizados \(documentos y matrices incrustados\)](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) seguirá siendo óptimo para sus datos y casos de uso. Es decir, para muchos escenarios, modelar sus datos de manera adecuada minimizará la necesidad de transacciones de múltiples documentos.

Para consideraciones adicionales sobre el uso de transacciones \(como el límite de tiempo de ejecución y el límite de tamaño del registro de operaciones\), consulte también [Consideraciones de producción](https://docs.mongodb.com/manual/core/transactions-production-consideration/) .

### Fragmentación  <a id="sharding"></a>

MongoDB usa [fragmentación](https://docs.mongodb.com/manual/reference/glossary/#std-term-sharding) para proporcionar escalado horizontal. Estos clústeres admiten implementaciones con grandes conjuntos de datos y operaciones de alto rendimiento. La fragmentación permite a los usuarios [dividir](https://docs.mongodb.com/manual/reference/glossary/#std-term-data-partition) una [colección](https://docs.mongodb.com/manual/reference/glossary/#std-term-collection) dentro de una base de datos para distribuir los documentos de la colección en varias [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancias o [fragmentos](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard) .

Para distribuir el tráfico de datos y aplicaciones en una colección fragmentada, MongoDB usa la [clave de fragmentación](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-shard-key) . La selección de la [clave de partición](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-shard-key) adecuada tiene implicaciones importantes para el rendimiento y puede habilitar o evitar el aislamiento de consultas y una mayor capacidad de escritura. Es importante considerar detenidamente el campo o los campos que se utilizarán como clave de partición.

Consulte [Fragmentación](https://docs.mongodb.com/manual/sharding/) y [claves de fragmentación](https://docs.mongodb.com/manual/core/sharding-shard-key/) para obtener más información.

### Índices  <a id="indexes"></a>

Utilice índices para mejorar el rendimiento de las consultas habituales. Cree índices en los campos que aparecen con frecuencia en las consultas y para todas las operaciones que devuelven resultados ordenados. MongoDB crea automáticamente un índice único en el `_id`campo.

Al crear índices, tenga en cuenta los siguientes comportamientos de los índices:

* Cada índice requiere al menos 8 kB de espacio de datos.
* Agregar un índice tiene un impacto negativo en el rendimiento de las operaciones de escritura. Para colecciones con una alta proporción de escritura a lectura, los índices son costosos ya que cada inserción también debe actualizar los índices.
* Las colecciones con una alta proporción de lectura a escritura a menudo se benefician de índices adicionales. Los índices no afectan las operaciones de lectura no indexadas.
* Cuando está activo, cada índice consume espacio en disco y memoria. Este uso puede ser significativo y debe rastrearse para la planificación de la capacidad, especialmente por preocupaciones sobre el tamaño del conjunto de trabajo.

Consulte [Estrategias de indexación](https://docs.mongodb.com/manual/applications/indexes/) para obtener más información sobre índices y [Analizar el rendimiento de las consultas](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/) . Además, el [generador de perfiles de](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) la base de [datos](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/) MongoDB puede ayudar a identificar consultas ineficientes.

### Gran cantidad de colecciones  <a id="large-number-of-collections"></a>

En determinadas situaciones, puede optar por almacenar información relacionada en varias colecciones en lugar de en una única colección.

Considere una colección de muestra `logs`que almacena documentos de registro para varios entornos y aplicaciones. La `logs`colección contiene documentos de la siguiente forma:

```text
{ log: "dev", ts: ..., info: ... }
{ log: "debug", ts: ..., info: ...}
```

Si el número total de documentos es bajo, puede agrupar los documentos en una colección por tipo. Para los registros, considere la posibilidad de mantener distintas colecciones de registros, como `logs_dev`y `logs_debug`. La `logs_dev` colección contendría solo los documentos relacionados con el entorno de desarrollo.

Generalmente, tener una gran cantidad de colecciones no tiene una penalización significativa en el rendimiento y da como resultado un muy buen rendimiento. Las colecciones distintas son muy importantes para el procesamiento por lotes de alto rendimiento.

Cuando utilice modelos que tengan una gran cantidad de colecciones, tenga en cuenta los siguientes comportamientos:

* Cada colección tiene una sobrecarga mínima de unos pocos kilobytes.
* Cada índice, incluido el índice de `_id`, requiere al menos 8 kB de espacio de datos.
* Para cada [base de datos](https://docs.mongodb.com/manual/reference/glossary/#std-term-database) , un solo archivo de espacio de nombres \(es decir `<database>.ns`\) almacena todos los metadatos para esa base de datos, y cada índice y colección tiene su propia entrada en el archivo de espacio de nombres. Consulte los [límites de longitud del espacio de nombres de](https://docs.mongodb.com/manual/reference/limits/#std-label-limit-namespace-length) lugares para conocer las limitaciones específicas.

### La colección contiene una gran cantidad de documentos pequeños  <a id="collection-contains-large-number-of-small-documents"></a>

Debería considerar la posibilidad de incrustarla por motivos de rendimiento si tiene una colección con una gran cantidad de documentos pequeños. Si puede agrupar estos documentos pequeños por alguna relación lógica _y los_ recupera con frecuencia por esta agrupación, podría considerar "agrupar" los documentos pequeños en documentos más grandes que contengan una serie de documentos incrustados.

"Recopilar" estos pequeños documentos en agrupaciones lógicas significa que las consultas para recuperar un grupo de documentos implican lecturas secuenciales y menos accesos aleatorios al disco. Además, "acumular" documentos y mover campos comunes a un documento más grande beneficia al índice en estos campos. Habría menos copias de los campos comunes _y_ habría menos entradas clave asociadas en el índice correspondiente. Consulte [Índices](https://docs.mongodb.com/manual/indexes/) para obtener más información sobre índices.

Sin embargo, si a menudo solo necesita recuperar un subconjunto de los documentos dentro del grupo, es posible que "acumular" los documentos no proporcione un mejor rendimiento. Además, si los documentos pequeños e independientes representan el modelo natural de los datos, debe mantener ese modelo.

### Optimización del almacenamiento para documentos pequeños  <a id="storage-optimization-for-small-documents"></a>

Cada documento de MongoDB contiene una cierta cantidad de gastos generales. Esta sobrecarga normalmente es insignificante, pero se vuelve significativa si todos los documentos tienen solo unos pocos bytes, como podría ser el caso si los documentos de su colección solo tienen uno o dos campos.

Considere las siguientes sugerencias y estrategias para optimizar la utilización del almacenamiento para estas colecciones:

* Utilice el `_id`campo de forma explícita.

  Los clientes de MongoDB agregan automáticamente un `_id`campo a cada documento y generan un [ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId) único de 12 bytes para el `_id` campo. Además, MongoDB siempre indexa el `_id`campo. Para documentos más pequeños, esto puede representar una cantidad significativa de espacio.

  Para optimizar el uso del almacenamiento, los usuarios pueden especificar un valor para el `_id`campo explícitamente al insertar documentos en la colección. Esta estrategia permite que las aplicaciones almacenen un valor en el `_id`campo que habría ocupado espacio en otra parte del documento.

  Puede almacenar cualquier valor en el `_id`campo, pero como este valor sirve como clave principal para los documentos de la colección, debe identificarlos de forma única. Si el valor del campo no es único, no puede servir como clave principal, ya que habría colisiones en la colección.

* Utilice nombres de campo más cortos.NOTA

  Acortar los nombres de campo reduce la expresividad y no proporciona un beneficio considerable para documentos más grandes y donde la sobrecarga del documento no es una preocupación importante. Los nombres de campo más cortos no reducen el tamaño de los índices, porque los índices tienen una estructura predefinida.

  En general, no es necesario utilizar nombres de campo cortos.

  MongoDB almacena todos los nombres de campo en cada documento. Para la mayoría de los documentos, esto representa una pequeña fracción del espacio utilizado por un documento; sin embargo, para documentos pequeños, los nombres de campo pueden representar una cantidad de espacio proporcionalmente grande. Considere una colección de documentos pequeños que se parezcan a los siguientes:

  ```text
  { last_name : "Smith", best_score: 3.9 }
  ```

  Si acorta el campo con el nombre `last_name`de `lname`y el campo con el nombre `best_score`de `score`, como se indica a continuación, podría ahorrar 9 bytes por documento.

  ```text
  { lname : "Smith", score : 3.9 }
  ```

* Incrustar documentos.

  En algunos casos, es posible que desee incrustar documentos en otros documentos y ahorrar en los gastos generales por documento. Ver [colección contiene una gran cantidad de documentos pequeños](https://docs.mongodb.com/manual/core/data-model-operations/#std-label-faq-developers-embed-documents) .

### Gestión del ciclo de vida de los datos  <a id="data-lifecycle-management"></a>

Las decisiones de modelado de datos deben tener en cuenta la gestión del ciclo de vida de los datos.

La función [Time to Live o TTL](https://docs.mongodb.com/manual/tutorial/expire-data/) de las colecciones expira los documentos después de un período de tiempo. Considere usar la función TTL si su aplicación requiere que algunos datos persistan en la base de datos durante un período de tiempo limitado.

Además, si su aplicación solo utiliza documentos insertados recientemente, considere [Colecciones limitadas](https://docs.mongodb.com/manual/core/capped-collections/) . Las colecciones limitadas brindan administración de _primero en_ entrar, _primero en salir_ \(FIFO\) de los documentos insertados y respaldan de manera eficiente las operaciones que insertan y leen documentos según el orden de inserción.

