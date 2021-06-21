# Índices comodín

MongoDB admite la creación de índices en un campo o conjunto de campos para admitir consultas. Dado que MongoDB admite esquemas dinámicos, las aplicaciones pueden consultar campos cuyos nombres no se pueden conocer de antemano o son arbitrarios.

_Nuevo en la versión MongoDB_ : 4.2

MongoDB 4.2 introduce índices comodín para admitir consultas en campos desconocidos o arbitrarios.

Considere una aplicación que captura datos definidos por el usuario debajo del `userMetadata`campo y admite consultas contra esos datos:

```text
{ "userMetadata" : { "likes" : [ "dogs", "cats" ] } }{ "userMetadata" : { "dislikes" : "pickles" } }{ "userMetadata" : { "age" : 45 } }{ "userMetadata" : "inactive" }
```

Los administradores quieren crear índices para admitir consultas en cualquier subcampo de `userMetadata`.

Un índice de comodín en `userMetadata` admita consultas de un solo campo en `userMetadata`, `userMetadata.likes`, `userMetadata.dislikes`, y `userMetadata.age`:

```text
db.userData.createIndex( { "userMetadata.$**" : 1 } )
```

El índice puede admitir las siguientes consultas:

```text
db.userData.find({ "userMetadata.likes" : "dogs" })db.userData.find({ "userMetadata.dislikes" : "pickles" })db.userData.find({ "userMetadata.age" : { $gt : 30 } })db.userData.find({ "userMetadata" : "inactive" })
```

Un índice que no sea comodín `userMetadata`solo puede admitir consultas sobre valores de `userMetadata`.IMPORTANTE

