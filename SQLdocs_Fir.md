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

## Week 2: Joins and Aggregations

### What I Learned
- Different types of JOINs
- Aggregation functions
- GROUP BY clauses
- HAVING filters
- Complex joins with multiple tables

### Types of JOINs
```sql
-- INNER JOIN (most common)
SELECT d.nombre, p.importe_inicial
FROM deudores d
INNER JOIN prestamos p ON d.id = p.id_deudor;

-- LEFT JOIN (all debtors, even without loans)
SELECT d.nombre, p.importe_inicial
FROM deudores d
LEFT JOIN prestamos p ON d.id = p.id_deudor;

-- Multiple joins
SELECT 
    d.nombre,
    p.importe_inicial,
    c.telefono1,
    dir.ciudad
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor;
```

### Aggregation Functions
```sql
-- Common aggregations
SELECT 
    estado,
    COUNT(*) as total_prestamos,
    SUM(importe_inicial) as suma_total,
    AVG(importe_inicial) as media,
    MIN(importe_inicial) as minimo,
    MAX(importe_inicial) as maximo
FROM prestamos
GROUP BY estado;

-- With formatting
SELECT 
    estado,
    COUNT(*) as total_prestamos,
    FORMAT(SUM(importe_inicial), 'C', 'es-ES') as suma_total,
    FORMAT(AVG(importe_inicial), 'C', 'es-ES') as media
FROM prestamos
GROUP BY estado;
```

### HAVING Clause
```sql
-- Find cities with many defaulted loans
SELECT 
    dir.ciudad,
    COUNT(*) as total_impagados,
    FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as deuda_total
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
JOIN direcciones dir ON d.id = dir.id_deudor
WHERE p.estado = 'IMPAGADO'
GROUP BY dir.ciudad
HAVING COUNT(*) > 10
ORDER BY total_impagados DESC;
```

### Practice Database Updates
```sql
-- Add contact info table
CREATE TABLE contactos (
    id INT PRIMARY KEY,
    id_deudor INT,
    telefono1 VARCHAR(15),
    telefono2 VARCHAR(15),
    email VARCHAR(100)
);

-- Add address table
CREATE TABLE direcciones (
    id INT PRIMARY KEY,
    id_deudor INT,
    tipo_via VARCHAR(50),
    nombre_via VARCHAR(100),
    numero VARCHAR(10),
    piso VARCHAR(10),
    ciudad VARCHAR(50),
    provincia VARCHAR(50),
    codigo_postal VARCHAR(5)
);

-- Insert sample data
INSERT INTO contactos VALUES 
(1, 1, '666111222', '911222333', 'juan@email.com'),
(2, 2, '666333444', NULL, 'maria@email.com');

INSERT INTO direcciones VALUES
(1, 1, 'Calle', 'Mayor', '10', '2B', 'Madrid', 'Madrid', '28001'),
(2, 2, 'Avenida', 'Libertad', '23', '4C', 'Barcelona', 'Barcelona', '08001');
```

### Sample Complex Queries
```sql
-- Complete debtor profile
SELECT 
    d.dni_nif,
    d.nombre,
    COUNT(p.id) as total_prestamos,
    FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as deuda_total,
    STRING_AGG(c.telefono1, ' / ') as telefonos,
    dir.ciudad
FROM deudores d
LEFT JOIN prestamos p ON d.id = p.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor
GROUP BY d.dni_nif, d.nombre, dir.ciudad
ORDER BY deuda_total DESC;

-- Defaulted loans by city and year
SELECT 
    dir.ciudad,
    YEAR(p.fecha_concesion) as año,
    COUNT(*) as total_impagados,
    FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as importe_total
FROM prestamos p
JOIN deudores d ON p.id_deudor = d.id
JOIN direcciones dir ON d.id = dir.id_deudor
WHERE p.estado = 'IMPAGADO'
GROUP BY dir.ciudad, YEAR(p.fecha_concesion)
ORDER BY dir.ciudad, año DESC;
```

