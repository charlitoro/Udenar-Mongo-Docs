# Crear un índice de pajar

IMPORTANTEDeprecación

MongoDB 4.4 desaprueba el índice [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/) y el [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch)comando. Utilice un [índice 2d](https://docs.mongodb.com/manual/core/2d/) con [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)o en su [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)lugar.

Un índice de pajar debe hacer referencia a dos campos: el campo de ubicación y un segundo campo. El segundo campo se utiliza para coincidencias exactas. Los índices de Haystack devuelven documentos basados ​​en la ubicación y una coincidencia exacta en un único criterio adicional. Estos índices no son necesariamente adecuados para devolver los documentos más cercanos a una ubicación en particular.

Para construir un índice de pajar, use la siguiente sintaxis:

```text
db.coll.createIndex( { <location field> : "geoHaystack" ,                       <additional field> : 1 } ,                     { bucketSize : <bucket value> } )
```

Para construir un índice de pajar, debe especificar la `bucketSize`opción al crear el índice. Una `bucketSize`de `5`crea un índice que agrupa los valores de ubicación que están dentro de las 5 unidades de la longitud y latitud especificadas. El `bucketSize`también determina la granularidad del índice. Puede ajustar el parámetro a la distribución de sus datos para que, en general, busque solo regiones muy pequeñas. Las áreas definidas por cubos pueden superponerse. Un documento puede existir en varios depósitos.EJEMPLO

Si tiene una colección con documentos que contienen campos similares a los siguientes:

```text
{ _id : 100, pos: { lng : 126.9, lat : 35.2 } , type : "restaurant"}{ _id : 200, pos: { lng : 127.5, lat : 36.1 } , type : "restaurant"}{ _id : 300, pos: { lng : 128.0, lat : 36.7 } , type : "national park"}
```

Las siguientes operaciones crean un índice de pajar con depósitos que almacenan claves dentro de 1 unidad de longitud o latitud.

```text
db.places.createIndex( { pos : "geoHaystack", type : 1 } ,                       { bucketSize : 1 } )
```

Este índice almacena el documento con un `_id`campo que tiene el valor `200`en dos depósitos diferentes:

* En un depósito que incluye el documento donde el `_id`campo tiene un valor de`100`
* En un depósito que incluye el documento donde el `_id`campo tiene un valor de`300`

Para consultar usando un índice de pajar, usa el [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch) comando. Consulte [Consultar un índice de Haystack](https://docs.mongodb.com/manual/tutorial/query-a-geohaystack-index/#std-label-geospatial-indexes-haystack-queries) .

De forma predeterminada, las consultas que utilizan un índice de pajar devuelven 50 documentos.

