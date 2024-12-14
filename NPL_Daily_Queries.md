# Daily NPL SQL Practice Queries 🔄

## 1. Morning Routine (Start your day with these)

### Client Searches
```sql
-- Basic client search (try different DNIs)
SELECT 
    d.dni_nif,
    d.nombre,
    p.numero_prestamo,
    FORMAT(p.importe_inicial, 'C', 'es-ES') as importe,
    p.estado
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
WHERE d.dni_nif = '12345678A';

-- Name search (try different names)
SELECT * FROM deudores WHERE nombre LIKE '%García%';
```

### Daily Summary
```sql
-- Portfolio status
SELECT 
    estado as loan_status,                                           -- Loan status
    COUNT(*) as total_prestamos,                      -- How many loans
    FORMAT(SUM(importe_inicial), 'C', 'es-ES') as importe_total  -- Total amount
FROM prestamos
GROUP BY estado;
```

## 2. Common Variations

### Different WHERE Conditions
```sql
-- Multiple conditions
WHERE d.dni_nif = '12345678A' 
    AND p.estado = 'IMPAGADO';

-- Date ranges
WHERE p.fecha_concesion BETWEEN '2023-01-01' AND '2023-12-31';
```

## 3. Practice Tips
1. Change one thing at a time
2. Verify your results
3. Understand each part
4. Document useful queries
