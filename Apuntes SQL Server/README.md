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
WHERE [Column Name 1] [COMPARISON OPERATOR] [Filtering Criteria]
GROUP BY
HAVING
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
> FROM Sales.vIndividualCustomer AS "vIC"
> WHERE "vIC".LastName = 'Zheng'
> ```


> [!NOTE]
> El Column Ordinal es la posición ordinal de una columna dentro de una tabla o vista. En SQL Server, no es posible seleccionar datos de una columna utilizando su posición ordinal directamente en una consulta `SELECT`. La posición ordinal se puede utilizar en cláusulas como `ORDER BY`. Al hacer queries, el column ordinal se asigna al orden de columna que aparece en la cláusula `SELECT`:
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