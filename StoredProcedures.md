# NPL Stored Procedures Guide 📚

## Table of Contents
1. [Week 1: Basic Procedures](#week-1-basic-procedures)
2. [Week 2: Data Quality](#week-2-data-quality)
3. [Week 3: Complex Reports](#week-3-complex-reports)
4. [Week 4: Integration](#week-4-integration)

## Week 1: Basic Procedures

### Client Search
```sql
CREATE PROCEDURE sp_BuscarCliente
    @busqueda VARCHAR(50)
AS
BEGIN
    SELECT 
        d.dni_nif,
        d.nombre,
        p.numero_prestamo,
        FORMAT(p.importe_inicial, 'C', 'es-ES') as importe,
        p.estado
    FROM deudores d
    LEFT JOIN prestamos p ON d.id = p.id_deudor
    WHERE 
        d.dni_nif LIKE '%' + @busqueda + '%'
        OR d.nombre LIKE '%' + @busqueda + '%';
END
GO

-- Use: EXEC sp_BuscarCliente @busqueda = 'García'
```

### Daily Summary
```sql
CREATE PROCEDURE sp_ResumenDiario
AS
BEGIN
    SELECT 
        COUNT(DISTINCT d.id) as total_deudores,
        FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as deuda_total,
        COUNT(CASE WHEN p.estado = 'IMPAGADO' THEN 1 END) as impagados
    FROM deudores d
    JOIN prestamos p ON d.id = p.id_deudor;
END
GO

-- Use: EXEC sp_ResumenDiario
```

## Week 2: Data Quality

### Validate Data
```sql
CREATE PROCEDURE sp_ValidarDatos
AS
BEGIN
    -- Check DNIs
    SELECT 'Invalid DNIs' as issue, COUNT(*) as total
    FROM deudores
    WHERE dni_nif NOT LIKE '[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][A-Z]';

    -- Check phones
    SELECT 'Invalid Phones' as issue, COUNT(*) as total
    FROM contactos
    WHERE telefono1 NOT LIKE '6%' 
        AND telefono1 NOT LIKE '9%'
        OR LEN(telefono1) != 9;
END
GO

-- Use: EXEC sp_ValidarDatos
```

### Clean Phone Numbers
```sql
CREATE PROCEDURE sp_LimpiarTelefonos
AS
BEGIN
    -- First, report what will be changed
    SELECT 
        'Before Cleaning' as stage,
        COUNT(*) as total_invalid
    FROM contactos
    WHERE telefono1 NOT LIKE '6%' 
        AND telefono1 NOT LIKE '9%'
        OR LEN(telefono1) != 9;

    -- Clean phone numbers
    UPDATE contactos
    SET telefono1 = 
        CASE 
            WHEN telefono1 LIKE '34%' THEN RIGHT(telefono1, 9)  -- Remove country code
            WHEN LEN(telefono1) != 9 THEN NULL                  -- Clear invalid numbers
            ELSE telefono1                                      -- Keep good numbers
        END;

    -- Report results
    SELECT 
        'After Cleaning' as stage,
        COUNT(*) as total_invalid
    FROM contactos
    WHERE telefono1 NOT LIKE '6%' 
        AND telefono1 NOT LIKE '9%'
        OR LEN(telefono1) != 9;
END
GO

-- Use: EXEC sp_LimpiarTelefonos
```

## Week 3: Complex Reports

### Portfolio Analysis
```sql
CREATE PROCEDURE sp_AnalisisCartera
    @fecha_inicio DATE = NULL,
    @fecha_fin DATE = NULL
AS
BEGIN
    SET @fecha_inicio = ISNULL(@fecha_inicio, DATEADD(MONTH, -1, GETDATE()));
    SET @fecha_fin = ISNULL(@fecha_fin, GETDATE());

    SELECT 
        estado,
        COUNT(*) as total_prestamos,
        FORMAT(SUM(importe_inicial), 'C', 'es-ES') as importe_total,
        FORMAT(AVG(importe_inicial), 'C', 'es-ES') as importe_medio
    FROM prestamos
    WHERE fecha_concesion BETWEEN @fecha_inicio AND @fecha_fin
    GROUP BY estado;
END
GO

-- Use: EXEC sp_AnalisisCartera '2024-01-01', '2024-01-31'
```

### Monthly Report
```sql
CREATE PROCEDURE sp_ReporteMensual
    @año INT = NULL,
    @mes INT = NULL
AS
BEGIN
    -- Use current year/month if not specified
    SET @año = ISNULL(@año, YEAR(GETDATE()));
    SET @mes = ISNULL(@mes, MONTH(GETDATE()));

    -- Monthly summary
    SELECT 
        'Monthly Overview' as report_type,
        COUNT(DISTINCT d.id) as total_deudores,
        COUNT(*) as total_prestamos,
        FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as cartera_total,
        FORMAT(AVG(p.importe_inicial), 'C', 'es-ES') as prestamo_medio
    FROM deudores d
    JOIN prestamos p ON d.id = p.id_deudor
    WHERE YEAR(p.fecha_concesion) = @año
        AND MONTH(p.fecha_concesion) = @mes;

    -- Status breakdown
    SELECT 
        estado,
        COUNT(*) as total,
        FORMAT(SUM(importe_inicial), 'C', 'es-ES') as importe_total
    FROM prestamos
    WHERE YEAR(fecha_concesion) = @año
        AND MONTH(fecha_concesion) = @mes
    GROUP BY estado;
END
GO

-- Use: EXEC sp_ReporteMensual 2024, 1  -- January 2024
```

## Week 4: Integration

### Export to Excel
```sql
CREATE PROCEDURE sp_ExportarExcel
    @estado VARCHAR(20) = NULL
AS
BEGIN
    SELECT 
        d.dni_nif as 'DNI/NIF',
        d.nombre as 'Nombre',
        FORMAT(p.importe_inicial, 'N2') as 'Importe',
        p.estado as 'Estado',
        FORMAT(p.fecha_concesion, 'dd/MM/yyyy') as 'Fecha'
    FROM deudores d
    JOIN prestamos p ON d.id = p.id_deudor
    WHERE p.estado = @estado
        OR @estado IS NULL
    ORDER BY p.fecha_concesion DESC;
END
GO

-- Use: EXEC sp_ExportarExcel @estado = 'IMPAGADO'
```

### Import Payments
```sql
CREATE PROCEDURE sp_ImportarPagos
    @fecha_proceso DATE = NULL
AS
BEGIN
    -- Use today if no date provided
    SET @fecha_proceso = ISNULL(@fecha_proceso, GETDATE());

    -- Create temp table for validation
    CREATE TABLE #PagosTmp (
        numero_prestamo VARCHAR(50),
        importe MONEY,
        fecha_pago DATE,
        estado_validacion VARCHAR(50)
    );

    -- Validate imported data
    UPDATE #PagosTmp
    SET estado_validacion = 
        CASE 
            WHEN importe <= 0 THEN 'Invalid Amount'
            WHEN fecha_pago > GETDATE() THEN 'Future Date'
            WHEN NOT EXISTS (
                SELECT 1 FROM prestamos 
                WHERE numero_prestamo = #PagosTmp.numero_prestamo
            ) THEN 'Loan Not Found'
            ELSE 'Valid'
        END;

    -- Report validation results
    SELECT 
        estado_validacion,
        COUNT(*) as total,
        FORMAT(SUM(importe), 'C', 'es-ES') as importe_total
    FROM #PagosTmp
    GROUP BY estado_validacion;

    -- Only import valid payments
    INSERT INTO pagos (numero_prestamo, importe, fecha_pago)
    SELECT 
        numero_prestamo,
        importe,
        fecha_pago
    FROM #PagosTmp
    WHERE estado_validacion = 'Valid';

    -- Cleanup
    DROP TABLE #PagosTmp;
END
GO

-- Use: EXEC sp_ImportarPagos '2024-01-27'
```

## Usage Tips
1. Always test with small data first
2. Document any modifications
3. Keep track of execution times
4. Back up before changes
