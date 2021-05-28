# Lista de verificación de seguridad

Este documento proporciona una lista de medidas de seguridad que debe implementar para proteger su instalación de MongoDB. La lista no pretende ser exhaustiva.

### Lista de verificación de preproducción / consideraciones  <a id="pre-production-checklist-considerations"></a>

#### ➤ Habilite el control de acceso y aplique la autenticación  <a id="arrow-enable-access-control-and-enforce-authentication"></a>

* Habilite el control de acceso y especifique el mecanismo de autenticación. Puede utilizar el mecanismo de autenticación SCRAM o x.509 de MongoDB o integrarlo con su infraestructura Kerberos / LDAP existente. La autenticación requiere que todos los clientes y servidores proporcionen credenciales válidas antes de que puedan conectarse al sistema.  


  Ver también:

  * [Autenticación](https://docs.mongodb.com/manual/core/authentication/)
  * [Habilitar control de acceso](https://docs.mongodb.com/manual/tutorial/enable-authentication/)

#### ➤ Configurar el control de acceso basado en roles  <a id="arrow-configure-role-based-access-control"></a>

* **Primero** cree un administrador de usuarios y luego cree usuarios adicionales. Cree un usuario de MongoDB único para cada persona / aplicación que acceda al sistema.
* Siga el principio de privilegio mínimo. Cree roles que definan los derechos de acceso exactos que requiere un conjunto de usuarios. Luego cree usuarios y asígneles solo los roles que necesitan para realizar sus operaciones. Un usuario puede ser una persona o una aplicación cliente.NOTA

  Un usuario puede tener privilegios en diferentes bases de datos. Si un usuario requiere privilegios en varias bases de datos, cree un único usuario con roles que otorguen los privilegios de base de datos aplicables en lugar de crear el usuario varias veces en diferentes bases de datos.  


  Ver también:

  * [Control de acceso basado en roles](https://docs.mongodb.com/manual/core/authorization/)
  * [Administrar usuarios y roles](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/)

#### ➤ Cifrar comunicación \(TLS / SSL\)  <a id="arrow-encrypt-communication--tls-ssl-"></a>

* Configure MongoDB para usar TLS / SSL para todas las conexiones entrantes y salientes. Usar TLS / SSL para cifrar la comunicación entre [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)y [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos)componentes de un despliegue MongoDB, así como entre todas las aplicaciones y MongoDB.

  A partir de la versión 4.0, MongoDB usa las bibliotecas nativas del sistema operativo TLS / SSL:

  | Plataforma | Biblioteca TLS / SSL |
  | :--- | :--- |
  | Ventanas | Canal seguro \(Schannel\) |
  | Linux / BSD | OpenSSL |
  | Mac OS | Transporte seguro |

  NOTA

  A partir de la versión 4.0, MongoDB deshabilita la compatibilidad con el cifrado TLS 1.0 en sistemas donde TLS 1.1+ está disponible. Para obtener más detalles, consulte [Deshabilitar TLS 1.0](https://docs.mongodb.com/manual/release-notes/4.0/#std-label-4.0-disable-tls) .  


  Consulte también: [Configurar `mongod`y `mongos`para TLS / SSL](https://docs.mongodb.com/manual/tutorial/configure-ssl/) .

#### ➤ Cifrar y proteger datos  <a id="arrow-encrypt-and-protect-data"></a>

* A partir de MongoDB Enterprise 3.2, puede cifrar datos en la capa de almacenamiento con el [cifrado en reposo](https://docs.mongodb.com/manual/core/security-encryption-at-rest/) nativo del motor de almacenamiento WiredTiger .
* Si no está utilizando el cifrado de WiredTiger en reposo, los datos de MongoDB deben cifrarse en cada host mediante un sistema de archivos, dispositivo o cifrado físico \(por ejemplo, dm-crypt\). Proteja los datos de MongoDB mediante permisos del sistema de archivos. Los datos de MongoDB incluyen archivos de datos, archivos de configuración, registros de auditoría y archivos de claves.
* Reúna los registros en un almacén de registros central. Estos registros contienen intentos de autenticación de la base de datos, incluida la dirección IP de origen.

#### ➤ Limitar la exposición de la red  <a id="arrow-limit-network-exposure"></a>

* Asegúrese de que MongoDB se ejecute en un entorno de red confiable y configure un firewall o grupos de seguridad para controlar el tráfico entrante y saliente para sus instancias de MongoDB.
* Deshabilite el acceso raíz SSH directo.
* Permita que solo los clientes de confianza accedan a las interfaces de red y los puertos en los que están disponibles las instancias de MongoDB.NOTA

  A partir de MongoDB 3.6, binarios de MongoDB [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)y [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#mongodb-binary-bin.mongos), se enlazan `localhost`de forma predeterminada. Desde las versiones 2.6 a 3.4 de MongoDB, solo los binarios de los paquetes oficiales de MongoDB RPM \(Red Hat, CentOS, Fedora Linux y derivados\) y DEB \(Debian, Ubuntu y derivados\) se vincularían `localhost`de forma predeterminada. Para obtener más información sobre este cambio, consulte [Cambios de compatibilidad de enlaces de Localhost](https://docs.mongodb.com/manual/release-notes/3.6-compatibility/#std-label-3.6-bind_ip-compatibility) .  


  Ver también:

  * [Fortalecimiento de la red y la configuración](https://docs.mongodb.com/manual/core/security-hardening/)
  * el [`net.bindIp`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-net.bindIp)ajuste de configuración
  * el [`security.clusterIpSourceWhitelist`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-security.clusterIpSourceWhitelist)ajuste de configuración
  * el campo [authenticationRestrictions](https://docs.mongodb.com/manual/reference/method/db.createUser/#std-label-db-createUser-authenticationRestrictions) al [`db.createUser()`](https://docs.mongodb.com/manual/reference/method/db.createUser/#mongodb-method-db.createUser)comando para especificar una lista blanca de IP por usuario.

#### ➤ Actividad del sistema de auditoría  <a id="arrow-audit-system-activity"></a>

* Rastree el acceso y los cambios a las configuraciones y los datos de la base de datos. [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) incluye una función de auditoría del sistema que puede registrar eventos del sistema \(por ejemplo, operaciones de usuario, eventos de conexión\) en una instancia de MongoDB. Estos registros de auditoría permiten el análisis forense y permiten a los administradores verificar los controles adecuados. Puede configurar filtros para registrar eventos específicos, como eventos de autenticación.  


  Ver también:

  * [Revisión de cuentas](https://docs.mongodb.com/manual/core/auditing/)
  * [Configurar auditoría](https://docs.mongodb.com/manual/tutorial/configure-auditing/)

#### ➤ Ejecute MongoDB con un usuario dedicado  <a id="arrow-run-mongodb-with-a-dedicated-user"></a>

* Ejecute procesos de MongoDB con una cuenta de usuario de sistema operativo dedicada. Asegúrese de que la cuenta tenga permisos para acceder a los datos, pero no permisos innecesarios.  


  Ver también: [Instalar MongoDB](https://docs.mongodb.com/manual/installation/)

#### ➤ Ejecute MongoDB con opciones de configuración segura  <a id="arrow-run-mongodb-with-secure-configuration-options"></a>

* MongoDB soporta la ejecución de código JavaScript para ciertas operaciones del lado del servidor: [`mapReduce`](https://docs.mongodb.com/manual/reference/command/mapReduce/#mongodb-dbcommand-dbcmd.mapReduce), [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#mongodb-query-op.-where), [`$accumulator`](https://docs.mongodb.com/manual/reference/operator/aggregation/accumulator/#mongodb-group-grp.-accumulator), y [`$function`](https://docs.mongodb.com/manual/reference/operator/aggregation/function/#mongodb-expression-exp.-function). Si no usa estas operaciones, deshabilite las secuencias de comandos del lado del servidor usando la [`--noscripting`](https://docs.mongodb.com/manual/reference/program/mongod/#std-option-mongod.--noscripting)opción en la línea de comando.
* Mantenga habilitada la validación de entrada. MongoDB habilita la validación de entrada de forma predeterminada a través de la [`net.wireObjectCheck`](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-net.wireObjectCheck)configuración. Esto asegura que todos los documentos almacenados por la [`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#mongodb-binary-bin.mongod)instancia sean [BSON](https://docs.mongodb.com/manual/reference/glossary/#std-term-BSON) válidos .  


  Consulte también: [Fortalecimiento de la configuración y la red](https://docs.mongodb.com/manual/core/security-hardening/)

#### ➤ Solicite una Guía de implementación técnica de seguridad \(cuando corresponda\)  <a id="arrow-request-a-security-technical-implementation-guide--where-applicable-"></a>

* La Guía de implementación técnica de seguridad \(STIG\) contiene pautas de seguridad para implementaciones dentro del Departamento de Defensa de los Estados Unidos. MongoDB Inc. proporciona su STIG, a pedido, para situaciones en las que sea necesario. Por favor, [solicitar una copia](http://www.mongodb.com/lp/contact/stig-requests) para obtener más información.

#### ➤ Considere el cumplimiento de los estándares de seguridad  <a id="arrow-consider-security-standards-compliance"></a>

* Para las aplicaciones que requieren el cumplimiento de HIPAA o PCI-DSS, consulte la [Arquitectura de referencia de seguridad de MongoDB](https://www.mongodb.com/collateral/mongodb-security-architecture) para obtener más información sobre cómo puede utilizar las capacidades de seguridad clave para crear una infraestructura de aplicaciones compatible.

### Verificaciones de producción periódicas / continuas [¶](https://docs.mongodb.com/manual/administration/security-checklist/#periodic-ongoing-production-checks) <a id="periodic-ongoing-production-checks"></a>

* Compruebe periódicamente el [CVE del producto MongoDB](https://www.mongodb.com/alerts) y actualice sus productos.
* Consulte las [fechas de finalización de la vida útil de MongoDB](https://www.mongodb.com/support-policy) y actualice su instalación de MongoDB. En general, intente mantenerse con la última versión.
* Asegúrese de que las políticas y los procedimientos de su sistema de gestión de seguridad de la información se extiendan a su instalación de MongoDB, incluida la realización de lo siguiente:
  * Aplique periódicamente parches a su máquina y revise las pautas.
  * Revise los cambios de políticas / procedimientos, especialmente los cambios en las reglas de su red para evitar la exposición inadvertida de MongoDB a Internet.
  * Revise los usuarios de la base de datos MongoDB y rótelos periódicamente.

