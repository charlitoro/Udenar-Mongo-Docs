---
description: >-
  Introducción al shell de mongo y comando básicos para la administración de
  datos y usuarios.
---

# Conexión Shell a Servidor Mongo

El shell de mongo es una interactiva interface de JavaScript para MongoDB. Mediante commandos en consola, el shell de mongo permite la construcción de queries y administración de operaciones con una simple estructura de consulta.

### Prerrequisitos

Asegurarse que MongoDB este instalado y corriendo su servicio antes de iniciar con el shell de mongo. El shell de mongo no necesita una instalación adicional, hace parte del paquete de MongoDB server.

{% hint style="info" %}
Agregar el **&lt;directorio de instalación de mongo&gt;/bin** al las variable de entorno **PATH** para sistemas UNI; para Windows ir a **Configuraciones avanzadas del sistema -&gt; Avanzado -&gt; Variables de Entorno** y agregar el path donde se encuentre instalado mongo.
{% endhint %}

### Servidor local en puerto por defecto

Es posible correr el shell de mongo sin ninguna opción en la linea de comando, esto es posible si el server tiene sus configuraciones por defecto, saliendo por **localhost** y por el puerto por defecto **27017**.

```bash
$ mongo
```

### Servidor local en un puerto no por defecto

Para acceder a un server local con una configuración de puerto distinto al por defecto, se incluye en el comando el parámetro **--port.**

```bash
$ mongo --port 28018
```

### Servidor remoto

Se especifica el host y/o el puerto,

Es posible especificar e una cadena de texto toda la configuración del servidor remoto

```bash
# host: example-host.com  puerto: 28018
$ mongo "mongodb://example-host.com:28018" 
```

Usando los parámetros del shell es posible enviar concatenado el host y el puerto con la opción **--host** en el comando

```bash
$ mongo --host example-host.com:28018
```

El puerto se puede separar en enviar como un parámetro mas usando **--host** y **--port** como opciones por separados en el mismo comando  

```bash
$ mongo --host example-host.com --port 28018
```

### Servidor con autenticación

Todos los gestores de bases de datos tienen autentocación por usuario y contraseña para acceder seguramente al servidor. 

Se especifica el usuario y la contraseña dentro de la cadena de texto junto al host y al puerto. Sí no se enviar la contraseña, el prompt de la terminal pedirá la contrasella del usuario.

```bash
# usuario: carlos
$ mongo "mongodb://carlos@example-host.com:28018/?authSource=admin"
```

Es posible enviar separado todos los parámetros para realizar una conexión al servidor con autenticacion de usuario contraseña. Se utiliza los parámetros **--username &lt;user&gt;** and **--password**, **--authenticationDatabase &lt;db&gt;**. Sí no se enviar la contraseña, el prompt de la terminal pedirá la contrasella del usuario.

```bash
$ mongo --username alice --password --authenticationDatabase admin --host example-host.com --port 28015
```



