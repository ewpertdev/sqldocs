# SQL Learning Journey

This repository documents my journey learning SQL (Microsoft SQL Server), broken down week by week.

## Week 1: SQL Fundamentals

### What I Learned
- Basic SQL query structure
- SELECT statements
- WHERE clauses
- ORDER BY for sorting
- LIKE patterns
- TOP clause (SQL Server's equivalent to LIMIT)
- DISTINCT keyword

### Key Commands 
```sql
SELECT column(s) FROM table;                -- Basic select
SELECT DISTINCT column FROM table;          -- Unique values
SELECT * FROM table WHERE condition;        -- Filtering
SELECT * FROM table ORDER BY column DESC;   -- Sorting
SELECT TOP n * FROM table;                  -- Limiting results (SQL Server syntax)
```

### Pattern Matching
- `%` : Matches any sequence of characters
- `_` : Matches any single character

### Common Operators
- Comparison: `=`, `>`, `<`, `>=`, `<=`, `<>` (not equal in SQL Server)
- Logical: `AND`, `OR`, `NOT`
- Other: `IN`, `BETWEEN`, `LIKE`

### Practice Database
```sql
-- Create the products table
CREATE TABLE products (
    id INT PRIMARY KEY,
    name NVARCHAR(100),
    price DECIMAL(10,2),
    category NVARCHAR(50)
);

-- Insert sample data
INSERT INTO products VALUES 
(1, N'Laptop', 999.99, N'Electronics'),
(2, N'Headphones', 99.99, N'Electronics'),
(3, N'Harry Potter Book', 19.99, N'Books'),
(4, N'Coffee Maker', 79.99, N'Kitchen'),
(5, N'Smartphone', 599.99, N'Electronics'),
(6, N'Smart Watch', 199.99, N'Electronics'),
(7, N'Blender', 49.99, N'Kitchen'),
(8, N'Science Book', 29.99, N'Books'),
(9, N'Tablet', 299.99, N'Electronics'),
(10, N'Toaster', 25.99, N'Kitchen');
```

### Sample Queries to Try
```sql
-- Find all Electronics
SELECT * FROM products 
WHERE category = N'Electronics';

-- Find items under $100
SELECT name, price 
FROM products 
WHERE price < 100 
ORDER BY price;

-- Find unique categories
SELECT DISTINCT category 
FROM products;

-- Find products with 'Smart' in the name
SELECT * FROM products 
WHERE name LIKE N'%Smart%';

-- Find the 3 most expensive products
SELECT TOP 3 * FROM products 
ORDER BY price DESC;

-- Find products between $50 and $200
SELECT * FROM products 
WHERE price BETWEEN 50 AND 200;

-- Find products in multiple categories
SELECT * FROM products 
WHERE category IN (N'Kitchen', N'Books');

-- Find the 5 cheapest products
SELECT TOP 5 * FROM products 
ORDER BY price ASC;

-- Find products NOT in Electronics
SELECT * FROM products 
WHERE category <> N'Electronics';
```

### SQL Server-Specific Notes
1. Use `NVARCHAR` instead of VARCHAR for string columns
2. Use `N` prefix for Unicode string literals (N'text')
3. Use `TOP` instead of LIMIT
4. Use `<>` for not equal (although `!=` also works)
5. Date format is typically 'YYYY-MM-DD'

### Resources Used
- SQL Server Management Studio (SSMS)
- [Microsoft SQL Documentation](https://docs.microsoft.com/en-us/sql/)
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)

### SQL Server Installation Paths
- **SQL Server Management Studio (SSMS):**
  ```
  C:\Program Files (x86)\Microsoft SQL Server Management Studio 19\Common7\IDE\Ssms.exe
  ```

- **SQL Server Default Instance:**
  ```
  C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Binn\sqlservr.exe
  ```

- **SQL Server Configuration Manager:**
  ```
  C:\Windows\SysWOW64\SQLServerManager16.msc
  ```

### Common Data Paths
- **Default Database Location:**
  ```
  C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\DATA
  ```

- **Default Backup Location:**
  ```
  C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup
  ```

Note: The paths might vary depending on:
- SQL Server version (e.g., MSSQL16 for SQL Server 2022)
- Instance name (MSSQLSERVER is the default)
- Custom installation paths

## Semana 1: Fundamentos SQL (continuación)

### Reglas de Validación DNI/NIF Español
- **Personas Físicas:**
  - Solo números (sin letra inicial)
  - X, Y, Z (números NIE para extranjeros)
- **Personas Jurídicas:**
  - Letras A-H: Sociedades (Anónimas, Limitadas, etc.)
  - Letra J: Sociedades Civiles
  - Letra N: Entidades Extranjeras
  - Letra P: Corporaciones Locales
  - Letra Q: Organismos Públicos
  - Letra R: Congregaciones Religiosas
  - Letra S: Órganos de la Administración
  - Letra U: Uniones Temporales de Empresas
  - Letra V: Otros tipos

### Consulta de Ejemplo para Validación de DNI
```sql
-- Crear una tabla para personas/entidades
CREATE TABLE entidades (
    id INT PRIMARY KEY,
    dni_nif NVARCHAR(9),
    nombre NVARCHAR(100),
    tipo_entidad NVARCHAR(50)
);

-- Función para determinar el tipo de entidad según DNI/NIF
CREATE FUNCTION dbo.ObtenerTipoEntidad(@dni_nif NVARCHAR(9))
RETURNS NVARCHAR(50)
AS
BEGIN
    DECLARE @primerCaracter NCHAR(1) = LEFT(@dni_nif, 1)
    DECLARE @tipoEntidad NVARCHAR(50)

    SET @tipoEntidad = 
        CASE 
            WHEN @primerCaracter IN ('X', 'Y', 'Z') THEN 'Persona Física (Extranjero)'
            WHEN @primerCaracter LIKE '[0-9]' THEN 'Persona Física'
            WHEN @primerCaracter LIKE '[A-H,J,N,P,Q,R,S,U,V]' THEN 'Persona Jurídica'
            ELSE 'DNI/NIF No Válido'
        END

    RETURN @tipoEntidad
END;

-- Consulta para clasificar entidades
SELECT 
    dni_nif,
    nombre,
    dbo.ObtenerTipoEntidad(dni_nif) as tipo_entidad
FROM entidades;
```

### Contexto de NPL (Créditos Morosos)
- Los NPL son préstamos que están en situación de impago o cerca de estarlo
- Comunes en el sector bancario y de cobro de deudas
- Es importante realizar seguimiento tanto de deudores individuales (personas físicas) como de empresas deudoras (personas jurídicas)

### Procedimientos Almacenados (Stored Procedures)
```sql
-- Ejemplo básico de Stored Procedure para crear una tabla
CREATE PROCEDURE sp_CrearTablaEntidades
AS
BEGIN
    -- Verificar si la tabla existe y borrarla si es necesario
    IF OBJECT_ID('entidades', 'U') IS NOT NULL
        DROP TABLE entidades;

    -- Crear la tabla
    CREATE TABLE entidades (
        id INT IDENTITY(1,1) PRIMARY KEY,
        dni_nif NVARCHAR(9),
        nombre NVARCHAR(100),
        tipo_entidad NVARCHAR(50)
    );
END;

-- Ejecutar el procedimiento
EXEC sp_CrearTablaEntidades;

-- Stored Procedure para insertar datos
CREATE PROCEDURE sp_InsertarEntidad
    @dni_nif NVARCHAR(9),
    @nombre NVARCHAR(100)
AS
BEGIN
    PRINT 'Iniciando inserción para DNI: ' + @dni_nif;
    
    INSERT INTO entidades (dni_nif, nombre, tipo_entidad)
    VALUES (@dni_nif, @nombre, dbo.ObtenerTipoEntidad(@dni_nif));
    
    PRINT 'Inserción completada';
END;
```

### Exportar Datos a CSV/Excel
```sql
-- Exportar a CSV usando BCP (Command Line)
-- Abrir CMD y ejecutar:
bcp "SELECT * FROM BaseDeDatos.dbo.entidades" queryout "C:\ruta\archivo.csv" -c -t"," -T -S nombreServidor

-- Exportar usando SSMS:
-- Click derecho en la base de datos > Tasks > Export Data
-- O usar esta consulta para generar CSV:
SELECT 
    dni_nif + ',' +
    nombre + ',' +
    tipo_entidad
FROM entidades
FOR XML PATH(''), TYPE;

-- También puedes usar SSIS (SQL Server Integration Services)
```

### Importar desde Excel/CSV
```sql
-- Usando BULK INSERT
BULK INSERT entidades
FROM 'C:\ruta\archivo.csv'
WITH (
    FIRSTROW = 2,              -- Saltar encabezados
    FIELDTERMINATOR = ',',     -- Delimitador de campos
    ROWTERMINATOR = '\n',      -- Delimitador de filas
    CODEPAGE = '65001'         -- UTF-8
);

-- Alternativa usando OPENROWSET
INSERT INTO entidades
SELECT *
FROM OPENROWSET(
    'Microsoft.ACE.OLEDB.12.0',
    'Excel 12.0;Database=C:\ruta\archivo.xlsx;HDR=YES',
    'SELECT * FROM [Hoja1$]'
);
```

### Tips Importantes
1. **Stored Procedures:**
   - Son más seguros que queries directas
   - Pueden ser reutilizados
   - Mejoran el rendimiento
   - Permiten control de transacciones

2. **Exportación/Importación:**
   - Siempre haz backup antes de importar
   - Verifica la codificación de caracteres (UTF-8)
   - Rev

### Ejecutar un SP
EXEC sp_CrearTablaEntidades;

### Ejecutar con parámetros
EXEC sp_InsertarEntidad 
    @dni_nif = '12345678A',
    @nombre = 'Juan Pérez';

### Manipulación de Texto en SQL Server
```sql
-- Ejemplos con DNI/NIF '12345678A'

-- Obtener caracteres desde la izquierda
SELECT LEFT(@dni_nif, 1)      -- Devuelve '1'
SELECT LEFT(@dni_nif, 2)      -- Devuelve '12'

-- Obtener caracteres desde la derecha
SELECT RIGHT(@dni_nif, 1)     -- Devuelve 'A'
SELECT RIGHT(@dni_nif, 2)     -- Devuelve '8A'

-- Obtener caracteres desde una posición específica
SELECT SUBSTRING(@dni_nif, 2, 1)   -- Devuelve '2' (2ª posición)
SELECT SUBSTRING(@dni_nif, 3, 1)   -- Devuelve '3' (3ª posición)
SELECT SUBSTRING(@dni_nif, 2, 3)   -- Devuelve '234' (desde 2ª posición, 3 caracteres)
```

### Nota Importante sobre Posiciones en SQL
- Las posiciones en SQL empiezan en 1, NO en 0 como en otros lenguajes de programación
- Sintaxis de SUBSTRING:
  ```sql
  SUBSTRING(texto, inicio, longitud)
  -- inicio: desde qué posición empezar (1 = primer carácter)
  -- longitud: cuántos caracteres tomar
  ```

### Funciones Útiles para Texto
- `LEFT(texto, n)`: Obtiene los primeros n caracteres
- `RIGHT(texto, n)`: Obtiene los últimos n caracteres
- `SUBSTRING(texto, inicio, longitud)`: Obtiene caracteres desde una posición específica
- `CHARINDEX(buscar, texto)`: Encuentra la posición de un carácter
- `LEN(texto)`: Obtiene la longitud del texto

### Insertar Datos de Ejemplo
```sql
-- Stored Procedure para insertar datos de ejemplo
CREATE PROCEDURE sp_InsertarDatosEjemplo
AS
BEGIN
    -- Limpiar tabla si es necesario
    -- TRUNCATE TABLE entidades;  -- Opcional: elimina registros existentes

    -- Insertar Personas Físicas (Natural Persons)
    INSERT INTO entidades (dni_nif, nombre, tipo_entidad)
    VALUES 
        ('12345678A', 'Juan Pérez García', 'Persona Física'),
        ('87654321B', 'María López Ruiz', 'Persona Física'),
        ('X1234567L', 'John Smith', 'Persona Física (Extranjero)'),
        ('Y7654321H', 'Marie Dubois', 'Persona Física (Extranjero)');

    -- Insertar Personas Jurídicas (Legal Entities)
    INSERT INTO entidades (dni_nif, nombre, tipo_entidad)
    VALUES
        ('A12345678', 'Empresa Tecnológica SA', 'Persona Jurídica'),
        ('B87654321', 'Consultora Digital SL', 'Persona Jurídica'),
        ('P1234567H', 'Ayuntamiento de Madrid', 'Persona Jurídica'),
        ('Q7654321B', 'Organismo Público Estatal', 'Persona Jurídica');
END;

-- Ejecutar el procedimiento
EXEC sp_InsertarDatosEjemplo;

-- Verificar los datos insertados
SELECT * FROM entidades;
```

### Tipos de Identificación
1. **Personas Físicas (Natural Persons)**
   - DNI regular: `12345678A` (8 números + letra)
   - NIE extranjeros: `X1234567L`, `Y7654321H` (X/Y/Z + 7 números + letra)

2. **Personas Jurídicas (Legal Entities)**
   - `A-H`: Sociedades (ej: `A12345678` para SA)
   - `B`: Sociedades Limitadas (ej: `B87654321` para SL)
   - `P`: Corporaciones Locales
   - `Q`: Organismos Públicos
   - etc.

### Verificar Datos Insertados
```sql
-- Ver todos los registros
SELECT * FROM entidades;

-- Ver por tipo
SELECT * FROM entidades WHERE tipo_entidad = 'Persona Física';
SELECT * FROM entidades WHERE tipo_entidad = 'Persona Jurídica';

-- Contar por tipo
SELECT tipo_entidad, COUNT(*) as total
FROM entidades
GROUP BY tipo_entidad;
```

## Next Steps
- Week 2: Joins and Aggregations
- Week 3: Subqueries and Functions
- Week 4: Database Design and Optimization