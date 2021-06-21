# Índices 2d

Utilice un `2d`índice para los datos almacenados como puntos en un plano bidimensional. El `2d`índice está diseñado para [pares de coordenadas heredados](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) utilizados en MongoDB 2.2 y versiones anteriores.

Utilice un `2d`índice si:

* su base de datos tiene [pares de coordenadas](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) heredados [heredados](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) de MongoDB 2.2 o anterior, _y_
* no tiene la intención de almacenar ningún dato de ubicación como objetos [GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON) .

Para obtener más información sobre consultas geoespaciales, consulte [Consultas geoespaciales](https://docs.mongodb.com/manual/geospatial-queries/) .

### Consideraciones  <a id="considerations"></a>

A partir de MongoDB 4.0, puede especificar una `key`opción en la [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)etapa de canalización para indicar la ruta del campo indexado que se utilizará. Esto permite que el [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)escenario se use en una colección que tiene múltiples `2d`índices y / o múltiples [índices de 2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) :

* Si su colección tiene múltiples `2d`índices y / o múltiples [índices de 2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) , debe usar la `key`opción para especificar la ruta del campo indexado a usar.
* Si no especifica el `key`, no puede tener un `2d`índice múltiple y / o un [índice 2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) múltiple, ya que sin el `key`, la selección de índice entre varios `2d`índices o `2dsphere`índices es ambigua.

NOTA

Si no especifica el `key`, y tiene como máximo solo un `2d`índice de índice y / o solo un `2d`índice de índice, MongoDB busca primero un `2d`índice para usar. Si `2d`no existe un índice, MongoDB busca un `2dsphere`índice para usar.

No utilice un `2d`índice si sus datos de ubicación incluyen objetos GeoJSON. Para indexar tanto [los pares de coordenadas heredados ](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy)_como los_ [objetos GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson) , use un índice [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) .

No puede utilizar un `2d`índice como [clave de fragmentación](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) al fragmentar una colección. Sin embargo, puede crear un índice geoespacial en una colección fragmentada utilizando un campo diferente como clave de fragmentación.

### Comportamiento  <a id="behavior"></a>

El `2d`índice admite cálculos en un [plano euclidiano plano](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geometry) . El `2d`índice también admite cálculos de _solo distancia_ en una esfera \(p . Ej . [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)\), Pero para cálculos _geométricos_ en una esfera \(p [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin). Ej. \), Almacene los datos como [objetos GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson) y use un `2dsphere`índice.

Un `2d`índice puede hacer referencia a dos campos. El primero debe ser el campo de ubicación. Un `2d`índice compuesto crea consultas que seleccionan primero en el campo de ubicación y luego filtra esos resultados según los criterios adicionales. Un `2d`índice compuesto puede cubrir consultas.

### `sparse`Propiedad  <a id="sparse-property"></a>

`2d`los índices son siempre [escasos](https://docs.mongodb.com/manual/core/index-sparse/) e ignoran la opción [escasa](https://docs.mongodb.com/manual/core/index-sparse/) . Si un documento carece de un `2d`campo de índice \(o el campo es `null`una matriz vacía\), MongoDB no agrega una entrada para el documento al `2d`índice. Para inserciones, MongoDB inserta el documento pero no lo agrega al `2d`índice.

Para un índice compuesto que incluye una `2d`clave de índice junto con claves de otros tipos, solo el `2d`campo de índice determina si el índice hace referencia a un documento.

### Opción de clasificación  <a id="collation-option"></a>

`2d`los índices solo admiten la comparación binaria simple y no admiten la opción de [clasificación](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation) .

Para crear un `2d`índice en una colección que tiene una intercalación no simple, debe especificar explícitamente `{collation: {locale: "simple"} }`al crear el índice.

