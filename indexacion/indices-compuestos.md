# Índices compuestos

MongoDB admite _índices compuestos_ , donde una única estructura de índice contiene referencias a múltiples campos [\[ 1 \]](https://docs.mongodb.com/manual/core/index-compound/#footnote-compound-index-field-limit) dentro de los documentos de una colección. El siguiente diagrama ilustra un ejemplo de un índice compuesto en dos campos:

![](../.gitbook/assets/image%20%2813%29.png)

| \[ [1](https://docs.mongodb.com/manual/core/index-compound/#ref-compound-index-field-limit-id1) \] | MongoDB impone un [`limit of 32 fields for any compound index`](https://docs.mongodb.com/manual/reference/limits/#mongodb-limit-Number-of-Indexed-Fields-in-a-Compound-Index). |
| :--- | :--- |


Los índices compuestos pueden admitir consultas que coincidan en varios campos.

### Crear un índice compuesto  <a id="create-a-compound-index"></a>

Para crear un [índice compuesto,](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) utilice una operación que se parezca al siguiente prototipo:

```text
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )
```

El valor del campo en la especificación del índice describe el tipo de índice para ese campo. Por ejemplo, un valor de `1`especifica un índice que ordena los elementos en orden ascendente. Un valor de `-1`especifica un índice que ordena los elementos en orden descendente. Para tipos de índices adicionales, consulte [tipos de índices](https://docs.mongodb.com/manual/indexes/#std-label-index-types) .

**IMPORTANTE:**

_A partir de MongoDB 4.4:_

* _Los índices compuestos pueden contener **un solo**_ [_campo de índice hash_](https://docs.mongodb.com/manual/core/index-hashed/) _._
* _Recibirá un error si intenta crear un índice compuesto que contenga más de un_ [_campo de índice hash_](https://docs.mongodb.com/manual/core/index-hashed/) _._

_En MongoDB 4.2 o anterior:_

* _Los índices compuestos **no** pueden contener un_ [_campo de índice con hash_](https://docs.mongodb.com/manual/core/index-hashed/) _._
* _Recibirá un error si intenta crear un índice compuesto que contenga un_ [_campo de índice hash_](https://docs.mongodb.com/manual/core/index-hashed/) _._

Considere una colección nombrada `products`que contiene documentos que se parecen al siguiente documento:

```text
{
 "_id": ObjectId(...),
 "item": "Banana",
 "category": ["food", "produce", "grocery"],
 "location": "4th Street Store",
 "stock": 4,
 "type": "cases"
}
```

La siguiente operación crea un índice ascendente en los campos `item`y `stock`:

```text
db.products.createIndex( { "item": 1, "stock": 1 } )
```

El orden de los campos enumerados en un índice compuesto es importante. El índice contendrá referencias a documentos ordenados primero por los valores del `item`campo y, dentro de cada valor del `item`campo, ordenados por valores del campo de valores. Consulte [Orden de clasificación](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-ascending-and-descending) para obtener más información.

Además de admitir consultas que coinciden en todos los campos de índice, los índices compuestos pueden admitir consultas que coinciden en el prefijo de los campos de índice. Es decir, los soportes del índice de consultas sobre el `item`terreno, así como tanto `item`y `stock`campos:

```text
db.products.find( { item: "Banana" } )
db.products.find( { item: "Banana", stock: { $gt: 5 } } )
```

Para obtener más información, consulte [Prefijos](https://docs.mongodb.com/manual/core/index-compound/#std-label-compound-index-prefix) .

### Orden de clasificación  <a id="sort-order"></a>

Los índices almacenan referencias a campos en orden ascendente \( `1`\) o descendente \( `-1`\). Para los índices de un solo campo, el orden de clasificación de las claves no importa porque MongoDB puede atravesar el índice en cualquier dirección. Sin embargo, para los [índices compuestos](https://docs.mongodb.com/manual/core/index-compound/#std-label-index-type-compound) , el orden de clasificación puede ser importante para determinar si el índice puede admitir una operación de clasificación.

Considere una colección `events`que contiene documentos con los campos `username`y `date`. Las aplicaciones pueden emitir consultas que devuelven resultados ordenados primero por `username`valores ascendentes y luego por valores descendentes \(es decir, del más reciente al último\) `date`, como por ejemplo:

```text
db.events.find().sort( { username: 1, date: -1 } )
```

o consultas que devuelven resultados ordenados primero por `username` valores descendentes y luego por `date`valores ascendentes , como:

```text
db.events.find().sort( { username: -1, date: 1 } )
```

El siguiente índice puede admitir estas dos operaciones de ordenación:

```text
db.events.createIndex( { "username" : 1, "date" : -1 } )
```

Sin embargo, el índice anterior **no puede** admitir la ordenación por `username`valores ascendentes y luego por `date`valores ascendentes , como los siguientes:

```text
db.events.find().sort( { username: 1, date: 1 } )
```

Para obtener más información sobre el orden de clasificación y los índices compuestos, consulte [Usar índices para ordenar los resultados de la consulta](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/) .

### Prefijos  <a id="prefixes"></a>

Los prefijos de índice son los subconjuntos _iniciales_ de campos indexados. Por ejemplo, considere el siguiente índice compuesto:

```text
{ "item": 1, "location": 1, "stock": 1 }
```

El índice tiene los siguientes prefijos de índice:

* `{ item: 1 }`
* `{ item: 1, location: 1 }`

Para un índice compuesto, MongoDB puede usar el índice para admitir consultas sobre los prefijos del índice. Como tal, MongoDB puede usar el índice para consultas en los siguientes campos:

* el `item`campo,
* el `item`campo _y_ el `location`campo,
* el `item`campo _y_ el `location`campo _y_ el `stock`campo.

MongoDB también puede usar el índice para admitir una consulta en los campos `item`y `stock`, ya que el `item`campo corresponde a un prefijo. Sin embargo, en este caso, el índice no sería tan eficiente para respaldar la consulta como lo sería si el índice estuviera solo en `item`y `stock`. Los campos de índice se analizan en orden; si una consulta omite un prefijo de índice en particular, no puede hacer uso de ningún campo de índice que siga a ese prefijo.

Dado que una consulta sobre `item`y `stock`omite el `location`prefijo de índice, no puede utilizar el `stock`campo de índice que sigue `location`. Solo el `item`campo del índice puede admitir esta consulta. Consulte [Crear índices para respaldar sus consultas](https://docs.mongodb.com/manual/tutorial/create-indexes-to-support-queries/#std-label-create-indexes-to-support-queries) para obtener más información.

MongoDB no puede usar el índice para admitir consultas que incluyan los siguientes campos, ya que sin el `item`campo, ninguno de los campos enumerados corresponde a un índice de prefijo:

* el `location`campo,
* el `stock`campo, o
* los campos `location`y `stock`.

Si tiene una colección que tiene tanto un índice compuesto como un índice en su prefijo \(por ejemplo, `{ a: 1, b: 1 }`y `{ a: 1 }`\), si ninguno de los índices tiene una restricción escasa o única, entonces puede eliminar el índice del prefijo \(por ejemplo `{ a: 1 }`\). MongoDB usará el índice compuesto en todas las situaciones en las que habría usado el índice de prefijo.

### Intersección de índice  <a id="index-intersection"></a>

A partir de la versión 2.6, MongoDB puede usar la [intersección de índices](https://docs.mongodb.com/manual/core/index-intersection/) para completar consultas. La elección entre crear índices compuestos que respalden sus consultas o confiar en la intersección de índices depende de las características específicas de su sistema. Consulte [Intersección de índice e índices compuestos](https://docs.mongodb.com/manual/core/index-intersection/#std-label-index-intersection-compound-indexes) para obtener más detalles.

### Consideraciones adicionales  <a id="additional-considerations"></a>

Las aplicaciones pueden encontrar un rendimiento reducido durante la creación de índices, incluido el acceso limitado de lectura / escritura a la colección. Para obtener más información sobre el proceso de generación de índices, consulte [Generaciones de índices en colecciones pobladas](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations) , incluida la sección [Generaciones de índices en entornos replicados](https://docs.mongodb.com/manual/core/index-creation/#std-label-index-operations-replicated-build) .

Algunos controladores pueden especificar índices, utilizando en `NumberLong(1)`lugar de `1`como especificación. Esto no tiene ningún efecto sobre el índice resultante.

