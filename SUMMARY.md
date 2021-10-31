# Table of contents

* [🍃 MongoDB](README.md)

## Introducción

* [Bases de datos NoSQL](introduccion/bases-de-datos-nosql.md)
* [Colecciones y Documentos](introduccion/colecciones-y-documentos.md)
* [Tipo de Dato BSON](introduccion/tipo-de-dato-bjson.md)

## Instalación

* [Versiones](instalacion/versiones-1.md)
* [Ediciones Community y Enterprise](instalacion/ediciones-community-y-enterprise.md)
* [Instalación y Configuración](instalacion/versiones/README.md)
  * [Versiones de MongoDB](instalacion/versiones/versiones-de-mongodb.md)

## El Shell de Mongo

* [Conexión Shell a Servidor Mongo](el-shell-de-mongo/conexion-shell-a-servidor-mongo.md)
* [Configuración Shell de Mongo](el-shell-de-mongo/configuracion-shell-de-mongo.md)
* [Tipo de Datos en Mongo Shell](el-shell-de-mongo/tipo-de-datos-en-mongo-shell.md)
* [Scripts para Mongo Shell](el-shell-de-mongo/scripts-para-mongo-shell.md)

## Operaciones CRUD

* [Inserción de Documentos](operaciones-crud/insercion-de-documentos.md)
* [Consulta de Documentos](operaciones-crud/consulta-de-docuementos/README.md)
  * [Documentos Incrustados/Anidados](operaciones-crud/consulta-de-docuementos/documentos-incrustado-anidados.md)
  * [Arrays](operaciones-crud/consulta-de-docuementos/arrays.md)
  * [Array de Documentos Incrustados](operaciones-crud/consulta-de-docuementos/array-de-documentos-incrustados.md)
  * [Proyección en Documentos](operaciones-crud/consulta-de-docuementos/proyeccion-de-campos.md)
* [Actualización de Documentos](operaciones-crud/actualizacion-de-documentos.md)
* [Eliminación de Documentos](operaciones-crud/eliminacion-de-documentos.md)
* [Búsqueda de Texto](operaciones-crud/busqueda-de-texto.md)
* [Consultas Geoespaciales](operaciones-crud/consultas-geoespaciales.md)

## Modelo de Datos

* [Introducción al Modelado de Datos](modelo-de-datos/introduccion-al-modelado-de-datos.md)
* [Validación del Esquema](modelo-de-datos/validacion-del-esquema.md)
* [Conceptos del Modelado de Datos](modelo-de-datos/conceptos-del-modelado-de-datos/README.md)
  * [Diseño de modelos de datos](modelo-de-datos/conceptos-del-modelado-de-datos/diseno-de-modelos-de-datos.md)
  * [Factores operativos y modelos de datos](modelo-de-datos/conceptos-del-modelado-de-datos/factores-operativos-y-modelos-de-datos.md)
* [Relaciones entre Documentos](modelo-de-datos/relaciones-entre-documentos/README.md)
  * [Modele relaciones uno a uno con documentos incrustados](modelo-de-datos/relaciones-entre-documentos/modele-relaciones-uno-a-uno-con-documentos-incrustados.md)
  * [Modelar relaciones de uno a varios con documentos incrustados](modelo-de-datos/relaciones-entre-documentos/modelar-relaciones-de-uno-a-varios-con-documentos-incrustados.md)
  * [Modelar relaciones de uno a varios con referencias de documentos](modelo-de-datos/relaciones-entre-documentos/modelar-relaciones-de-uno-a-varios-con-referencias-de-documentos.md)
