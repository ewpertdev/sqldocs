# Common NPL Tasks & Solutions 🛠️

## 1. Data Quality Checks

### Check Invalid DNIs
```sql
-- Find invalid DNI format
SELECT dni_nif, nombre
FROM deudores
WHERE dni_nif NOT LIKE '[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][A-Z]';
```

### Check Phone Numbers
```sql
-- Find invalid phone numbers
SELECT id, telefono1
FROM contactos
WHERE telefono1 NOT LIKE '6%'      -- Mobile
    AND telefono1 NOT LIKE '9%'    -- Landline
    OR LEN(telefono1) != 9;        -- Spanish numbers = 9 digits
```

## 2. Common Reports

### Portfolio Summary
```sql
-- Basic summary
SELECT 
    COUNT(DISTINCT d.id) as total_deudores,
    COUNT(*) as total_prestamos,
    FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as cartera_total
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor;
```

### Defaulted Loans
```sql
-- Find recent defaults
SELECT 
    d.dni_nif,
    d.nombre,
    p.numero_prestamo,
    p.fecha_concesion,
    FORMAT(p.importe_inicial, 'C', 'es-ES') as importe
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
WHERE p.estado = 'IMPAGADO'
ORDER BY p.fecha_concesion DESC;
```

## 3. Excel Exports

### Format for Excel
```sql
-- Excel-friendly format
SELECT 
    d.dni_nif as 'DNI/NIF',
    d.nombre as 'Nombre',
    FORMAT(p.importe_inicial, 'N2') as 'Importe',
    FORMAT(p.fecha_concesion, 'dd/MM/yyyy') as 'Fecha'
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor;
```

## 4. Tips & Best Practices
1. Always verify data before exports
2. Document special cases
3. Keep track of common issues
4. Save successful queries
