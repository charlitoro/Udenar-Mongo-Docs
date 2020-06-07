# Eliminación de Documentos

Se utiliza los siguientes método en Mongo shell:

* **db.collection.deleteMany\(\)**

      Elimina todos los documentos que coinciden con los **filter** de una colección.

```javascript
db.colección.deleteMany ( 
   < filter > , 
   { 
      writeConcern :  < document > , 
      colation :  < document > 
   } 
)
```

**Eliminar todos los documentos de una inventory colección.**

```javascript
db.inventario.deleteMany({})
```

**Eliminar todos los documentos que coinciden bajo una condición.**

Aquí se puede especificar criterios o filtros que identifiquen los documentos a eliminar. Los filtros usan la misma sintaxis que las operaciones de lectura.

A continuación especificamos condiciones de igual

```javascript
{  < campo1 >:< valor1 > ,  ...  }
```

El siguiente ejemplo elimina todos los documentos de la `inventory` colección donde el estado es igual a "A"

```javascript
db.inventario.deleteMany({  estado  :  "A"  })
```

* **db.collection.deleteOne\(\)**

         Elimina un solo documento de una colección.

```javascript
db.colección.deleteOne ( 
   < filter > , 
   { 
      writeConcern :  < document > , 
      colation :  < document > 
   } 
)
```

**Eliminar solo un documento que coincida con una condición**

Para eliminar como máximo un solo documento que coincida con un filtro especificado \(aunque varios documentos puedan coincidir con el filtro especificado\), se utiliza este método.

El siguiente ejemplo elimina el primer documento donde su estado está en `"D"`:  


```javascript
db.inventario.deleteOne(  {  estado :  "D"  }  )
```

