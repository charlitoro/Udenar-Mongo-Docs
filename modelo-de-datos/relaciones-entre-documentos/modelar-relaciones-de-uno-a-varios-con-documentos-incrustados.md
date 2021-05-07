# Modelar relaciones de uno a varios con documentos incrustados

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que utiliza documentos [incrustados](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) para describir una relación de uno a varios entre los datos conectados. Incrustar datos conectados en un solo documento puede reducir el número de operaciones de lectura necesarias para obtener datos. En general, debe estructurar su esquema para que su aplicación reciba toda la información requerida en una sola operación de lectura.

### Patrón de documento incrustado  <a id="embedded-document-pattern"></a>

Considere el siguiente ejemplo que mapea las relaciones entre usuarios y direcciones múltiples. El ejemplo ilustra la ventaja de incrustar sobre las referencias si necesita ver muchas entidades de datos en el contexto de otra. En esta relación de uno a muchos entre `patron`y `address`datos, el `patron`tiene varias `address`entidades.

En el modelo de datos normalizado, los `address`documentos contienen una referencia al `patron`documento.

```text
// patron document
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address documents
{
   patron_id: "joe", // reference to patron document
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}

{
   patron_id: "joe",
   street: "1 Some Other Street",
   city: "Boston",
   state: "MA",
   zip: "12345"
}
```

Si su aplicación recupera con frecuencia los `address`datos con la `name`información, entonces su aplicación necesita emitir múltiples consultas para resolver las referencias. Un esquema más óptimo sería incrustar las `address`entidades de datos en los `patron`datos, como en el siguiente documento:

```text
{
   "_id": "joe",
   "name": "Joe Bookreader",
   "addresses": [
                {
                  "street": "123 Fake Street",
                  "city": "Faketon",
                  "state": "MA",
                  "zip": "12345"
                },
                {
                  "street": "1 Some Other Street",
                  "city": "Boston",
                  "state": "MA",
                  "zip": "12345"
                }
              ]
 }
```

Con el modelo de datos integrado, su aplicación puede recuperar la información completa del usuario con una sola consulta.

### Patrón de subconjunto  <a id="subset-pattern"></a>

Un problema potencial con el [patrón de documento incrustado](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#std-label-one-to-many-embedded-document-pattern) es que puede dar lugar a documentos grandes, especialmente si el campo incrustado no está acotado. En este caso, puede usar el patrón de subconjunto para acceder solo a los datos que requiere la aplicación, en lugar del conjunto completo de datos incrustados.

Considere un sitio de comercio electrónico que tenga una lista de reseñas de un producto:

```text
{
  "_id": 1,
  "name": "Super Widget",
  "description": "This is the most useful item in your toolbox.",
  "price": { "value": NumberDecimal("119.99"), "currency": "USD" },
  "reviews": [
    {
      "review_id": 786,
      "review_author": "Kristina",
      "review_text": "This is indeed an amazing widget.",
      "published_date": ISODate("2019-02-18")
    },
    {
      "review_id": 785,
      "review_author": "Trina",
      "review_text": "Nice product. Slow shipping.",
      "published_date": ISODate("2019-02-17")
    },
    ...
    {
      "review_id": 1,
      "review_author": "Hans",
      "review_text": "Meh, it's okay.",
      "published_date": ISODate("2017-12-06")
    }
  ]
}
```

Las reseñas están ordenadas en orden cronológico inverso. Cuando un usuario visita la página de un producto, la aplicación carga las diez reseñas más recientes.

En lugar de almacenar todas las reseñas con el producto, puede dividir la colección en dos colecciones:

* La `product`colección almacena información sobre cada producto, incluidas las diez revisiones más recientes del producto:

```text
{
  "_id": 1,
  "name": "Super Widget",
  "description": "This is the most useful item in your toolbox.",
  "price": { "value": NumberDecimal("119.99"), "currency": "USD" },
  "reviews": [
    {
      "review_id": 786,
      "review_author": "Kristina",
      "review_text": "This is indeed an amazing widget.",
      "published_date": ISODate("2019-02-18")
    }
    ...
    {
      "review_id": 776,
      "review_author": "Pablo",
      "review_text": "Amazing!",
      "published_date": ISODate("2019-02-16")
    }
  ]
}
```

* La `review`colección almacena todas las reseñas. Cada reseña contiene una referencia al producto para el que fue escrita.

```text
{
  "review_id": 786,
  "product_id": 1,
  "review_author": "Kristina",
  "review_text": "This is indeed an amazing widget.",
  "published_date": ISODate("2019-02-18")
}
{
  "review_id": 785,
  "product_id": 1,
  "review_author": "Trina",
  "review_text": "Nice product. Slow shipping.",
  "published_date": ISODate("2019-02-17")
}
...
{
  "review_id": 1,
  "product_id": 1,
  "review_author": "Hans",
  "review_text": "Meh, it's okay.",
  "published_date": ISODate("2017-12-06")
}
```

Al almacenar las diez revisiones más recientes en la `product` colección, solo el subconjunto requerido de los datos generales se devuelve en la llamada a la `product`colección. Si un usuario desea ver reseñas adicionales, la aplicación realiza una llamada a la `review` colección.

**CONSEJO**

Al considerar dónde dividir sus datos, la parte de los datos a la que se accede con mayor frecuencia debe ir a la colección que la aplicación carga primero. En este ejemplo, el esquema se divide en diez revisiones porque ese es el número de revisiones visibles en la aplicación de forma predeterminada.

**CONSEJO**

Para aprender a usar el patrón de subconjunto para modelar relaciones uno a uno entre colecciones, consulte [Modelar relaciones uno a uno con documentos incrustados](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/#std-label-data-modeling-example-one-to-one) .

#### Compensaciones del patrón de subconjunto  <a id="trade-offs-of-the-subset-pattern"></a>

El uso de documentos más pequeños que contienen datos a los que se accede con más frecuencia reduce el tamaño general del conjunto de trabajo. Estos documentos más pequeños dan como resultado un rendimiento de lectura mejorado para los datos a los que la aplicación accede con más frecuencia.

Sin embargo, el patrón de subconjunto da como resultado la duplicación de datos. En el ejemplo, las reseñas se mantienen tanto en la `product`colección como en la `reviews`colección. Deben tomarse medidas adicionales para garantizar que las revisiones sean coherentes entre cada recopilación. Por ejemplo, cuando un cliente edita su reseña, es posible que la aplicación deba realizar dos operaciones de escritura: una para actualizar la `product`colección y otra para actualizar la `reviews`colección.

También debe implementar la lógica en su aplicación para asegurarse de que las reseñas de la `product`colección sean siempre las diez reseñas más recientes de ese producto.

#### Otros casos de uso de muestra  <a id="other-sample-use-cases"></a>

Además de las reseñas de productos, el patrón de subconjunto también puede ser una buena opción para almacenar:

* Comentarios en una publicación de blog, cuando solo desea mostrar los comentarios más recientes o mejor calificados de forma predeterminada.
* Miembros del elenco en una película, cuando solo quieres mostrar a los miembros del elenco con los roles más grandes de forma predeterminada.

