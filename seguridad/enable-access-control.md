# Habilitar el control de acceso

### Descripción general  <a id="overview"></a>

Habilitar el control de acceso en una implementación de MongoDB fuerza la autenticación, lo que requiere que los usuarios se identifiquen. Al acceder a una implementación de MongoDB que tiene el control de acceso habilitado, los usuarios solo pueden realizar acciones según lo determinen sus roles.

El siguiente tutorial habilita el control de acceso en una [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancia independiente y usa el [mecanismo de autenticación predeterminado](https://docs.mongodb.com/manual/core/authentication-mechanisms/#std-label-authentication-mechanism-default) . Para conocer todos los mecanismos de autenticación admitidos, consulte [Mecanismos de autenticación](https://docs.mongodb.com/manual/core/authentication/#std-label-available-authentication-mechanisms) .

### Administrador de usuarios  <a id="user-administrator"></a>

Con el control de acceso habilitado, asegúrese de tener un usuario [`userAdmin`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-userAdmin)o un [`userAdminAnyDatabase`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-userAdminAnyDatabase)rol en la `admin`base de datos. Este usuario puede administrar usuarios y roles tales como: crear usuarios, otorgar o revocar roles de usuarios y crear o modificar roles aduaneros.

### Procedimiento  <a id="procedure"></a>

El siguiente procedimiento primero agrega un administrador de usuarios a una instancia de MongoDB que se ejecuta sin control de acceso y luego habilita el control de acceso.

**NOTA:** La instancia de ejemplo de MongoDB usa [`port 27017`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--port)y el directorio del `/var/lib/mongodb`directorio de datos . El ejemplo asume la existencia del directorio de datos `/var/lib/mongodb`. Especifique un directorio de datos diferente según corresponda.

**1**

#### Inicie MongoDB sin control de acceso.  <a id="start-mongodb-without-access-control"></a>

Inicie una [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancia independiente sin control de acceso.

Por ejemplo, abra una terminal y emita lo siguiente:

```text
mongod --port 27017 --dbpath /var/lib/mongodb
```

**2**

#### Conéctese a la instancia.  <a id="connect-to-the-instance"></a>

Por ejemplo, abra una nueva terminal y conecte un [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo) shell a la instancia:

```text
mongo --port 27017 
```

Especifique opciones de línea de comando adicionales según corresponda para conectar el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell a su implementación, como `--host`.

**3**

#### Cree el administrador de usuarios.  <a id="create-the-user-administrator"></a>

Desde el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell, agregue un usuario con el [`userAdminAnyDatabase`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-userAdminAnyDatabase)rol en la `admin`base de datos. Incluya roles adicionales según sea necesario para este usuario. Por ejemplo, lo siguiente crea el usuario `myUserAdmin`en la `admin`base de datos con el [`userAdminAnyDatabase`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-userAdminAnyDatabase)rol y el [`readWriteAnyDatabase`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-readWriteAnyDatabase)rol.

**CONSEJO:** _A partir de la versión 4.2 del_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_shell, puede usar el_ [_`passwordPrompt()`_](https://docs.mongodb.com/manual/reference/method/passwordPrompt/#mongodb-method-passwordPrompt)_método junto con varios métodos / comandos de autenticación / administración de usuario para solicitar la contraseña en lugar de especificar la contraseña directamente en la llamada al método / comando. Sin embargo, aún puede especificar la contraseña directamente como lo haría con versiones anteriores del_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_shell._

```text
use admin
db.createUser(
  {
    user: "myUserAdmin",
    pwd: passwordPrompt(), // or cleartext password
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
)

```

**NOTA**_: La base de datos donde crea el usuario \(en este ejemplo `admin`\) es la_ [_base de datos de autenticación_](https://docs.mongodb.com/manual/core/security-users/#std-label-user-authentication-database) _del usuario . Aunque el usuario se autenticaría en esta base de datos, el usuario puede tener roles en otras bases de datos; es decir, la base de datos de autenticación del usuario no limita los privilegios del usuario._

**4**

#### Reinicie la instancia de MongoDB con control de acceso.  <a id="re-start-the-mongodb-instance-with-access-control"></a>

1. Cierra la [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancia. Por ejemplo, desde el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell, emita el siguiente comando:

   ```text
   db.adminCommand( { shutdown: 1 } )
   ```

2. Sal del [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)caparazón.
3. Inicie [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)con el control de acceso habilitado.
   * Si inicia [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)desde la línea de comando, agregue la [`--auth`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--auth)opción de línea de comando:

     ```text
     mongod --auth --port 27017 --dbpath /var/lib/mongodb
     ```

   * Si comienza [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)utilizando un [archivo de configuración](https://docs.mongodb.com/manual/reference/configuration-options/#std-label-configuration-options) , agregue la [`security.authorization`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-security.authorization)configuración del archivo de configuración:

     ```text
     security:    authorization: enabled
     ```

Los clientes que se conectan a esta instancia ahora deben autenticarse como usuarios de MongoDB. Los clientes solo pueden realizar acciones según lo determinado por sus roles asignados.

**5**

#### Conéctese y autentíquese como administrador de usuarios.  <a id="connect-and-authenticate-as-the-user-administrator"></a>

Usando el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)caparazón, puede:

* Conéctese con la autenticación pasando las credenciales de usuario, o
* Conéctese primero sin autenticación y luego emita el [`db.auth()`](https://docs.mongodb.com/manual/reference/method/db.auth/#mongodb-method-db.auth)método para autenticarse.

Autenticar durante la conexiónAutenticar después de la conexión

Iniciar un [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell con el [`-u <username>`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--username), [`-p`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--password)y las [`--authenticationDatabase <database>`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--authenticationDatabase)opciones de línea de comandos:

```text
mongo --port 27017  --authenticationDatabase "admin" -u "myUserAdmin" -p
```

Introduzca su contraseña cuando se le solicite.6

#### Cree usuarios adicionales según sea necesario para su implementación.  <a id="create-additional-users-as-needed-for-your-deployment"></a>

Una vez autenticado como administrador de usuarios, utilícelo [`db.createUser()`](https://docs.mongodb.com/manual/reference/method/db.createUser/#mongodb-method-db.createUser)para crear usuarios adicionales. Puede asignar [roles integrados](https://docs.mongodb.com/manual/reference/built-in-roles/) o [roles definidos](https://docs.mongodb.com/manual/core/security-user-defined-roles/) por el usuario a los usuarios.

La siguiente operación agrega un usuario `myTester`a la `test` base de datos que tiene un [`readWrite`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-readWrite)rol en la `test` base de datos y también un [`read`](https://docs.mongodb.com/manual/reference/built-in-roles/#mongodb-authrole-read)rol en la `reporting` base de datos.

**CONSEJO**_: A partir de la versión 4.2 del_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_shell, puede usar el_ [_`passwordPrompt()`_](https://docs.mongodb.com/manual/reference/method/passwordPrompt/#mongodb-method-passwordPrompt)_método junto con varios métodos / comandos de autenticación / administración de usuario para solicitar la contraseña en lugar de especificar la contraseña directamente en la llamada al método / comando. Sin embargo, aún puede especificar la contraseña directamente como lo haría con versiones anteriores del_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_shell._

```text
use test
db.createUser(
  {
    user: "myTester",
    pwd:  passwordPrompt(),   // or cleartext password
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)

```

**NOTA**_: La base de datos donde crea el usuario \(en este ejemplo `test`\) es la_ [_base de datos de autenticación de_](https://docs.mongodb.com/manual/core/security-users/#std-label-user-authentication-database) _ese usuario . Aunque el usuario se autenticaría en esta base de datos, el usuario puede tener roles en otras bases de datos; es decir, la base de datos de autenticación del usuario no limita los privilegios del usuario._

Después de crear los usuarios adicionales, desconecte el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell.

**7**

#### Conéctese a la instancia y autentíquese como `myTester`.  <a id="connect-to-the-instance-and-authenticate-as-mytester"></a>

Después de desconectar el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell como `myUserAdmin`, vuelva a conectarlo como `myTester`. Usted puede:

* Conéctese con la autenticación pasando las credenciales de usuario, o
* Conéctese primero sin autenticación y luego emita el [`db.auth()`](https://docs.mongodb.com/manual/reference/method/db.auth/#mongodb-method-db.auth)método para autenticarse.

**Autenticar durante la conexión**

Iniciar un [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell con el [`-u <username>`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--username), [`-p`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--password)y las [`--authenticationDatabase <database>`](https://docs.mongodb.com/manual/reference/program/mongo/#std-option-mongo.--authenticationDatabase)opciones de línea de comandos:

```text
mongo --port 27017 -u "myTester" --authenticationDatabase "test" -p
```

Ingrese la contraseña del usuario cuando se le solicite.8

**Autenticar después de la conexión**

\*\*\*\*

Conecte la [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)carcasa a [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod):

```text
mongo --port 27017
```

 En el [`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)shell, cambie a la base de datos de autenticación \(en este caso, `admin`\) y use el [`db.auth(<username>, <pwd>)`](https://docs.mongodb.com/manual/reference/method/db.auth/#mongodb-method-db.auth) método para autenticarse:

**CONSEJO:** _A partir de la versión 4.2 del_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_shell, puede usar el_ [_`passwordPrompt()`_](https://docs.mongodb.com/manual/reference/method/passwordPrompt/#mongodb-method-passwordPrompt)_método junto con varios métodos / comandos de autenticación / administración de usuario para solicitar la contraseña en lugar de especificar la contraseña directamente en la llamada al método / comando. Sin embargo, aún puede especificar la contraseña directamente como lo haría con versiones anteriores del_ [_`mongo`_](https://docs.mongodb.com/manual/reference/program/mongo/#mongodb-binary-bin.mongo)_shell._

```text
use admin
db.auth("myUserAdmin", passwordPrompt()) // or cleartext password
```

#### _8_ <a id="insert-a-document-as-mytester"></a>

#### Inserte un documento como `myTester`.  <a id="insert-a-document-as-mytester"></a>

Como `myTester`, tiene privilegios para realizar operaciones de lectura y escritura en la `test`base de datos \(así como para realizar operaciones de lectura en la `reporting`base de datos\). Una vez autenticado como `myTester`, inserte un documento en una colección en la `test` base de datos. Por ejemplo, puede realizar la siguiente operación de inserción en la `test`base de datos:

```text
db.foo.insert( { x: 1, y: 1 } )
```

CONSEJO Ver también:

[Administrar usuarios y roles](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/) .

### Consideraciones adicionales  <a id="additional-considerations"></a>

#### Conjuntos de réplicas y clústeres fragmentados  <a id="replica-sets-and-sharded-clusters"></a>

Los conjuntos de réplicas y los clústeres fragmentados requieren autenticación interna entre los miembros cuando el control de acceso está habilitado. Para obtener más detalles, consulte [Autenticación interna / de membresía](https://docs.mongodb.com/manual/core/security-internal-authentication/) .

#### Excepción de localhost [¶](https://docs.mongodb.com/manual/tutorial/enable-authentication/#localhost-exception) <a id="localhost-exception"></a>

Puede crear usuarios antes o después de habilitar el control de acceso. Si habilita el control de acceso antes de crear cualquier usuario, MongoDB proporciona una [excepción de host local](https://docs.mongodb.com/manual/core/security-users/#std-label-localhost-exception) que le permite crear un administrador de usuarios en la `admin`base de datos. Una vez creado, debe autenticarse como administrador de usuarios para crear usuarios adicionales según sea necesario.

