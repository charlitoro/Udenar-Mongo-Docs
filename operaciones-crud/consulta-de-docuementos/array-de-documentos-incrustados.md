# Array de Documentos Incrustados

Los ejemplos de esta pagina utilizan la colección **inventory.** Para llenar la colección **inventory,** ejecuta el siguiente comando:

```javascript
db.inventory.insertMany( [
   { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
   { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
   { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
   { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```

## Consulta de un Documento Anidado en un Array

El siguiente ejemplo busca todos los documentos donde un elemento del array **instock** coincida con un documento:

```javascript
db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )
```

Para una consulta de igualdad en todo el documento insertado o anidado requiere un documento que coincidan exactamente con el documento buscado, incluso en el orden de los campos. La siguiente consulta no coincide con algún documento en la colección **inventory:**  

```javascript
db.inventory.find( { "instock": { qty: 5, warehouse: "A" } } )
```

## Condición Especifica Sobre el Campo de un Array de Documentos

### Condición sobre un campo insertado en un array de documentos

Si no se conoce la posición del documento insertado en el array, se concatena el nombre del campo array, con un punto \(.\) y el nombre del campo del documento insertado en el array.

El siguiente ejemplo busca todos los documentos donde el array **instock** tiene al menos un documento que coincida con en campo **qty,** donde su valor sea menor o igual \(**$lte**\) que **20:** 

```javascript
db.inventory.find( { 'instock.qty': { $lte: 20 } } )
```

### **Usando la posición en el array para consultar un campo de un documento insertado**

Utilizar la **notación punto,** es posible especificar la condición de consulta para el campo el del documento en una posición especifica del array.

{% hint style="success" %}
Cuando una consulta usa la **notación punto**, el campo y la posición deben estar entre comillas.
{% endhint %}

El siguiente ejemplo busca todos los documentos donde el array **instock** tiene en su primer elemento un documento que contenga el campo **qtr**, cuyo valor sea menor o igual \(**$lte**\) que **20**:

```javascript
db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )
```

## Condiciones Multiples para un Array de Documentos

Cuando tienes condiciones en más de un campo anidado dentro de un array de documentos, puede especificar la consulta de modo que un solo documento cumpla con al condición o cualquier combinación de documentos en el array cumpla con las condiciones.

### Documento que cumple multiples condiciones en campos anidados

Use el operador **$elemMatch** para especificar multiples criterios de búsqueda en un array de documentos de modo que al menos uno de los documentos cumpla con todas los criterios de búsqueda. 

El siguiente ejemplo consulta por documentos donde el array **instock** tiene al menos un documento que contenga el campo **qty** igual a **5** y **warehouse** igual a **A:**

```javascript
db.inventory.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } )
```

El siguiente ejemplo, busca los documentos donde el array **instock** tiene al menos un documento insertado que contenga el campo **qty** mayor que **10** y menor o igual que **20:**

```javascript
db.inventory.find( { "instock": { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } } ) 
```

### **Combinación de elementos que cumplen el criterio**

Sí la condición de la consulta compuesta en un array no usa el operado **$elemMatch**, la consulta selecciona aquellos documentos cuyo array contiene combinación de elementos que cumplan las condiciones.

La siguiente consulta busca los documentos donde cualquiera documento del array **instock** tenga el campo **qty** mayor que **10** y cualquier documento \(pero no necesariamente el mismo documento\) en el array que tenga el campo **qty** menor o igual que **20:**

```javascript
db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } )
```

El siguiente ejemplo consulta los documentos donde el array **instock** tiene al menos un documento que contiene el campo **qty** igual a **5** y almeno un documento \(pero no necesariamente el mismo documento\) que contenga el campo **warehouse** sea igual a **A:**

```javascript
db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } )
```

  ****   

### 

