# Especificar un idioma para el índice de texto



Este tutorial describe cómo [especificar el idioma predeterminado asociado con el índice de texto](https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/#std-label-specify-default-language-text-index) y también cómo [crear índices de texto para colecciones que contienen documentos en diferentes idiomas](https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/#std-label-select-from-multiple-languages-for-text-index) .

### Especificar el idioma predeterminado para un `text`índice  <a id="specify-the-default-language-for-a-text-index"></a>

El idioma predeterminado asociado con los datos indexados determina las reglas para analizar las raíces de las palabras \(es decir, la derivación\) e ignorar las palabras vacías. El idioma predeterminado para los datos indexados es `english`.

Para especificar un idioma diferente, use la `default_language`opción al crear el `text`índice. Consulte [Idiomas de búsqueda de texto](https://docs.mongodb.com/manual/reference/text-search-languages/#std-label-text-search-languages) para conocer los idiomas disponibles `default_language`.

El siguiente ejemplo crea para la `quotes`colección un `text` índice en el `content`campo y establece el `default_language`en `spanish`:

```text
db.quotes.createIndex(   { content : "text" },   { default_language: "spanish" })
```

### Crear un `text`índice para una colección en varios idiomas  <a id="create-a-text-index-for-a-collection-in-multiple-languages"></a>

#### Especificar el idioma del índice dentro del documento  <a id="specify-the-index-language-within-the-document"></a>

Si una colección contiene documentos o documentos incrustados que están en diferentes idiomas, incluya un campo nombrado `language`en los documentos o documentos incrustados y especifique como su valor el idioma para ese documento o documento incrustado.

MongoDB utilizará el idioma especificado para ese documento o documento incrustado al crear el `text`índice:

* El idioma especificado en el documento anula el idioma predeterminado del `text`índice.
* El idioma especificado en un documento incrustado anula el idioma especificado en un documento adjunto o el idioma predeterminado para el índice.

Consulte [Idiomas de búsqueda de texto](https://docs.mongodb.com/manual/reference/text-search-languages/#std-label-text-search-languages) para obtener una lista de los idiomas admitidos.

Por ejemplo, una colección `quotes`contiene documentos en varios idiomas que incluyen el `language`campo en el documento y / o el documento incrustado según sea necesario:

```text
{   _id: 1,   language: "portuguese",   original: "A sorte protege os audazes.",   translation:     [        {           language: "english",           quote: "Fortune favors the bold."        },        {           language: "spanish",           quote: "La suerte protege a los audaces."        }    ]}{   _id: 2,   language: "spanish",   original: "Nada hay más surrealista que la realidad.",   translation:      [        {          language: "english",          quote: "There is nothing more surreal than reality."        },        {          language: "french",          quote: "Il n'y a rien de plus surréaliste que la réalité."        }      ]}{   _id: 3,   original: "is this a dagger which I see before me.",   translation:   {      language: "spanish",      quote: "Es este un puñal que veo delante de mí."   }}
```

Si crea un `text`índice en el `quote`campo con el idioma predeterminado de inglés.

```text
db.quotes.createIndex( { original: "text", "translation.quote": "text" } )
```

Luego, para los documentos y documentos incrustados que contienen el `language` campo, el `text`índice usa ese idioma para analizar las raíces de las palabras y otras características lingüísticas.

Para documentos incrustados que no contienen el `language`campo,

* Si el documento adjunto contiene el `language`campo, el índice utiliza el idioma del documento para el documento incrustado.
* De lo contrario, el índice utiliza el idioma predeterminado para los documentos incrustados.

Para los documentos que no contienen el `language`campo, el índice utiliza el idioma predeterminado, que es el inglés.

#### Utilice cualquier campo para especificar el idioma de un documento  <a id="use-any-field-to-specify-the-language-for-a-document"></a>

Para utilizar un campo con un nombre distinto de `language`, incluya la `language_override`opción al crear el índice.

Por ejemplo, proporcione el siguiente comando para usarlo `idioma`como nombre de campo en lugar de `language`:

```text
db.quotes.createIndex( { quote : "text" },                       { language_override: "idioma" } )
```

Los documentos de la `quotes`colección pueden especificar un idioma con el `idioma`campo:

```text
{ _id: 1, idioma: "portuguese", quote: "A sorte protege os audazes" }{ _id: 2, idioma: "spanish", quote: "Nada hay más surrealista que la realidad." }{ _id: 3, idioma: "english", quote: "is this a dagger which I see before me" }
```

