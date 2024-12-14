# NPL SQL Week 2: Joins and Aggregations 📊

## Day 1-2: Advanced Joins

### Multiple Table Joins
```sql
-- Get complete debtor profile
SELECT 
    d.dni_nif,
    d.nombre,
    p.numero_prestamo,
    p.importe_inicial,
    c.telefono1,
    dir.ciudad
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor         -- Get their loans
LEFT JOIN contactos c ON d.id = c.id_deudor    -- Maybe has contacts
LEFT JOIN direcciones dir ON d.id = dir.id_deudor;  -- Maybe has address
```

### Different Join Types
```sql
-- INNER JOIN (only matching records)
SELECT d.nombre, p.importe_inicial 
FROM deudores d
INNER JOIN prestamos p ON d.id = p.id_deudor;

-- LEFT JOIN (all debtors, even without loans)
SELECT d.nombre, ISNULL(p.importe_inicial, 0) as deuda
FROM deudores d
LEFT JOIN prestamos p ON d.id = p.id_deudor;
```

## Day 3-4: Advanced Aggregations

### Complex Grouping
```sql
-- Debt by city and status
SELECT 
    dir.ciudad,
    p.estado,
    COUNT(*) as total_prestamos,
    FORMAT(SUM(p.importe_inicial), 'C', 'es-ES') as deuda_total
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
JOIN direcciones dir ON d.id = dir.id_deudor
GROUP BY dir.ciudad, p.estado
ORDER BY dir.ciudad, deuda_total DESC;
```

### Having Clause
```sql
-- Find cities with high default rates
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

## Day 5: Practice & Review

### Key Concepts:
1. JOIN types (INNER, LEFT, RIGHT)
2. Multiple table joins
3. Complex GROUP BY
4. HAVING filters
5. Aggregation functions

### Common Mistakes to Avoid:
1. Missing JOIN conditions
2. Wrong JOIN type
3. Forgetting GROUP BY columns
4. Mixing aggregates with non-aggregates 