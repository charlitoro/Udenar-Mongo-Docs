# Consultar un 2dsphereíndice

Las siguientes secciones describen las consultas admitidas por el `2dsphere`índice.

### Objetos GeoJSON delimitados por un polígono  <a id="geojson-objects-bounded-by-a-polygon"></a>

El [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)operador consulta los datos de ubicación que se encuentran dentro de un polígono GeoJSON. Los datos de su ubicación deben almacenarse en formato GeoJSON. Utilice la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $geoWithin :                           { $geometry :                             { type : "Polygon" ,                               coordinates : [ <coordinates> ]                      } } } } )
```

El siguiente ejemplo selecciona todos los puntos y formas que existen por completo dentro de un polígono GeoJSON:

```text
db.places.find( { loc :                  { $geoWithin :                    { $geometry :                      { type : "Polygon" ,                        coordinates : [ [                                          [ 0 , 0 ] ,                                          [ 3 , 6 ] ,                                          [ 6 , 1 ] ,                                          [ 0 , 0 ]                                        ] ]                } } } } )
```

### Intersecciones de objetos GeoJSON  <a id="intersections-of-geojson-objects"></a>

El [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects)operador consulta las ubicaciones que se cruzan con un objeto GeoJSON específico. Una ubicación se cruza con el objeto si la intersección no está vacía. Esto incluye documentos que tienen un borde compartido.

El [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects)operador usa la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $geoIntersects :                           { $geometry :                             { type : "<GeoJSON object type>" ,                               coordinates : [ <coordinates> ]                      } } } } )
```

El siguiente ejemplo se utiliza [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects)para seleccionar todos los puntos y formas indexados que se cruzan con el polígono definido por la `coordinates`matriz.

```text
db.places.find( { loc :                  { $geoIntersects :                    { $geometry :                      { type : "Polygon" ,                        coordinates: [ [                                         [ 0 , 0 ] ,                                         [ 3 , 6 ] ,                                         [ 6 , 1 ] ,                                         [ 0 , 0 ]                                       ] ]                } } } } )
```

### Proximidad a un punto GeoJSON  <a id="proximity-to-a-geojson-point"></a>

Las consultas de proximidad devuelven los puntos más cercanos al punto definido y clasifican los resultados por distancia. Una consulta de proximidad sobre datos de GeoJSON requiere un `2dsphere`índice.

Para consultar la proximidad a un punto GeoJSON, use el [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)operador. La distancia está en metros.

El [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)usa la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $near :                           { $geometry :                              { type : "Point" ,                                coordinates : [ <longitude> , <latitude> ] } ,                             $maxDistance : <distance in meters>                      } } } )
```

Para ver ejemplos, consulte [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near).

Consulte también el [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)operador y la [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear) etapa de canalización de agregación.

### Puntos dentro de un círculo definidos en una esfera  <a id="points-within-a-circle-defined-on-a-sphere"></a>

Para seleccionar todas las coordenadas de la cuadrícula en un "casquete esférico" en una esfera, utilice [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)con el [`$centerSphere`](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#mongodb-query-op.-centerSphere)operador. Especifique una matriz que contenga:

* Las coordenadas de la cuadrícula del punto central del círculo.
* El radio del círculo medido en radianes. Para calcular radianes, consulte [Calcular distancia mediante geometría esférica](https://docs.mongodb.com/manual/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/) .

Utilice la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $geoWithin :                           { $centerSphere :                              [ [ <x>, <y> ] , <radius> ] }                      } } )
```

El siguiente ejemplo consulta las coordenadas de la cuadrícula y devuelve todos los documentos dentro de un radio de 10 millas de longitud `88 W`y latitud `30 N`. El ejemplo convierte la distancia, 10 millas, a radianes dividiendo por el radio ecuatorial aproximado de la Tierra, 3963,2 millas:

```text
db.places.find( { loc :                  { $geoWithin :                    { $centerSphere :                       [ [ -88 , 30 ] , 10 / 3963.2 ]                } } } )
```

