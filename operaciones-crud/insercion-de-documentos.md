# Inserción de Documentos

### Insertar un Solo Documento

El método **db.collection.insertOne\(&lt;{data}&gt;\)** inserta un documento en la colección.

El siguiente ejemplo inserta un nuevo documento en la colección **inventario**. Sí el documento a insertar no tiene especificado el atributo _id, Mongo agregara el atributo **\_id** con un tipo de dato **ObjectId**._  ****

```javascript
> db.inventario.insertOne({ 
...      nombre: "Gel Antibacterial", 
...      cantidad: 20, 
...      tags: ["limpieza, salud"], 
...      tamaño: { h: 28, w: 35.5, unidad: "litro" } 
...   }
...)
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5edbf177fb7383970816b0f1")
}
```

**insertOne\(\)** retorna un documento que contiene el nuevo **\_id** insertado. Para verificar el nuevo documento insertado con todos sus atributos ejecutamos:

```javascript
> db.inventario.find({nombre: "Gel Antibacterial"})
```

### Insertar Multiple Documentos

_Desde la version 3.2_

El método **db.collection.insertMany\(&lt;\[data\]&gt;\)** permite insertar multiples documentos en una colección, pasando un array de documentos como parámetro del método**:**

```javascript
> db.inventatio.insertMany([
...    { nombre: "Periodico", cantidad: 25, tags: ["noticias", "red"], tamaño: { h: 14, w: 21, unidad: "cm" } },
...    { nombre: "Teclado", cantidad: 30, tags: ["tecnologia", "hardware"], tamaño: { h: 15, w: 30, unidad: "cm" } },
...    { nombre: "Guitarra", cantidad: 5, tags: ["musica"], tamaño: { h: 40, w: 120, unidad: "cm" } }
...  ])
{
    "acknowledged" : true,
    "insertedIds" : [ 
        ObjectId("5edbf177fb7383970816b0a1"),
        ObjectId("5edbf177fb7383910816b0c6"),
        ObjectId("5edbf177fb7383970716b084")
    ]
}
```

**insertMany\(\)** retorna los documentos que contiene los nuevos **\_id** insertados. Para verificar los nuevos documento insertado con todos sus atributos ejecutamos:

```javascript
> db.inventario.find( {} )
```

### Funcionamiento

#### Creación de colección

Si la colección a la que se le inserta uno o multiples documentos no existe, Mongo creara esta colección e insertara los documentos.  

#### Campo \_id

En MongoDB, cada documento almacenado en una colección requiere un campo id único que actue como una llave primaria \(primary key\). Sí un documento insertado omite __el campo **\_**_**id,**_ Mongo automaticamente generara un tipo de dato **ObjectId** en el campo _**\_id**_.

