# Introducción al Modelado de Datos

El desafío clave en el modelado de datos es equilibrar las necesidades de la aplicación, las características de rendimiento del motor de la base de datos y los patrones de recuperación de datos. Al diseñar modelos de datos, siempre tenga en cuenta el uso de los datos por parte de la aplicación \(es decir, consultas, actualizaciones y procesamiento de los datos\), así como la estructura inherente de los datos en sí.

### Esquema flexible  <a id="flexible-schema"></a>

A diferencia de las bases de datos SQL, donde debe determinar y declarar el esquema de una tabla antes de insertar datos, las [colecciones](https://docs.mongodb.com/manual/reference/glossary/#std-term-collection) de MongoDB , por defecto, no requieren que sus [documentos](https://docs.mongodb.com/manual/core/document/) tengan el mismo esquema. Es decir:

* No es necesario que los documentos de una sola colección tengan el mismo conjunto de campos y el tipo de datos de un campo puede diferir entre los documentos de una colección.
* Para cambiar la estructura de los documentos en una colección, como agregar nuevos campos, eliminar campos existentes o cambiar los valores de campo a un nuevo tipo, actualice los documentos a la nueva estructura.

Esta flexibilidad facilita la asignación de documentos a una entidad o un objeto. Cada documento puede coincidir con los campos de datos de la entidad representada, incluso si el documento tiene una variación sustancial de otros documentos de la colección.

Sin embargo, en la práctica, los documentos de una colección comparten una estructura similar y puede aplicar [reglas de validación de documentos](https://docs.mongodb.com/manual/core/schema-validation/) para una colección durante las operaciones de actualización e inserción. Consulte [Validación de esquema](https://docs.mongodb.com/manual/core/schema-validation/) para obtener más detalles.

### Estructura del documento  <a id="document-structure"></a>

La decisión clave en el diseño de modelos de datos para aplicaciones MongoDB gira en torno a la estructura de los documentos y cómo la aplicación representa las relaciones entre los datos. MongoDB permite incrustar datos relacionados en un solo documento.

#### Datos incrustados  <a id="embedded-data"></a>

Los documentos integrados capturan las relaciones entre los datos al almacenar los datos relacionados en una estructura de documento única. Los documentos de MongoDB permiten incrustar estructuras de documentos en un campo o matriz dentro de un documento. Estos modelos de datos _desnormalizados_ permiten que las aplicaciones recuperen y manipulen datos relacionados en una sola operación de base de datos.![Modelo de datos con campos incrustados que contienen toda la informaci&#xF3;n relacionada.](https://docs.mongodb.com/manual/images/data-model-denormalized.bakedsvg.svg)Click para agrandar

Para muchos casos de uso en MongoDB, el modelo de datos desnormalizado es óptimo.

Consulte [Modelos de datos integrados](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) para conocer las fortalezas y debilidades de la inserción de documentos.

#### Referencias  <a id="references"></a>

Las referencias almacenan las relaciones entre los datos al incluir enlaces o _referencias_ de un documento a otro. Las aplicaciones pueden resolver estas [referencias](https://docs.mongodb.com/manual/reference/database-references/) para acceder a los datos relacionados. En términos generales, estos son modelos de datos _normalizados_ .![Modelo de datos utilizando referencias para vincular documentos.  Tanto el documento de \`\` contacto &apos;&apos; como el documento de \`\` acceso &apos;&apos; contienen una referencia al documento de \`\` usuario &apos;&apos;.](https://docs.mongodb.com/manual/images/data-model-normalized.bakedsvg.svg)Click para agrandar

Consulte [Modelos de datos normalizados](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-referencing) para conocer los puntos fuertes y débiles del uso de referencias.

### Atomicidad de las operaciones de escritura  <a id="atomicity-of-write-operations"></a>

#### Atomicidad de documento único  <a id="single-document-atomicity"></a>

En MongoDB, una operación de escritura es atómica al nivel de un solo documento, incluso si la operación modifica varios documentos incrustados _dentro de_ un solo documento.

Un modelo de datos desnormalizado con datos incrustados combina todos los datos relacionados en un solo documento en lugar de normalizarlos en múltiples documentos y colecciones. Este modelo de datos facilita las operaciones atómicas.

Cuando una sola operación de escritura \(por ejemplo [`db.collection.updateMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#mongodb-method-db.collection.updateMany)\) modifica varios documentos, la modificación de cada documento es atómica, pero la operación en su conjunto no es atómica.

Al realizar operaciones de escritura de varios documentos, ya sea a través de una sola operación de escritura o de varias operaciones de escritura, otras operaciones pueden intercalarse.

Para situaciones que requieren atomicidad de lecturas y escrituras en múltiples documentos \(en una o varias colecciones\), MongoDB admite transacciones de múltiples documentos:

* **En la versión 4.0** , MongoDB admite transacciones de múltiples documentos en conjuntos de réplicas.
* **En la versión 4.2** , MongoDB introduce transacciones distribuidas, que agrega soporte para transacciones de múltiples documentos en clústeres fragmentados e incorpora el soporte existente para transacciones de múltiples documentos en conjuntos de réplicas.

Para obtener detalles sobre las transacciones en MongoDB, consulte la página [Transacciones](https://docs.mongodb.com/manual/core/transactions/) .

#### Transacciones de varios documentos  <a id="multi-document-transactions"></a>

Para situaciones que requieren atomicidad de lecturas y escrituras en múltiples documentos \(en una o varias colecciones\), MongoDB admite transacciones de múltiples documentos:

* **En la versión 4.0** , MongoDB admite transacciones de múltiples documentos en conjuntos de réplicas.
* **En la versión 4.2** , MongoDB introduce transacciones distribuidas, que agrega soporte para transacciones de múltiples documentos en clústeres fragmentados e incorpora el soporte existente para transacciones de múltiples documentos en conjuntos de réplicas.

Para obtener detalles sobre las transacciones en MongoDB, consulte la página [Transacciones](https://docs.mongodb.com/manual/core/transactions/) .IMPORTANTE

En la mayoría de los casos, la transacción de múltiples documentos genera un mayor costo de rendimiento que la escritura de un solo documento, y la disponibilidad de transacciones de múltiples documentos no debe reemplazar el diseño de esquemas efectivo. Para muchos escenarios, el [modelo de datos desnormalizados \(documentos y matrices incrustados\)](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) seguirá siendo óptimo para sus datos y casos de uso. Es decir, para muchos escenarios, modelar sus datos de manera adecuada minimizará la necesidad de transacciones de múltiples documentos.

Para consideraciones adicionales sobre el uso de transacciones \(como el límite de tiempo de ejecución y el límite de tamaño del registro de operaciones\), consulte también [Consideraciones de producción](https://docs.mongodb.com/manual/core/transactions-production-consideration/) .CONSEJOVer también:

[Consideraciones de atomicidad](https://docs.mongodb.com/manual/core/data-model-operations/#std-label-data-model-atomicity)

### Uso y rendimiento de datos  <a id="data-use-and-performance"></a>

Al diseñar un modelo de datos, considere cómo las aplicaciones utilizarán su base de datos. Por ejemplo, si su aplicación solo usa documentos insertados recientemente, considere usar [Colecciones limitadas](https://docs.mongodb.com/manual/core/capped-collections/) . O si las necesidades de su aplicación son principalmente operaciones de lectura en una colección, agregar índices para admitir consultas comunes puede mejorar el rendimiento.

Consulte [Factores operativos y modelos de datos](https://docs.mongodb.com/manual/core/data-model-operations/) para obtener más información sobre estas y otras consideraciones operativas que afectan los diseños de modelos de datos.

### Más información  <a id="learn-more"></a>

#### Presentaciones de MongoDB.live 2020  <a id="mongodb.live-2020-presentations"></a>

Para aprender cómo incorporar el modelo de datos flexible en su esquema, consulte las siguientes presentaciones de **MongoDB.live 2020** :

* Obtenga información sobre las relaciones entre entidades en MongoDB y ejemplos de sus implementaciones con el [modelado de datos con MongoDB](https://www.mongodb.com/presentations/data-modeling-with-mongodb) .
* Aprenda patrones de diseño de modelado de datos avanzados que puede incorporar a su esquema con los [patrones de diseño de esquemas avanzados](https://www.mongodb.com/presentations/advanced-schema-design-patterns) .

#### Guía de modernización de aplicaciones  <a id="application-modernization-guide"></a>

Para obtener más información sobre el modelado de datos con MongoDB, descargue la [Guía de modernización de aplicaciones de MongoDB](https://www.mongodb.com/modernize?tck=docs_server) .

La descarga incluye los siguientes recursos:

* Presentación sobre la metodología de modelado de datos con MongoDB
* Documento técnico que cubre las mejores prácticas y consideraciones para migrar a MongoDB desde un modelo de datos [RDBMS](https://docs.mongodb.com/manual/reference/glossary/#std-term-RDBMS)
* Hacer referencia al esquema MongoDB con su equivalente RDBMS
* Cuadro de mando de modernización de aplicaciones

