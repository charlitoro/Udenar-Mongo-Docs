# Búsqueda de Texto

**Descripción general**

MongoDB admite operaciones de consulta que realizan una búsqueda de texto de contenido de cadena. Para realizar la búsqueda de texto , MongoDB usa un índice de texto \([text index](https://docs.mongodb.com/manual/core/index-text/#std-label-index-feature-text)\) y el operador  [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)

**Nota :** Las vistas no soportan la búsqueda de texto.

**Ejemplo :**

Este ejemplo demuestra cómo crear un índice de texto y usarlo para buscar cafeterías, solo con campos de texto.

Crea una colección de tiendas con los siguientes documentos:

```text
db.stores.insert(
   [
     { _id: 1, name: "Java Hut", description: "Coffee and cakes" },
     { _id: 2, name: "Burger Buns", description: "Gourmet hamburgers" },
     { _id: 3, name: "Coffee Shop", description: "Just coffee" },
     { _id: 4, name: "Clothes Clothes Clothes", description: "Discount clothing" },
     { _id: 5, name: "Java Shopping", description: "Indonesian goods" }
   ]
)
```

#### **Text Index** <a id="text-index"></a>

MongoDB proporciona índices de texto \( [text indexes](https://docs.mongodb.com/manual/core/index-text/#std-label-index-feature-text)\) para admitir consultas de búsqueda de texto sobre contenido de cadenas. Los`text index`pueden incluir cualquier campo cuyo valor sea una cadena o una matriz de elementos de cadena.

Para realizar consultas de búsqueda de texto, debe tener un `text index` en su colección. Una colección solo puede tener **un** índice de búsqueda de texto, pero ese índice puede cubrir varios campos.

Por ejemplo, puede ejecutar lo siguiente en un [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell para permitir la búsqueda de texto en los campos `name`y `description`:

```text
db.stores.createIndex( { name: "text", description: "text" } )
```

#### `$text`Operador  <a id="-text-operator"></a>

Utilice el [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)operador de consulta para realizar búsquedas de texto en una colección con un [índice de texto](https://docs.mongodb.com/manual/core/index-text/#std-label-index-feature-text) .

[`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)tokenizará la cadena de búsqueda usando espacios en blanco y la mayoría de los signos de puntuación como delimitadores, y realizará una lógica `OR`de todos esos tokens en la cadena de búsqueda.

Por ejemplo, puede utilizar la siguiente consulta para buscar todas las tiendas que contengan los términos de la lista "café", "tienda" y "java":

```text
db.stores.find( { $text: { $search: "java coffee shop" } } )
```

**Frase exacta** 

También puede buscar frases exactas envolviéndolas entre comillas dobles. Si la `$search`cadena incluye una frase y términos individuales, la búsqueda de texto solo coincidirá con los documentos que incluyan la frase.

Por ejemplo, a continuación se encontrarán todos los documentos que contengan "cafetería":

```text
db.stores.find( { $text: { $search: "\"coffee shop\"" } } )
```

Para obtener más información, consulte [Frases](https://docs.mongodb.com/manual/reference/operator/query/text/#std-label-text-operator-phrases) .

  
**Exclusión de términos** 

Para excluir una palabra, puede anteponer un carácter " `-`". Por ejemplo, para buscar todas las tiendas que contengan "java" o "tienda" pero no "café", utilice lo siguiente:

```text
db.stores.find( { $text: { $search: "java shop -coffee" } } )
```

**Ordenar** 

MongoDB devolverá sus resultados en orden sin clasificar de forma predeterminada. Sin embargo, las consultas de búsqueda de texto calcularán una puntuación de relevancia para cada documento que especifica qué tan bien un documento coincide con la consulta.

Para ordenar los resultados en orden de puntuación de relevancia, debe proyectar explícitamente el campo y ordenarlo:[`$meta`](https://docs.mongodb.com/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta) `textScore`

```text
db.stores.find(
   { $text: { $search: "java coffee shop" } },
   { score: { $meta: "textScore" } }
).sort( { score: { $meta: "textScore" } } )
```

La búsqueda de texto también está disponible en la canalización de agregación.

### Soporte de idiomas  <a id="language-support"></a>

MongoDB admite la búsqueda de texto en varios idiomas. Consulte [Idiomas de búsqueda de texto](https://docs.mongodb.com/manual/reference/text-search-languages/) para obtener una lista de los idiomas admitidos.

Para los datos alojados en MongoDB Atlas, [Atlas Search](https://docs.atlas.mongodb.com/atlas-search/) proporciona soporte para idiomas adicionales. Para ver la lista completa de idiomas admitidos por Atlas Search, consulte los [Analizadores de idiomas de Atlas Search](https://docs.atlas.mongodb.com/reference/atlas-search/analyzers/language/) .

