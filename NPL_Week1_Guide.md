# NPL SQL Week 1: Fundamentals Guide 📚

## Day 1: Database Setup

### Create Base Tables
```sql
-- Create entities table
CREATE TABLE entidades (
    id INT IDENTITY(1,1) PRIMARY KEY,
    dni_nif NVARCHAR(9),
    nombre NVARCHAR(100),
    tipo_entidad NVARCHAR(50)
);

-- Create debts table
CREATE TABLE deudas (
    id INT IDENTITY(1,1) PRIMARY KEY,
    numero_contrato NVARCHAR(50),
    importe_deuda MONEY,
    estado NVARCHAR(20),
    fecha_alta DATE,
    id_entidad INT,
    FOREIGN KEY (id_entidad) REFERENCES entidades(id)
);
```

### Core Functions
```sql
-- Entity type detection from DNI/NIF
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
GO
```

### Basic Procedures
```sql
-- Insert new entity
CREATE PROCEDURE sp_InsertarEntidad
    @dni_nif NVARCHAR(9),
    @nombre NVARCHAR(100)
AS
BEGIN
    INSERT INTO entidades (dni_nif, nombre, tipo_entidad)
    VALUES (@dni_nif, @nombre, dbo.ObtenerTipoEntidad(@dni_nif));
END;
GO

-- Insert new debt
CREATE PROCEDURE sp_InsertarDeuda
    @numero_contrato NVARCHAR(50),
    @importe_deuda MONEY,
    @dni_nif NVARCHAR(9)
AS
BEGIN
    DECLARE @id_entidad INT

    SELECT @id_entidad = id 
    FROM entidades 
    WHERE dni_nif = @dni_nif;

    INSERT INTO deudas (
        numero_contrato, 
        importe_deuda, 
        estado,
        fecha_alta,
        id_entidad
    )
    VALUES (
        @numero_contrato, 
        @importe_deuda, 
        'PENDIENTE',
        GETDATE(),
        @id_entidad
    );
END;
GO
```

### Sample Data
```sql
-- Insert sample entities
EXEC sp_InsertarEntidad '12345678A', 'Juan Pérez';
EXEC sp_InsertarEntidad 'B12345678', 'Empresa ABC SL';
EXEC sp_InsertarEntidad 'X1234567L', 'John Smith';

-- Insert sample debts
EXEC sp_InsertarDeuda 'LOAN001', 5000.00, '12345678A';
EXEC sp_InsertarDeuda 'LOAN002', 15000.00, 'B12345678';
EXEC sp_InsertarDeuda 'LOAN003', 7500.00, 'X1234567L';
```

### Basic Queries
```sql
-- Find client by DNI
SELECT dni_nif, nombre 
FROM entidades 
WHERE dni_nif = '12345678A';

-- Find all debts for a client
SELECT 
    e.dni_nif,
    e.nombre,
    d.numero_contrato,
    FORMAT(d.importe_deuda, 'C', 'es-ES') as importe,
    d.estado
FROM entidades e
JOIN deudas d ON e.id = d.id_entidad
WHERE e.dni_nif = '12345678A';

-- Summary by entity type
SELECT 
    tipo_entidad,
    COUNT(*) as total_entidades,
    FORMAT(SUM(d.importe_deuda), 'C', 'es-ES') as deuda_total
FROM entidades e
JOIN deudas d ON e.id = d.id_entidad
GROUP BY tipo_entidad;
```

## Key Points to Remember
1. Always validate DNI/NIF format
2. Use proper data types (MONEY for amounts)
3. Maintain referential integrity
4. Document procedures

## Common Mistakes to Avoid
1. Forgetting foreign key constraints
2. Not handling NULL values
3. Missing transaction control
4. Incorrect date formats

## Day 1-2: Basic Queries

### Essential Searches
```sql
-- Find client by DNI
SELECT dni_nif, nombre FROM deudores WHERE dni_nif = '12345678A';

-- Find client by name
SELECT nombre, dni_nif FROM deudores WHERE nombre LIKE '%García%';
```

### Basic Joins
```sql
-- Client with their loans
SELECT 
    d.dni_nif,
    d.nombre,
    p.numero_prestamo,
    FORMAT(p.importe_inicial, 'C', 'es-ES') as importe
FROM deudores d                               -- Start with debtors
JOIN prestamos p ON d.id = p.id_deudor;      -- Connect with their loans
```

## Day 3-4: Reports & Counting

### Daily Summary
```sql
SELECT 
    estado,
    COUNT(*) as total,
    FORMAT(SUM(importe_inicial), 'C', 'es-ES') as importe_total
FROM prestamos
GROUP BY estado;
```

## Day 5: Review & Practice

### Key Concepts:
1. Basic SELECT syntax
2. WHERE conditions
3. JOINs
4. GROUP BY
5. Formatting numbers

### Add birth date to entities table
```sql
ALTER TABLE entidades
ADD fecha_nacimiento DATE;
```

### Update sample data with birth dates
```sql
UPDATE entidades
SET fecha_nacimiento = DATEADD(YEAR, -30, GETDATE())
WHERE dni_nif = '12345678A';
```

### Find people over 20
```sql
CREATE PROCEDURE sp_BuscarMayores20
AS
BEGIN
    SELECT 
        dni_nif,
        nombre,
        fecha_nacimiento,
        DATEDIFF(YEAR, fecha_nacimiento, GETDATE()) as edad
    FROM entidades
    WHERE 
        tipo_entidad LIKE 'Persona Física%'
        AND DATEDIFF(YEAR, fecha_nacimiento, GETDATE()) > 20;
END;
GO
```

### Delete people over 20
```sql
CREATE PROCEDURE sp_EliminarMayores20
AS
BEGIN
    BEGIN TRANSACTION;
    
    -- First delete their debts (due to foreign key)
    DELETE d
    FROM deudas d
    JOIN entidades e ON d.id_entidad = e.id
    WHERE 
        e.tipo_entidad LIKE 'Persona Física%'
        AND DATEDIFF(YEAR, e.fecha_nacimiento, GETDATE()) > 20;
    
    -- Then delete the entities
    DELETE FROM entidades
    WHERE 
        tipo_entidad LIKE 'Persona Física%'
        AND DATEDIFF(YEAR, fecha_nacimiento, GETDATE()) > 20;
    
    COMMIT;
END;
GO

### Execute procedures
EXEC sp_BuscarMayores20;
EXEC sp_EliminarMayores20;
