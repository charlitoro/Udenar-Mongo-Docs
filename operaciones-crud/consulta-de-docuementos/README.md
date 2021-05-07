# Consulta de Documentos

En anteriores secciones se ha utilizado el método **db.collection.find\(\)** para la consulta de algunos documentos insertados. Los ejemplos de esta pagina utilizan la colección **inventory.** Para llenar la colección **inventory,** ejecuta el siguiente comando:

```javascript
db.inventory.insertMany([
    { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
    { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
    { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
    { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },        
    { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```

### Todos los Documentos de una Colección

Para seleccionar todos los documentos de una colección, se pasa un documento vació como parámetro de filtro en el método. El parámetro de filtro determina el criterio de selección. 

```javascript
db.inventory.find( {} )

{ "_id" : ObjectId("5edc172afb7383970816b0f2"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
{ "_id" : ObjectId("5edc172afb7383970816b0f3"), "item" : "notebook", "qty" : 50, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "A" }
{ "_id" : ObjectId("5edc172afb7383970816b0f4"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
{ "_id" : ObjectId("5edc172afb7383970816b0f5"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "D" }
{ "_id" : ObjectId("5edc172afb7383970816b0f6"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "cm" }, "status" : "A" }
```

Esta operación corresponde a la siguiente sentencia SQL de un gestor de bases de datos relacional:

```sql
SELECT * FROM inventory
```

### Condiciones de Igualdad

Para especificar condiciones de igualdad, usar la expresión **&lt;campo&gt;:&lt;valor&gt;** en el parámetro filtro de la consulta:

```sql
{ <campo>: <valor>, ... }
```

El siguiente ejemplo selecciona desde la colección **inventory** todos los documentos donde el campo **status** sea igual a **"D":**

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { status: "D" } )

{ "_id" : ObjectId("5edc172afb7383970816b0f4"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
{ "_id" : ObjectId("5edc172afb7383970816b0f5"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "D" }

```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT * FROM inventory WHERE status = "D"
```
{% endtab %}
{% endtabs %}

### Condiciones Especificas Usando Operaciones

El parámetro filtro de una consulta puede usar **operadores** para condiciones especificas de la siguiente forma: 

```javascript
{ <campo1>: { <operador1>: <valor1> }, ... }
```

El siguiente ejemplo retorna todos los documentos desde la colección **inventory** donde el campo **status** sea igual a **"A"** o "**D**":

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT * FROM inventory WHERE status in ("A", "D")
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Si bien se puede utilizar el operador **$or**, use el operador **$in** en lugar del operador **$or** cuando se realicen operaciones de igualdad en el mismo campo.  
{% endhint %}

### Condición AND

Una consulta compuesta puede especificar condiciones para más de un campo en los documentos de la colección. Una conjunción lógica **AND** conecta las clausulas de una consulta compuesta para que la consulta seleccione los documentos de la colección que coincidan con todas las condiciones.

El siguiente ejemplo retorna todos los documentos donde el **status** sea igual "**A**" y **qty** sea menor que \(**$lt**\) 30:

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT * FROM inventory WHERE status = "A" AND qty < 30
```
{% endtab %}
{% endtabs %}

Mira más [operaciones de comparación](https://docs.mongodb.com/manual/reference/operator/query-comparison/#query-selectors-comparison).

### Condición OR

Usando el operador **$or**, se puede especificar una consulta compuesta que una cada clausula con un **OR** lógico para que esta consulta seleccione los documentos que coincidan con al menos una de las condiciones:

El siguiente ejemplo retorna todos los documentos donde el **status** es igual a "**A**" o **qty** es menor que \(**$lt**\) 30:  ****

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT * FROM inventory WHERE status = "A" OR qty < 30
```
{% endtab %}
{% endtabs %}

### Condiciones AND y OR

En el siguiente ejemplo, la consulta compuesta retorna todos los documentos donde el **status** sea igual a "**A**" y  **qty** sea menor que 30 o **item** inicie con el carácter **p:**

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Mongo soporta las consultas con expresiones regulare**s $regex** para realizar coincidencias de patron de cadena.  
{% endhint %}

