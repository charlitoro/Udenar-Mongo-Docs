# Elementos internos del índice 2d



Este documento proporciona una explicación más detallada de los aspectos internos de los `2d`índices geoespaciales de MongoDB . Este material no es necesario para las operaciones normales o el desarrollo de aplicaciones, pero puede ser útil para la resolución de problemas y para una mayor comprensión.

### Cálculo de valores de Geohash para `2d`índices  <a id="calculation-of-geohash-values-for-2d-indexes"></a>

Cuando crea un índice geoespacial en [pares de coordenadas heredados](https://docs.mongodb.com/manual/reference/glossary/#std-term-legacy-coordinate-pairs) , MongoDB calcula los valores de [geohash](https://docs.mongodb.com/manual/reference/glossary/#std-term-geohash) para los pares de coordenadas dentro del [rango de ubicación](https://docs.mongodb.com/manual/tutorial/build-a-2d-index/#std-label-geospatial-indexes-range) especificado y luego indexa los valores de geohash.

Para calcular un valor de geohash, divida de forma recursiva un mapa bidimensional en cuadrantes. Luego asigne a cada cuadrante un valor de dos bits. Por ejemplo, una representación de dos bits de cuatro cuadrantes sería:

```text
01  1100  10
```

Estos valores de dos bits \( `00`, `01`, `10`, y `11`\) representan cada uno de los cuadrantes y todos los puntos dentro de cada cuadrante. Para un geohash con dos bits de resolución, todos los puntos del cuadrante inferior izquierdo tendrían un geohash de `00`. El cuadrante superior izquierdo tendría el geohash de `01`. La parte inferior derecha y la parte superior derecha tendrían un geohash de `10` y `11`, respectivamente.

Para proporcionar precisión adicional, continúe dividiendo cada cuadrante en subcuadrantes. Cada subcuadrante tendría el valor geohash del cuadrante contenedor concatenado con el valor del subcuadrante. El geohash para el cuadrante superior derecho es `11`, y la geohash para las sub-cuadrantes sería \(en sentido horario desde la parte superior izquierda\): `1101`, `1111`, `1110`, y `1100`, respectivamente.

### Documentos de múltiples ubicaciones para `2d`índices  <a id="multi-location-documents-for-2d-indexes"></a>

NOTA

Si bien los `2d`índices geoespaciales no admiten más de un campo geoespacial en un documento, puede usar un [índice de claves](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multi-key) múltiples para indexar múltiples pares de coordenadas en un solo documento. En el ejemplo más simple, puede tener un campo \(p `locs`. Ej. \) Que contiene una matriz de coordenadas, como en el siguiente ejemplo:

```text
db.places.save( {  locs : [ [ 55.5 , 42.3 ] ,           [ -74 , 44.74 ] ,           { lng : 55.5 , lat : 42.3 } ]} )
```

Los valores de la matriz pueden ser matrices, como en `[ 55.5, 42.3 ]`, o documentos incrustados, como en `{ lng : 55.5 , lat : 42.3 }`.

A continuación, puede crear un índice geoespacial en el `locs`campo, como se muestra a continuación:

```text
db.places.createIndex( { "locs": "2d" } )
```

También puede modelar los datos de ubicación como un campo dentro de un documento incrustado. En este caso, el documento contendría un campo \(p `addresses`. Ej. \) Que contiene una serie de documentos donde cada documento tiene un campo \(p `loc:`. Ej. \) Que contiene coordenadas de ubicación. Por ejemplo:

```text
db.records.save( {  name : "John Smith",  addresses : [ {                 context : "home" ,                 loc : [ 55.5, 42.3 ]                } ,                {                 context : "work",                 loc : [ -74 , 44.74 ]                }              ]} )
```

Luego, podría crear el índice geoespacial en el `addresses.loc`campo como en el siguiente ejemplo:

```text
db.records.createIndex( { "addresses.loc": "2d" } )
```

