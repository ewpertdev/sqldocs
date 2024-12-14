# NPL Asset Management Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Database Structure](#database-structure)
3. [Core Functions](#core-functions)
4. [Common Tasks](#common-tasks)
5. [Reporting](#reporting)
6. [Troubleshooting](#troubleshooting)
7. [Understanding JOINs in Detail](#understanding-joins-in-detail)

## Introduction
This documentation covers the SQL procedures and queries used for NPL (Non-Performing Loans) asset management.

### Purpose
- Manage debtor information
- Track loan amounts
- Generate reports
- Analyze portfolio data

## Database Structure

### Tables

#### Entidades (Entities)
```sql
CREATE TABLE entidades (
    id INT IDENTITY(1,1) PRIMARY KEY,
    dni_nif NVARCHAR(9),
    nombre NVARCHAR(100),
    tipo_entidad NVARCHAR(50)
);
```

#### Deudas (Debts)
```sql
CREATE TABLE deudas (
    id INT IDENTITY(1,1) PRIMARY KEY,
    numero_contrato NVARCHAR(50),
    importe_deuda MONEY,
    id_entidad INT FOREIGN KEY REFERENCES entidades(id)
);
```

## Core Functions

### Entity Type Detection
The system automatically detects entity types from DNI/NIF patterns:
```sql
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
```

### Core Procedures

#### Creating Tables
```sql
CREATE PROCEDURE sp_CrearTablaEntidades
AS
BEGIN
    IF OBJECT_ID('entidades', 'U') IS NOT NULL
        DROP TABLE entidades;
    
    CREATE TABLE entidades (
        id INT IDENTITY(1,1) PRIMARY KEY,
        dni_nif NVARCHAR(9),
        nombre NVARCHAR(100),
        tipo_entidad NVARCHAR(50)
    );
END;

CREATE PROCEDURE sp_CrearTablaDeudas
AS
BEGIN
    IF OBJECT_ID('deudas', 'U') IS NOT NULL
        DROP TABLE deudas;
    
    CREATE TABLE deudas (
        id INT IDENTITY(1,1) PRIMARY KEY,
        numero_contrato NVARCHAR(50),
        importe_deuda MONEY,
        id_entidad INT,
        FOREIGN KEY (id_entidad) REFERENCES entidades(id)
    );
END;
```

#### Data Entry
```sql
CREATE PROCEDURE sp_InsertarEntidad
    @dni_nif NVARCHAR(9),
    @nombre NVARCHAR(100)
AS
BEGIN
    INSERT INTO entidades (dni_nif, nombre, tipo_entidad)
    VALUES (@dni_nif, @nombre, dbo.ObtenerTipoEntidad(@dni_nif));
END;

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

    INSERT INTO deudas (numero_contrato, importe_deuda, id_entidad)
    VALUES (@numero_contrato, @importe_deuda, @id_entidad);
END;
```

## Common Tasks

### 1. Adding New Entities and Debts
```sql
-- Add new entity
EXEC sp_InsertarEntidad '12345678A', 'Juan Pérez';

-- Add debt for entity
EXEC sp_InsertarDeuda 'CONT-001', 5000.00, '12345678A';
```

### 2. Viewing Portfolio Data
```sql
-- View total debt by entity
SELECT 
    e.nombre AS 'Nombre',
    e.tipo_entidad AS 'Tipo',
    COUNT(d.id) AS 'Número de Préstamos',
    CONCAT('€ ', FORMAT(SUM(d.importe_deuda), 'N2')) AS 'Deuda Total'
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
GROUP BY e.nombre, e.tipo_entidad
ORDER BY SUM(d.importe_deuda) DESC;
```

### Excel Export Format
```sql
-- Excel-friendly format
SELECT 
    d.dni_nif as 'DNI/NIF',          -- Spanish ID
    d.nombre as 'Nombre',            -- Name in Spanish
    FORMAT(p.importe_inicial, 'N2') as 'Importe',  -- 2 decimal places
    FORMAT(p.fecha_concesion, 'dd/MM/yyyy') as 'Fecha'  -- Spanish date format
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor;
```

### Find specific loans
```sql
-- Find specific loans
SELECT d.dni_nif, d.nombre, p.importe_inicial
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
WHERE p.estado = 'IMPAGADO'              -- Defaulted loans
    AND p.importe_inicial > 50000        -- Big loans
    AND d.dni_nif LIKE '123%';          -- Specific client group
```

## Reporting

### 1. Portfolio Summary Reports
```sql
-- Overall Portfolio Summary
SELECT 
    tipo_entidad AS 'Tipo de Entidad',
    COUNT(DISTINCT e.id) AS 'Total Deudores',
    COUNT(d.id) AS 'Total Préstamos',
    CONCAT('€ ', FORMAT(SUM(d.importe_deuda), 'N2')) AS 'Deuda Total',
    CONCAT('€ ', FORMAT(AVG(d.importe_deuda), 'N2')) AS 'Deuda Media'
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
GROUP BY tipo_entidad;
```

### 2. Debt Range Analysis
```sql
-- Distribution of debts by amount ranges
WITH DebtRanges AS (
    SELECT 
        e.nombre,
        e.tipo_entidad,
        d.importe_deuda,
        CASE 
            WHEN d.importe_deuda <= 10000 THEN '0-10K'
            WHEN d.importe_deuda <= 50000 THEN '10K-50K'
            WHEN d.importe_deuda <= 100000 THEN '50K-100K'
            ELSE 'Over 100K'
        END AS range_category
    FROM entidades e
    JOIN deudas d ON e.id = d.id_entidad
)
SELECT 
    range_category AS 'Rango',
    COUNT(*) AS 'Número de Préstamos',
    CONCAT('€ ', FORMAT(SUM(importe_deuda), 'N2')) AS 'Suma Total',
    CONCAT('€ ', FORMAT(AVG(importe_deuda), 'N2')) AS 'Media'
FROM DebtRanges
GROUP BY range_category
ORDER BY 
    CASE range_category
        WHEN '0-10K' THEN 1
        WHEN '10K-50K' THEN 2
        WHEN '50K-100K' THEN 3
        ELSE 4
    END;
```

### 3. Top Debtors Report
```sql
-- Top 10 debtors by total debt
SELECT TOP 10
    e.nombre AS 'Deudor',
    e.tipo_entidad AS 'Tipo',
    COUNT(d.id) AS 'Número de Préstamos',
    CONCAT('€ ', FORMAT(SUM(d.importe_deuda), 'N2')) AS 'Deuda Total',
    CONCAT('€ ', FORMAT(MAX(d.importe_deuda), 'N2')) AS 'Préstamo Mayor'
FROM entidades e
JOIN deudas d ON e.id = d.id_entidad
GROUP BY e.nombre, e.tipo_entidad
ORDER BY SUM(d.importe_deuda) DESC;
```

### 4. Data Validation Report
```sql
-- Check for data quality issues
SELECT 
    'Entidades sin deudas' AS 'Tipo',
    COUNT(*) AS 'Cantidad'
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
WHERE d.id IS NULL

UNION ALL

SELECT 
    'DNI/NIF inválidos',
    COUNT(*)
FROM entidades
WHERE tipo_entidad = 'DNI/NIF No Válido'

UNION ALL

SELECT 
    'Deudas sobre €100K',
    COUNT(*)
FROM deudas
WHERE importe_deuda > 100000;
```

### 5. Export Format
```sql
-- Export-friendly format for Excel
SELECT 
    e.dni_nif AS 'DNI/NIF',
    e.nombre AS 'Nombre',
    e.tipo_entidad AS 'Tipo',
    d.numero_contrato AS 'Contrato',
    FORMAT(d.importe_deuda, 'N2') AS 'Importe'
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
ORDER BY e.nombre, d.importe_deuda DESC;
```

## Understanding JOINs in Detail

### Basic JOIN Concept
A JOIN connects two tables based on related columns. In our case:
- `entidades` table has an `id` column
- `deudas` table has an `id_entidad` column that references `entidades.id`

### Visual Example
```
Table entidades:                    Table deudas:
id | dni_nif    | nombre           id | numero_contrato | id_entidad
---+------------+--------          ---+----------------+-----------
1  | 12345678A  | Juan            1  | CONT-001       | 1
2  | X1234567L  | Marie           2  | CONT-002       | 1
3  | A12345678  | Empresa         3  | CONT-003       | 2
                                  4  | CONT-004       | 1
```

### Types of JOINs Explained

#### 1. INNER JOIN
```sql
-- Shows only entities that have debts
SELECT e.nombre, d.importe_deuda
FROM entidades e
INNER JOIN deudas d ON e.id = d.id_entidad;

Result:
nombre    | importe_deuda
----------+--------------
Juan      | 5000         -- from CONT-001
Juan      | 3000         -- from CONT-002
Juan      | 1000         -- from CONT-004
Marie     | 2000         -- from CONT-003
```

#### 2. LEFT JOIN (What we use most)
```sql
-- Shows ALL entities, even those without debts
SELECT e.nombre, d.importe_deuda
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad;

Result:
nombre    | importe_deuda
----------+--------------
Juan      | 5000
Juan      | 3000
Juan      | 1000
Marie     | 2000
Empresa   | NULL         -- Included despite no debts!
```

### Step-by-Step JOIN Construction

1. **Start with basic entity selection:**
```sql
SELECT e.nombre
FROM entidades e;
```

2. **Add the JOIN clause:**
```sql
SELECT e.nombre
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad;
```

3. **Add debt information:**
```sql
SELECT 
    e.nombre,
    d.importe_deuda
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad;
```

4. **Add grouping for totals:**
```sql
SELECT 
    e.nombre,
    COUNT(d.id) AS 'Número de Préstamos',
    SUM(d.importe_deuda) AS 'Deuda Total'
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
GROUP BY e.nombre;
```

5. **Add formatting and ordering:**
```sql
SELECT 
    e.nombre AS 'Nombre',
    COUNT(d.id) AS 'Número de Préstamos',
    CONCAT('€ ', FORMAT(SUM(d.importe_deuda), 'N2')) AS 'Deuda Total'
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
GROUP BY e.nombre
ORDER BY SUM(d.importe_deuda) DESC;
```

### Common JOIN Patterns

#### Multiple Conditions
```sql
-- Join with multiple conditions
SELECT e.nombre, d.importe_deuda
FROM entidades e
LEFT JOIN deudas d ON 
    e.id = d.id_entidad 
    AND d.importe_deuda > 1000;
```

#### Filtering After JOIN
```sql
-- First JOIN, then filter
SELECT e.nombre, d.importe_deuda
FROM entidades e
LEFT JOIN deudas d ON e.id = d.id_entidad
WHERE d.importe_deuda > 1000;
```

### Best Practices for JOINs
1. Always use table aliases (e.g., 'e' for entidades)
2. Specify the join type explicitly (LEFT, INNER, etc.)
3. Use meaningful column names in results
4. Consider performance with large datasets
5. Use appropriate indexes on JOIN columns

## Portfolio Analysis

### Loan Status and Year
```sql
-- Group by loan status and year
SELECT 
    estado,
    YEAR(fecha_concesion) as año,
    COUNT(*) as total,
    FORMAT(SUM(importe_inicial), 'C', 'es-ES') as importe
FROM prestamos
GROUP BY estado, YEAR(fecha_concesion)
ORDER BY año DESC, estado;
```