### Key Points to Remember
1. Always check JOIN conditions
2. GROUP BY must include non-aggregated columns
3. HAVING filters after grouping
4. Use LEFT JOIN when you want all records from left table

### SQL Server-Specific Notes
1. Use STRING_AGG for concatenation
2. FORMAT function for currency
3. YEAR() for date parts
4. Multiple JOIN performance considerations

## Week 3: Subqueries and Functions

### What I Learned
- Subqueries in SELECT, FROM, WHERE
- User-Defined Functions
- Common Table Expressions (CTEs)
- Window Functions
- Error Handling

### Subqueries
```sql
-- Find debtors with above-average loans
SELECT d.dni_nif, d.nombre, p.importe_inicial
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
WHERE p.importe_inicial > (
    SELECT AVG(importe_inicial) 
    FROM prestamos
);

-- Find debtors with multiple defaulted loans
SELECT dni_nif, nombre
FROM deudores
WHERE id IN (
    SELECT id_deudor
    FROM prestamos
    WHERE estado = 'IMPAGADO'
    GROUP BY id_deudor
    HAVING COUNT(*) > 1
);
```

### User-Defined Functions
```sql
-- Function to calculate total debt for a client
CREATE FUNCTION dbo.fn_DeudaTotal
    (@id_deudor INT)
RETURNS MONEY
AS
BEGIN
    DECLARE @total MONEY;
    
    SELECT @total = SUM(importe_inicial)
    FROM prestamos
    WHERE id_deudor = @id_deudor;
    
    RETURN ISNULL(@total, 0);
END;
GO

-- Use the function
SELECT 
    d.dni_nif,
    d.nombre,
    dbo.fn_DeudaTotal(d.id) as deuda_total
FROM deudores d;
```

### Common Table Expressions (CTEs)
```sql
-- Complex analysis using CTE
WITH DeudaClientes AS (
    SELECT 
        d.id,
        d.dni_nif,
        d.nombre,
        COUNT(p.id) as total_prestamos,
        SUM(p.importe_inicial) as deuda_total
    FROM deudores d
    JOIN prestamos p ON d.id = p.id_deudor
    GROUP BY d.id, d.dni_nif, d.nombre
),
EstadisticasCiudad AS (
    SELECT 
        dir.ciudad,
        AVG(dc.deuda_total) as deuda_media
    FROM DeudaClientes dc
    JOIN direcciones dir ON dc.id = dir.id_deudor
    GROUP BY dir.ciudad
)
SELECT 
    dc.*,
    dir.ciudad,
    ec.deuda_media as deuda_media_ciudad
FROM DeudaClientes dc
JOIN direcciones dir ON dc.id = dir.id_deudor
JOIN EstadisticasCiudad ec ON dir.ciudad = ec.ciudad
WHERE dc.deuda_total > ec.deuda_media;
```

### Window Functions
```sql
-- Rank debtors by debt amount
SELECT 
    d.dni_nif,
    d.nombre,
    FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as deuda_total,
    RANK() OVER (ORDER BY SUM(p.importe_inicial) DESC) as ranking
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
GROUP BY d.dni_nif, d.nombre;

-- Running totals by date
SELECT 
    fecha_concesion,
    importe_inicial,
    SUM(importe_inicial) OVER (
        ORDER BY fecha_concesion
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as total_acumulado
FROM prestamos;
```

### Error Handling
```sql
-- Safe function with error handling
CREATE FUNCTION dbo.fn_ValidarDNI
    (@dni VARCHAR(9))
RETURNS BIT
AS
BEGIN
    DECLARE @isValid BIT = 0;
    
    BEGIN TRY
        IF @dni LIKE '[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][A-Z]'
            SET @isValid = 1;
    END TRY
    BEGIN CATCH
        SET @isValid = 0;
    END CATCH
    
    RETURN @isValid;
END;
GO

-- Use with error handling
BEGIN TRY
    SELECT * FROM deudores
    WHERE dbo.fn_ValidarDNI(dni_nif) = 1;
END TRY
BEGIN CATCH
    SELECT 
        ERROR_NUMBER() as ErrorNumber,
        ERROR_MESSAGE() as ErrorMessage;
END CATCH;
```

