# Documentos Incrustado/Anidados

Los ejemplos de esta pagina utilizan la colección **inventory.** Para llenar la colección **inventory,** ejecuta el siguiente comando:

```javascript
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```

### Documento Incrustado/Anidado

El schema de mongo permite anidar documentos dentro de otro. Cuanto se trabaja con la estructura **JSON** es claro mirar como un objeto puede tener varios niveles de profundidad:

```javascript
// Json ejemplo de objetos anidados
{
    "nivel-1": {
        "nivel-2": {
            "nivel-3": {
                "nivel-4": {
                    "campo-1": "contenido",
                    "campo-2": "contenido"
                }
            }
        }
    }
}
```

En la inserción de un documento se inserta en un campo otros documentos de una manera similar al ejemplo anterior. Esa es la forma como se incrustan documentos dentro de otros en mongo.

### Consultar un Documento Incrustado/Anidado

Para una especifica condición de igualdad de un campo que esta en un documento incrustado, use la estructura **{ &lt;campo&gt;: &lt;valor&gt; }** donde **&lt;valor&gt;** es el documento a encontrar.

La siguiente consulta retorna todos los documentos donde el campo **size** del documento es igual a **{ h:14, w: 21,  uom: "cm" } :**

```javascript
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )
```

Las coincidencias de igualdad donde todo el documento incrustado requiera una coincidencia exacta del documento **&lt;valor&gt;,** incluido el orden de los campos. ****Por ejemplo, la siguiente consulta no coincide con ningún documento:

```javascript
db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  )
```

### Consulta de un Campo Anidado

Para especificar la condición de búsqueda de un campo insertado o anidado, usar la **notación punto** \(campo.campoAnidado\): 

#### Condición igualdad de un campo incrustado

El siguiente ejemplo retorna todos los elementos donde el campo **uom** incrustado en el campo **size** sea igual a "**in**":

```javascript
db.inventory.find( { "size.uom": "in" } )
```

#### Condición especifica usando operadores

Una consulta de un campo puede usar operadores dentro de una condición de la siguiente forma:

```javascript
{ <campo1>: { <operador1>: <valor1> }, ... }
```

La siguiente consulta usa el operador menor que \(**$lt**\) en el campo **h** incrustado en el campo **size:**

```javascript
db.inventory.find( { "size.h": { $lt: 15 } } )
```

#### Condición AND

La siguiente consulta retorna todos los documentos donde el campo anidado **h**  es menor que  **15**, el campo anidado **uom** sea igual a "**in**" y el campo **status** sea igual a "**D**" :

```javascript
db.inventory.find( { "size.h": { $lt: 15 }, "size.uom": "in", status: "D" } )
```

