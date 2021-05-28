# Autenticación

La autenticación es el proceso de verificar la identidad de un cliente. Cuando el control de acceso, es decir, la [autorización](https://docs.mongodb.com/manual/core/authorization/) , está habilitado, MongoDB requiere que todos los clientes se autentiquen para determinar su acceso.

Aunque la autenticación y la [autorización](https://docs.mongodb.com/manual/core/authorization/) están estrechamente relacionadas, la autenticación es distinta de la autorización. La autenticación verifica la identidad de un usuario; la autorización determina el acceso del usuario verificado a los recursos y operaciones.

### Métodos de autenticación  <a id="authentication-methods"></a>

Para autenticarse como usuario, debe proporcionar un nombre de usuario, contraseña y la [base de datos de autenticación](https://docs.mongodb.com/manual/reference/program/mongo/#std-label-mongo-shell-authentication-options) asociada con ese usuario.

Para autenticarse usando el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell, haga lo siguiente:

* Utilice las [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)opciones de autenticación de línea de comandos \( [`--username`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--username), [`--password`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--password)y [`--authenticationDatabase`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--authenticationDatabase)\) cuando se conecta a la [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)o [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)ejemplo, o
* Conéctese primero a la instancia [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)o [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)y luego ejecute el [`authenticate`](https://docs.mongodb.com/manual/reference/command/authenticate/#mongodb-dbcommand-dbcmd.authenticate)comando o el [`db.auth()`](https://docs.mongodb.com/manual/reference/method/db.auth/#mongodb-method-db.auth)método en la [base de datos de autenticación](https://docs.mongodb.com/manual/reference/program/mongo/#std-label-mongo-shell-authentication-options) .

**IMPORTANTE:** _La autenticación varias veces como usuarios diferentes **no** elimina las credenciales de los usuarios previamente autenticados. Esto puede llevar a que una conexión tenga más permisos de los previstos por el usuario y hace que las operaciones dentro de una_ [_sesión lógica_](https://docs.mongodb.com/manual/reference/server-sessions/) _generen un error._

Para ver ejemplos de autenticación mediante un controlador MongoDB, consulte la [documentación del controlador](https://docs.mongodb.com/ecosystem/drivers/) .

### Mecanismos de autenticación  <a id="authentication-mechanisms"></a>

MongoDB admite una serie de [mecanismos](https://docs.mongodb.com/manual/core/authentication-mechanisms/#std-label-security-authentication-mechanisms) de [autenticación](https://docs.mongodb.com/manual/core/authentication-mechanisms/#std-label-security-authentication-mechanisms) que los clientes pueden utilizar para verificar su identidad. Estos mecanismos permiten que MongoDB se integre en su sistema de autenticación existente.

MongoDB admite múltiples mecanismos de autenticación:

* [SCRAM](https://docs.mongodb.com/manual/core/security-scram/#std-label-authentication-scram) \( _predeterminado_ \)
* [Autenticación de certificado x.509](https://docs.mongodb.com/manual/core/security-x.509/#std-label-security-auth-x509) .

Además de admitir los mecanismos mencionados anteriormente, MongoDB Enterprise también admite los siguientes mecanismos:

* [Autenticación de proxy LDAP](https://docs.mongodb.com/manual/core/authentication-mechanisms-enterprise/#std-label-security-auth-ldap) y
* [Autenticación Kerberos](https://docs.mongodb.com/manual/core/authentication-mechanisms-enterprise/#std-label-security-auth-kerberos) .

### Autenticación interna  <a id="internal-authentication"></a>

Además de verificar la identidad de un cliente, MongoDB puede requerir que los miembros de conjuntos de réplicas y clústeres fragmentados [autentiquen su pertenencia](https://docs.mongodb.com/manual/core/security-internal-authentication/#std-label-inter-process-auth) a su respectivo conjunto de réplicas o clúster fragmentado. Consulte [Autenticación interna / de membresía](https://docs.mongodb.com/manual/core/security-internal-authentication/#std-label-inter-process-auth) para obtener más información.

### Autenticación en clústeres fragmentados  <a id="authentication-on-sharded-clusters"></a>

En los clústeres fragmentados, los clientes generalmente se autentican directamente en las [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)instancias. Sin embargo, algunas operaciones de mantenimiento pueden requerir la autenticación directamente en un fragmento específico. Para obtener más información sobre la autenticación y los clústeres fragmentados, consulte [Usuarios de clústeres fragmentados](https://docs.mongodb.com/manual/core/security-users/#std-label-sharding-security) .

