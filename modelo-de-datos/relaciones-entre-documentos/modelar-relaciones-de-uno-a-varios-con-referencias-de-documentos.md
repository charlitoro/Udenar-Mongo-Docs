# Modelar relaciones de uno a varios con referencias de documentos

### Descripción general  <a id="overview"></a>

Esta página describe un modelo de datos que utiliza [referencias](https://docs.mongodb.com/manual/core/data-model-design/#std-label-data-modeling-referencing) entre documentos para describir relaciones de uno a varios entre datos conectados.

### Patrón  <a id="pattern"></a>

Considere el siguiente ejemplo que mapea las relaciones entre editor y libro. El ejemplo ilustra la ventaja de hacer referencia a la incrustación para evitar la repetición de la información del editor.

Incrustar el documento del editor dentro del documento del libro conduciría a la **repetición** de los datos del editor, como muestran los siguientes documentos:

Para evitar la repetición de los datos del editor, utilice _referencias_ y mantenga la información del editor en una colección separada de la colección de libros.

```text
{
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",
   publisher: {
              name: "O'Reilly Media",
              founded: 1980,
              location: "CA"
            }
}

{
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",
   publisher: {
              name: "O'Reilly Media",
              founded: 1980,
              location: "CA"
            }
}
```

Cuando se utilizan referencias, el crecimiento de las relaciones determina dónde almacenar la referencia. Si el número de libros por editorial es pequeño con un crecimiento limitado, a veces puede resultar útil almacenar la referencia del libro dentro del documento de la editorial. De lo contrario, si el número de libros por editor es ilimitado, este modelo de datos conduciría a matrices crecientes y mutables, como en el siguiente ejemplo:

```text
{
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA",
   books: [123456789, 234567890, ...]
}

{
    _id: 123456789,
    title: "MongoDB: The Definitive Guide",
    author: [ "Kristina Chodorow", "Mike Dirolf" ],
    published_date: ISODate("2010-09-24"),
    pages: 216,
    language: "English"
}

{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English"
}
```

Para evitar matrices crecientes y cambiantes, almacene la referencia del editor dentro del documento del libro:

```text
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",
   publisher_id: "oreilly"
}

{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",
   publisher_id: "oreilly"
}
```

