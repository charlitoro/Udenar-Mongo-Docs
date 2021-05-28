# Especificar el nombre del textíndice

**NOTA:** _Cambiado en MongoDB 4.2 , A partir de la versión 4.2, para_ [_featureCompatibilityVersion_](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) _establecido en `"4.2"`o superior, MongoDB elimina el_ [_`Index Name Length`_](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length)_límite de 127 bytes como máximo. En versiones anteriores o versiones de MongoDB con_ [_featureCompatibilityVersion_](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv) _\(fCV\) establecido en `"4.0"`, los nombres de índice deben estar dentro de_ [_`limit`_](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Index-Name-Length)_._

El nombre predeterminado para el índice consta de cada nombre de campo indexado concatenado `_text`. Por ejemplo, el siguiente comando crea un `text`índice en los campos `content`, `users.comments`y `users.profiles`:

```text
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   }
)
```

El nombre predeterminado del índice es:

```text
"content_text_users.comments_text_users.profiles_text"
```

### Especifique un nombre para el `text`índice  <a id="specify-a-name-for-text-index"></a>

Puedes pasar la `name`opción al [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método:

```text
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   },
   {
     name: "MyTextIndex"
   }
)
```

### Utilice el nombre del índice para soltar un `text`índice  <a id="use-the-index-name-to-drop-a-text-index"></a>

Ya sea que el índice de [texto](https://docs.mongodb.com/manual/core/index-text/) tenga el nombre predeterminado o que haya especificado un nombre para el índice de [texto](https://docs.mongodb.com/manual/core/index-text/) , para eliminar el índice de [texto](https://docs.mongodb.com/manual/core/index-text/) , pase el nombre del índice al [`db.collection.dropIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndex/#mongodb-method-db.collection.dropIndex)método.

Por ejemplo, considere el índice creado por la siguiente operación:

```text
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   },
   {
     name: "MyTextIndex"
   }
)
```

Luego, para eliminar este índice de texto, pase el nombre `"MyTextIndex"`al [`db.collection.dropIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndex/#mongodb-method-db.collection.dropIndex)método, como se muestra a continuación:

```text
db.collection.dropIndex("MyTextIndex")
```

Para obtener los nombres de los índices, use el [`db.collection.getIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.getIndexes/#mongodb-method-db.collection.getIndexes)método.

