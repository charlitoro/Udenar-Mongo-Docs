# Arrays

Los ejemplos de esta pagina utilizan la colección **inventory.** Para llenar la colección **inventory,** ejecuta el siguiente comando:

```javascript
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
   { item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
   { item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
   { item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
   { item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }
]);
```

### Consultar por Campo Array

Para condiciones especificas de igualdad en un array, use el documento **{ &lt;campo&gt;: &lt;valor&gt; }** donde **&lt;valor&gt;** es un array exacto, incluido el orden de los elementos del array 

El siguiente ejemplo consulta todos los documentos donde el valor del campo **tags** es un array con exactamente dos elementos, "**red**" y "**black**", en ese especifico orden:

```javascript
db.inventory.find( { tags: ["red", "blank"] } )
```

Sí, en lugar, se quiere buscar un array que contenga ambos elementos, "**red**" y "**black**", sin tener en cuenta el orden de los elementos, se utiliza el operador **$all:** 

```javascript
db.inventory.find( { tags: { $all: ["red", "blank"] } } )
```

### Consultar Por el Elemento de un Array

En las consultad donde un campo array contiene al menos uno de los elementos con un especifico valor, usar el filtro **{ &lt;campo&gt;: &lt;valor&gt; }** donde el **&lt;valor&gt;** es el elemento valor buscado:

El siguiente ejemplo consulta todos los documentos donde el campo **tags** es un array que contiene la cadena "**red**" como uno de sus elementos:

```javascript
db.inventory.find( { tags: "red" } )
```

Para condiciones especificas donde buscamos por el valor de un elemento de un array, use operadores en el documento de filtrado de la consulta:

```javascript
{ <campo array>: { <operador1>: <valor1>, ... } }
```

La siguiente consulta retorna todos los documentos donde el array **dim\_cm** contenga al menos en uno de sus elementos un valor mayor que \(**$gt**\) 25:

```javascript
db.inventory.find( { dim_cm: { $gt: 25 } } )
```

### Multiples Condiciones para Elementos de un Array

Cuando especificamos multiples condiciones sobre elementos de una array, es posible realizar la consulta de modo que un solo elemento del array cumpla con esta condición o cualquiera de los elementos cumpla con la condición.

#### Consultar con filtro compuesto sobre elementos de un array

El siguiente ejemplo consulta por documentos donde el array **dim\_cm** contiene elementos que en alguna combinación satisface la condición; por ejemplo, un elemento pude satisface la condición mayor que \(**$gt**\) **15** y otro elemento puede satisfacer la condición menor que \(**$lt**\) **20**, o una elemento pude satisfacer ambas condiciones:

```javascript
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )
```

#### Consultar por un array de elementos que cumplan con varias criterios

Utilizar el operador **$elemMatch** para especificar multiple criterios sobre elementos de un array donde al menos one de los elementos cumpla todos los criterios especificados:

El siguiente ejemplo consulta por los documentos donde el array **dim\_cm** contenga al menos un elemento que sea mayor que\($**gt**\) **22** y menor que \(**$lt**\) **30**:

```javascript
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )
```

#### Consultar elemento por el indice de posición en el array

Utilizando la **notación punto,** es posible la condición de consulta por un elemento en una index o posición en el array. Los array inician por indice zero\(0\).

El siguiente ejemplo consulta todos los documentos donde el segundo elemento en el array **dim\_cm** es mayor que **25**:

```javascript
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )
```

#### Consultar por el tamaño de un array 

Utilice el operador **$size** para consultar por el numero de elementos de un array. El siguiente ejemplo consulta los documentos donde el array **tags** tenga 3 elementos:

```javascript
db.inventory.find( { "tags": { $size: 3 } } )
```