### Key Points to Remember
1. Subqueries can slow performance - use indexes
2. CTEs make complex queries readable
3. Window functions are powerful for analysis
4. Always handle errors in functions

### SQL Server-Specific Notes
1. Use ISNULL() for NULL handling
2. Window functions require ORDER BY
3. CTEs must be followed by a SELECT
4. Error handling is T-SQL specific

## Week 4: Database Design and Optimization

### What I Learned
- Index design and maintenance
- Query optimization
- Database normalization
- Performance monitoring
- Backup strategies

### Index Design
```sql
-- Create index for frequent searches
CREATE INDEX IX_Deudores_DNI 
ON deudores(dni_nif);

-- Create index for loan status
CREATE INDEX IX_Prestamos_Estado 
ON prestamos(estado)
INCLUDE (importe_inicial, fecha_concesion);

-- Check index usage
SELECT 
    OBJECT_NAME(s.object_id) as TableName,
    i.name as IndexName,
    s.user_seeks,
    s.user_scans
FROM sys.dm_db_index_usage_stats s
JOIN sys.indexes i ON s.object_id = i.object_id 
    AND s.index_id = i.index_id
WHERE database_id = DB_ID('YourDatabaseName');
```

### Query Optimization
```sql
-- Before optimization
SELECT d.*, p.*, c.*
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
JOIN contactos c ON d.id = c.id_deudor;

-- After optimization
SELECT 
    d.dni_nif,
    d.nombre,
    p.numero_prestamo,
    p.importe_inicial,
    c.telefono1
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
JOIN contactos c ON d.id = c.id_deudor
WHERE p.estado = 'IMPAGADO';
```

### Performance Monitoring
```sql
-- Check slow queries
SELECT TOP 10
    qs.total_elapsed_time / qs.execution_count as avg_time,
    qs.execution_count,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((qs.statement_end_offset - qs.statement_start_offset)/2)+1) as query
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY avg_time DESC;

-- Check database size
SELECT 
    name,
    size/128.0 as size_mb,
    size/128.0 - CAST(FILEPROPERTY(name, 'SpaceUsed') AS int)/128.0 as free_mb
FROM sys.database_files;
```

### Backup Strategy
```sql
-- Full backup
BACKUP DATABASE YourDatabase
TO DISK = 'C:\Backups\YourDatabase.bak'
WITH INIT;

-- Transaction log backup
BACKUP LOG YourDatabase
TO DISK = 'C:\Backups\YourDatabase_Log.trn';

-- Restore database
RESTORE DATABASE YourDatabase
FROM DISK = 'C:\Backups\YourDatabase.bak'
WITH RECOVERY;
```

### Maintenance Tasks
```sql
-- Update statistics
UPDATE STATISTICS deudores;
UPDATE STATISTICS prestamos;

-- Rebuild fragmented indexes
ALTER INDEX ALL ON deudores REBUILD;
ALTER INDEX ALL ON prestamos REBUILD;

-- Clean up old data
DELETE FROM prestamos 
WHERE estado = 'PAGADO' 
    AND fecha_concesion < DATEADD(YEAR, -10, GETDATE());
```

### Key Points to Remember
1. Index only frequently searched columns
2. Monitor query performance regularly
3. Back up data according to schedule
4. Clean up old data periodically

### SQL Server-Specific Notes
1. Use SQL Server Management Studio for monitoring
2. Schedule maintenance tasks
3. Set up alerts for performance issues
4. Document all optimizations
