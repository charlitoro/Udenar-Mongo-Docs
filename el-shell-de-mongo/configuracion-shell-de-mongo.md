---
description: Configuraciones básicas de la terminal de commandos de mongo
---

# Configuración Shell de Mongo

### Personalización del prompt

Al tratarse de una interface de JavaScript, es posible crear pequeños script que permitan modificar ciertas variables que utiliza el shell de mongo. La variable **prompt** almacena un string con el texto de presentación de inicio de cada linea de comando. Es aconsejable poder tener información adicional en el **prompt** para tener una referencia que nos permita identificar rápidamente donde estemos trabajando.     

{% tabs %}
{% tab title="Numerado" %}
```javascript
> cmdCount = 1;
> prompt = function() {
>     return (cmdCount++) + "> ";
> }
> // Nuevo prompt
1>
2>
```
{% endtab %}

{% tab title="Base Dato y Host " %}
```javascript
> host = db.serverStatus().host;
> 
> prompt = function() {
>     return db+"@"+host+"$ ";
> }
> // Nuevo prompt
test@example-host$ 
test@example-host$   
```
{% endtab %}

{% tab title="Tiempo y Documentos" %}
```javascript
> prompt = function() {
>     return "Uptime:"+db.serverStatus().uptime+" Documents:"+db.stats().objects+" > ";
> }
> // Nuevo prompt
Uptime:5897 Documents:6 > 
```
{% endtab %}
{% endtabs %}

Estas funciones pueden ser ejecutadas en el prompt de mongo pero an el momento que salgamos del shell de mongo, las configuraciones de pierden. Para mantener la configuración deseada se puede establecer la función en el archivo de configuración **.mongorc.js.**

###   **Cambiar el tamaño de lote**

El método **db.collection.find\(\)** retorna un **cursor** con el resultado; sin embargo, si en el shell de mongo, si el cursor retornado no esta asignado a una variable, luego el cursor es automáticamente iterado hasta 20 veces para imprimir hasta los primeros 20 documentos que coincidan con la consulta. El shell de mongo le pedirá que lo escriba para iterar otras 20 veces. Se puede establecer el atributo **DBQuery.shellBatchSize** para cambiar el numero de documentos del valor predeterminado de 20

```javascript
DBQuery.shellBatchSize = 10;
```

