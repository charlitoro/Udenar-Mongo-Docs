# Índices geoHaystack

IMPORTANTEDeprecación

MongoDB 4.4 desaprueba el índice [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/) y el [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch)comando. Utilice un [índice 2d](https://docs.mongodb.com/manual/core/2d/) con [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)o en su [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)lugar.

Un `geoHaystack`índice es un índice especial que está optimizado para devolver resultados en áreas pequeñas. `geoHaystack`Los índices mejoran el rendimiento en consultas que utilizan geometría plana.

Para consultas que usan geometría esférica, un **índice de 2 esferas es una mejor opción** que un índice de pajar. [Los índices 2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) permiten la reordenación de campos; `geoHaystack`los índices requieren que el primer campo sea el campo de ubicación. Además, los `geoHaystack` índices solo se pueden usar a través de comandos y, por lo tanto, siempre devuelven todos los resultados a la vez.

### Comportamiento  <a id="behavior"></a>

`geoHaystack`Los índices crean "depósitos" de documentos de la misma área geográfica para mejorar el rendimiento de las consultas limitadas a esa área. Cada segmento de un `geoHaystack`índice contiene todos los documentos dentro de una proximidad específica a una longitud y latitud determinadas.

### `sparse`Propiedad  <a id="sparse-property"></a>

`geoHaystack`los índices son [dispersos](https://docs.mongodb.com/manual/core/index-sparse/) por defecto e ignoran la opción [sparse: true](https://docs.mongodb.com/manual/core/index-sparse/) . Si un documento carece de un `geoHaystack`campo de índice \(o el campo es `null`una matriz vacía\), MongoDB no agrega una entrada para el documento al `geoHaystack`índice. Para inserciones, MongoDB inserta el documento pero no lo agrega al `geoHaystack`índice.

`geoHaystack`los índices incluyen una `geoHaystack`clave de índice y una clave de índice no geoespacial; sin embargo, solo el `geoHaystack`campo de índice determina si el índice hace referencia a un documento.

#### Opción de clasificación  <a id="collation-option"></a>

`geoHaystack`los índices solo admiten la comparación binaria simple y no admiten la [intercalación](https://docs.mongodb.com/manual/reference/bson-type-comparison-order/#std-label-collation) .

Para crear un `geoHaystack`índice en una colección que tiene una intercalación no simple, debe especificar explícitamente `{collation: {locale: "simple"} }`al crear el índice.

### Crear `geoHaystack`índice  <a id="create-geohaystack-index"></a>

Para crear un `geoHaystack`índice, consulte [Crear un índice de pajar](https://docs.mongodb.com/manual/tutorial/build-a-geohaystack-index/) . Para obtener información y un ejemplo sobre cómo [consultar un](https://docs.mongodb.com/manual/tutorial/query-a-geohaystack-index/) índice de pajar, consulte [Consultar un índice de pajar](https://docs.mongodb.com/manual/tutorial/query-a-geohaystack-index/) .

