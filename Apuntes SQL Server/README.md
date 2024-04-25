# SQL Server consejos y tips

## Conceptos de bases de datos en SQL Server:
- **Una instancia** -> Puede pensarse como una instalación de SQL Server, por lo que cada vez que creamos un nuevo servicio y luego instalamos SQL Server, estamos creando una instancia de SQL Server. Cada instancia tiene una IAM network única. Es una copia de SQL Server que se ejecuta en un servidor. Un equipo puede tener instalada una o varias instancias de SQL Server, cada una de las cuales puede contener una o varias bases de datos. Las instancias son independientes entre sí y pueden ejecutarse en el mismo servidor físico o en diferentes servidores
- **Una base de datos** -> En alto nivel es una colección organizada de datos. Tiene esquemas y datos dentro, pero la base de datos solo es el contenedor de almacenaje y organización de estos. Cada base de datos contiene uno o varios grupos de propiedad de objetos denominados esquemas. Dentro de cada esquema, hay objetos de base de datos como tablas, vistas y procedimientos almacenados. Las bases de datos se almacenan en el sistema de archivos en archivos, que pueden agruparse en grupos de archivos
- **El esquema** -> Es un agrupamiento físico de tablas de manera lógica. El esquema es físico, porque los datos de la base de datos se guardan en disco con la forma que da el esquema. Es como un contenedor lógico dentro de una base de datos que agrupa objetos de base de datos, como tablas, vistas, y procedimientos almacenados. El esquema lógico de la base de datos describe las restricciones o reglas lógicas que se aplicarán a los datos. Su principal objetivo es comprender las entidades de datos, incluidas sus relaciones y atributos. En las tablas podemos ver que tienen la siguiente estructura: nombreEsquema.nombreTabla
- **Las tablas** -> Es un objeto de base de datos que almacena datos en filas y columnas. Cada fila representa un registro y cada columna representa un atributo del registro. Las tablas son los contenedores principales de datos en SQL Server y se utilizan para almacenar y organizar información estructurada. Las tablas pueden contener diferentes tipos de datos, como fechas, nombres, importes en moneda o números, y se pueden diseñar para almacenar información específica
- **Las vistas** -> Es una tabla virtual que se define por una consulta SELECT sobre una o más tablas. Los datos contenidos en una vista no están almacenados físicamente en la vista, sino que se generan dinámicamente cada vez que se consulta la vista. Las vistas se utilizan comúnmente como filtros de las tablas de las que recogen datos, actuando como mecanismos de seguridad que permiten a los usuarios tener acceso a los datos sin necesidad de acceder directamente a las tablas subyacentes. Además, las vistas pueden simplificar y personalizar la visualización de los datos para el usuario. Se suelen usar para mostrar consultas que se suelen hacer constantemente.
## Organización:
En el objeto principal que iniciamos, aparecen:
- **Databases** -> Son las bases de datos
- **Security** -> Son los roles de seguridad
- **Server Objects** -> Contiene objetos que son globales para el servidor, como endpoints, registros de errores y configuraciones de servidor
- **Replication** -> Permite copiar y distribuir datos y objetos de una base de datos a otra y mantenerlos sincronizados.
- **Manager** -> Herramientas o interfaces de administración que permiten gestionar y monitorear el servidor y sus recursos.
- **XEvent Profiler** ->  Es una herramienta que permite monitorear y analizar eventos que ocurren en el servidor, como consultas ejecutadas, errores, conexiones de clientes, etc., utilizando el sistema de eventos extendido (XEvent) de SQL Server.

## Antes de empezar:
SQL Server es case-insensitive por lo que da igual escribir con mayúsculas o minúsculas, pero es buena práctica usar los query statements en mayúsculas.
- Se ponen en azul oscuro y mayúsculas:
    - Las **palabras reservadas** (SELECT, FROM y WHERE)
    - Las **instrucciones DDL (Data Definition Language)** (CREATE, ALTER, DROP, TRUNCATE, RENAME, COMMENT)
    - Las **instrucciones DML (Data Manipulation Language)** (SELECT, INSET, UPDATE, DELETE)
- Se ponen en rosa y mayúsculas:
    - Las **aggregate functions** (SUM, MAX, MIN, COUNT, DISTINCT, AVG, )
