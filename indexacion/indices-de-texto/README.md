# Índices de texto

### Descripción general  <a id="overview"></a>

MongoDB proporciona [índices de texto](https://docs.mongodb.com/manual/core/index-text/#std-label-index-feature-text) para admitir consultas de búsqueda de texto sobre contenido de cadenas. `text`los índices pueden incluir cualquier campo cuyo valor sea una cadena o una matriz de elementos de cadena.

### Versiones  <a id="versions"></a>

| `text` Versión del índice | Descripción |
| :--- | :--- |
| Versión 3 | MongoDB presenta una versión 3 del `text`índice. La versión 3 es la versión predeterminada de los `text`índices creados en MongoDB 3.2 y posteriores. |
| Versión 2 | MongoDB 2.6 presenta una versión 2 del `text`índice. La versión 2 es la versión predeterminada de los `text`índices creados en las series MongoDB 2.6 y 3.0. |
| Versión 1 | MongoDB 2.4 presenta una versión 1 del `text`índice. MongoDB 2.4 solo admite la versión `1`. |

Para anular la versión predeterminada y especificar una versión diferente, incluya la opción `{ "textIndexVersion": <version> }`al crear el índice.

### Crear índice de texto  <a id="create-text-index"></a>

IMPORTANTE

Una colección puede tener como máximo **un** `text` índice.

Atlas Search \(disponible en [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) \) admite múltiples índices de búsqueda de texto completo en una sola colección. Para obtener más información, consulte la [documentación de Atlas Search](https://docs.atlas.mongodb.com/atlas-search/) .

Para crear un `text`índice, use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método. Para indexar un campo que contiene una cadena o una matriz de elementos de cadena, incluya el campo y especifique el literal de cadena `"text"`en el documento de índice, como en el siguiente ejemplo:

```text
db.reviews.createIndex( { comments: "text" } )
```

Puede indexar varios campos para el `text`índice. El siguiente ejemplo crea un `text`índice en los campos `subject`y `comments`:

```text
db.reviews.createIndex(   {     subject: "text",     comments: "text"   } )
```

Un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/) puede incluir `text` claves de índice en combinación con claves de índice ascendente / descendente. Para obtener más información, consulte [Índice compuesto](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-compound) .

Para eliminar un `text`índice, use el nombre del índice. Consulte [Usar el nombre del índice para eliminar un `text`índice](https://docs.mongodb.com/manual/tutorial/avoid-text-index-name-limit/#std-label-drop-text-index) para obtener más información.

#### Especificar pesos  <a id="specify-weights"></a>

Para un `text`índice, el _peso_ de un campo indexado denota la importancia del campo en relación con los otros campos indexados en términos de la puntuación de búsqueda de texto.

Para cada campo indexado en el documento, MongoDB multiplica el número de coincidencias por el peso y suma los resultados. Con esta suma, MongoDB calcula la puntuación del documento. Consulte al [`$meta`](https://docs.mongodb.com/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta) operador para obtener detalles sobre la devolución y la clasificación por puntajes de texto.

El peso predeterminado es 1 para los campos indexados. Para ajustar los pesos de los campos indexados, incluya la `weights`opción en el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método.

Para obtener más información sobre el uso de ponderaciones para controlar los resultados de una búsqueda de texto, consulte [Controlar](https://docs.mongodb.com/manual/tutorial/control-results-of-text-search/) los resultados de la búsqueda con ponderaciones .

#### Índices de texto comodín  <a id="wildcard-text-indexes"></a>

NOTA

Los índices de texto comodín son distintos de los [índices comodín](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-index-core) . Los índices comodín no pueden admitir consultas mediante el [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text) operador.

Si bien los índices de texto comodín y los índices [comodín](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-index-core) comparten el `$**`patrón de campo comodín , son tipos de índices distintos. Solo los índices de texto comodín admiten el [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)operador.

Al crear un `text`índice en varios campos, también puede utilizar el especificador comodín \( `$**`\). Con un índice de texto comodín, MongoDB indexa todos los campos que contienen datos de cadena para cada documento de la colección. El siguiente ejemplo crea un índice de texto utilizando el especificador comodín:

```text
db.collection.createIndex( { "$**": "text" } )
```

Este índice permite la búsqueda de texto en todos los campos con contenido de cadena. Un índice de este tipo puede ser útil con datos muy desestructurados si no está claro qué campos incluir en el índice de texto o para consultas ad-hoc.

Los índices de texto comodín son `text`índices de varios campos. Como tal, puede asignar pesos a campos específicos durante la creación del índice para controlar la clasificación de los resultados. Para obtener más información sobre el uso de ponderaciones para controlar los resultados de una búsqueda de texto, consulte [Controlar](https://docs.mongodb.com/manual/tutorial/control-results-of-text-search/) los resultados de la búsqueda con ponderaciones .

Los índices de texto comodín, como con todos los índices de texto, pueden formar parte de índices compuestos. Por ejemplo, lo siguiente crea un índice compuesto en el campo `a`, así como el especificador comodín:

```text
db.collection.createIndex( { a: 1, "$**": "text" } )
```

Al igual que con todos [los índices de texto compuestos](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-compound) , dado que `a`precede a la clave del índice de texto, para realizar una [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)búsqueda con este índice, el predicado de consulta debe incluir condiciones de coincidencia de igualdad `a`. Para obtener información sobre índices de texto compuesto, consulte [Índices de texto compuesto](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-compound) .

### Insensibilidad a  minúsculas  <a id="case-insensitivity"></a>

_Modificado en la versión 3.2_ .

El `text`índice de la versión 3 admite los idiomas comunes `C`, simples `S`y para los idiomas turcos, los `T`pliegues de casos especiales como se especifica en [Pliegue de casos de base de datos de caracteres Unicode 8.0](http://www.unicode.org/Public/8.0.0/ucd/CaseFolding.txt) .

El pliegue de mayúsculas y minúsculas expande la insensibilidad entre mayúsculas y minúsculas del `text` índice para incluir caracteres con diacríticos, como `é`y `É`, y caracteres de alfabetos no latinos, como "И" y "и" en el alfabeto cirílico.

La versión 3 del `text`índice también es [insensible a los diacríticos](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-diacritic-insensitivity) . Como tal, el índice también no distingue entre `é`, `É`, `e`, y `E`.

Las versiones anteriores del `text`índice no distinguen entre mayúsculas y minúsculas `[A-z]`solo para ; es decir, no distingue entre mayúsculas y minúsculas solo para caracteres latinos no diacríticos. Para todos los demás caracteres, las versiones anteriores del índice de texto los tratan como distintos.

### Insensibilidad diacrítica  <a id="diacritic-insensitivity"></a>

_Modificado en la versión 3.2_ .

Con la versión 3, el `text`índice es insensible a los diacríticos. Es decir, el índice no distingue entre caracteres que contienen signos diacríticos y su homólogo no marcado, como por ejemplo `é`, `ê`y `e`. Más específicamente, el `text`índice elimina los caracteres categorizados como signos diacríticos en la [lista de accesorios de la base de datos de caracteres Unicode 8.0](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt) .

La versión 3 del `text`índice tampoco [distingue](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-case-insensitivity) entre [mayúsculas](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-case-insensitivity) y [minúsculas](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-case-insensitivity) para los caracteres con diacríticos. Como tal, el índice también no distingue entre `é`, `É`, `e`, y `E`.

Las versiones anteriores del `text`índice tratan los caracteres con signos diacríticos como distintos.

### Delimitadores de tokenización  <a id="tokenization-delimiters"></a>

_Modificado en la versión 3.2_ .

Para tokenización, versión 3 `text`índice utiliza los delimitadores categorizados bajo `Dash`, `Hyphen`, `Pattern_Syntax`, `Quotation_Mark`, `Terminal_Punctuation`, y `White_Space`en [Unicode 8.0 Base de Datos de Carácter Lista Prop](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt) .

Por ejemplo, si se les da una cadena `"Il a dit qu'il «était le meilleur joueur du monde»"`, las `text`golosinas de índice `«`, `»`y espacios como delimitadores.

Las versiones anteriores del índice se tratan `«`como parte del término `"«était"`y `»`como parte del término `"monde»"`.

### Entradas de índice  <a id="index-entries"></a>

`text`index tokeniza y deriva los términos en los campos indexados para las entradas del índice. `text`index almacena una entrada de índice para cada término derivado único en cada campo indexado para cada documento de la colección. El índice utiliza [un](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-supported-languages) sufijo simple [específico del lenguaje](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-supported-languages) .

### Idiomas admitidos y palabras  <a id="supported-languages-and-stop-words"></a>

MongoDB admite la búsqueda de texto en varios idiomas. `text`índices caen las palabras vacías específicos del idioma \(por ejemplo, en Inglés, `the`, `an`, `a`, `and`, etc.\) y un uso simple sufijo específico del idioma derivada. Para obtener una lista de los idiomas admitidos, consulte [Idiomas de búsqueda de texto](https://docs.mongodb.com/manual/reference/text-search-languages/#std-label-text-search-languages) .

Si especifica un valor de idioma de `"none"`, entonces el `text`índice usa una tokenización simple sin lista de palabras vacías ni derivación.

Para especificar un idioma para el `text`índice, consulte [Especificar un idioma para el índice de texto](https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/) .

### `sparse`Propiedad  <a id="sparse-property"></a>

`text`los índices son siempre [escasos](https://docs.mongodb.com/manual/core/index-sparse/) e ignoran la opción [escasa](https://docs.mongodb.com/manual/core/index-sparse/) . Si un documento carece de un `text`campo de índice \(o el campo es `null`una matriz vacía\), MongoDB no agrega una entrada para el documento al `text`índice. Para inserciones, MongoDB inserta el documento pero no lo agrega al `text`índice.

Para un índice compuesto que incluye una `text`clave de índice junto con claves de otros tipos, solo el `text`campo de índice determina si el índice hace referencia a un documento. Las otras claves no determinan si el índice hace referencia a los documentos o no.

### Restricciones  <a id="restrictions"></a>

#### Un índice de texto por colección  <a id="one-text-index-per-collection"></a>

Una colección puede tener como máximo **un** `text` índice.

Atlas Search \(disponible en [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) \) admite múltiples índices de búsqueda de texto completo en una sola colección. Para obtener más información, consulte la [documentación de Atlas Search](https://docs.atlas.mongodb.com/atlas-search/) .

#### Búsqueda de texto y sugerencias  <a id="text-search-and-hints"></a>

No puede usar [`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#mongodb-method-cursor.hint)si la consulta incluye una [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)expresión de consulta.

#### Índice de texto y clasificación  <a id="text-index-and-sort"></a>

Las operaciones de clasificación no pueden obtener el orden de clasificación de un `text`índice, ni siquiera de un [índice de texto compuesto](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-compound) ; es decir, las operaciones de clasificación no pueden utilizar el orden en el índice de texto.

#### Índice compuesto  <a id="compound-index"></a>

Un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/) puede incluir una `text` clave de índice en combinación con claves de índice ascendente / descendente. Sin embargo, estos índices compuestos tienen las siguientes restricciones:

* Un `text`índice compuesto no puede incluir ningún otro tipo de índice especial, como campos de índice [geoespacial](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial) o de [claves múltiples](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multi-key) .
* Si el `text`índice compuesto incluye claves que **preceden a** la `text`clave de índice, para realizar una [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)búsqueda, el predicado de consulta debe incluir **condiciones de coincidencia de igualdad** en las claves anteriores.
* Al crear un `text`índice compuesto , todas las `text`claves de índice deben enumerarse de forma adyacente en el documento de especificación del índice.

Consulte también [Índice de texto y Ordenar](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-and-sort) para conocer las limitaciones adicionales.

Para ver un ejemplo de un índice de texto compuesto, consulte [Limitar el número de entradas escaneadas](https://docs.mongodb.com/manual/tutorial/limit-number-of-items-scanned-for-text-search/) .

#### Suelta un índice de texto  <a id="drop-a-text-index"></a>

Para eliminar un `text`índice, pase el _nombre_ del índice al [`db.collection.dropIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndex/#mongodb-method-db.collection.dropIndex)método. Para obtener el nombre del índice, ejecute el [`db.collection.getIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.getIndexes/#mongodb-method-db.collection.getIndexes)método.

Para obtener información sobre el esquema de nomenclatura predeterminado para los `text`índices y sobre cómo anular el nombre predeterminado, consulte [Especificar el nombre del `text`índice](https://docs.mongodb.com/manual/tutorial/avoid-text-index-name-limit/) .

#### Opción de clasificación  <a id="collation-option"></a>

`text`los índices solo admiten la comparación binaria simple y no admiten la [intercalación](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation) .

Para crear un `text`índice en una colección que tiene una intercalación no simple, debe especificar explícitamente `{collation: {locale: "simple"} }`al crear el índice.

### Requisitos de almacenamiento y costos de rendimiento  <a id="storage-requirements-and-performance-costs"></a>

`text` Los índices tienen los siguientes requisitos de almacenamiento y costos de rendimiento:

* `text`los índices pueden ser grandes. Contienen una entrada de índice para cada palabra post-raíz única en cada campo indexado para cada documento insertado.
* La creación de un `text`índice es muy similar a la creación de un gran índice de varias claves y llevará más tiempo que la creación de un índice simple ordenado \(escalar\) sobre los mismos datos.
* Cuando cree un `text`índice grande en una colección existente, asegúrese de tener un límite suficientemente alto para los descriptores de archivos abiertos. Consulte la [configuración recomendada](https://docs.mongodb.com/manual/reference/ulimit/) .
* `text` Los índices afectarán el rendimiento de la inserción porque MongoDB debe agregar una entrada de índice para cada palabra post-derivada única en cada campo indexado de cada nuevo documento fuente.
* Además, los `text`índices no almacenan frases o información sobre la proximidad de palabras en los documentos. Como resultado, las consultas de frases se ejecutarán de manera mucho más efectiva cuando toda la colección quepa en la RAM.

### Soporte de búsqueda de texto  <a id="text-search-support"></a>

El `text`índice admite [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)operaciones de consulta. Para ver ejemplos de búsqueda de texto, consulte [`$text reference page`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text). Para ver ejemplos de [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)operaciones en canalizaciones de agregación, consulte [Búsqueda de texto](https://docs.mongodb.com/manual/tutorial/text-search-in-aggregation/) en la canalización de [agregación](https://docs.mongodb.com/manual/tutorial/text-search-in-aggregation/) .

