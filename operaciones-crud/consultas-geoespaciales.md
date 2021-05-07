# Consultas Geoespaciales

MongoDB admite operaciones de consulta en datos geoespaciales. Esta sección presenta las características geoespaciales de MongoDB.

### Datos geoespaciales  <a id="geospatial-data"></a>

En MongoDB, puede almacenar datos geoespaciales como objetos [GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson) o como [pares de coordenadas heredados](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) .

#### Objetos GeoJSON  <a id="geojson-objects"></a>

Para calcular la geometría sobre una esfera similar a la Tierra, almacene sus datos de ubicación como [objetos GeoJSON](https://docs.mongodb.com/manual/reference/geojson/) .

Para especificar datos de GeoJSON, use un documento incrustado con:

* un campo llamado `type`que especifica el [tipo de objeto GeoJSON](https://docs.mongodb.com/manual/reference/geojson/) y
* un campo llamado `coordinates`que especifica las coordenadas del objeto.

  Si especifica las coordenadas de latitud y longitud, enumere primero la **longitud** y luego la **latitud** :

  * Los valores de longitud válidos están entre `-180`y `180`, ambos inclusive.
  * Los valores de latitud válidos están entre `-90`y `90`, ambos inclusive.

```text
<field>: { type: <GeoJSON type> , coordinates: <coordinates> }
```

Por ejemplo, para especificar un [punto GeoJSON](https://docs.mongodb.com/manual/reference/geojson/#std-label-geojson-point) :

```text
location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
}
```

Para obtener una lista de los objetos GeoJSON compatibles con MongoDB, así como ejemplos, consulte [Objetos GeoJSON](https://docs.mongodb.com/manual/reference/geojson/) .

Las consultas geoespaciales de MongoDB sobre objetos GeoJSON se calculan en una esfera; MongoDB utiliza el [sistema de](https://docs.mongodb.com/manual/reference/glossary/#std-term-WGS84) referencia [WGS84](https://docs.mongodb.com/manual/reference/glossary/#std-term-WGS84) para consultas geoespaciales sobre objetos GeoJSON.  
Pares de coordenadas heredados 

Para calcular distancias en un plano euclidiano, almacene sus datos de ubicación como pares de coordenadas heredados y use un [`2d`](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2d)índice. MongoDB admite cálculos de superficies esféricas en pares de coordenadas heredados a través de un [`2dsphere`](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere)índice al convertir los datos al tipo de punto GeoJSON.

Para especificar datos como pares de coordenadas heredados, puede usar una matriz \( _preferido_ \) o un documento incrustado.Especificar mediante una matriz \( _preferido_ \):

```text
<field>: [ <x>, <y> ]
```

Si especifica coordenadas de latitud y longitud, indique primero la **longitud** y luego la **latitud** ; es decir

```text
<field>: [<longitude>, <latitude> ]
```

* Los valores de longitud válidos están entre `-180`y `180`, ambos inclusive.
* Los valores de latitud válidos están entre `-90`y `90`, ambos inclusive.

Especificar a través de un documento incrustado:

```text
<field>: { <field1>: <x>, <field2>: <y> }
```

Si especifica coordenadas de latitud y longitud, el primer campo, independientemente del nombre del campo, debe contener el valor de **longitud** y el segundo campo, el valor de **latitud** ; es decir

```text
<field>: { <field1>: <longitude>, <field2>: <latitude> }
```

* Los valores de longitud válidos están entre `-180`y `180`, ambos inclusive.
* Los valores de latitud válidos están entre `-90`y `90`, ambos inclusive.

Para especificar pares de coordenadas heredados, se prefieren las matrices sobre un documento incrustado, ya que algunos idiomas no garantizan el orden asociativo de mapas.  


### Índices geoespaciales <a id="geospatial-indexes"></a>

MongoDB proporciona los siguientes tipos de índices geoespaciales para respaldar las consultas geoespaciales.

#### `2dsphere` <a id="2dsphere"></a>

[Los](https://docs.mongodb.com/manual/core/2dsphere/) índices de [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) admiten consultas que calculan [geometrías en una esfera similar a la tierra](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geometry) .

Para crear un `2dsphere`índice, use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método y especifique el literal de cadena `"2dsphere"`como tipo de índice:

```text
db.collection.createIndex( { <location field> : "2dsphere" } )
```

donde `<location field>`es un campo cuyo valor es un [objeto GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson) o un [par de coordenadas heredado](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) .

Para obtener más información sobre el `2dsphere`índice, consulte [`2dsphere`Índices](https://docs.mongodb.com/manual/core/2dsphere/) .

#### `2d` <a id="2d"></a>

[Los](https://docs.mongodb.com/manual/core/2d/) índices [2D](https://docs.mongodb.com/manual/core/2d/) admiten consultas que calculan [geometrías en un plano bidimensional](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geometry) . Aunque el índice puede admitir [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)consultas que calculan en una esfera, si es posible, utilice el [`2dsphere`](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere)índice para consultas esféricas.

Para crear un `2d`índice, use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex) método, especificando el campo de ubicación como clave y el literal de cadena `"2d"`como tipo de índice:

```text
db.collection.createIndex( { <location field> : "2d" } )
```

donde `<location field>`es un campo cuyo valor es un [par de coordenadas heredado](https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy) .

Para obtener más información sobre el `2d`índice, consulte [`2d`Índices](https://docs.mongodb.com/manual/core/2d/) .

#### Índices geoespaciales y colecciones fragmentadas  <a id="geospatial-indexes-and-sharded-collections"></a>

No puede utilizar un índice geoespacial como [clave de fragmentación](https://docs.mongodb.com/manual/reference/glossary/#std-term-shard-key) al fragmentar una colección. Sin embargo, puede crear un índice geoespacial en una colección fragmentada utilizando un campo diferente como clave de fragmentación.

Las siguientes operaciones geoespaciales se admiten en colecciones fragmentadas:

* [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear) etapa de agregación
* [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)y [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)operadores de consulta \(a partir de MongoDB 4.0\)

A partir de MongoDB 4.0, [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)y [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)las consultas se admite para colecciones fragmentados.

En versiones anteriores de MongoDB, las consultas [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)y las [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere)colecciones fragmentadas no son compatibles; en su lugar, para los clústeres fragmentados, debe usar la [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)etapa de agregación o el `geoNear`comando \(disponible en MongoDB 4.0 y versiones anteriores\).

También puede consultar datos geoespaciales para un clúster fragmentado mediante [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin)y [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects).

#### Consultas cubiertas [¶](https://docs.mongodb.com/manual/geospatial-queries/#covered-queries) <a id="covered-queries"></a>

[Los índices geoespaciales](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial) no pueden [cubrir una consulta](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries) .  


**NOTA**

Para consultas esféricas, use el `2dsphere`resultado del índice.

El uso de `2d`índice para consultas esféricas puede conducir a resultados incorrectos, como el uso del `2d`índice para consultas esféricas que envuelven los polos.

#### Operadores de consultas geoespaciales  <a id="geospatial-query-operators"></a>

MongoDB proporciona los siguientes operadores de consultas geoespaciales:

| Nombre | Descripción |
| :--- | :--- |
| [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects) | Selecciona geometrías que se cruzan con una geometría [GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON) . El índice [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) admite archivos [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects). |
| [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin) | Selecciona geometrías dentro de una [geometría GeoJSON](https://docs.mongodb.com/manual/reference/geojson/#std-label-geospatial-indexes-store-geojson) delimitante . Los índices [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) y [2d son](https://docs.mongodb.com/manual/core/2d/) compatibles [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin). |
| [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near) | Devuelve objetos geoespaciales próximos a un punto. Requiere un índice geoespacial. Los índices [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) y [2d son](https://docs.mongodb.com/manual/core/2d/) compatibles [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near). |
| [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere) | Devuelve objetos geoespaciales próximos a un punto de una esfera. Requiere un índice geoespacial. Los índices [2dsphere](https://docs.mongodb.com/manual/core/2dsphere/) y [2d son](https://docs.mongodb.com/manual/core/2d/) compatibles [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere). |

Para obtener más detalles, incluidos ejemplos, consulte la página de referencia individual.  


#### Etapa de agregación geoespacial  <a id="geospatial-aggregation-stage"></a>

MongoDB proporciona la siguiente [etapa de canalización de agregación](https://docs.mongodb.com/manual/core/aggregation-pipeline/) geoespacial :

<table>
  <thead>
    <tr>
      <th style="text-align:left">Etapa</th>
      <th style="text-align:left">Descripci&#xF3;n</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear"><code>$geoNear</code></a>
      </td>
      <td style="text-align:left">
        <p>Devuelve un flujo ordenado de documentos seg&#xFA;n la proximidad a un
          punto geoespacial. Incorpora la funcionalidad de <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/match/#mongodb-pipeline-pipe.-match"><code>$match</code></a>,
          <a
          href="https://docs.mongodb.com/manual/reference/operator/aggregation/sort/#mongodb-pipeline-pipe.-sort"><code>$sort</code>
            </a>y <a href="https://docs.mongodb.com/manual/reference/operator/aggregation/limit/#mongodb-pipeline-pipe.-limit"><code>$limit</code></a>para
            los datos geoespaciales. Los documentos de salida incluyen un campo de
            distancia adicional y pueden incluir un campo de identificaci&#xF3;n de
            ubicaci&#xF3;n.</p>
        <p><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear"><code>$geoNear</code></a>requiere
          un <a href="https://docs.mongodb.com/manual/core/geospatial-indexes/">&#xED;ndice geoespacial</a> .</p>
      </td>
    </tr>
  </tbody>
</table>

Para obtener más detalles, incluidos ejemplos, consulte la [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear) página de referencia.

### Modelos geoespaciales  <a id="geospatial-models"></a>

Las consultas geoespaciales de MongoDB pueden interpretar la geometría en una superficie plana o una esfera.

`2dsphere` los índices solo admiten consultas esféricas \(es decir, consultas que interpretan geometrías en una superficie esférica\).

`2d`los índices admiten consultas planas \(es decir, consultas que interpretan geometrías en una superficie plana\) y algunas consultas esféricas. Si bien los `2d` índices admiten algunas consultas esféricas, el uso de `2d`índices para estas consultas esféricas puede generar errores. Si es posible, utilice `2dsphere`índices para consultas esféricas.

La siguiente tabla enumera los operadores de consulta geoespacial, consulta admitida, utilizados por cada operación geoespacial:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Operaci&#xF3;n</th>
      <th style="text-align:left">Consulta esf&#xE9;rica / plana</th>
      <th style="text-align:left">Notas</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near"><code>$near</code></a>(
        Punto de centroide <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson">GeoJSON</a> en
        esta l&#xED;nea y la l&#xED;nea siguiente, &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere">2dsphere</a> )</td>
      <td
      style="text-align:left">Esf&#xE9;rico</td>
        <td style="text-align:left">Consulte tambi&#xE9;n el <a href="https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere"><code>$nearSphere</code></a>operador,
          que proporciona la misma funcionalidad cuando se usa con <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson">GeoJSON</a> y
          un &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere">2dsphere</a> .</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near"><code>$near</code></a>(
        <a
        href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy">coordenadas heredadas</a>, &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2d">2d</a> )</td>
      <td
      style="text-align:left">Departamento</td>
        <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere"><code>$nearSphere</code></a>(
        Punto <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson">GeoJSON</a> ,
        &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere">2dsphere</a> )</td>
      <td
      style="text-align:left">Esf&#xE9;rico</td>
        <td style="text-align:left">
          <p>Proporciona la misma funcionalidad que la <a href="https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near"><code>$near</code></a>operaci&#xF3;n
            que usa un punto <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-geojson">GeoJSON</a> y
            un &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere">2dsphere</a> .</p>
          <p>Para consultas esf&#xE9;ricas, puede ser preferible utilizar <a href="https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere"><code>$nearSphere</code></a>que
            especifique expl&#xED;citamente las consultas esf&#xE9;ricas en el nombre
            en lugar del <a href="https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near"><code>$near</code></a>operador.</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere"><code>$nearSphere</code></a>(
        <a
        href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geospatial-legacy">coordenadas heredadas</a>, &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2d">2d</a> )</td>
      <td
      style="text-align:left">Esf&#xE9;rico</td>
        <td style="text-align:left">Utilice puntos <a href="https://docs.mongodb.com/manual/reference/glossary/#std-term-GeoJSON">GeoJSON en su</a> lugar.</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin"><code>$geoWithin</code></a>:
        { <a href="https://docs.mongodb.com/manual/reference/operator/query/geometry/#mongodb-query-op.-geometry"><code>$geometry</code></a>:
        ...}</td>
      <td style="text-align:left">Esf&#xE9;rico</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin"><code>$geoWithin</code></a>:
        { <a href="https://docs.mongodb.com/manual/reference/operator/query/box/#mongodb-query-op.-box"><code>$box</code></a>:
        ...}</td>
      <td style="text-align:left">Departamento</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin"><code>$geoWithin</code></a>:
        { <a href="https://docs.mongodb.com/manual/reference/operator/query/polygon/#mongodb-query-op.-polygon"><code>$polygon</code></a>:
        ...}</td>
      <td style="text-align:left">Departamento</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin"><code>$geoWithin</code></a>:
        { <a href="https://docs.mongodb.com/manual/reference/operator/query/center/#mongodb-query-op.-center"><code>$center</code></a>:
        ...}</td>
      <td style="text-align:left">Departamento</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#mongodb-query-op.-geoWithin"><code>$geoWithin</code></a>:
        { <a href="https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#mongodb-query-op.-centerSphere"><code>$centerSphere</code></a>:
        ...}</td>
      <td style="text-align:left">Esf&#xE9;rico</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#mongodb-query-op.-geoIntersects"><code>$geoIntersects</code></a>
      </td>
      <td style="text-align:left">Esf&#xE9;rico</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear"><code>$geoNear</code></a>etapa
        de agregaci&#xF3;n ( &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2dsphere">2dsphere</a> )</td>
      <td
      style="text-align:left">Esf&#xE9;rico</td>
        <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear"><code>$geoNear</code></a>etapa
        de agregaci&#xF3;n ( &#xED;ndice <a href="https://docs.mongodb.com/manual/geospatial-queries/#std-label-geo-2d">2d</a> )</td>
      <td
      style="text-align:left">Departamento</td>
        <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

### Ejemplo  <a id="example"></a>

Crea una colección `places`con los siguientes documentos:

```text
db.places.insert( {    name: "Central Park",   location: { type: "Point", coordinates: [ -73.97, 40.77 ] },   category: "Parks"} );db.places.insert( {   name: "Sara D. Roosevelt Park",   location: { type: "Point", coordinates: [ -73.9928, 40.7193 ] },   category: "Parks"} );db.places.insert( {   name: "Polo Grounds",   location: { type: "Point", coordinates: [ -73.9375, 40.8303 ] },   category: "Stadiums"} );
```

La siguiente operación crea un `2dsphere`índice en el `location`campo:

```text
db.places.createIndex( { location: "2dsphere" } )
```

La siguiente consulta utiliza el [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near)operador para devolver documentos que están al menos a 1000 metros y como máximo a 5000 metros del punto GeoJSON especificado, ordenados en orden de más cercano a más lejano:

```text
db.places.find(   {     location:       { $near:          {            $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },            $minDistance: 1000,            $maxDistance: 5000          }       }   })
```

La siguiente operación utiliza la [`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear)operación de agregación para devolver documentos que coinciden con el filtro de consulta `{ category: "Parks" }`, ordenados de más cerca a más lejos del punto GeoJSON especificado:

```text
db.places.aggregate( [   {      $geoNear: {         near: { type: "Point", coordinates: [ -73.9667, 40.78 ] },         spherical: true,         query: { category: "Parks" },         distanceField: "calcDistance"      }   }] )
```