- Se ponen en gris y mayúsculas:
    - Los **operadores lógicos** (AND, OR, NOT, XOR, LIKE, IN, BETWEEN, EXISTS, ALL, ANY o SOME). Estos siguen la "predicate logic" o "symbolic logic", que básicamente es el conocimiento de cómo diferentes expresiones booleanas y lógicas se comportan combinadamente. Ej: AND tiene preferencia sobre OR.
    - Los **operadores de conjunto** (INNER JOIN, LEFT JOIN, LEFT OUTER JOIN, RIGHT JOIN, RIGHT OUTER JOIN, FULL JOIN y FULL OUTER JOIN)
- Se ponen en rojo:
    - Las **REGEX** se ponen con comillas simples (').
    - Los **Wildcard Characters o Operators** ("%", "_", "[]", "^", "-", "{}") se ponen con comillas simples. Estos operadores se ponen con la cláusula LIKE. Para más info: [Vete a este enlace](https://www.w3schools.com/sql/sql_wildcards.asp). Ejemplos de esto son:
```
SELECT *
FROM HumanResources.vEmployee
WHERE FirstName LIKE 'M_%' 
```
```
SELECT *
FROM HumanResources.vEmployee
WHERE FirstName LIKE 'D[a, o]n' 
```
    - Los **Strings** se ponen con comillas simples ('). Por ejemplo en un "Literal SELECT Statement":
```
SELECT 'Cristina Correa'
```
- Se ponen en negrita:
    - Los **números** se ponen como siempre sin comillas y siguen negros pero en negrita
- Cuando hay **espacios** se ponen corchetes ([]) o comillas dobles (") cuando por ejemplo queremos poner alias. Por ejemplo:
```
SELECT TOP 10 
	FirstName AS [Customer First Name], 
  LastName AS [Customer Last Name]
FROM Sales.Customer
```
o
```
SELECT TOP 10 
	FirstName AS "Customer First Name", 
  LastName AS "Customer Last Name"
FROM Sales.Customer
```

## Buenas prácticas
> [!NOTE]
> Es buena práctica separar las cláusulas de sql en diferentes líneas para que se pueda leer mejor. Por ejemplo:
> 
> Esto está bien:
> ```
> SELECT customerName, contactFirstName, contactLastName FROM customers.customers WHERE contactFirstName LIKE 'J%E'
> ```
> Pero es más legible y mejor poner:
> ```
> SELECT customerName, contactFirstName, contactLastName
> FROM customers.customers 
> WHERE contactFirstName 
> LIKE 'J%E'
> ```
> Aunque esto por ejemplo está regulinchi:
>  ```
> SELECT 
> TOP 500 FirstName, MiddleName, LastName
> FROM Person.Person
> ```
> ```
> SELECT 
> TOP 10 
> PERCENT *
> FROM Production.Product
> ```
> Y esto está mucho mejor:
> ```
> SELECT TOP 500 FirstName, MiddleName, LastName
> FROM Person.Person
> ```
> ```
> SELECT TOP 10 PERCENT *
> FROM Production.Product
> ```
> Hay que tener cuidado de cómo hacer bien y legibles las consultas

- Una buena práctica es saber qué cláusulas utilizar para simplificar y hacer más legibles las queries. Por ejemplo
    - Al igual que podemos escribir esto así:
```
SELECT *
FROM HumanResources.vEmployee
WHERE FirstName = 'Chris' OR FirstName = 'Steve' 
	OR FirstName = 'Michael' OR FirstName = 'Thomas'
```
```
SELECT *
FROM Sales.vStoreWithDemographics
WHERE AnnualSales >= 1000000 AND AnnualSales <= 2000000
```
    - Es mucho mejor utilizar cláusulas como estas:
```
SELECT *
FROM HumanResources.vEmployee
WHERE FirstName IN ('Chris', 'Steve', 'Michael', 'Thomas')
```
```
SELECT *
FROM Sales.vStoreWithDemographics
WHERE AnnualSales BETWEEN 1000000 AND 2000000
```

- En vez de utilizar el Wildcard Operator de los corchetes, deberías de usar REGEX. Ya que la performance de la base de datos tiende a empeorar si usamos los corchetes. 
- Sabiendo que un valor NULL y un espacio en blanco son diferentes porque el NULL significa nada, es decir, que no hay un dato específico para ese valor particular, mientras que el espacio en blanco es un valor conocido y puede ser comparado con otros valores de cadena. En SQL, puedes buscar valores que sean espacios en blanco utilizando la cláusula `WHERE [Column Name] = ' '` . Para filtrar los valores NULL podemos hacer lo siguiente: `WHERE [Column Name] IS NULL` o `WHERE [Column Name] IS NOT NULL`. Por ejemplo:
```
SELECT *
FROM Person.Person
WHERE MiddleName IS NULL
```
    - Pero lo siguiente estaría muy mal y no funcionaría:
```
SELECT *
FROM Person.Person
WHERE MiddleName = NULL
```
```
SELECT *
FROM Person.Person
WHERE MiddleName = 'NULL'
```

## Configuración
- Si vamos a **Tools > Options > Text Editor > All Languages** y buscamos la opción **Settings > Line numbers** y hacemos click para marcar esa opción, podemos ver los números a la izquierda de todas las queries que hagamos.

## Queries
- **Abrir el espacio para hacer queries** -> Seleccionamos la base de datos y le damos al botón `New Query`
- **Cambiar la base de datos del espacio para hacer queries** -> Seleccionamos en el desplegable donde aparece la base de datos que estamos utilizando, otra base de datos a la que nos queramos cambiar
- **Ejecutar Query** -> Una vez que hayamos escrito la query, le damos al botón de `Execute` o presionamos `F5`. La query que se ejecuta es la query que estemos seleccionando (highlight) o todo lo que esté en el espacio de editar queries. Si tenemos dos queries en el espacio, seleccionamos una y le damos a ejecutar, solo se ejecutará la query que hayamos seleccionado.


## FORMATOS:
**El formato básico es el siguiente:**
```
SELECT [Column Name 1], [Column Name 2], ..., [Column Name n] 
FROM [Database Name].[Schema Name].[Table Name / View Name] 
WHERE [Column Name] [COMPARISON OPERATOR] [some value]
```
Ej:
```
SELECT NationalIDNumber, JobTitle, HireDate
FROM HumanResources.Employee
WHERE BirthDate <= '1/1/1980' 
```
- Si ya estamos conectados en `New Query` a la base de datos que queremos, no tenemos que especificarla

**El formato más completo es el siguiente:**
```
SELECT [Column Name 1] AS [Some Alias], [Column Name 2], ..., [Column Name n] 
FROM [Database Name].[Schema Name].[Table Name / View Name] 
WHERE [Column Name 1] {comparison operator} [Filtering Criteria]
GROUP BY [Column Name 1], [Column Name 2], ..., [Column Name n] 
HAVING [Aggregate function] {comparison operator} [Filtering Criteria]
ORDER BY [ColumnName / ColumnOrdinal / ColumnAlias] [ASC / DESC]
```
**El formato completo simplificado es:**
```
SELECT 
FROM
WHERE 
GROUP BY
HAVING
ORDER BY 
```
> [!WARNING]
> Aunque las queries que escribimos se hagan de la forma anterior, tenemos que saber que el orden en el que se evalúa el código de SQL es el siguiente: 
> ```
> FROM 
> WHERE 
> GROUP BY 
> HAVING 
> SELECT 
> ORDER BY
> ```
> Es por eso que si hacemos lo siguiente:
> ```
> SELECT FirstName, LastName AS "Last Name"
> FROM Sales.vIndividualCustomer
> WHERE "Last Name" = 'Zheng'
> ```
> Nos da un **error** (*Invalid column name 'Last Name'*) porque el alias de la columna del `SELECT` se evalúa en SQL después del `WHERE` y este último no reconoce al alias. 
> Pero si en vez de poner un alias en el `SELECT`, se lo pones en el `FROM`, entonces sí que se evalúa porque el `FROM` va antes que el `WHERE` a la hora de evaluarse:
> ```
> SELECT FirstName, LastName
> FROM Sales.vIndividualCustomer AS vIC
> WHERE vIC.LastName = 'Zheng'
> ```


> [!NOTE]
> **1-** El Column Ordinal es la posición ordinal de una columna dentro de una tabla o vista. En SQL Server, no es posible seleccionar datos de una columna utilizando su posición ordinal directamente en una consulta `SELECT`. La posición ordinal se puede utilizar en cláusulas como `ORDER BY`. Al hacer queries, el column ordinal se asigna al orden de columna que aparece en la cláusula `SELECT`:
> Ej:
> ```
> SELECT FirstName, LastName
> FROM Sales.vIndividualCustomer
> ORDER BY 2 DESC
> ```
> El column ordinal de FirstName es 1 y el column ordinal de LastName es 2, por lo que aquí lo que estamos haciendo es:
> ```
> SELECT FirstName, LastName
> FROM Sales.vIndividualCustomer
> ORDER BY LastName DESC
> ```
> **2-** Para ordenar con `ORDER BY`, por defecto es de forma ascendente (ASC), y si se quiere ordenar las columnas de varias formas, se tiene que poner entre comas. Si se quiere por ejemplo el apellido de forma ascendente y el nombre de forma descendente, lo que deberíamos de poner es lo siguiente: 
> ```
> SELECT FirstName, LastName
> FROM Sales.vIndividualCustomer
> ORDER BY LastName, FirstName DESC
> ```
> Aquí se ordena el **LastName** de forma ascendente (esta al ser la opción por defecto, sería lo mismo que poner: `ORDER BY LastName ASC, FirstName DESC`) y el **FirstName** de forma descendente.
> - Pero hay que tener en cuenta que el `ORDER BY` **no es conmutativo**, el orden de los factores SÍ que altera el resultado. Esto se puede ver en estos dos ejemplos ya que los resultados que dan no son iguales:
> ```
> SELECT FirstName, LastName
> FROM Sales.vIndividualCustomer
> ORDER BY LastName, FirstName DESC
> ```
> Se ordena primero por orden ascendente los apellidos y luego por orden descendente los nombres dando los siguientes resultados:
> | |FirstName|LastName|
> |-|---------|--------|
> |1| Xavier  |  Adams |
> |2| Wyatt   |  Adams |
> |3| Thomas  |  Adams |
> |4| Sydney  |  Adams |
> 
> ```
> SELECT FirstName, LastName
> FROM Sales.vIndividualCustomer
> ORDER BY FirstName DESC, LastName
> ```
> Se ordena primero por orden descendente los nombres y luego por orden ascendente los apellidos dando los siguientes resultados:
> | |FirstName|LastName|
> |-|---------|--------|
> |1|   Zoe   | Bailey |
> |2|   Zoe   | Bell   |
> |3|   Zoe   | Brooks |
> |4|   Zoe   | Cook   |
>
> **3-** Los alias en las tablas en la cláusula `FROM` son obligatorios cuando se usa cualquier tipo de `JOIN`. Esto es porque si dos tablas tienen un atributo que se llama de la misma manera, habrá un conflicto como el que vamos a poner a continuación:
> 
> Si tenemos dos tablas: La primera **Production.Product** que se ve así:
> | |ProductID| Name    |ProductLine|ProductSubcategoryID|
> |-|---------|---------|-----------|--------------------|
> |1|    1    | Blade   |  NULL     |         1          |
> |2|    2    | Decal 1 |  M        |         6          |
> |3|    3    | Decal 2 |  R        |         6          |
> |4|    4    | Fork End|  NULL     |         3          |
> 
> Y tenemos la segunda tabla llamada **Production.ProductSubcategory** que se ve así:
> 
> | | ProductSubcategoryID | ProductCategoryID |     Name       |
> |-|----------------------|-------------------|----------------|
> |1|          1           |        1          | Mountain Bikes |
> |2|          2           |        1          | Road Bikes     |
> |3|          3           |        1          | Brakes         |
> |4|          4           |        2          | Chains         |
> 
> Si hacemos una query como esta donde no explicamos si queremos el atributo Name de la primera tabla o de la segunda, nos dará un fallo (*Ambiguous column name 'Name'.*):
> ```
> SELECT Name
> FROM Production.Product P
> INNER JOIN Production.ProductSubcategory PS
> ON P.ProductSubcategoryID = PS.ProductSubcategoryID
> ```
> Mientras que si utilizamos los alias en la cláusula `SELECT`, no nos dará ningún fallo:
> ```
> SELECT 
> 	P.Name AS "Product Name", 
> 	P.ProductLine, 
> 	PS.Name AS "Sub Category Name"
> FROM Production.Product P
> INNER JOIN Production.ProductSubcategory PS
> ON P.ProductSubcategoryID = PS.ProductSubcategoryID
> ```
> **4-** Cuando usamos `GROUP BY` normalmente tendremos que utilizar **aggregate functions** como SUM(), COUNT(), MAX() en las Select clause o la Having clause
> 
> **5-** Se puede hacer `GROUP BY` de varias columnas como podemos ver en el siguiente ejemplo donde obtenemos la suma total de las ventas por territorio y por vendedor durante el año 2006, mostrando el nombre del territorio, el nombre del vendedor y el total de ventas para cada combinación de territorio y vendedor, y ordenando los resultados por territorio y luego por vendedor. Acuérdate de que se pueden poner dos parámetros unidos con "+" para formar uno solo como pasa con "FirstName" y "LastName" que forman el "Sales Person Name". Si no se separar con comas (,) cuentan como un único parámetro y por eso hay que añadirles un espacio (" ") para que los nombres en vez de ser "CristinaCorrea" sean "Cristina Correa".
> ```
> SELECT
>   ST.Name AS "Territory Name",
>   P.FirstName + ' ' + P.LastName AS "Sales Person Name",
>   SUM(TotalDue) AS "Total Sales"
> FROM Sales.SalesOrderHeader SOH
> INNER JOIN Sales.SalesPerson SP
> ON SP.BusinessEntityID = SOH.SalesPersonID
> INNER JOIN Person.Person P
> ON P.BusinessEntityID = SP.BusinessEntityID
> INNER JOIN Sales.SalesTerritory ST
> ON ST.TerritoryID = SOH.TerritoryID
> WHERE OrderDate BETWEEN '1/1/2006' AND '12/31/2006'
> GROUP BY ST.Name, P.FirstName + ' ' + P.LastName
> ORDER BY 1, 2
> ```
> El resultado sería:
>
> | | Territory Name | Sales Person Name | Total Sales |
> |-|----------------|-------------------|-------------|
> |1|     Canada     |   Garrett Vargas  |1340859.9994 |
> |2|     Canada     |   Jae Pak         |2842719.9744 |
> |3|     Canada     |   José Saraiva    |1184324.6949 |
> |4|     Central    |   Stephen Jian    |75455.4235   |
>
> **6-** Sabemos que la `HAVING` clause es fundamentalmente similar a la `WHERE` clause, con la diferencia de que mientras `WHERE` filtra filas de datos basándose en los valores de columnas, `HAVING` se utiliza para filtrar grupos basados en sus funciones de agregación (aggregate functions). Podemos ver cómo funciona fácilmente con este ejemplo:
>
> Primero haremos una query donde nos devuelva los territorios y sus ventas totales anuales en el 2006 ordenadas según el nombre del territorio:
> ```
> SELECT
>   ST.Name AS "Territory Name",
>   SUM(TotalDue) AS "Total Sales - 2006"
> FROM Sales.SalesOrderHeader SOH
> INNER JOIN Sales.SalesTerritory ST
> ON ST.TerritoryID = SOH.TerritoryID
> WHERE OrderDate BETWEEN '1/1/2006' AND '12/31/2006'
> GROUP BY ST.Name
> ORDER BY 1
> ```
> Esto nos devolverá 18 filas con los siguientes resultados (solo pongo los 4 primeros):
> | | Territory Name | Total Sales |
> |-|----------------|-------------|
> |1|   Australia    |2380484.8387 |
> |2|   Canada       |6130230.7354 |
> |3|   Central      |2959946.9341 |
> |4|   France       |1535232.8963 |
>
> Ahora gracias al `HAVING` no nos va a devolver todos los territorios y sus ventas totales anuales en el 2006 ordenadas según el nombre del territorio, sino que solo nos va a devolver los territorios que hayan tenido unas ventas superiores a 4 millones:
> ```
> SELECT
>   ST.Name AS "Territory Name",
>   SUM(TotalDue) AS "Total Sales - 2006"
> FROM Sales.SalesOrderHeader SOH
> INNER JOIN Sales.SalesTerritory ST
> ON ST.TerritoryID = SOH.TerritoryID
> WHERE OrderDate BETWEEN '1/1/2006' AND '12/31/2006'
> GROUP BY ST.Name
> HAVING SUM(TotalDue) > 4000000
> ORDER BY 1
> ```
> Esto nos dará 3 filas que representan los únicos territorios con ventas mayores a 4 millones:
> | | Territory Name | Total Sales |
> |-|----------------|-------------|
> |1|   Canada       |6130230.7354 |
> |2|   Northwest    |4855977.1032 |
> |3|   Southwest    |8489320.9825 |
>