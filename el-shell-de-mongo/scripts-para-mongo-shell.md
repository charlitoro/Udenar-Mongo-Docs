---
description: >-
  El shell de mongo permite escribir scripts en JavaScript para manipular los
  datos en MongoDB o administrar operaciones.
---

# Scripts para Mongo Shell

### Abrir Nuevas Conexiones

Desde el shell o desde un archivo JavaScript, es posible instanciar una conexión a mongo utilizando el constructor de mongo **Mongo\(\):**

```javascript
> new Mongo();
> new Mongo(<host>);
> new Mongo(<host:puerto>);
```

El siguiente ejemplo instancia una nueva conexión con **MongoDB** que se ejecuta en **localhost** en el puerto predeterminado y establece la variable global **db** a la base de datos myDatabase usando el método **getDB\(\):**

```javascript
> conn = new Mongo();
> db = conn.getDB("myDatabase");
```

Si la conexión requiere de aseso, es posible utilizar el método **db.auth\(\)** para autenticarse.

Adicionalmente, el método **connect\(\)** recibe una cadena de texto con todos los parámetros de conexión. El siguiente ejemplo se conecta al **localhost** por el puerto **27018** y la base de datos **myDatabase:**

```javascript
> db = connect("localhost:27020/myDatabase");
```

{% hint style="success" %}
 Lista de [métodos](https://docs.mongodb.com/manual/reference/method/) del shell de mongo
{% endhint %}

###  **Modo Interactivo y Scripts**

{% hint style="success" %}
En la version 4.2, el shell de mongo proporciona un método **isInteractive\(\)** que retorna un valor **booleano** que indica si estas en modo interactivo o script.
{% endhint %}

Consideraciones para escribir scripts en mongo:

* Para establecer la variable global **db**, se utiliza el metodo **getDB\(\)** o **connect\(\).**
* No es posible usar comando de ayuda alguno del shell \(e.g. **`use <dbname>`**, **`show dbs`**, etc.\) in los archivos JavaScript porque no son validos en el lenguaje JavaScript.

La siguiente tabla muestra los comandos ayuda del shell y su equivalente en JavaScript.  

<table>
  <thead>
    <tr>
      <th style="text-align:left">Shell</th>
      <th style="text-align:left">JavaScript Equivalents</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>show dbs,  show databases</b> 
      </td>
      <td style="text-align:left"><b>db.adminCommand( </b>&apos;listDatabases&apos;<b> )</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>use &lt;db&gt;</b>
      </td>
      <td style="text-align:left"><b>db = db.getSiblingDB( &apos;</b>&lt;db&gt;<b>&apos; )</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>show collections</b>
      </td>
      <td style="text-align:left"><b>db.getCollectionNames()</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>show users</b>
      </td>
      <td style="text-align:left"><b>db.getUsers()</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>show roles</b>
      </td>
      <td style="text-align:left"><b>db.getRoles( {showBuiltinRoles: true} )</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>show log &lt;logname&gt;</b>
      </td>
      <td style="text-align:left"><b>db.adminCommand( { &apos;getLog&apos; :  &apos;&lt;logname&gt;&apos; } )</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>show logs</b>
      </td>
      <td style="text-align:left"><b>db.adminCommand( { &apos;getLog&apos; :  &apos;*&apos; } )</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>it</b>
      </td>
      <td style="text-align:left">
        <p><b>cursor = db.collection.find() </b>
        </p>
        <p><b>if ( cursor.hasNext() ){ </b>
        </p>
        <p><b>    cursor.next(); </b>
        </p>
        <p><b>}</b>
        </p>
      </td>
    </tr>
  </tbody>
</table>

En el modo interactivo o consola, mongo inprime lso resultados incluido el contenido de todos los cursores\( documentos \). En un script este usa la función **print\(\)** de JavaScript o la función de mongo **printjson\(\)**, función que retorna un objeto **JSON**.

```javascript
// Imprime todos los items del docuemnto en el cursor resultado

> cursor = db.collection.find();
> while ( cursor.hasNext() ) {
>    printjson( cursor.next() );
> }
```

### Scripts

Desde la linea de comandos es posible evaluar script de JavaScript. Con la opción  **--eval**  se envia el fragmento de lineas:

```javascript
> mongo test --eval "printjson(db.getCollectionNames())"
```

Retorna los nombres de las colecciones en la base de datos.

#### Ejecutar un archivo JavaScript

Los archivos JavaScript tienen la extensión **.js** y mongo tiene la posibilidad de ejecutar directamente este tipo de archivos:

```javascript
> mongo localhost:27017/test myjsfile.js
```

Esta operación realiza la conexión con la base de datos **test** accediendo por  **localhost** puerto **27017** \(puerto por defecto\).

Otra alternativa para ejecitar un archivo **.js** es usando la función **load\(\)**:

```javascript
> load("myjstest.js")
```

Esta función carga y ejecuta el código del archivo **myjstest.js.**

El método **load\(\)** admite path relativos y absolutos. Sí el directorio del shell de mongo es **/data/db,** y el archivo **myjstest.js** esta en el directorio **/data/db/scripts,** entonces los siguientes llamados son equivalentes:

```javascript
> load("scripts/myjstest.js") // path relativo
> load("/data/db/scripts/myjstest.js") // path absoluto
```

