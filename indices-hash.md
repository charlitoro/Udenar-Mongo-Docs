# Índices hash

Los índices hash mantienen entradas con hash de los valores del campo indexado.

Los índices hash admiten la [fragmentación](https://docs.mongodb.com/manual/sharding/) mediante claves de fragmentos hash. [La fragmentación basada en hash](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding) utiliza un índice hash de un campo como clave de fragmentación para particionar datos en su clúster fragmentado.

El uso de una clave de fragmento hash para fragmentar una colección da como resultado una distribución más uniforme de los datos. Consulte [Fragmentación hash](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding) para obtener más detalles.

### Función hash  <a id="hashing-function"></a>

Los índices hash utilizan una función hash para calcular el hash del valor del campo de índice. [\[ 1 \]](https://docs.mongodb.com/manual/core/index-hashed/#footnote-hashvalue) La función hash contrae los documentos incrustados y calcula el hash para todo el valor, pero no admite índices de varias claves \(es decir, matrices\). Específicamente, crear un índice hash en un campo que contiene una matriz _o_ intentar insertar una matriz en un campo indexado hash devuelve un error.CONSEJO

MongoDB calcula automáticamente los hash al resolver consultas utilizando índices hash. Las aplicaciones **no** necesitan calcular hashes.

| \[ [1](https://docs.mongodb.com/manual/core/index-hashed/#ref-hashvalue-id1) \] | A partir de la versión 4.0, el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell proporciona el método [`convertShardKeyToHashed()`](https://docs.mongodb.com/manual/reference/method/convertShardKeyToHashed/#mongodb-method-convertShardKeyToHashed). Este método usa la misma función hash que el índice hash y se puede usar para ver cuál sería el valor hash de una clave. |
| :--- | :--- |


### Crear un índice hash  <a id="create-a-hashed-index"></a>

Para crear un [índice hash](https://docs.mongodb.com/manual/core/index-hashed/#std-label-index-type-hashed) , especifique `hashed`como el valor de la clave de índice, como en el siguiente ejemplo:

```text
db.collection.createIndex( { _id: "hashed" } )
```

### Crear un índice hash compuesto  <a id="create-a-compound-hashed-index"></a>

_Nuevo en la versión 4.4_ .

A partir de MongoDB 4.4, MongoDB admite la creación de índices compuestos que incluyen un solo campo hash. Para crear un índice hash compuesto, especifique `hashed`como el valor de cualquier clave de índice individual al crear el índice:

```text
db.collection.createIndex( { "fieldA" : 1, "fieldB" : "hashed", "fieldC" : -1 } )
```

Los índices compuestos con hash requieren [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) establecido en `4.4`.CONSEJOVer también:

[Fragmentación hash](https://docs.mongodb.com/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding)

### Consideraciones  <a id="considerations"></a>

#### Documentos incrustados  <a id="embedded-documents"></a>

La función hash contrae los documentos incrustados y calcula el hash para todo el valor, pero no admite índices de varias claves \(es decir, matrices\). Específicamente, crear un índice hash en un campo que contiene una matriz _o_ intentar insertar una matriz en un campo indexado hash devuelve un error.

#### Restricción única  <a id="unique-constraint"></a>

MongoDB no admite la especificación de una restricción única en un `hashed` índice. En su lugar, puede crear un índice adicional sin hash con la restricción única en ese campo. MongoDB puede usar ese índice sin hash para imponer la unicidad en el campo.

#### 2 53 Límite  <a id="253-limit"></a>

ADVERTENCIA

Los `hashed`índices de MongoDB truncan los números de punto flotante a enteros de 64 bits antes del hash. Por ejemplo, un `hashed`índice de almacenaría el mismo valor para un campo que sostenía un valor de `2.3`, `2.2`y `2.9`. Para evitar colisiones, no utilice un `hashed`índice para números de coma flotante que no se puedan convertir de manera confiable a enteros de 64 bits \(y luego de nuevo a coma flotante\). Los `hashed`índices de MongoDB no admiten valores de coma flotante superiores a 2 53 .

Para ver cuál sería el valor hash de una clave, consulte [`convertShardKeyToHashed()`](https://docs.mongodb.com/manual/reference/method/convertShardKeyToHashed/#mongodb-method-convertShardKeyToHashed).

#### PowerPC y 2 63  <a id="powerpc-and-263"></a>

Para [índices hash](https://docs.mongodb.com/manual/core/index-hashed/) , MongoDB 4.2 asegura que el valor hash para el valor de coma flotante 2 63 en PowerPC sea consistente con otras plataformas.

Aunque los [índices hash](https://docs.mongodb.com/manual/core/index-hashed/) en un campo que puede contener valores de punto flotante superiores a 2 53 es una configuración no admitida, los clientes aún pueden insertar documentos donde el campo indexado tenga el valor 2 63 .

Para enumerar todos los `hashed`índices de todas las colecciones en su implementación, puede usar la siguiente operación en el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell:

```text
db.adminCommand("listDatabases").databases.forEach(function(d){   let mdb = db.getSiblingDB(d.name);   mdb.getCollectionInfos({ type: "collection" }).forEach(function(c){      let currentCollection = mdb.getCollection(c.name);      currentCollection.getIndexes().forEach(function(idx){        let idxValues = Object.values(Object.assign({}, idx.key));        if (idxValues.includes("hashed")) {          print("Hashed index: " + idx.name + " on " + d.name + "." + c.name);          printjson(idx);        };      });   });});
```

Para comprobar si el campo indexado contiene el valor 2 63 , ejecute la siguiente operación para la colección y el campo indexado:

* Si el tipo de campo indexado es un escalar y nunca un documento:

  ```text
  // substitute the actual collection name for <collection>// substitute the actual indexed field name for <indexfield>db.<collection>.find( { <indexfield>: Math.pow(2,63) } );
  ```

* Si el tipo de campo indexado es un documento \(o un escalar\), puede ejecutar:

  ```text
  // substitute the actual collection name for <collection>// substitute the actual indexed field name for <indexfield>db.<collection>.find({    $where: function() {        function findVal(obj, val) {            if (obj === val)                return true;            for (const child in obj) {                if (findVal(obj[child], val)) {                    return true;                }            }            return false;        }        return findVal(this.<indexfield>, Math.pow(2, 63));    }})
  ```

