# Control de acceso basado en roles

MongoDB emplea el control de acceso basado en roles \(RBAC\) para controlar el acceso a un sistema MongoDB. A un usuario se le otorgan uno o más [roles](https://docs.mongodb.com/manual/core/authorization/#std-label-roles) que determinan el acceso del usuario a los recursos y operaciones de la base de datos. Fuera de las asignaciones de funciones, el usuario no tiene acceso al sistema.

### Habilitar el control de acceso  <a id="enable-access-control"></a>

MongoDB no habilita el control de acceso de forma predeterminada. Puede habilitar la autorización mediante [`--auth`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--auth)la [`security.authorization`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-security.authorization)configuración o . La habilitación de [la autenticación interna](https://docs.mongodb.com/manual/core/security-internal-authentication/) también habilita la autorización del cliente.

Una vez que se habilita el control de acceso, los usuarios deben [autenticarse](https://docs.mongodb.com/manual/core/authentication/) .

### Roles  <a id="roles"></a>

Un rol otorga privilegios para realizar las [acciones](https://docs.mongodb.com/manual/reference/privilege-actions/#std-label-security-user-actions) especificadas en el [recurso](https://docs.mongodb.com/manual/reference/resource-document/) . Cada privilegio se especifica explícitamente en el rol o se hereda de otro rol o de ambos.

#### Privilegios  <a id="privileges"></a>

Un privilegio consta de un recurso especificado y las acciones permitidas en el recurso.

Un [recurso](https://docs.mongodb.com/manual/reference/resource-document/) es una base de datos, colección, conjunto de colecciones o el clúster. Si el recurso es el clúster, las acciones afiliadas afectan el estado del sistema en lugar de una base de datos o colección específica. Para obtener información sobre los documentos de recursos, consulte el [Documento de recursos](https://docs.mongodb.com/manual/reference/resource-document/) .

Una [acción](https://docs.mongodb.com/manual/reference/privilege-actions/) especifica la operación permitida en el recurso. Para conocer las acciones disponibles, consulte [Acciones de privilegio](https://docs.mongodb.com/manual/reference/privilege-actions/) .

#### Privilegios heredados  <a id="inherited-privileges"></a>

Un rol puede incluir uno o más roles existentes en su definición, en cuyo caso el rol hereda todos los privilegios de los roles incluidos.

Un rol puede heredar privilegios de otros roles en su base de datos. Un rol creado en la `admin`base de datos puede heredar privilegios de roles en cualquier base de datos.

#### Ver los privilegios del rol  <a id="view-role-s-privileges"></a>

Puede ver los privilegios de un rol emitiendo el [`rolesInfo`](https://docs.mongodb.com/manual/reference/command/rolesInfo/#mongodb-dbcommand-dbcmd.rolesInfo) comando con los campos `showPrivileges`y `showBuiltinRoles`configurados en `true`.

### Usuarios y roles  <a id="users-and-roles"></a>

Puede asignar roles a los usuarios durante la creación de usuarios. También puede actualizar los usuarios existentes para otorgar o revocar roles. Para obtener una lista completa de los métodos de administración de usuarios, consulte [Administración de usuarios](https://docs.mongodb.com/manual/reference/method/#std-label-user-management-methods)

Un usuario asignado a un rol recibe todos los privilegios de ese rol. Un usuario puede tener múltiples roles. Al asignar roles de usuario en varias bases de datos, un usuario creado en una base de datos puede tener permisos para actuar en otras bases de datos.NOTA

El primer usuario creado en la base de datos debe ser un administrador de usuarios que tenga privilegios para administrar a otros usuarios. Consulte [Habilitar control de acceso](https://docs.mongodb.com/manual/tutorial/enable-authentication/) .

### Roles integrados y roles definidos por el usuario  <a id="built-in-roles-and-user-defined-roles"></a>

MongoDB proporciona [roles integrados](https://docs.mongodb.com/manual/reference/built-in-roles/) que brindan un conjunto de privilegios comúnmente necesarios en un sistema de base de datos.

Si estos roles integrados no pueden proporcionar el conjunto de privilegios deseado, MongoDB proporciona métodos para crear y modificar [roles definidos por el usuario](https://docs.mongodb.com/manual/core/security-user-defined-roles/) .

