# Crear un índice 2d

Para construir un `2d`índice geoespacial , use el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método y especifique `2d`. Utilice la siguiente sintaxis:

```text
db.<collection>.createIndex( { <location field> : "2d" ,                               <additional field> : <value> } ,                             { <index-specification options> } )
```

El `2d`índice utiliza las siguientes opciones opcionales de especificación de índice:

```text
{ min : <lower bound> , max : <upper bound> ,  bits : <bit precision> }
```

### Definir rango de ubicación para un `2d`índice  <a id="define-location-range-for-a-2d-index"></a>

De forma predeterminada, un `2d`índice asume longitud y latitud y tiene límites de -180 **inclusivos** y 180 **no inclusivos** . Si los documentos contienen datos de coordenadas fuera del rango especificado, MongoDB devuelve un error.IMPORTANTE

Los límites predeterminados permiten que las aplicaciones inserten documentos con latitudes no válidas superiores a 90 o inferiores a -90. El comportamiento de las consultas geoespaciales con tales puntos no válidos no está definido.

En los `2d`índices, puede cambiar el rango de ubicación.

Puede crear un `2d`índice geoespacial con un rango de ubicación diferente al predeterminado. Utilice las opciones `min`y `max`al crear el índice. Utilice la siguiente sintaxis:

```text
db.collection.createIndex( { <location field> : "2d" } ,                           { min : <lower bound> , max : <upper bound> } )
```

### Definir la precisión de ubicación para un `2d`índice  <a id="define-location-precision-for-a-2d-index"></a>

De forma predeterminada, un `2d`índice en pares de coordenadas heredados usa 26 bits de precisión, lo que equivale aproximadamente a 2 pies o 60 centímetros de precisión usando el rango predeterminado de -180 a 180. La precisión se mide por el tamaño en bits de los valores [geohash](https://docs.mongodb.com/manual/reference/glossary/#std-term-geohash) usados para almacenar datos de ubicación. Puede configurar índices geoespaciales con hasta 32 bits de precisión.

La precisión del índice no afecta la precisión de la consulta. Las coordenadas reales de la cuadrícula siempre se utilizan en el procesamiento final de la consulta. Las ventajas de una menor precisión son una menor sobrecarga de procesamiento para las operaciones de inserción y el uso de menos espacio. Una ventaja de una mayor precisión es que las consultas escanean porciones más pequeñas del índice para devolver resultados.

Para configurar una precisión de ubicación diferente a la predeterminada, use la `bits`opción al crear el índice. Utilice la siguiente sintaxis:

```text
db.<collection>.createIndex( {<location field> : "<index type>"} ,                             { bits : <bit precision> } )
```

Para obtener información sobre los aspectos internos de los valores de geohash, consulte [Cálculo de valores de Geohash para `2d`índices](https://docs.mongodb.com/manual/core/geospatial-indexes/#std-label-geospatial-indexes-geohash) .

