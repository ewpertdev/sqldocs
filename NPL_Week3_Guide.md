# NPL SQL Week 3: Subqueries and Functions 🔍

## Day 1-2: Subqueries

### Basic Subqueries
```sql
-- Find debtors with above-average loans
SELECT 
    d.dni_nif,
    d.nombre,
    p.importe_inicial
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
WHERE p.importe_inicial > (
    SELECT AVG(importe_inicial) 
    FROM prestamos
);

-- Find debtors with multiple defaults
SELECT 
    d.dni_nif,
    d.nombre,
    COUNT(*) as total_impagados
FROM deudores d
WHERE d.id IN (
    SELECT id_deudor
    FROM prestamos
    WHERE estado = 'IMPAGADO'
    GROUP BY id_deudor
    HAVING COUNT(*) > 1
);
```

## Day 3-4: Functions

### Custom Functions
```sql
-- Calculate total debt for a client
CREATE FUNCTION fn_DeudaTotal
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

-- Validate Spanish DNI/NIE
CREATE FUNCTION fn_ValidarDNI
    (@dni_nif VARCHAR(9))
RETURNS BIT
AS
BEGIN
    RETURN 
        CASE 
            WHEN @dni_nif LIKE '[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][A-Z]' THEN 1
            WHEN @dni_nif LIKE '[XYZ][0-9][0-9][0-9][0-9][0-9][0-9][0-9][A-Z]' THEN 1
            ELSE 0
        END;
END;
GO
```

## Day 5: Common Table Expressions (CTEs)

### Complex Analysis
```sql
-- Portfolio analysis with CTEs
WITH DeudaClientes AS (
    SELECT 
        d.id,
        d.dni_nif,
        d.nombre,
        COUNT(*) as total_prestamos,
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
    ec.deuda_media,
    CASE 
        WHEN dc.deuda_total > ec.deuda_media THEN 'Alto Riesgo'
        ELSE 'Riesgo Normal'
    END as nivel_riesgo
FROM DeudaClientes dc
JOIN direcciones dir ON dc.id = dir.id_deudor
JOIN EstadisticasCiudad ec ON dir.ciudad = ec.ciudad;
```

### Key Concepts:
1. Subqueries in WHERE, FROM, SELECT
2. User-defined functions
3. CTEs for complex analysis
4. Error handling
5. Data validation

### Common Mistakes to Avoid:
1. Uncorrelated vs correlated subqueries
2. Function performance impact
3. CTE limitations
4. Proper error handling
</rewritten_file> 