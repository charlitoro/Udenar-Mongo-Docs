# Versiones de MongoDB

**MongoDB 2.2**

Se ven cambios significativos en la estandarización de preferencias de lectura en los controladores de MongoDB, también hace posible garantizar que los datos en un clúster fragmentado distribuido geográficamente siempre estén más cerca de la aplicación que utilizará más esos datos, también tiene mejoras en concurrencia, este mejora el rendimiento en fallas de bloqueos de página, mejora el paralelismo en la aplicación de escrituras en secundarias.

**MongoDB 2.4**

Contiene nuevas características como la de búsqueda de contenido de texto en las bases de datos MongoDB, mejoras en soporte geoespacial, además de poder agregar un nuevo [índice hash](https://docs.mongodb.com/manual/core/index-hashed/#index-type-hashed) a los documentos de índice utilizando hashes de valores de campo, también se añaden cambios a los operadores de actualización, y las limitaciones adicionales para Map-Reduce y `$where`Operaciones, además de agregar mejores en rendimiento notorias.

**MongoDB 2.6**

En esta versión se evidencian mejoras de agregación, otorgando la capacidad de devolver conjuntos de resultados de cualquier tamaño, ya sea devolviendo un cursor o escribiendo la salida en una colección. Además, la canalización de agregación admite variables y agrega nuevas operaciones para manejar conjuntos y redactar datos, la búsqueda de texto ahora está habilitada de manera predeterminado, mejoras en los sistemas de actualización e inserción incluyen operaciones adicionales y mejoras que aumentan la consistencia de los datos modificados.

**MongoDB 3.0**

Presenta una API de motor de almacenamiento conectable que permite a terceros desarrollar motores de almacenamiento para MongoDB,  tiene un nuevo soporte para el motor de almacenamiento **WiredTiger** que admite todas las funciones de MongoDB, incluidas las operaciones que informan sobre el servidor, la base de datos y las estadísticas de recopilación, MongoDB 3.0 ya no implementa la asignación dinámica de registros y desprecia **paddingFactor,** y en cuanto a los conjuntos de réplicas pueden tener hasta 50 miembros.

**MongoDB 3.2**

A partir de MongoDB 3.2, MongoDB reduce el tiempo de conmutación por error del conjunto de réplicas y acelera la detección de múltiples primarios simultáneos, los servidores de configuración para un clúster fragmentado se pueden implementar como un conjunto de réplica. Los servidores de configuración del conjunto de réplicas deben ejecutar el motor de almacenamiento **WiredTiger**, MongoDB 3.2 presenta la `readConcern`opción de consulta para conjuntos de réplicas y fragmentos de conjuntos de réplica , además de que en esta versión brinda la capacidad de validar documentos durante las actualizaciones e inserciones. Las reglas de validación se especifican por colección.

**MongoDB 3.4**

Clústeres fragmentados ya no admiten el uso de **mongod** instancias duplicadas \(SCCC\) como servidores de configuración, llega **Sharding Zones** que reemplaza los fragmentos de reconocimiento de etiquetas disponibles en versiones anteriores,  además de que  MongoDB puede realizar migraciones de fragmentos paralelos. Al igual que en las versiones anteriores, un fragmento puede participar en una migración como máximo a la vez. Al observar esta restricción, para un clúster fragmentado con _n_ fragmentos, MongoDB puede realizar a lo sumo _n / 2_ \(redondeadas hacia abajo\) migraciones fragmentarias simultáneas.

**MongoDB 3.6**

MongoDB agrega nuevas etapas de canalización de agregación, a esto le agrega un ayudante [`db.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.aggregate/#db.aggregate)para realizar agregaciones que no dependen de una colección subyacente, como las que comienzan con [`$currentOp`](https://docs.mongodb.com/manual/reference/operator/aggregation/currentOp/#pipe._S_currentOp)o [`$listLocalSessions`](https://docs.mongodb.com/manual/reference/operator/aggregation/listLocalSessions/#pipe._S_listLocalSessions)

Se crea una nueva variable de agregación **Remove**  permite la exclusión condicional de un campo, agrega soporte para zonas horarias a los operadores de fecha de agregación.

**MongoDB 4.0**

A partir de la versión 4.0, MongoDB desprecia el motor de almacenamiento MMAPv1**,** elimina el soporte para la replicación maestro-esclavo en desuso. Antes de poder actualizar a MongoDB 4.0, si su implementación utiliza la replicación maestro-esclavo, debe actualizar a un conjunto de réplicas.

\*\*\*\*

