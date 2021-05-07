# Modele relaciones uno a uno con documentos incrustados

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que utiliza documentos [incrustados](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-embedding) para describir una relación uno a uno entre los datos conectados. Incrustar datos conectados en un solo documento puede reducir el número de operaciones de lectura necesarias para obtener datos. En general, debe estructurar su esquema para que su aplicación reciba toda la información requerida en una sola operación de lectura.

### Patrón de documento incrustado  <a id="embedded-document-pattern"></a>

Considere el siguiente ejemplo que mapea las relaciones entre usuarios y direcciones. El ejemplo ilustra la ventaja de incrustar sobre las referencias si necesita ver una entidad de datos en el contexto de la otra. En esta relación uno a uno entre `patron`y `address`datos, el `address`pertenece a `patron`.

En el modelo de datos normalizados, el `address`documento contiene una referencia al `patron`documento.

```text
// patron document
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address document
{
   patron_id: "joe", // reference to patron document
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```

Si los `address`datos se recuperan con frecuencia con la `name` información, entonces con la referencia, su aplicación necesita emitir múltiples consultas para resolver la referencia. El mejor modelo de datos sería incrustar los `address`datos en los `patron`datos, como en el siguiente documento:

```text
{
   _id: "joe",
   name: "Joe Bookreader",
   address: {
              street: "123 Fake Street",
              city: "Faketon",
              state: "MA",
              zip: "12345"
            }
}
```

Con el modelo de datos integrado, su aplicación puede recuperar la información completa del usuario con una sola consulta.

### Patrón de subconjunto  <a id="subset-pattern"></a>

Un problema potencial con el [patrón de documento incrustado](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#std-label-one-to-many-embedded-document-pattern) es que puede generar documentos grandes que contienen campos que la aplicación no necesita. Estos datos innecesarios pueden causar una carga adicional en su servidor y ralentizar las operaciones de lectura. En su lugar, puede utilizar el patrón de subconjunto para recuperar el subconjunto de datos al que se accede con más frecuencia en una única llamada a la base de datos.

Considere una aplicación que muestra información sobre películas. La base de datos contiene una `movie`colección con el siguiente esquema:

```text
{
  "_id": 1,
  "title": "The Arrival of a Train",
  "year": 1896,
  "runtime": 1,
  "released": ISODate("01-25-1896"),
  "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
  "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
  "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
  "lastupdated": ISODate("2015-08-15T10:06:53"),
  "type": "movie",
  "directors": [ "Auguste Lumière", "Louis Lumière" ],
  "imdb": {
    "rating": 7.3,
    "votes": 5043,
    "id": 12
  },
  "countries": [ "France" ],
  "genres": [ "Documentary", "Short" ],
  "tomatoes": {
    "viewer": {
      "rating": 3.7,
      "numReviews": 59
    },
    "lastUpdated": ISODate("2020-01-09T00:02:53")
  }
}
```

Actualmente, la `movie`colección contiene varios campos que la aplicación no necesita para mostrar una descripción general simple de una película, como `fullplot`información de clasificación. En lugar de almacenar todos los datos de la película en una sola colección, puede dividir la colección en dos colecciones:

* La `movie`colección contiene información básica sobre una película. Estos son los datos que carga la aplicación por defecto:

```text
// movie collection

{
  "_id": 1,
  "title": "The Arrival of a Train",
  "year": 1896,
  "runtime": 1,
  "released": ISODate("1896-01-25"),
  "type": "movie",
  "directors": [ "Auguste Lumière", "Louis Lumière" ],
  "countries": [ "France" ],
  "genres": [ "Documentary", "Short" ],
}
```

La `movie_details`colección contiene datos adicionales a los que se accede con menos frecuencia para cada película:

```text
// movie_details collection

{
  "_id": 156,
  "movie_id": 1, // reference to the movie collection
  "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
  "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
  "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
  "lastupdated": ISODate("2015-08-15T10:06:53"),
  "imdb": {
    "rating": 7.3,
    "votes": 5043,
    "id": 12
  },
  "tomatoes": {
    "viewer": {
      "rating": 3.7,
      "numReviews": 59
    },
    "lastUpdated": ISODate("2020-01-29T00:02:53")
  }
}
```

Este método mejora el rendimiento de lectura porque requiere que la aplicación lea menos datos para cumplir con su solicitud más común. La aplicación puede realizar una llamada adicional a la base de datos para recuperar los datos a los que se accede con menos frecuencia si es necesario.

**CONSEJO**

Al considerar dónde dividir sus datos, la parte de los datos a la que se accede con mayor frecuencia debe ir a la colección que la aplicación carga primero.

**CONSEJO**

Para aprender a usar el patrón de subconjunto para modelar relaciones de uno a varios entre colecciones, consulte [Modelar](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#std-label-data-modeling-example-one-to-many) relaciones de uno a varios [con documentos incrustados](https://docs.mongodb.com/manual/tutorial/model-embedded-one-to-many-relationships-between-documents/#std-label-data-modeling-example-one-to-many) .

#### Compensaciones del patrón de subconjunto  <a id="trade-offs-of-the-subset-pattern"></a>

El uso de documentos más pequeños que contienen datos a los que se accede con más frecuencia reduce el tamaño general del conjunto de trabajo. Estos documentos más pequeños dan como resultado un rendimiento de lectura mejorado y hacen que haya más memoria disponible para la aplicación.

Sin embargo, es importante comprender su aplicación y la forma en que carga los datos. Si divide sus datos en varias colecciones de manera incorrecta, su aplicación a menudo necesitará realizar varios viajes a la base de datos y depender de las `JOIN`operaciones para recuperar todos los datos que necesita.

Además, dividir sus datos en muchas colecciones pequeñas puede aumentar el mantenimiento requerido de la base de datos, ya que puede resultar difícil rastrear qué datos se almacenan en qué colección.



