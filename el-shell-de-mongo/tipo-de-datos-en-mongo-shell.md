# Tipo de Datos en Mongo Shell

**Tipo Fecha**

Mongo shell proporciona varios métodos para devolver la fecha, ya sea como cadena o como un objeto tipo Date, aquí se tiene: 

```javascript
Date() // Método que devuelve la fecha actual como una cadena.
new Date() // Constructor que devuelve un objeto de tipo Date. 
```

Para devolver la fecha como una cadena, usando el `Date()`método, se hace de la siguiente manera:

```javascript
var  myDateString = Fecha ();
```

Para imprimir el valor de la variable, escriba el nombre de la variable en el shell, como se muestra a continuación y en la segunda linea el valor del resultado de la variable: 

```javascript
myDateString
// Mié  19 Dic  2012 01 : 03 : 25 GMT - 0500 ( EST )
```

**ObjectId**

Proporciona la `ObjectId()`clase contenedora alrededor del tipo de datos ObjectId**.** 

```javascript
new ObjectId
```

  
**NumberLong**

Trata todos los números como valores de punto flotante de forma predeterminada, El Mongo Shell proporciona el `NumberLong()` contenedor para manejar enteros de 64 bits.

```javascript
NumberLong ( "2090845886852" )
```

#### NumberInt

El Mongo Shell proporciona al `NumberInt()` constructor para especificar explícitamente enteros de 32 bits.

#### NumberDecimal

El Mongo shell proporciona al `NumberDecimal()`constructor para especificar explícitamente valores de punto flotante basados ​​en decimales de 128 bits capaces de emular el redondeo decimal con precisión exacta. Esta funcionalidad está destinada a aplicaciones que manejan datos monetarios , como cálculos financieros, impositivos y científicos.

