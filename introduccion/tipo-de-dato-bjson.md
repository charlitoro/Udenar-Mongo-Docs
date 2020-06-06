---
description: BSON es el tipo de dato como mongo guarda los documentos
---

# Tipo de Dato BSON

**BSON** es el tipo de dato como **Mongo** almacena los documentos y realiza procesamiento de datos remotamente. **BSON** un formato binario serealizado que tiene una gran similitud al tipo de dato **JSON**, con la diferencia que **BSON** almacena binariamente los datos para mayor rendimiento.  

Cada tipo en un **BSON** puede ser identificado por un valor numérico o de cadena descritos en la siguiente lista.  

| Type | Number | Alias |
| :--- | :--- | :--- |
| Double | 1 | "double" |
| String | 2 | "string" |
| Object | 3 | "object" |
| Array | 4 | "array" |
| Binary data | 5 | "binData" |
| ObjectId | 7 | "objectId" |
| Boolean | 8 | "bool" |
| Date | 9 | "data" |
| Null | 10 | "null" |
| Regular Expression | 11 | "regex" |
| JavaScript | 13 | "javascript" |
| 32-bit integer | 16 | "int" |
| Timestamp | 17 | "timestamp" |
| 64-bin integer | 18 | "long" |

Algunos de los tipos mencionados anteriormente tienen ciertas particularidades especiales descritas a continuación:

### ObjectId

Es un tipo de dato pequeño, unico, rapido de generar y ordenado que esta compuesto por un tamaño de 12 bytes:

* 4 bytes de un valor timestamp, con la fecha de creación del objetoId
* 5 bytes con un valor generado aleatoriamente
* 3 bytes con un valor contador incremental

En MongoDB, cada documento almacenado en una colección requiere un campo _id único que sirve como una llave primaria. Si el campo id es omitido en la creación o inserción de un documento, los drivers de Mongo automáticamente generan un **ObjectId** para el campo **\_**_**id.**

### **String**

El tipo de dato **String** dentro del formato **BSON** son **UTF-8**. En general, los drivers de cada lenguaje de programacion convierte el formato string a **UTF-8** en la serialización y deserialización de **BSON.** Adicionalmente, las peticiones $regex de Mongo soportan UTF-8 en expresiones regulares.

### Timestamps

Timestamp es un estándar internacional de formato de rastreo de tiempo como un total de acumulado de segundos. Es un numero que inicia su recuento desde la época de Unix, específicamente la fecha es 1 de enero de 1970 en UTC. En ese sentido un numero timestamp representa el numero de segundo que han transcurrido hasta un momento determinado, y es un formato usado para el procesamiento con fechas.

**BSON** tiene un timestamp especial interno en Mongo que no esta asociado con el tipo de dato regular **Date:**

* Los 32 bits mas significantes con un valor **time\_t\(**Segundos desde la época de Unix**\)**
* Los 32 bits menos significantes son un valor incremental ordinario dentro de un segundo dado

```javascript
// Insersion de un documento con un timestamp vacio
db.test.insertOne({ts: new Timestamp()});

// Resulado del documento creado
{"_id": ObjectId("542c2b97bac0595474108b48"), "ts": Timmestamp(1412180887, 1)}
```

El servidor remplaza el valor vació para el campo **ts** con el valor timestamp del tiempo actual de inserción del documento.

### Date

Date es un tipo 64-bit integer que representa el numero de mili segundos desde la época Unix. Los valores negativo representan fechas antes de 1970.

{% tabs %}
{% tab title="new Date\(\)" %}
```javascript
// Generar un valor Date usando el constructor new Date() de mongo shell
var myDate = new Date();
```
{% endtab %}

{% tab title="ISODate\(\)" %}
```javascript
// Generar Date usado el constructor ISODate() de mongo shell
var myIsoDate = ISODate();
```
{% endtab %}

{% tab title="String Date" %}
```javascript
// Retornar la fecha como string de un objeto Date
myDate.toString()

"Wed May 27 2020 16:16:26 GMT-0500 (Colombia Standard Time)"

```
{% endtab %}

{% tab title="Obtener Mes" %}
```javascript
// Retornar el mes de un objeto Date
myDate.getMonth()
// Retorna el numero del mes, donde Enero es 0
4
```
{% endtab %}
{% endtabs %}

 

 

