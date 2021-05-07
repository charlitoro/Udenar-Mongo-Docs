# Diseño de modelos de datos

Los modelos de datos efectivos respaldan las necesidades de su aplicación. La consideración clave para la estructura de sus documentos es la decisión de [incrustar](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) o [utilizar referencias](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-referencing) .

### Modelos de datos integrados  <a id="embedded-data-models"></a>

Con MongoDB, puede incrustar datos relacionados en una única estructura o documento. Estos esquemas se conocen generalmente como modelos "desnormalizados" y aprovechan los ricos documentos de MongoDB. Considere el siguiente diagrama:![Modelo de datos con campos incrustados que contienen toda la informaci&#xF3;n relacionada.](https://docs.mongodb.com/manual/images/data-model-denormalized.bakedsvg.svg)Click para agrandar

Los modelos de datos integrados permiten que las aplicaciones almacenen información relacionada en el mismo registro de base de datos. Como resultado, es posible que las aplicaciones necesiten emitir menos consultas y actualizaciones para completar operaciones comunes.

En general, utilice modelos de datos integrados cuando:

* tiene relaciones "contiene" entre entidades. Consulte [Modelos de relaciones uno a uno con documentos incrustados](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/#std-label-data-modeling-example-one-to-one) .
* tienes relaciones de uno a varios entre entidades. En estas relaciones, los documentos "muchos" o secundarios siempre aparecen o se ven en el contexto de los documentos "uno" o principales. Consulte [Modelar relaciones de uno a varios con documentos incrustados](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#std-label-data-modeling-example-one-to-many) .

En general, la incrustación proporciona un mejor rendimiento para las operaciones de lectura, así como la capacidad de solicitar y recuperar datos relacionados en una sola operación de base de datos. Los modelos de datos integrados permiten actualizar los datos relacionados en una sola operación de escritura atómica.

Para acceder a los datos dentro de los documentos incrustados, utilice [la notación de puntos](https://docs.mongodb.com/manual/reference/glossary/#std-term-dot-notation) para "acceder" a los documentos incrustados. Consulte la [consulta de datos en matrices](https://docs.mongodb.com/manual/tutorial/query-arrays/#std-label-read-operations-arrays) y la [consulta de datos en documentos incrustados](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/#std-label-read-operations-embedded-documents) para obtener más ejemplos sobre cómo acceder a datos en matrices y documentos incrustados.

#### Modelo de datos integrado y límite de tamaño del documento  <a id="embedded-data-model-and-document-size-limit"></a>

Los documentos en MongoDB deben ser más pequeños que [`maximum BSON document size`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-BSON-Document-Size).

Para datos binarios masivos, considere [GridFS](https://docs.mongodb.com/manual/core/gridfs/) .

### Modelos de datos normalizados  <a id="normalized-data-models"></a>

Los modelos de datos normalizados describen relaciones utilizando [referencias](https://docs.mongodb.com/manual/reference/database-references/) entre documentos.![Modelo de datos utilizando referencias para vincular documentos.  Tanto el documento de \`\` contacto &apos;&apos; como el documento de \`\` acceso &apos;&apos; contienen una referencia al documento de \`\` usuario &apos;&apos;.](https://docs.mongodb.com/manual/images/data-model-normalized.bakedsvg.svg)Click para agrandar

En general, utilice modelos de datos normalizados:

* cuando la incrustación daría lugar a la duplicación de datos, pero no proporcionaría suficientes ventajas de rendimiento de lectura para compensar las implicaciones de la duplicación.
* para representar relaciones de varios a varios más complejas.
* para modelar grandes conjuntos de datos jerárquicos.

Para unirse a colecciones, MongoDB proporciona las etapas de agregación:

* [`$lookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#mongodb-pipeline-pipe.-lookup) \(Disponible a partir de MongoDB 3.2\)
* [`$graphLookup`](https://docs.mongodb.com/manual/reference/operator/aggregation/graphLookup/#mongodb-pipeline-pipe.-graphLookup) \(Disponible a partir de MongoDB 3.4\)

MongoDB también proporciona referencias para unir datos entre colecciones.

Para obtener un ejemplo de modelos de datos normalizados, consulte [Modelar relaciones de uno a varios con referencias de documentos](https://docs.mongodb.com/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/#std-label-data-modeling-publisher-and-books) .

Para ver ejemplos de varios modelos de árbol, consulte [Estructuras de árbol del modelo](https://docs.mongodb.com/manual/applications/data-models-tree-structures/) .

### Lectura adicional  <a id="further-reading"></a>

Para obtener más información sobre el modelado de datos con MongoDB, descargue la [Guía de modernización de aplicaciones de MongoDB](https://www.mongodb.com/modernize?tck=docs_server) .

La descarga incluye los siguientes recursos:

* Presentación sobre la metodología de modelado de datos con MongoDB
* Documento técnico que cubre las mejores prácticas y consideraciones para migrar a MongoDB desde un modelo de datos [RDBMS](https://docs.mongodb.com/manual/reference/glossary/#std-term-RDBMS)
* Hacer referencia al esquema MongoDB con su equivalente RDBMS
* Cuadro de mando de modernización de aplicaciones