Los índices comodín no están diseñados para reemplazar la planificación de índices basada en cargas de trabajo. Para obtener más información sobre cómo crear índices para respaldar consultas, consulte [Crear](https://docs.mongodb.com/manual/tutorial/create-indexes-to-support-queries/#std-label-create-indexes-to-support-queries) índices para respaldar [sus consultas](https://docs.mongodb.com/manual/tutorial/create-indexes-to-support-queries/#std-label-create-indexes-to-support-queries) . Para obtener documentación completa sobre las limitaciones del índice de comodines, consulte [Restricciones del índice de comodines](https://docs.mongodb.com/manual/reference/index-wildcard-restrictions/#std-label-wildcard-index-restrictions) .

### Crear índice comodín  <a id="create-wildcard-index"></a>

IMPORTANTE

El [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) debe ser para crear índices comodín. Para obtener instrucciones sobre cómo configurar la fCV, consulte [Establecer versión de compatibilidad de](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-set-fcv) funciones [en implementaciones de MongoDB 4.4](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-set-fcv) .[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) `4.2`

Puede crear índices comodín utilizando el [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes)comando de la base de datos o sus ayudantes de shell, [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)o [`createIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndexes/#mongodb-method-db.collection.createIndexes).

#### Crear un índice comodín en un campo  <a id="create-a-wildcard-index-on-a-field"></a>

Para indexar el valor de un campo específico:

```text
db.collection.createIndex( { "fieldA.$**" : 1 } )
```

Con este índice comodín, MongoDB indexa todos los valores de `fieldA`. Si el campo es un documento o una matriz anidados, el índice comodín se repite en el documento o la matriz y almacena el valor de todos los campos en el documento o la matriz.

Por ejemplo, los documentos de la `product_catalog`colección pueden contener un `product_attributes`campo. El `product_attributes`campo puede contener campos anidados arbitrarios, incluidos documentos y matrices incrustados:

```text
{  "product_name" : "Spy Coat",  "product_attributes" : {    "material" : [ "Tweed", "Wool", "Leather" ]    "size" : {      "length" : 72,      "units" : "inches"    }  }}{  "product_name" : "Spy Pen",  "product_attributes" : {     "colors" : [ "Blue", "Black" ],     "secret_feature" : {       "name" : "laser",       "power" : "1000",       "units" : "watts",     }  }}
```

La siguiente operación crea un índice comodín en el `product_attributes`campo:

```text
db.products_catalog.createIndex( { "product_attributes.$**" : 1 } )
```

El índice comodín puede admitir consultas arbitrarias de un solo campo en `product_attributes`sus campos integrados:

```text
db.products_catalog.find( { "product_attributes.size.length" : { $gt : 60 } } )db.products_catalog.find( { "product_attributes.material" : "Leather" } )db.products_catalog.find( { "product_attributes.secret_feature.name" : "laser" } )
```

NOTA

La sintaxis del índice comodín específico de la ruta es incompatible con la `wildcardProjection`opción. Consulte las [Opciones de `wildcard`índices](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-option) para obtener más información.

Para ver un ejemplo, consulte [Crear un índice comodín en una ruta de un solo campo](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-onepath) .

#### Crear un índice comodín en todos los campos  <a id="create-a-wildcard-index-on-all-fields"></a>

Para indexar el valor de todos los campos en un documento \(excluyendo `_id`\), especifique `"$**"`como clave de índice:

```text
db.collection.createIndex( { "$**" : 1 } )
```

Con este índice comodín, MongoDB indexa todos los campos de cada documento de la colección. Si un campo determinado es un documento o una matriz anidados, el índice comodín se repite en el documento / matriz y almacena el valor de todos los campos en el documento / matriz.

Para ver un ejemplo, consulte [Crear un índice comodín en todas las rutas de campo](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-allpaths) .NOTA

Los índices comodín omiten el `_id`campo de forma predeterminada. Para incluir el `_id`campo en el índice comodín, debe incluirlo explícitamente en el `wildcardProjection`documento. Consulte [Opciones de `wildcard`índices](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-option) para obtener más información.

#### Cree un índice comodín en varios campos específicos  <a id="create-a-wildcard-index-on-multiple-specific-fields"></a>

Para indexar los valores de varios campos específicos en un documento:

```text
db.collection.createIndex(  { "$**" : 1 },  { "wildcardProjection" :    { "fieldA" : 1, "fieldB.fieldC" : 1 }  })
```

Con este índice comodín, MongoDB indexa todos los valores de los campos especificados para cada documento de la colección. Si un campo determinado es un documento o una matriz anidados, el índice comodín se repite en el documento / matriz y almacena el valor de todos los campos en el documento / matriz.NOTA

Los índices comodín no admiten la combinación de declaraciones de inclusión y exclusión en el `wildcardProjection`documento, _excepto_ cuando se incluye explícitamente el `_id`campo. Para obtener más información sobre `wildcardProjection`, consulte [Opciones para `wildcard`índices](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-option) .

Para obtener un ejemplo, consulte [Incluir campos específicos en la cobertura del índice comodín](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-inclusion) .

#### Cree un índice comodín que excluya varios campos específicos  <a id="create-a-wildcard-index-that-excludes-multiple-specific-fields"></a>

Para indexar los campos de todos los campos en un documento _excluyendo_ rutas de campo específicas:

```text
db.collection.createIndex(  { "$**" : 1 },  { "wildcardProjection" :    { "fieldA" : 0, "fieldB.fieldC" : 0 }  })
```

Con este índice comodín, MongoDB indexa todos los campos de cada documento de la colección, _excluyendo_ las rutas de campo especificadas. Si un campo determinado es un documento o una matriz anidados, el índice comodín se repite en el documento / matriz y almacena los valores de todos los campos en el documento / matriz.

Para obtener un ejemplo, consulte [Omitir campos específicos de la cobertura del índice comodín](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-exclusion) .NOTA

Los índices comodín no admiten la combinación de declaraciones de inclusión y exclusión en el `wildcardProjection`documento, _excepto_ cuando se incluye explícitamente el `_id`campo. Para obtener más información sobre `wildcardProjection`, consulte [Opciones para `wildcard`índices](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#std-label-createIndex-method-wildcard-option) .

### Consideraciones  <a id="considerations"></a>

* Los índices comodín pueden admitir como máximo _un_ campo en cualquier predicado de consulta determinado. Para obtener más información sobre la compatibilidad con consultas de índice comodín, consulte [Compatibilidad con consultas / clasificación de índice comodín](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-index-query-sort-support) .
* El [featureCompatibilityVersion](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) debe ser para crear índices comodín. Para obtener instrucciones sobre cómo configurar la fCV, consulte [Establecer versión de compatibilidad de](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-set-fcv) funciones [en implementaciones de MongoDB 4.4](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-set-fcv) .[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod) `4.2`
* Los índices comodín omiten el campo \_id de forma predeterminada. Para incluir el `_id` campo en el índice comodín, debe incluirlo explícitamente en el documento wildcardProjection \(es decir `{ "_id" : 1 }`\).
* Puede crear varios índices comodín en una colección.
* Un índice comodín puede cubrir los mismos campos que otros índices de la colección.
* Los índices comodín son índices [dispersos](https://docs.mongodb.com/manual/core/index-sparse/) y solo contienen entradas para documentos que tienen el campo indexado, incluso si el campo de índice contiene un valor nulo.

### Comportamiento  <a id="behavior"></a>

Los índices comodín tienen un comportamiento específico al indexar campos que son un objeto \(es decir, un documento incrustado\) o una matriz:

* Si el campo es un objeto, el índice comodín desciende al objeto e indexa su contenido. El índice de comodines continúa descendiendo a los documentos incrustados adicionales que encuentra.
* Si el campo es una matriz, el índice comodín atraviesa la matriz e indexa cada elemento:
  * Si un elemento de la matriz es un objeto, el índice comodín desciende al objeto para indexar su contenido como se describe anteriormente.
  * Si el elemento es una matriz, es decir, una matriz que está incrustada directamente dentro de la matriz principal, entonces el índice comodín _no_ atraviesa la matriz incrustada, sino que indexa _toda la_ matriz como un valor único.
* Para todos los demás campos, registre el valor primitivo \(no objeto / matriz\) en el índice.

El índice comodín continúa atravesando cualquier objeto o arreglo adicional anidado hasta que alcanza un valor primitivo \(es decir, un campo que no es un objeto o arreglo\). Luego indexa este valor primitivo, junto con la ruta completa a ese campo.

Por ejemplo, considere el siguiente documento:

```text
{  "parentField" : {    "nestedField" : "nestedValue",    "nestedObject" : {      "deeplyNestedField" : "deeplyNestedValue"    },    "nestedArray" : [      "nestedArrayElementOne",      [ "nestedArrayElementTwo" ]    ]  }}
```

Un índice comodín que incluye `parentField`registros de las siguientes entradas:

* `"parentField.nestedField" : "nestedValue"`
* `"parentField.nestedObject.deeplyNestedField" : "deeplyNestedValue"`
* `"parentField.nestedArray" : "nestedArrayElementOne"`
* `"parentField.nestedArray" : ["nestedArrayElementTwo"]`

Tenga en cuenta que los registros de `parentField.nestedArray`no incluyen la posición de la matriz para cada elemento. Los índices comodín ignoran las posiciones de los elementos de la matriz al registrar el elemento en el índice. Los índices comodín aún pueden admitir consultas que incluyen índices de matriz explícitos. Consulte [Consultas con índices de matriz explícitos](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-query-support-explicit-array-indices) para obtener más información.

Para obtener más información sobre el comportamiento del índice comodín con objetos anidados, consulte [Objetos anidados](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-index-nested-objects) .

Para obtener más información sobre el comportamiento del índice comodín con matrices anidadas, consulte [Matrices anidadas](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-index-nested-arrays) .

#### Objetos anidados  <a id="nested-objects"></a>

Cuando un índice comodín encuentra un objeto anidado, desciende al objeto e indexa su contenido. Por ejemplo:

```text
{  "parentField" : {    "nestedField" : "nestedValue",    "nestedArray" : ["nestedElement"]    "nestedObject" : {      "deeplyNestedField" : "deeplyNestedValue"    }  }}
```

Un índice comodín que incluye `parentField`desciende al objeto para recorrer e indexar su contenido:

* Para cada campo que sea en sí mismo un objeto \(es decir, un documento incrustado\), descienda al objeto para indexar su contenido.
* Para cada campo que sea una matriz, recorra la matriz e indexe su contenido.
* Para todos los demás campos, registre el valor primitivo \(no objeto / matriz\) en el índice.

El índice comodín continúa atravesando cualquier objeto o arreglo adicional anidado hasta que alcanza un valor primitivo \(es decir, un campo que no es un objeto o arreglo\). Luego indexa este valor primitivo, junto con la ruta completa a ese campo.

Dado el documento de muestra, el índice comodín agrega los siguientes registros al índice:

* `"parentField.nestedField" : "nestedValue"`
* `"parentField.nestedObject.deeplyNestedField" : "deeplyNestedValue"`
* `"parentField.nestedArray" : "nestedElement"`

Para obtener más información sobre el comportamiento del índice comodín con matrices anidadas, consulte [Matrices anidadas](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-index-nested-arrays) .

#### Matrices anidadas  <a id="nested-arrays"></a>

Cuando un índice comodín encuentra una matriz anidada, intenta atravesar la matriz para indexar sus elementos. Si la matriz es en sí misma un elemento de una matriz principal \(es decir, una matriz incrustada\), el índice comodín registra la matriz completa como un valor en lugar de atravesar su contenido. Por ejemplo:

```text
{  "parentArray" : [    "arrayElementOne",    [ "embeddedArrayElement" ],    "nestedObject" : {      "nestedArray" : [        "nestedArrayElementOne",        "nestedArrayElementTwo"      ]    }  ]}
```

Un índice comodín que incluye `parentArray`desciende a la matriz para recorrer e indexar su contenido:

* Para cada elemento que sea una matriz \(es decir, una matriz incrustada\), indexe la matriz _completa_ como un valor.
* Para cada elemento que sea un objeto, descienda al objeto para recorrer e indexar su contenido.
* Para todos los demás campos, registre el valor primitivo \(no objeto / matriz\) en el índice.

El índice comodín continúa atravesando cualquier objeto o arreglo adicional anidado hasta que alcanza un valor primitivo \(es decir, un campo que no es un objeto o arreglo\). Luego indexa este valor primitivo, junto con la ruta completa a ese campo.

Dado el documento de muestra, el índice comodín agrega los siguientes registros al índice:

* `"parentArray" : "arrayElementOne"`
* `"parentArray" : ["embeddedArrayElement"]`
* `"parentArray.nestedObject.nestedArray" : "nestedArrayElementOne"`
* `"parentArray.nestedObject.nestedArray" : "nestedArrayElementTwo"`

Tenga en cuenta que los registros de `parentField.nestedArray`no incluyen la posición de la matriz para cada elemento. Los índices comodín ignoran las posiciones de los elementos de la matriz al registrar el elemento en el índice. Los índices comodín aún pueden admitir consultas que incluyen índices de matriz explícitos. Consulte [Consultas con índices de matriz explícitos](https://docs.mongodb.com/manual/core/index-wildcard/#std-label-wildcard-query-support-explicit-array-indices) para obtener más información.CONSEJOVer también:

[`Nested Depth for BSON Documents`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Nested-Depth-for-BSON-Documents)

### Restricciones  <a id="restrictions"></a>

* No puede fragmentar una colección mediante un índice comodín. Cree un índice que no sea comodín en el campo o los campos en los que desea fragmentar. Para obtener más información sobre la selección de claves de fragmentos, consulte [Claves de fragmentos](https://docs.mongodb.com/manual/core/sharding-shard-key/#std-label-sharding-shard-key) .
* No puede crear un índice [compuesto](https://docs.mongodb.com/manual/core/index-compound/) .
* No puede especificar las siguientes propiedades para un índice comodín:
  * [TTL](https://docs.mongodb.com/manual/core/index-ttl/)
  * [Único](https://docs.mongodb.com/manual/core/index-unique/)
* No puede crear los siguientes tipos de índices utilizando la sintaxis de comodines:
  * [2d \(geoespacial\)](https://docs.mongodb.com/manual/core/geospatial-indexes/)
  * [2dsphere \(geoespacial\)](https://docs.mongodb.com/manual/core/2dsphere/)
  * [Troceado](https://docs.mongodb.com/manual/core/index-hashed/)

IMPORTANTE

Los índices comodín son distintos e incompatibles con los [índices de texto comodín](https://docs.mongodb.com/manual/core/index-text/#std-label-text-index-wildcard) . Los índices comodín no pueden admitir consultas mediante el [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)operador.

Para obtener documentación completa sobre las restricciones de creación de índices comodín, consulte [Propiedades o tipos de índices incompatibles](https://docs.mongodb.com/manual/reference/index-wildcard-restrictions/#std-label-wildcard-index-restrictions-create) .

### Soporte de consulta / clasificación de índice comodín  <a id="wildcard-index-query-sort-support"></a>

#### Consultas cubiertas  <a id="covered-queries"></a>

Los índices comodín pueden admitir una [consulta cubierta ](https://docs.mongodb.com/manual/core/query-optimization/#std-label-covered-queries)**solo si se cumplen** todas las condiciones siguientes:

* El planificador de consultas selecciona el índice comodín para satisfacer el predicado de la consulta.
* El predicado de la consulta especifica _exactamente_ un campo cubierto por el índice comodín.
* La proyección excluye `_id`e incluye explícitamente _solo_ el campo de consulta.
* El campo de consulta especificado nunca es una matriz.

Considere el siguiente índice comodín en la `employees`colección:

```text
db.products.createIndex( { "$**" : 1 } )
```

La siguiente operación consulta para un solo campo `lastName`y proyecta todos los demás campos del documento resultante:

```text
db.products.find(  { "lastName" : "Doe" },  { "_id" : 0, "lastName" : 1 })
```

Suponiendo que lo especificado `lastName`nunca es una matriz, MongoDB puede usar el `$**`índice comodín para admitir una consulta cubierta.

#### Predicados de consulta de campo múltiple  <a id="multi-field-query-predicates"></a>

Los índices comodín pueden admitir como máximo _un_ campo de predicado de consulta. Es decir:

* MongoDB no puede utilizar un índice que no sea comodín para satisfacer una parte de un predicado de consulta y un índice comodín para satisfacer otra.
* MongoDB no puede usar un índice comodín para satisfacer una parte de un predicado de consulta y otro índice comodín para satisfacer otro.
* Incluso si un único índice comodín pudiera admitir varios campos de consulta, MongoDB puede usar el índice comodín para admitir solo uno de los campos de consulta. Todos los campos restantes se resuelven sin índice.

Sin embargo, MongoDB puede usar el mismo índice comodín para satisfacer cada argumento independiente de los operadores de consulta [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#mongodb-query-op.-or)o agregación [`$or`](https://docs.mongodb.com/manual/reference/operator/aggregation/or/#mongodb-expression-exp.-or).

#### Consultas con Ordenar  <a id="queries-with-sort"></a>

MongoDB puede usar un índice comodín para satisfacer el **solo si** todo lo siguiente es verdadero:[`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#mongodb-method-cursor.sort)

* El planificador de consultas selecciona el índice comodín para satisfacer el predicado de la consulta.
* Las [`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#mongodb-method-cursor.sort)especifica **solamente** el campo de consulta predicado.
* El campo especificado nunca es una matriz.

Si no se cumplen las condiciones anteriores, MongoDB no puede usar el índice comodín para la ordenación. MongoDB no admite [`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#mongodb-method-cursor.sort) operaciones que requieran un índice diferente al del predicado de consulta. Para obtener más información, consulte [Intersección y ordenación de índices](https://docs.mongodb.com/manual/core/index-intersection/#std-label-index-intersection-sort) .

Considere el siguiente índice comodín en la `products`colección:

```text
db.products.createIndex( { "product_attributes.$**" : 1 } )
```

La siguiente operación consulta para un solo campo `product_attributes.price`y ordena en ese mismo campo:

```text
db.products.find(  { "product_attributes.price" : { $gt : 10.00 } },).sort(  { "product_attributes.price" : 1 })
```

Suponiendo que lo especificado `price`nunca es una matriz, MongoDB puede usar el `product_attributes.$**`índice comodín para satisfacer tanto el [`find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#mongodb-method-db.collection.find)y [`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#mongodb-method-cursor.sort).

#### Patrones de consulta no admitidos  <a id="unsupported-query-patterns"></a>

* Los índices comodín no pueden admitir la condición de consulta que comprueba si un campo no existe.
* Los índices comodín no pueden admitir la condición de consulta que comprueba si un campo es o no igual a un documento o una matriz.
* Los índices comodín no pueden admitir la condición de consulta que comprueba si un campo no es igual a nulo.

Para obtener más información, consulte [Patrones de consulta y agregación no admitidos](https://docs.mongodb.com/manual/reference/index-wildcard-restrictions/#std-label-wildcard-index-restrictions-query-aggregation) .

#### Consultas con índices de matriz explícitos  <a id="queries-with-explicit-array-indices"></a>

Los índices comodín de MongoDB no registran la posición de la matriz de ningún elemento dado en una matriz durante la indexación. Sin embargo, MongoDB aún puede seleccionar el índice comodín para responder a una consulta que incluye una ruta de campo con uno o más índices de matriz explícitos \(por ejemplo, `parentArray.0.nestedArray.0`\). Debido a la creciente complejidad de definir límites de índice para cada matriz anidada consecutiva, MongoDB no considera que el índice comodín responda a una ruta de campo determinada en la consulta si esa ruta contiene más que `8`índices de matriz explícitos. MongoDB aún puede considerar el índice comodín para responder a otras rutas de campo en la consulta.

Por ejemplo:

```text
{  "parentObject" : {    "nestedArray" : [       "elementOne",       {         "deeplyNestedArray" : [ "elementTwo" ]       }     ]  }}
```

MongoDB puede seleccionar un índice comodín que incluye `parentObject`satisfacer las siguientes consultas:

* `"parentObject.nestedArray.0" : "elementOne"`
* `"parentObject.nestedArray.1.deeplyNestedArray.0" : "elementTwo"`

Si una ruta de campo determinada en el predicado de consulta especifica más de 8 índices de matriz explícitos, MongoDB no considera el índice comodín para responder a esa ruta de campo. En cambio, MongoDB selecciona otro índice elegible para responder a la consulta _o_ realiza un análisis de la colección.

Tenga en cuenta que los índices comodín en sí mismos no tienen ningún límite en la profundidad a la que atraviesan un documento mientras lo indexan; la limitación solo se aplica a consultas que especifican explícitamente índices de matriz exactos. Al emitir las mismas consultas sin los índices de matriz explícitos, MongoDB puede seleccionar el índice comodín para responder a la consulta:

* `"parentObject.nestedArray" : "elementOne"`
* `"parentObject.nestedArray.deeplyNestedArray" : "elementTwo"`

CONSEJOVer también:

[`Nested Depth for BSON Documents`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Nested-Depth-for-BSON-Documents)

