# Ediciones Community y Enterprise

MongoDB Enterprise proporciona varias funciones que no están disponibles en la edición de MongoDB Community, como:

* Motor de almacenamiento en memoria
* Auditoria
* Autenticación Kerberos
* Autenticación de proxy LDAP y autorización LDAP 
* Cifrado en Rest

**Motor de almacenamiento en memoria**

A partir de MongoDB Enterprise versión 3.2.6, el motor de almacenamiento en memoria es parte de la disponibilidad general \(GA\) en las compilaciones de 64 bits. Además de algunos metadatos y datos de diagnóstico, el motor de almacenamiento en memoria no mantiene ningún dato en disco, incluidos datos de configuración, índices, credenciales de usuario, etc.

Al evitar las E / S de disco, el motor de almacenamiento en memoria permite una latencia más predecible de las operaciones de la base de datos.

**Auditoria**

MongoDB Enterprise incluye una capacidad de auditoría **mongod** e **mongos** instancias. La función de auditoría permite a los administradores y usuarios rastrear la actividad del sistema para implementaciones con múltiples usuarios y aplicaciones.

El sistema de auditoría escribe cada evento de auditoría en un búfer en memoria de eventos de auditoría. MongoDB escribe este búfer en el disco periódicamente. Para los eventos recopilados de cualquier conexión individual, los eventos tienen un orden total: si MongoDB escribe un evento en el disco, el sistema garantiza que ha escrito todos los eventos anteriores para esa conexión al disco.

Si una entrada de evento de auditoría corresponde a una operación que afecta el estado duradero de la base de datos, como una modificación en los datos, MongoDB siempre escribirá el evento de auditoría en el disco _antes de_ escribir en el glosario  esa entrada.

Es decir, antes de agregar una operación al diario, MongoDB escribe todos los eventos de auditoría en la conexión que activó la operación, hasta e incluyendo la entrada para la operación.

Estas garantías de auditoría requieren que MongoDB se ejecute con **journaling** habilitado.

**Autenticación Kerberos**

Kerberos es un protocolo de autenticación estándar de la industria para grandes sistemas cliente / servidor. Kerberos permite que MongoDB y las aplicaciones aprovechen la infraestructura y los procesos de autenticación existentes.

**Autenticación de proxy LDAP y autorización LDAP** 

MongoDB Enterprise admite consultas en un servidor LDAP para los grupos LDAP a los que pertenece el usuario autenticado. MongoDB asigna los nombres distinguidos \(DN\) de cada grupo devuelto a roles en la **admin** base de datos. MongoDB autoriza al usuario en función de los roles asignados y sus privilegios asociados. 

El proceso de autorización de LDAP se resume a continuación:

1. Un cliente se autentica en MongoDB, proporcionando las credenciales de un usuario.
2. Si el nombre de usuario requiere la asignación a un DN LDAP antes de vincularse con el servidor LDAP, MongoDB puede aplicar transformaciones basadas en la configuración **security.ldap.userToDNMapping** configurada.
3. MongoDB se une a un servidor LDAP especificado al **security.ldap.servers** usar el nombre de usuario proporcionado o, si se aplicó una transformación, el nombre de usuario transformado.

   MongoDB usa el enlace simple por defecto, pero también puede usar el `sasl`enlace si está configurado en security.ldap.bind.method y **security.ldap.bind.saslMechanisms**

   Si una transformación requiere consultar el servidor LDAP, o si el servidor LDAP no permite enlaces anónimos, MongoDB usa el nombre de usuario y la contraseña especificados **security.ldap.bind.queryUser** y se **security.ldap.bind.queryPassword** vincula al servidor LDAP antes de intentar autenticar las credenciales de usuario proporcionadas.

4. El servidor LDAP devuelve el resultado del intento de enlace a MongoDB. En caso de éxito, MongoDB intenta autorizar al usuario.
5. El servidor MongoDB intenta asignar el nombre de usuario a un usuario en la `$external`base de datos, asignándole al usuario cualquier rol o privilegio asociado a un usuario coincidente. Si MongoDB no puede encontrar un usuario coincidente, la autenticación falla.
6. El cliente puede realizar aquellas acciones para las cuales MongoDB otorgó los roles o privilegios de usuario autenticado.

**Cifrado en Rest**

Presenta una opción de cifrado nativa para el motor de almacenamiento WiredTiger. Esta característica permite a MongoDB cifrar archivos de datos de modo que solo las partes con la clave de descifrado puedan decodificar y leer los datos.

