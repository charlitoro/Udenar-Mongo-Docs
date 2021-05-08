# Índices

Los índices admiten la ejecución eficiente de consultas en MongoDB. Sin índices, MongoDB debe realizar un _escaneo de colección_ , es decir, escanear cada documento en una colección, para seleccionar aquellos documentos que coincidan con la declaración de consulta. Si existe un índice apropiado para una consulta, MongoDB puede usar el índice para limitar la cantidad de documentos que debe inspeccionar.

Los índices son estructuras de datos especiales [\[ 1 \]](https://docs.mongodb.com/manual/indexes/#footnote-b-tree) que almacenan una pequeña parte del conjunto de datos de la colección en una forma fácil de recorrer. El índice almacena el valor de un campo específico o un conjunto de campos, ordenados por el valor del campo. El orden de las entradas de índice admite coincidencias de igualdad eficientes y operaciones de consulta basadas en rangos. Además, MongoDB puede devolver resultados ordenados utilizando el orden en el índice.

El siguiente diagrama ilustra una consulta que selecciona y ordena los documentos coincidentes mediante un índice:![Diagrama de una consulta que usa un &#xED;ndice para seleccionar y devolver resultados ordenados.  El &#xED;ndice almacena valores de \`\` puntuaci&#xF3;n &apos;&apos; en orden ascendente.  MongoDB puede recorrer el &#xED;ndice en orden ascendente o descendente para devolver resultados ordenados.](https://docs.mongodb.com/manual/images/index-for-sort.bakedsvg.svg)

Básicamente, los índices en MongoDB son similares a los índices en otros sistemas de bases de datos. MongoDB define índices a nivel de [colección](https://docs.mongodb.com/manual/reference/glossary/#std-term-collection) y admite índices en cualquier campo o subcampo de los documentos de una colección MongoDB.

### `_id`Índice predeterminado  <a id="default-_id-index"></a>

MongoDB crea un [índice único](https://docs.mongodb.com/manual/core/index-unique/#std-label-index-type-unique) en el campo [\_id](https://docs.mongodb.com/manual/core/document/#std-label-document-id-field) durante la creación de una colección. El `_id`índice evita que los clientes inserten dos documentos con el mismo valor para el `_id`campo. No puede colocar este índice en el `_id`campo.NOTA

En [clústeres fragmentados](https://docs.mongodb.com/manual/reference/glossary/#std-term-sharded-cluster) , si _no_ usa el `_id`campo como [clave de fragmento](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) , su aplicación **debe** garantizar la unicidad de los valores en el `_id`campo para evitar errores. Esto se hace con mayor frecuencia mediante el uso de un [ObjectId](https://docs.mongodb.com/manual/reference/glossary/#std-term-ObjectId) estándar generado automáticamente .

### Crear un índice  <a id="create-an-index"></a>

➤ Utilice el menú desplegable **Seleccione su idioma** en la parte superior derecha para establecer el idioma de los ejemplos en esta página.

Para crear un índice en [Mongo Shell](https://docs.mongodb.com/manual/tutorial/getting-started/) , use [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex).

```text
db.collection.createIndex( <key and index type specification>, <options> )
```

El siguiente ejemplo crea un índice descendente de clave única en el `name`campo:

```text
db.collection.createIndex( { name: -1 } )
```

El [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método solo crea un índice si aún no existe un índice de la misma especificación.

| \[ [1](https://docs.mongodb.com/manual/indexes/#ref-b-tree-id2) \] | Los índices de MongoDB utilizan una estructura de datos de árbol B. |
| :--- | :--- |


#### Nombres de índice  <a id="index-names"></a>

El nombre predeterminado para un índice es la concatenación de las claves indexadas y la dirección de cada clave en el índice \(es decir, 1 o -1\) utilizando guiones bajos como separador. Por ejemplo, un índice creado en `{ item : 1, quantity: -1 }`tiene el nombre `item_1_quantity_-1`.

Puede crear índices con un nombre personalizado, como uno que sea más legible por humanos que el predeterminado. Por ejemplo, considere una aplicación que consulta con frecuencia la `products`colección para completar datos en el inventario existente. El siguiente [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) método crea un índice `item`y se `quantity`nombra `query for inventory`:

```text
db.products.createIndex(  { item: 1, quantity: -1 } ,  { name: "query for inventory" })
```

Puede ver los nombres de los índices utilizando el [`db.collection.getIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.getIndexes/#mongodb-method-db.collection.getIndexes) método. No puede cambiar el nombre de un índice una vez creado. En su lugar, debe eliminar y volver a crear el índice con un nombre nuevo.

### Tipos de índice  <a id="index-types"></a>

MongoDB proporciona varios tipos de índices diferentes para admitir tipos específicos de datos y consultas.

#### Campo único  <a id="single-field"></a>

Además del `_id`índice definido por MongoDB, MongoDB admite la creación de índices ascendentes / descendentes definidos por el usuario en un [solo campo de un documento](https://docs.mongodb.com/manual/core/index-single/) .![Diagrama de un &#xED;ndice en el campo \`\` puntuaci&#xF3;n &apos;&apos; \(ascendente\).](https://docs.mongodb.com/manual/images/index-ascending.bakedsvg.svg)

Para un índice de un solo campo y operaciones de clasificación, el orden de clasificación \(es decir, ascendente o descendente\) de la clave de índice no importa porque MongoDB puede atravesar el índice en cualquier dirección.

Consulte [Índices de campo único](https://docs.mongodb.com/manual/core/index-single/) y [Ordenar con un índice de campo único](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/#std-label-sort-results-single-field) para obtener más información sobre índices de campo único.

#### Índice compuesto  <a id="compound-index"></a>

MongoDB también admite índices definidos por el usuario en varios campos, es decir [, índices compuestos](https://docs.mongodb.com/manual/core/index-compound/) .

El orden de los campos enumerados en un índice compuesto tiene importancia. Por ejemplo, si un índice compuesto consta de `{ userid: 1, score: -1 }`, el índice ordena primero por `userid`y luego, dentro de cada `userid` valor, ordena por `score`.![Diagrama de un &#xED;ndice compuesto en el campo \`\` userid &apos;&apos; \(ascendente\) y el campo \`\` score &apos;&apos; \(descendente\).  El &#xED;ndice ordena primero por el campo \`\` ID de usuario &apos;&apos; y luego por el campo \`\` puntuaci&#xF3;n &apos;&apos;.](https://docs.mongodb.com/manual/images/index-compound-key.bakedsvg.svg)

Para índices compuestos y operaciones de clasificación, el orden de clasificación \(es decir, ascendente o descendente\) de las claves de índice puede determinar si el índice puede admitir una operación de clasificación. Consulte [Orden de clasificación](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-ascending-and-descending) para obtener más información sobre el impacto del orden de índice en los resultados de los índices compuestos.

Consulte [Índices compuestos](https://docs.mongodb.com/manual/core/index-compound/) y [Ordenar en varios campos](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/#std-label-sort-on-multiple-fields) para obtener más información sobre índices compuestos.

#### Índice Multikey  <a id="multikey-index"></a>

MongoDB utiliza [índices de varias claves](https://docs.mongodb.com/manual/core/index-multikey/) para indexar el contenido almacenado en matrices. Si indexa un campo que contiene un valor de matriz, MongoDB crea entradas de índice independientes para _cada_ elemento de la matriz. Estos [índices de varias claves](https://docs.mongodb.com/manual/core/index-multikey/) permiten que las consultas seleccionen documentos que contienen matrices haciendo coincidir el elemento o elementos de las matrices. MongoDB determina automáticamente si se debe crear un índice de varias claves si el campo indexado contiene un valor de matriz; no es necesario especificar explícitamente el tipo de múltiples claves.![Diagrama de un &#xED;ndice de varias claves en el campo \`\` addr.zip &apos;&apos;.  El campo \`\` addr &apos;&apos; contiene una serie de documentos de direcci&#xF3;n.  Los documentos de direcci&#xF3;n contienen el campo \`\` zip &apos;&apos;.](https://docs.mongodb.com/manual/images/index-multikey.bakedsvg.svg)

Consulte [Índices de múltiples claves](https://docs.mongodb.com/manual/core/index-multikey/) y [Límites](https://docs.mongodb.com/manual/core/multikey-index-bounds/) de índices de múltiples claves para obtener más información sobre los índices de múltiples claves.

#### Índice geoespacial  <a id="geospatial-index"></a>

Para admitir consultas eficientes de datos de coordenadas geoespaciales, MongoDB proporciona dos índices especiales: índices [2d](https://docs.mongodb.com/manual/core/2d/) que usan geometría plana al devolver resultados e [índices 2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) que usan geometría esférica para devolver resultados.

Consulte [`2d`Index Internals](https://docs.mongodb.com/manual/core/geospatial-indexes/) para obtener una introducción de alto nivel a los índices geoespaciales.

#### Índices de texto  <a id="text-indexes"></a>

MongoDB proporciona un `text`tipo de índice que admite la búsqueda de contenido de cadena en una colección. Estos índices de texto no almacenan palabras _vacías_ específicas del idioma \(por ejemplo, "el", "a", "o"\) y _derivan_ las palabras en una colección para almacenar solo palabras raíz.

Consulte [Índices de texto](https://docs.mongodb.com/manual/core/index-text/) para obtener más información sobre los índices de texto y la búsqueda.

#### Índices hash  <a id="hashed-indexes"></a>

Para admitir la [fragmentación basada en hash](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding) , MongoDB proporciona un tipo de [índice hash](https://docs.mongodb.com/manual/core/index-hashed/) , que indexa el hash del valor de un campo. Estos índices tienen una distribución de valores más aleatoria a lo largo de su rango, pero _solo_ admiten coincidencias de igualdad y no pueden admitir consultas basadas en rangos.

### Propiedades del índice  <a id="index-properties"></a>

#### Índices únicos  <a id="unique-indexes"></a>

La propiedad [única](https://docs.mongodb.com/manual/core/index-unique/) de un índice hace que MongoDB rechace valores duplicados para el campo indexado. Aparte de la restricción única, los índices únicos son funcionalmente intercambiables con otros índices de MongoDB.

#### Índices parciales  <a id="partial-indexes"></a>

_Nuevo en la versión 3.2_ .

[Los índices parciales](https://docs.mongodb.com/manual/core/index-partial/) solo indexan los documentos de una colección que cumplen una expresión de filtro especificada. Al indexar un subconjunto de los documentos de una colección, los índices parciales tienen menores requisitos de almacenamiento y menores costos de rendimiento para la creación y el mantenimiento de índices.

Los índices parciales ofrecen un superconjunto de la funcionalidad de los índices dispersos y deberían preferirse a los índices dispersos.

#### Índices  <a id="sparse-indexes"></a>

La propiedad [dispersa](https://docs.mongodb.com/manual/core/index-sparse/) de un índice asegura que el índice solo contenga entradas para documentos que tienen el campo indexado. El índice omite los documentos que _no_ tienen el campo indexado.

Puede combinar la opción de índice disperso con la opción de índice único para evitar la inserción de documentos que tengan valores duplicados para los campos indexados y omitir los documentos de indexación que carecen de los campos indexados.

#### Índices TTL  <a id="ttl-indexes"></a>

[Los índices TTL](https://docs.mongodb.com/manual/core/index-ttl/) son [índices](https://docs.mongodb.com/manual/core/index-ttl/) especiales que MongoDB puede usar para eliminar automáticamente documentos de una colección después de un cierto período de tiempo. Esto es ideal para ciertos tipos de información, como datos de eventos generados por máquinas, registros e información de sesiones que solo necesitan persistir en una base de datos durante un período de tiempo finito.

Consulte: [Caducar datos de colecciones configurando TTL](https://docs.mongodb.com/manual/tutorial/expire-data/) para obtener instrucciones de implementación.

#### Índices ocultos  <a id="hidden-indexes"></a>

_Nuevo en la versión 4.4_ .

[Los índices ocultos](https://docs.mongodb.com/manual/core/index-hidden/) no son visibles para el [planificador de consultas](https://docs.mongodb.com/manual/core/query-plans/) y no se pueden utilizar para admitir una consulta.

Al ocultar un índice al planificador, los usuarios pueden evaluar el impacto potencial de eliminar un índice sin eliminarlo realmente. Si el impacto es negativo, el usuario puede mostrar el índice en lugar de tener que volver a crear un índice eliminado. Y debido a que los índices se mantienen completamente mientras están ocultos, los índices están inmediatamente disponibles para su uso una vez que se muestran.

A excepción del `_id`índice, puede ocultar cualquier índice.

### Uso del índice  <a id="index-use"></a>

Los índices pueden mejorar la eficiencia de las operaciones de lectura. El tutorial [Analizar el rendimiento de las consultas](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/) proporciona un ejemplo de las estadísticas de ejecución de una consulta con y sin índice.

Para obtener información sobre cómo MongoDB elige un índice para usar, consulte el [optimizador de consultas](https://docs.mongodb.com/manual/core/query-plans/#std-label-read-operations-query-optimization) .

### Índices y colación  <a id="indexes-and-collation"></a>

_Nuevo en la versión 3.4_ .

[La intercalación](https://docs.mongodb.com/manual/reference/collation/) permite a los usuarios especificar reglas específicas del idioma para la comparación de cadenas, como reglas para letras y tildes.

➤ Utilice el menú desplegable **Seleccione su idioma** en la parte superior derecha para establecer el idioma de los ejemplos en esta página.

Para utilizar un índice para las comparaciones de cadenas, una operación también debe especificar la misma intercalación. Es decir, un índice con una intercalación no puede admitir una operación que realiza comparaciones de cadenas en los campos indexados si la operación especifica una intercalación diferente.

Por ejemplo, la colección `myColl`tiene un índice en un campo de cadena `category`con la configuración regional de clasificación `"fr"`.

```text
db.myColl.createIndex( { category: 1 }, { collation: { locale: "fr" } } )
```

La siguiente operación de consulta, que especifica la misma intercalación que el índice, puede usar el índice:

```text
db.myColl.find( { category: "cafe" } ).collation( { locale: "fr" } )
```

Sin embargo, la siguiente operación de consulta, que de forma predeterminada utiliza el intercalador binario "simple", no puede utilizar el índice:

```text
db.myColl.find( { category: "cafe" } )
```

Para un índice compuesto donde las claves de prefijo de índice no son cadenas, matrices ni documentos incrustados, una operación que especifica una intercalación diferente aún puede usar el índice para admitir comparaciones en las claves de prefijo de índice.

Por ejemplo, la colección `myColl`tiene un índice compuesto sobre los campos numéricos `score`y `price`y el campo de cadena `category`; el índice se crea con la configuración regional de intercalación `"fr"`para las comparaciones de cadenas:

```text
db.myColl.createIndex(   { score: 1, price: 1, category: 1 },   { collation: { locale: "fr" } } )
```

Las siguientes operaciones, que utilizan la `"simple"`intercalación binaria para las comparaciones de cadenas, pueden utilizar el índice:

```text
db.myColl.find( { score: 5 } ).sort( { price: 1 } )db.myColl.find( { score: 5, price: { $gt: NumberDecimal( "10" ) } } ).sort( { price: 1 } )
```

La siguiente operación, que usa la `"simple"`intercalación binaria para las comparaciones de cadenas en el `category`campo indexado , puede usar el índice para completar solo la `score: 5`parte de la consulta:

```text
db.myColl.find( { score: 5, category: "cafe" } )
```

Para obtener más información sobre la intercalación, consulte la [página de referencia de](https://docs.mongodb.com/manual/reference/collation/) la [intercalación](https://docs.mongodb.com/manual/reference/collation/) .

Los siguientes índices solo admiten la comparación binaria simple y no admiten la [intercalación](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation) :

* índices de [texto](https://docs.mongodb.com/manual/core/index-text/) ,
* Índices [2d](https://docs.mongodb.com/manual/core/2d/) , y
* índices [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/) .

### Consultas cubiertas  <a id="covered-queries"></a>

Cuando los criterios de consulta y la [proyección](https://docs.mongodb.com/manual/reference/glossary/#std-term-projection) de una consulta incluyen _solo_ los campos indexados, MongoDB devuelve los resultados directamente del índice _sin_ escanear ningún documento ni traer documentos a la memoria. Estas consultas cubiertas pueden ser _muy_ eficientes.![Diagrama de una consulta que usa solo el &#xED;ndice para coincidir con los criterios de la consulta y devolver los resultados.  MongoDB no necesita inspeccionar datos fuera del &#xED;ndice para completar la consulta.](https://docs.mongodb.com/manual/images/index-for-covered-query.bakedsvg.svg)

Para obtener más información sobre consultas cubiertas, consulte [Consulta cubierta](https://docs.mongodb.com/manual/core/query-optimization/#std-label-read-operations-covered-query) .

### Intersección de índice  <a id="index-intersection"></a>

MongoDB puede usar la [intersección de índices](https://docs.mongodb.com/manual/core/index-intersection/) para completar consultas. Para consultas que especifican condiciones de consulta compuestas, si un índice puede cumplir una parte de una condición de consulta y otro índice puede cumplir otra parte de la condición de consulta, entonces MongoDB puede usar la intersección de los dos índices para cumplir con la consulta. Si el uso de un índice compuesto o el uso de una intersección de índices es más eficiente depende de la consulta en particular y del sistema.

Para obtener detalles sobre la intersección de índices, consulte [Intersección de índices](https://docs.mongodb.com/manual/core/index-intersection/) .

### Restricciones  <a id="restrictions"></a>

Se aplican ciertas restricciones a los índices, como la longitud de las claves de índice o el número de índices por colección. Consulte [Limitaciones del índice](https://docs.mongodb.com/manual/reference/limits/#std-label-index-limitations) para obtener más detalles.

### Consideraciones adicionales  <a id="additional-considerations"></a>

Aunque los índices pueden mejorar el rendimiento de las consultas, los índices también presentan algunas consideraciones operativas. Consulte [Consideraciones operativas para índices](https://docs.mongodb.com/manual/core/data-model-operations/#std-label-data-model-indexes) para obtener más información.

Las aplicaciones pueden encontrar un rendimiento reducido durante la creación de índices, incluido el acceso limitado de lectura / escritura a la colección. Para obtener más información sobre el proceso de generación de índices, consulte [Generaciones de índices en colecciones pobladas](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations) , incluida la sección [Generaciones de índices en entornos replicados](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations-replicated-build) .

Algunos controladores pueden especificar índices, utilizando en `NumberLong(1)`lugar de `1`como especificación. Esto no tiene ningún efecto sobre el índice resultante.

