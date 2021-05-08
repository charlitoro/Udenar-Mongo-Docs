# Limite el número de entradas escaneadas

Este tutorial describe cómo crear índices para limitar el número de entradas de índice escaneadas en busca de consultas que incluyan una [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text) expresión y condiciones de igualdad.

Una colección `inventory`contiene los siguientes documentos:

```text
{ _id: 1, dept: "tech", description: "lime green computer" }{ _id: 2, dept: "tech", description: "wireless red mouse" }{ _id: 3, dept: "kitchen", description: "green placemat" }{ _id: 4, dept: "kitchen", description: "red peeler" }{ _id: 5, dept: "food", description: "green apple" }{ _id: 6, dept: "food", description: "red potato" }
```

Considere el caso de uso común que realiza búsquedas de texto por departamentos _individuales_ , como:

```text
db.inventory.find( { dept: "kitchen", $text: { $search: "green" } } )
```

Para limitar la búsqueda de texto para escanear solo aquellos documentos dentro de un específico `dept`, cree un índice compuesto que _primero_ especifique una clave de índice ascendente / descendente en el campo `dept`y luego una `text`clave de índice en el campo `description`:

```text
db.inventory.createIndex(   {     dept: 1,     description: "text"   })
```

Luego, la búsqueda de texto dentro de un departamento en particular limitará el escaneo de documentos indexados. Por ejemplo, la siguiente consulta escanea solo aquellos documentos con `dept`igual a `kitchen`:

```text
db.inventory.find( { dept: "kitchen", $text: { $search: "green" } } )
```

NOTA

* Un `text`índice compuesto no puede incluir ningún otro tipo de índice especial, como campos de índice [geoespacial](https://docs.mongodb.com/manual/geospatial-queries/#std-label-index-feature-geospatial) o de [claves múltiples](https://docs.mongodb.com/manual/core/index-multikey/#std-label-index-type-multi-key) .
* Si el `text`índice compuesto incluye claves que **preceden a** la `text`clave de índice, para realizar una [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#mongodb-query-op.-text)búsqueda, el predicado de consulta debe incluir **condiciones de coincidencia de igualdad** en las claves anteriores.
* Al crear un `text`índice compuesto , todas las `text`claves de índice deben enumerarse de forma adyacente en el documento de especificación del índice.

CONSEJOVer también:

[Índices de texto](https://docs.mongodb.com/manual/core/index-text/)

