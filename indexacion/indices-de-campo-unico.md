# Índices de campo único

MongoDB proporciona soporte completo para índices en cualquier campo en una [colección](https://docs.mongodb.com/manual/reference/glossary/#std-term-collection) de [documentos](https://docs.mongodb.com/manual/reference/glossary/#std-term-document) . De forma predeterminada, todas las colecciones tienen un índice en el [campo \_id](https://docs.mongodb.com/manual/indexes/#std-label-index-type-id) , y las aplicaciones y los usuarios pueden agregar índices adicionales para admitir consultas y operaciones importantes.

Este documento describe índices ascendentes / descendentes en un solo campo.![Diagrama de un &#xED;ndice en el campo \`\` puntuaci&#xF3;n &apos;&apos; \(ascendente\).](https://docs.mongodb.com/manual/images/index-ascending.bakedsvg.svg)

### Cree un índice ascendente en un solo campo  <a id="create-an-ascending-index-on-a-single-field"></a>

Considere una colección nombrada `records`que contiene documentos que se parecen al siguiente documento de muestra:

```text
{  "_id": ObjectId("570c04a4ad233577f97dc459"),  "score": 1034,  "location": { state: "NY", city: "New York" }}
```

La siguiente operación crea un índice ascendente en el `score` campo de la `records`colección:

```text
db.records.createIndex( { score: 1 } )
```

El valor del campo en la especificación del índice describe el tipo de índice para ese campo. Por ejemplo, un valor de `1`especifica un índice que ordena los elementos en orden ascendente. Un valor de `-1`especifica un índice que ordena los elementos en orden descendente. Para tipos de índices adicionales, consulte [tipos de índices](https://docs.mongodb.com/manual/indexes/#std-label-index-types) .

El índice creado admitirá consultas que se seleccionen en el campo `score`, como las siguientes:

```text
db.records.find( { score: 2 } )db.records.find( { score: { $gt: 10 } } )
```

### Crear un índice en un campo incrustado  <a id="create-an-index-on-an-embedded-field"></a>

Puede crear índices en campos dentro de documentos incrustados, al igual que puede indexar campos de nivel superior en documentos. Los índices de los campos incrustados difieren de los [índices de los documentos incrustados](https://docs.mongodb.com/manual/core/index-single/#std-label-index-embedded-documents) , que incluyen el contenido completo hasta el máximo [`index size`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Key-Limit)del documento incrustado en el índice. En cambio, los índices en los campos incrustados le permiten utilizar una "notación de puntos" para introspectar los documentos incrustados.

Considere una colección nombrada `records`que contiene documentos que se parecen al siguiente documento de muestra:

```text
{  "_id": ObjectId("570c04a4ad233577f97dc459"),  "score": 1034,  "location": { state: "NY", city: "New York" }}
```

La siguiente operación crea un índice en el `location.state` campo:

```text
db.records.createIndex( { "location.state": 1 } )
```

El índice creado admitirá consultas que se seleccionen en el campo `location.state`, como las siguientes:

```text
db.records.find( { "location.state": "CA" } )db.records.find( { "location.city": "Albany", "location.state": "NY" } )
```

### Crear un índice en un documento incrustado  <a id="create-an-index-on-embedded-document"></a>

También puede crear índices en el documento incrustado como un todo.

Considere una colección nombrada `records`que contiene documentos que se parecen al siguiente documento de muestra:

```text
{  "_id": ObjectId("570c04a4ad233577f97dc459"),  "score": 1034,  "location": { state: "NY", city: "New York" }}
```

El `location`campo es un documento incrustado, que contiene los campos incrustados `city`y `state`. El siguiente comando crea un índice en el `location` campo como un todo:

```text
db.records.createIndex( { location: 1 } )
```

La siguiente consulta puede usar el índice en el `location`campo:

```text
db.records.find( { location: { city: "New York", state: "NY" } } )
```

NOTA

Aunque la consulta puede utilizar el índice, el conjunto de resultados no incluye el documento de muestra anterior. Al realizar coincidencias de igualdad en documentos incrustados, el orden de los campos es importante y los documentos incrustados deben coincidir exactamente. Consulte [Consultar documentos incrustados](https://docs.mongodb.com/manual/reference/method/db.collection.find/#std-label-query-embedded-documents) para obtener más información sobre la consulta de documentos incrustados.

### Consideraciones adicionales  <a id="additional-considerations"></a>

Las aplicaciones pueden encontrar un rendimiento reducido durante la creación de índices, incluido el acceso limitado de lectura / escritura a la colección. Para obtener más información sobre el proceso de generación de índices, consulte [Generaciones de índices en colecciones pobladas](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations) , incluida la sección [Generaciones de índices en entornos replicados](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations-replicated-build) .

Algunos controladores pueden especificar índices, utilizando en `NumberLong(1)`lugar de `1`como especificación. Esto no tiene ningún efecto sobre el índice resultante.

