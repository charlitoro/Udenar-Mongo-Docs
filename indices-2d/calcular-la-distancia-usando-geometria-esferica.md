# Calcular la distancia usando geometría esférica



ADVERTENCIA

Para consultas esféricas, use el `2dsphere`resultado del índice.

El uso de `2d`índice para consultas esféricas puede conducir a resultados incorrectos, como el uso del `2d`índice para consultas esféricas que envuelven los polos.

El `2d`índice admite consultas que calculan distancias en un plano euclidiano \(superficie plana\). El índice también admite los siguientes operadores de consulta y comandos que calculan distancias utilizando geometría esférica:NOTA

Si bien el `2d`índice admite consultas básicas que utilizan la distancia esférica , considere pasar a un `2dsphere`índice si sus datos son principalmente longitud y latitud.

* [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)
* [`$centerSphere`](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#mongodb-query-op.-centerSphere)
* [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)
* [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)etapa de tubería con la `spherical: true`opción

IMPORTANTE

Las operaciones mencionadas utilizan radianes para la distancia. Otros operadores de consulta esférica no lo hacen, como [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin).

Para que los operadores de consultas esféricas funcionen correctamente, debe convertir distancias a radianes y convertir de radianes a las unidades de distancia utilizadas por su aplicación.

Para convertir:

* _distancia a radianes_ : divida la distancia por el radio de la esfera \(por ejemplo, la Tierra\) en las mismas unidades que la medida de la distancia.
* _radianes a distancia_ : multiplique la medida en radianes por el radio de la esfera \(por ejemplo, la Tierra\) en el sistema de unidades al que desea convertir la distancia.

El radio ecuatorial de la Tierra es de aproximadamente `3,963.2` millas o `6,378.1`kilómetros.

La siguiente consulta devolvería documentos de la `places` colección dentro del círculo descrito por el centro `[ -74, 40.74 ]` con un radio de `100`millas:

```text
db.places.find( { loc: { $geoWithin: { $centerSphere: [ [ -74, 40.74 ] ,                                                     100 / 3963.2 ] } } } )
```

NOTA

Si especifica las coordenadas de latitud y longitud, enumere primero la **longitud** y luego la **latitud** :

* Los valores de longitud válidos están entre `-180`y `180`, ambos inclusive.
* Los valores de latitud válidos están entre `-90`y `90`, ambos inclusive.

