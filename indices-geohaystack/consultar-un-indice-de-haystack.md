# Consultar un índice de Haystack

IMPORTANTEDeprecación

MongoDB 4.4 desaprueba el índice [geoHaystack](https://docs.mongodb.com/manual/core/geohaystack/) y el [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch)comando. Utilice un [índice 2d](https://docs.mongodb.com/manual/core/2d/) con [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)o en su [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)lugar.

Un índice de pajar es un `2d`índice geoespacial especial que está optimizado para devolver resultados en áreas pequeñas. Para crear un índice de pajar, consulte [Crear un índice de pajar](https://docs.mongodb.com/manual/tutorial/build-a-geohaystack-index/#std-label-geospatial-indexes-haystack-index) .

Para consultar un índice de pajar, use el [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch)comando. Debe especificar tanto las coordenadas como el campo adicional a [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#mongodb-dbcommand-dbcmd.geoSearch). Por ejemplo, para devolver todos los documentos con el valor `restaurant`en el `type`campo cerca del punto de ejemplo, el comando se parecería a:

```text
db.runCommand( { geoSearch : "places" ,                 search : { type: "restaurant" } ,                 near : [-74, 40.74] ,                 maxDistance : 10 } )
```

NOTA

Los índices de Haystack no son adecuados para consultas de la lista completa de documentos más cercanos a una ubicación en particular. Los documentos más cercanos podrían estar más distantes en comparación con el tamaño del cubo.NOTA

Actualmente, los índices de pajar no admiten las [operaciones de consulta esférica](https://docs.mongodb.com/manual/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/) .

El [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)método no puede acceder al índice del pajar.

