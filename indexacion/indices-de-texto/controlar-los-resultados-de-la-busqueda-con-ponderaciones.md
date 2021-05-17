# Controlar los resultados de la búsqueda con ponderaciones

La búsqueda de texto asigna una puntuación a cada documento que contiene el término de búsqueda en los campos indexados. La puntuación determina la relevancia de un documento para una consulta de búsqueda determinada.

Para un `text`índice, el _peso_ de un campo indexado denota la importancia del campo en relación con los otros campos indexados en términos de la puntuación de búsqueda de texto.

Para cada campo indexado en el documento, MongoDB multiplica el número de coincidencias por el peso y suma los resultados. Con esta suma, MongoDB calcula la puntuación del documento. Consulte al [`$meta`](https://docs.mongodb.com/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta) operador para obtener detalles sobre la devolución y la clasificación por puntajes de texto.

El peso predeterminado es 1 para los campos indexados. Para ajustar los pesos de los campos indexados, incluya la `weights`opción en el [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método.ADVERTENCIA

Elija los pesos con cuidado para evitar la necesidad de volver a indexar.

Una colección `blog`tiene los siguientes documentos:

```text
{  _id: 1,  content: "This morning I had a cup of coffee.",  about: "beverage",  keywords: [ "coffee" ]}{  _id: 2,  content: "Who doesn't like cake?",  about: "food",  keywords: [ "cake", "food", "dessert" ]}
```

Para crear un `text`índice con diferentes pesos de campo para el `content`campo y el `keywords`campo, incluya la `weights` opción al [`createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex)método. Por ejemplo, el siguiente comando crea un índice en tres campos y asigna pesos a dos de los campos:

```text
db.blog.createIndex(   {     content: "text",     keywords: "text",     about: "text"   },   {     weights: {       content: 10,       keywords: 5     },     name: "TextIndex"   } )
```

El `text`índice tiene los siguientes campos y pesos:

* `content` tiene un peso de 10,
* `keywords` tiene un peso de 5, y
* `about` tiene el peso predeterminado de 1.

Estos pesos denotan la importancia relativa de los campos indexados entre sí. Por ejemplo, una coincidencia de términos en el `content`campo tiene:

* `2`veces \(es decir `10:5`\) el impacto como coincidencia de términos en el `keywords`campo y
* `10`veces \(es decir `10:1`\) el impacto como coincidencia de términos en el `about`campo.

NOTA

Para los datos alojados en MongoDB Atlas, [Atlas Search](https://docs.atlas.mongodb.com/atlas-search/) proporciona una puntuación personalizada más sólida que los `text`índices. Para obtener más información, consulte la documentación de Atlas Search [Scoring](https://docs.atlas.mongodb.com/reference/atlas-search/scoring/) .

