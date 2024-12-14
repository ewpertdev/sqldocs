# NPL SQL Week 1: Fundamentals Guide 📚

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
