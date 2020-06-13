# Proyección en Documentos

Por defecto, las consultas en **MongoDB** retorna todos los campos de los documentos encontrados. Para limitar la cantidad de datos que **MongoDB** envía, pude incluir un documento de proyección que limite los campos a retornar.

 Los ejemplos de esta pagina utilizan la colección **inventory.** Para llenar la colección **inventory,** ejecuta el siguiente comando:

```javascript
db.inventory.insertMany( [
  { item: "journal", status: "A", size: { h: 14, w: 21, uom: "cm" }, instock: [ { warehouse: "A", qty: 5 } ] },
  { item: "notebook", status: "A",  size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "C", qty: 5 } ] },
  { item: "paper", status: "D", size: { h: 8.5, w: 11, uom: "in" }, instock: [ { warehouse: "A", qty: 60 } ] },
  { item: "planner", status: "D", size: { h: 22.85, w: 30, uom: "cm" }, instock: [ { warehouse: "A", qty: 40 } ] },
  { item: "postcard", status: "A", size: { h: 10, w: 15.25, uom: "cm" }, instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```

## Retornar Todos los Campos

Si no especifica un documento de proyección, el método **db.collection.find\(\)** retorna todos los campos en los documentos encontrados.

El siguiente ejemplo retorna todos los documentos de todos los documentos en la colección **inventory** donde el **status** sea igual a "**A**":

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { status: "A" } )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT * from inventory WHERE status = "A"
```
{% endtab %}
{% endtabs %}

## Retornar Campos Específicos y el Campo \_id

Una proyección puede incluir explícitamente varios campos estableciendo el **&lt;campo&gt;** como **1** en el documento de proyección. La siguiente operación retorna todos los documentos que coincidan. En el conjunto de resultado, solo el campo **item, status** y, por defecto, el campo **\_id** son retornados en los documentos encontrados.

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1 } )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT _id, item, status from inventory WHERE status = "A"
```
{% endtab %}
{% endtabs %}

## Omitir el Campo \_id

Puede remover el campo **\_id** del resultado configurando en **0** en el documento proyección, como en el siguiente ejemplo:

{% tabs %}
{% tab title="Mongo Shell" %}
```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1, _id: 0 } )
```
{% endtab %}

{% tab title="SQL" %}
```sql
SELECT item, status from inventory WHERE status = "A"
```
{% endtab %}
{% endtabs %}

## Retornar Todo Menos Los Campos Excluidos

En lugar de listar los campos a retornar en lo documento encontrado, puede usar una proyección para excluir campos. El siguiente ejemplo retorna todos los campos excepto por el **status** y el **instock** en los documentos encontrados:  

```javascript
db.inventory.find( { status: "A" }, { status: 0, instock: 0 } )
```

## Retornar Campos Específicos en Documentos Anidados

Puede especificar los campos a retornar de un documento anidado. Use la **notación punto** para referirse al campo del documento anidado y establecer en **1** en el documento de proyección.

El siguiente ejemplo retorna:

* **\_id** \(por defecto\)
* **item**
* **status**
* **uom** en el documento **size**

El campo **uom** permanece dentro del documento **size.**

```javascript
db.inventory.find(
   { status: "A" },
   { item: 1, status: 1, "size.uom": 1 }
)
```

## Omitir Campos Específicos en Documentos Anidados

Puede omitir campos especifico de un documento anidado. Use la **notación punto** para referirse al campo del documento anidado en el documento en el documento de proyección y establecer en **0.**

El siguiente ejemplo especifica una proyección que excluye el campo **uom** del documento. Todos los otros campos son retornados en los documentos encontrados: 

```javascript
db.inventory.find(
   { status: "A" },
   { "size.uom": 0 }
)
```

## Proyección en Documento Anidado en un Array 

Use la **notación punto** para proyectar específicos campos dentro de un documento insertado en un Array. 

El siguiente ejemplo especifica una proyección que retorna:

* **\_id** \(por defecto\)
* **item**
* **status**
* **qty** del documento insertado en el array **instock**

```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1, "instock.qty": 1 } )
```

## Proyectar Específicos Elementos de un Array

Para campos que contengan arrays, **MongoDB** proporciona la siguiente operación de proyección para manipular arrays: **$elemMatch, $slice,** and **$**.

El siguiente ejemplo usa la operación de proyección **$slice** que retorna el ultimo elemento en el array **instock:**

```javascript
db.inventory.find( { status: "A" }, { item: 1, status: 1, instock: { $slice: -1 } } )
```

**$elemMatch, $slice,** y **$,** son la única manera de proyectar elementos específicos para incluir en el array retornado. No puede proyectar elementos específicos de un array usando la posición de uno de los index del array; e.j. la proyección **{ "instock.0":  1 }** no proyectara el array con el primer elemento.

