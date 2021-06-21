# Consultar un índice 2d

Las siguientes secciones describen las consultas admitidas por el `2d`índice.

### Puntos dentro de una forma definida en una superficie plana  <a id="points-within-a-shape-defined-on-a-flat-surface"></a>

Para seleccionar todos los pares de coordenadas heredados que se encuentran dentro de una forma determinada en una superficie plana, utilice el [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)operador junto con un operador de forma. Utilice la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $geoWithin :                            { $box|$polygon|$center : <coordinates>                      } } } )
```

Las siguientes consultas para documentos dentro de un rectángulo definido por `[ 0 , 0 ]`en la esquina inferior izquierda y por `[ 100 , 100 ]`en la esquina superior derecha.

```text
db.places.find( { loc :                  { $geoWithin :                     { $box : [ [ 0 , 0 ] ,                                [ 100 , 100 ] ]                 } } } )
```

Las siguientes consultas para documentos que están dentro del círculo centrados en `[ -74 , 40.74 ]`y con un radio de `10`:

```text
db.places.find( { loc: { $geoWithin :                          { $center : [ [-74, 40.74 ] , 10 ]                } } } )
```

Para ver la sintaxis y ejemplos de cada forma, consulte lo siguiente:

* [`$box`](https://docs.mongodb.com/manual/reference/operator/query/box/#mongodb-query-op.-box)
* [`$polygon`](https://docs.mongodb.com/manual/reference/operator/query/polygon/#mongodb-query-op.-polygon)
* [`$center`](https://docs.mongodb.com/manual/reference/operator/query/center/#mongodb-query-op.-center) \(define un círculo\)

### Puntos dentro de un círculo definidos en una esfera  <a id="points-within-a-circle-defined-on-a-sphere"></a>

MongoDB admite consultas esféricas rudimentarias en `2d`índices planos por motivos heredados. En general, los cálculos esféricos deben utilizar un `2dsphere` índice, como se describe en [`2dsphere`Índices](https://docs.mongodb.com/manual/core/2dsphere/) .

Para consultar pares de coordenadas heredados en un "casquete esférico" en una esfera, utilícelo [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)con el [`$centerSphere`](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#mongodb-query-op.-centerSphere)operador. Especifique una matriz que contenga:

* Las coordenadas de la cuadrícula del punto central del círculo.
* El radio del círculo medido en radianes. Para calcular radianes, consulte [Calcular distancia mediante geometría esférica](https://docs.mongodb.com/manual/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/) .

Utilice la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $geoWithin :                            { $centerSphere : [ [ <x>, <y> ] , <radius> ] }                      } } )
```

La siguiente consulta de ejemplo devuelve todos los documentos dentro de un radio de 10 millas de longitud `88 W`y latitud `30 N`. El ejemplo convierte la distancia a radianes dividiendo la distancia por el radio ecuatorial aproximado de la Tierra, 3963,2 millas:

```text
db.<collection>.find( { loc : { $geoWithin :                                 { $centerSphere :                                    [ [ 88 , 30 ] , 10 / 3963.2 ]                      } } } )
```

### Proximidad a un punto en una superficie plana  <a id="proximity-to-a-point-on-a-flat-surface"></a>

Las consultas de proximidad devuelven los pares de coordenadas heredados más cercanos al punto definido y clasifican los resultados por distancia. Utilice el [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)operador. El operador requiere un `2d`índice.

El [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)operador usa la siguiente sintaxis:

```text
db.<collection>.find( { <location field> :                         { $near : [ <x> , <y> ]                      } } )
```

Para ver ejemplos, consulte [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near).

### Coincidencias exactas en una superficie plana  <a id="exact-matches-on-a-flat-surface"></a>

No puede utilizar un `2d`índice para devolver una coincidencia exacta para un par de coordenadas. Utilice un índice escalar, ascendente o descendente en un campo que almacena coordenadas para devolver coincidencias exactas.

En el siguiente ejemplo, la [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find) operación devolverá una coincidencia exacta en una ubicación si tiene un `{ 'loc': 1}`índice:

```text
db.<collection>.find( { loc: [ <x> , <y> ] } )
```

Esta consulta devolverá cualquier documento con el valor de `[ <x> , <y> ]`.

