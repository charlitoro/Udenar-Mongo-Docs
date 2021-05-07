# Validación del Esquema

MongoDB proporciona la capacidad de realizar la validación del esquema durante las actualizaciones y las inserciones.

### Especificar reglas de validación  <a id="specify-validation-rules"></a>

Las reglas de validación se basan en cada colección.

Para especificar reglas de validación al crear una nueva colección, úsela [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#mongodb-method-db.createCollection)con la `validator`opción.

Para agregar validación de documentos a una colección existente, use el [`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-dbcommand-dbcmd.collMod)comando con la `validator`opción.

MongoDB también proporciona las siguientes opciones relacionadas:

* `validationLevel` opción, que determina qué tan estrictamente aplica MongoDB las reglas de validación a los documentos existentes durante una actualización, y
* `validationAction`, que determina si MongoDB debe `error`rechazar y rechazar los documentos que violan las reglas de validación o `warn`sobre las violaciones en el registro pero permiten documentos no válidos.

### Esquema JSON  <a id="json-schema"></a>

_Nuevo en la versión 3.6_ .

A partir de la versión 3.6, MongoDB admite la validación de esquemas JSON. Para especificar la validación del esquema JSON, use el [`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#mongodb-query-op.-jsonSchema)operador en su `validator`expresión.NOTA

JSON Schema es el medio recomendado para realizar la validación del esquema.

Por ejemplo, el siguiente ejemplo especifica las reglas de validación mediante el esquema JSON:

```text
db.createCollection("students", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: [ "name", "year", "major", "address" ],
         properties: {
            name: {
               bsonType: "string",
               description: "must be a string and is required"
            },
            year: {
               bsonType: "int",
               minimum: 2017,
               maximum: 3017,
               description: "must be an integer in [ 2017, 3017 ] and is required"
            },
            major: {
               enum: [ "Math", "English", "Computer Science", "History", null ],
               description: "can only be one of the enum values and is required"
            },
            gpa: {
               bsonType: [ "double" ],
               description: "must be a double if the field exists"
            },
            address: {
               bsonType: "object",
               required: [ "city" ],
               properties: {
                  street: {
                     bsonType: "string",
                     description: "must be a string if the field exists"
                  },
                  city: {
                     bsonType: "string",
                     description: "must be a string and is required"
                  }
               }
            }
         }
      }
   }
})
```

Para obtener más información, consulte [`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#mongodb-query-op.-jsonSchema).

### Otras expresiones de consulta  <a id="other-query-expressions"></a>

Además de la validación del esquema JSON que utiliza el [`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#mongodb-query-op.-jsonSchema)operador de consulta, MongoDB admite la validación con [otros operadores de consulta](https://docs.mongodb.com/manual/reference/operator/query/#std-label-query-selectors) , con la excepción de:

* [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#mongodb-query-op.-near),
* [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#mongodb-query-op.-nearSphere),
* [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text),
* [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where), y
* [`$expr`](https://docs.mongodb.com/manual/reference/operator/query/expr/#mongodb-query-op.-expr)con [`$function`](https://docs.mongodb.com/manual/reference/operator/aggregation/function/#mongodb-expression-exp.-function)expresiones.

Por ejemplo, el siguiente ejemplo especifica las reglas del validador mediante la expresión de consulta:

```text
db.createCollection( "contacts",
   { validator: { $or:
      [
         { phone: { $type: "string" } },
         { email: { $regex: /@mongodb\.com$/ } },
         { status: { $in: [ "Unknown", "Incomplete" ] } }
      ]
   }
} )
```

**CONSEJO**

[operadores de consulta](https://docs.mongodb.com/manual/reference/operator/query/#std-label-query-selectors)

### Comportamiento  <a id="behavior"></a>

La validación se produce durante las actualizaciones y las inserciones. Cuando agrega validación a una colección, los documentos existentes no se someten a controles de validación hasta que se modifican.

#### Documentos existentes  <a id="existing-documents"></a>

La `validationLevel`opción determina qué operaciones MongoDB aplica las reglas de validación:

* Si `validationLevel`es `strict`\(el valor predeterminado\), MongoDB aplica reglas de validación a todas las inserciones y actualizaciones.
* Si `validationLevel`es así `moderate`, MongoDB aplica reglas de validación a inserciones y actualizaciones de documentos existentes que ya cumplen con los criterios de validación. Con el `moderate`nivel, no se comprueba la validez de las actualizaciones de los documentos existentes que no cumplen los criterios de validación.

Por ejemplo, cree una `contacts`colección con los siguientes documentos:

```text
db.contacts.insert([
   { "_id": 1, "name": "Anne", "phone": "+1 555 123 456", "city": "London", "status": "Complete" },
   { "_id": 2, "name": "Ivan", "city": "Vancouver" }
])
```

Emita el siguiente comando para agregar un validador a la `contacts` colección:

```text
db.runCommand( {
   collMod: "contacts",
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "phone", "name" ],
      properties: {
         phone: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         name: {
            bsonType: "string",
            description: "must be a string and is required"
         }
      }
   } },
   validationLevel: "moderate"
} )
```

La `contacts`colección ahora tiene un validador con `moderate` validationLevel:

* Si intentó actualizar el documento con `_id`de `1`, MongoDB aplicaría las reglas de validación ya que el documento existente coincide con los criterios.
* Por el contrario, MongoDB no se aplicará reglas de validación a las actualizaciones al documento con el `_id`del `2`ya que no cumple con las reglas de validación.

Para desactivar la validación del todo, se puede establecer `validationLevel`a `off`.

#### Aceptar o rechazar documentos no válidos  <a id="accept-or-reject-invalid-documents"></a>

La `validationAction`opción determina cómo MongoDB maneja los documentos que violan las reglas de validación:

* Si `validationAction`es `error`\(el valor predeterminado\), MongoDB rechaza cualquier inserción o actualización que viole los criterios de validación.
* Si `validationAction`es así `warn`, MongoDB registra cualquier infracción pero permite que continúe la inserción o actualización.

Por ejemplo, cree una `contacts2`colección con el siguiente validador de esquema JSON:

```text
db.createCollection( "contacts2", {
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "phone" ],
      properties: {
         phone: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         email: {
            bsonType : "string",
            pattern : "@mongodb\.com$",
            description: "must be a string and match the regular expression pattern"
         },
         status: {
            enum: [ "Unknown", "Incomplete" ],
            description: "can only be one of the enum values"
         }
      }
   } },
   validationAction: "warn"
} )
```

Con el `warn` [`validationAction`](https://docs.mongodb.com/manual/reference/command/collMod/#mongodb-collflag-validationAction), MongoDB registra cualquier infracción, pero permite que continúe la inserción o actualización.

Por ejemplo, la siguiente operación de inserción viola la regla de validación:

```text
db.contacts2.insert( { name: "Amanda", status: "Updated" } )
```

Sin embargo, dado que `validationAction`es `warn`solo, MongoDB solo registra el mensaje de violación de validación y permite que la operación continúe:

```text
2017-12-01T12:31:23.738-05:00 W STORAGE  [conn1] Document would fail validation collection: example.contacts2 doc: { _id: ObjectId('5a2191ebacbbfc2bdc4dcffc'), name: "Amanda", status: "Updated" }
```

### Restricciones  <a id="restrictions"></a>

No se puede especificar un validador para las colecciones en las `admin`, `local`y `config`las bases de datos.

No puede especificar un validador para `system.*`colecciones.

### Omitir la validación de documentos  <a id="bypass-document-validation"></a>

Los usuarios pueden omitir la validación de documentos usando la `bypassDocumentValidation`opción.

Los siguientes comandos pueden omitir la validación por operación usando la nueva opción `bypassDocumentValidation`:

* [`applyOps`](https://docs.mongodb.com/manual/reference/command/applyOps/#mongodb-dbcommand-dbcmd.applyOps) mando
* [`findAndModify`](https://docs.mongodb.com/manual/reference/command/findAndModify/#mongodb-dbcommand-dbcmd.findAndModify)comando y [`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#mongodb-method-db.collection.findAndModify)método
* [`mapReduce`](https://docs.mongodb.com/manual/reference/command/mapReduce/#mongodb-dbcommand-dbcmd.mapReduce)comando y [`db.collection.mapReduce()`](https://docs.mongodb.com/manual/reference/method/db.collection.mapReduce/#mongodb-method-db.collection.mapReduce)método
* [`insert`](https://docs.mongodb.com/manual/reference/command/insert/#mongodb-dbcommand-dbcmd.insert) mando
* [`update`](https://docs.mongodb.com/manual/reference/command/update/#mongodb-dbcommand-dbcmd.update) mando
* [`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out)y [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)etapas para el [`aggregate`](https://docs.mongodb.com/manual/reference/command/aggregate/#mongodb-dbcommand-dbcmd.aggregate)comando y el [`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate)método

Para las implementaciones que han habilitado el control de acceso, para evitar la validación de documentos, el usuario autenticado debe tener [`bypassDocumentValidation`](https://docs.mongodb.com/manual/reference/privilege-actions/#mongodb-authaction-bypassDocumentValidation)acción. Los roles incorporados [`dbAdmin`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-dbAdmin)y [`restore`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-restore)proporcionan esta acción.  


