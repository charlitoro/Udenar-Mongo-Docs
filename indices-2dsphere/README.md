# Índices 2dsphere

### Descripción general  <a id="overview"></a>

Un `2dsphere`índice admite consultas que calculan geometrías en una esfera similar a la tierra. `2dsphere`index admite todas las consultas geoespaciales de MongoDB: consultas de inclusión, intersección y proximidad. Para obtener más información sobre consultas geoespaciales, consulte [Consultas geoespaciales](https://docs.mongodb.com/manual/geospatial-queries/) .

El `2dsphere`índice admite datos almacenados como [objetos GeoJSON ](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson)_y_ [pares de coordenadas heredados](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) \(consulte también [`2dsphere`Restricciones de campos indexados](https://docs.mongodb.com/manual/core/2dsphere/#std-label-2dsphere-data-restrictions) \). Para los pares de coordenadas heredados, el índice convierte los datos a GeoJSON [`Point`](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-point).

### Versiones  <a id="versions"></a>

| `2dsphere` Versión del índice | Descripción |
| :--- | :--- |
| Versión 3 | MongoDB 3.2 presenta una versión 3 de `2dsphere`índices. La versión 3 es la versión predeterminada de los `2dsphere`índices creados en MongoDB 3.2 y posteriores. |
| Versión 2 | MongoDB 2.6 presenta una versión 2 de `2dsphere`índices. La versión 2 es la versión predeterminada de los `2dsphere`índices creados en las series MongoDB 2.6 y 3.0. |

Para anular la versión predeterminada y especificar una versión diferente, incluya la opción `{ "2dsphereIndexVersion": <version> }`al crear el índice.

#### `sparse`Propiedad  <a id="sparse-property"></a>

Los `2dsphere`índices de la versión 2 y posteriores siempre son [escasos](https://docs.mongodb.com/manual/core/index-sparse/) e ignoran la opción [escasa](https://docs.mongodb.com/manual/core/index-sparse/) . Si un documento carece de un `2dsphere`campo de índice \(o el campo es `null`una matriz vacía\), MongoDB no agrega una entrada para el documento al índice. Para inserciones, MongoDB inserta el documento pero no lo agrega al `2dsphere`índice.

Para un índice compuesto que incluye una `2dsphere`clave de índice junto con claves de otros tipos, solo el `2dsphere`campo de índice determina si el índice hace referencia a un documento.

Las versiones anteriores de MongoDB solo admiten `2dsphere (Version 1)` índices. `2dsphere (Version 1)`los índices _no_ son escasos de forma predeterminada y rechazarán documentos con `null`campos de ubicación.

#### Objetos GeoJSON adicionales  <a id="additional-geojson-objects"></a>

Version 2 y posteriores `2dsphere`índices incluye soporte para el objeto GeoJSON adicional: [`MultiPoint`](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-multipoint), [`MultiLineString`](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-multilinestring), [`MultiPolygon`](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-multipolygon), y [`GeometryCollection`](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-geometrycollection). Para obtener detalles sobre todos los objetos GeoJSON compatibles, consulte [Objetos GeoJSON](https://docs.mongodb.com/manual/reference/geojson/) .

### Consideraciones  <a id="considerations"></a>

#### `geoNear`y `$geoNear`restricciones  <a id="geonear-and--geonear-restrictions"></a>

A partir de MongoDB 4.0, puede especificar una `key`opción en la [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)etapa de canalización para indicar la ruta del campo indexado que se utilizará. Esto permite que el [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)escenario se use en una colección que tiene múltiples `2dsphere`índices y / o múltiples índices [2d](https://docs.mongodb.com/manual/core/2d/) :

* Si su colección tiene múltiples `2dsphere`índices y / o múltiples índices [2d](https://docs.mongodb.com/manual/core/2d/) , debe usar la `key`opción para especificar la ruta del campo indexado a usar.
* Si no especifica `key`, no puede tener un `2dsphere`índice múltiple y / o un índice [2d](https://docs.mongodb.com/manual/core/2d/) múltiple, ya que sin él `key`, la selección de índice entre varios `2d`índices o `2dsphere`índices es ambigua.

NOTA

Si no especifica el `key`, y tiene como máximo solo un `2dsphere`índice de índice y / o solo un `2dsphere`índice de índice, MongoDB busca primero un `2d`índice para usar. Si `2d`no existe un índice, MongoDB busca un `2dsphere`índice para usar.

#### Restricciones de claves de fragmentos  <a id="shard-key-restrictions"></a>

No puede utilizar un `2dsphere`índice como [clave de fragmentación](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) al fragmentar una colección. Sin embargo, puede crear un índice geoespacial en una colección fragmentada utilizando un campo diferente como clave de fragmentación.

#### `2dsphere`Restricciones de campos indexados  <a id="2dsphere-indexed-field-restrictions"></a>

Los campos con índices [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) deben contener datos de geometría en forma de [pares](https://docs.mongodb.com/manual/reference/glossary/#std-term-legacy-coordinate-pairs) de [coordenadas](https://docs.mongodb.com/manual/reference/glossary/#std-term-legacy-coordinate-pairs) o datos [GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON) . Si intenta insertar un documento con datos que no son de geometría en un `2dsphere`campo indexado, o crea un `2dsphere`índice en una colección donde el campo indexado tiene datos que no son de geometría, la operación fallará.

### Crear un `2dsphere`índice  <a id="create-a-2dsphere-index"></a>

Para crear un `2dsphere`índice, use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método y especifique el literal de cadena `"2dsphere"`como tipo de índice:

```text
db.collection.createIndex( { <location field> : "2dsphere" } )
```

donde `<location field>`es un campo cuyo valor es un [objeto GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson) o un [par de coordenadas heredado](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) .

A diferencia de un índice [2d](https://docs.mongodb.com/manual/core/2d/) compuesto que puede hacer referencia a un campo de ubicación y a otro campo, un índice [compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) `2dsphere` puede hacer referencia a múltiples campos de ubicación y no ubicación.

Para los siguientes ejemplos, considere una colección `places`con documentos que almacenan datos de ubicación como [GeoJSON Point](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-point) en un campo llamado `loc`:

```text
db.places.insert(   {      loc : { type: "Point", coordinates: [ -73.97, 40.77 ] },      name: "Central Park",      category : "Parks"   })db.places.insert(   {      loc : { type: "Point", coordinates: [ -73.88, 40.78 ] },      name: "La Guardia Airport",      category : "Airport"   })
```

#### Crear un `2dsphere`índice  <a id="create-a-2dsphere-index-1"></a>

La siguiente operación crea un índice [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) en el campo de ubicación `loc`:

```text
db.places.createIndex( { loc : "2dsphere" } )
```

#### Crear un índice compuesto con `2dsphere`clave de índice  <a id="create-a-compound-index-with-2dsphere-index-key"></a>

Un [índice compuesto](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) puede incluir una `2dsphere`clave de índice en combinación con claves de índice no geoespaciales. Por ejemplo, la siguiente operación crea un índice compuesto donde la primera clave `loc`es una `2dsphere`clave de índice y las claves restantes `category`y `names`son claves de índice no geoespaciales, específicamente claves descendentes \( `-1`\) y ascendentes \( `1`\) respectivamente.

```text
db.places.createIndex( { loc : "2dsphere" , category : -1, name: 1 } )
```

A diferencia del índice [2d](https://docs.mongodb.com/manual/core/2d/) , un `2dsphere`índice compuesto no requiere que el campo de ubicación sea el primer campo indexado. Por ejemplo:

```text
db.places.createIndex( { category : 1 , loc : "2dsphere" } )
```