* [Estructura en Árbol](modelo-de-datos/estructura-en-arbol/README.md)
  * [Modelo de estructuras de árbol con referencias principales](modelo-de-datos/estructura-en-arbol/modelo-de-estructuras-de-arbol-con-referencias-principales.md)
  * [Modelo de estructuras de árbol  con referencias](modelo-de-datos/estructura-en-arbol/modelo-de-estructuras-de-arbol-con-referencias.md)
  * [Modelo de estructuras de árbol  con una matriz de antepasados](modelo-de-datos/estructura-en-arbol/modelo-de-estructuras-de-arbol-con-una-matriz-de-antepasados.md)
  * [Modelo de estructuras de árbol con rutas materializadas](modelo-de-datos/estructura-en-arbol/modelo-de-estructuras-de-arbol-con-rutas-materializadas.md)
  * [Modelo de estructuras de árbol con conjuntos anidados](modelo-de-datos/estructura-en-arbol/modelo-de-estructuras-de-arbol-con-conjuntos-anidados.md)

## Transacciones

* [Acerca de las transacciones](transacciones/untitled.md)
* [Drivers API](transacciones/drivers-api.md)
* [Consideraciones de producción](transacciones/consideraciones-de-produccion.md)
* [Consideraciones de producción (clústeres fragmentados)](transacciones/consideraciones-de-produccion-clusteres-fragmentados.md)
* [Transacciones y operaciones](transacciones/transacciones-y-operaciones.md)

## Indexación

* [Índices](indexacion/untitled.md)
* [Índices de campo único](indexacion/indices-de-campo-unico.md)
* [Índices compuestos](indexacion/indices-compuestos.md)
* [Índices Multikey](indexacion/indices-multikey/README.md)
  * [Límites de índice de varias ciudades](indexacion/indices-multikey/limites-de-indice-de-varias-ciudades.md)
* [Índices de texto](indexacion/indices-de-texto/README.md)
  * [Especificar un idioma para el índice de texto](indexacion/indices-de-texto/especificar-un-idioma-para-el-indice-de-texto.md)
  * [Especificar el nombre del textíndice](indexacion/indices-de-texto/especificar-el-nombre-del-textindice.md)
  * [Controlar los resultados de la búsqueda con ponderaciones](indexacion/indices-de-texto/controlar-los-resultados-de-la-busqueda-con-ponderaciones.md)
  * [Limite el número de entradas escaneadas](indexacion/indices-de-texto/limite-el-numero-de-entradas-escaneadas.md)

***

* [Índices comodín](indices-comodin/README.md)
  * [Restricciones del índice de comodines](indices-comodin/restricciones-del-indice-de-comodines.md)
* [Índices 2dsphere](indices-2dsphere/README.md)
  * [Consultar un 2dsphereíndice](indices-2dsphere/consultar-un-2dsphereindice.md)
* [Índices 2d](indices-2d/README.md)
  * [Crear un índice 2d](indices-2d/crear-un-indice-2d.md)
  * [Consultar un índice 2d](indices-2d/consultar-un-indice-2d.md)
  * [Elementos internos del índice 2d](indices-2d/elementos-internos-del-indice-2d.md)
  * [Calcular la distancia usando geometría esférica](indices-2d/calcular-la-distancia-usando-geometria-esferica.md)
* [Índices geoHaystack](indices-geohaystack/README.md)
  * [Crear un índice de pajar](indices-geohaystack/crear-un-indice-de-pajar.md)
  * [Consultar un índice de Haystack](indices-geohaystack/consultar-un-indice-de-haystack.md)
* [Índices hash](indices-hash.md)
* [Propiedades del índice](propiedades-del-indice/README.md)
  * [Índices TTL](propiedades-del-indice/indices-ttl/README.md)
    * [Caducar datos de colecciones configurando TTL](propiedades-del-indice/indices-ttl/caducar-datos-de-colecciones-configurando-ttl.md)
  * [Índices únicos](propiedades-del-indice/indices-unicos.md)

## Seguridad

* [Lista de verificación de seguridad](seguridad/untitled.md)
* [Habilitar el control de acceso](seguridad/enable-access-control.md)
* [Autenticación](seguridad/authentication.md)
* [Control de acceso basado en roles](seguridad/role-based-access-control.md)
