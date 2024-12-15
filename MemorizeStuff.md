# Essential SQL Knowledge for NPL Work 🧠

## 1. Basic Syntax (The Building Blocks)

### SELECT Statement Structure
```sql
SELECT column1, column2       -- What you want
FROM table_name              -- Where to get it
WHERE condition;             -- Filter criteria
```

### Common WHERE Conditions
```sql
WHERE estado = 'IMPAGADO'              -- Equals
WHERE importe > 1000                   -- Greater than
WHERE fecha BETWEEN '2023-01-01' AND '2023-12-31'
WHERE ciudad IN ('Madrid', 'Barcelona')
WHERE telefono IS NULL                 -- Missing data
```

## 2. JOIN Types (The Connections)

### INNER JOIN
```sql
-- Only matching records
SELECT d.nombre, o.importe
FROM deudores d
INNER JOIN operaciones o ON d.id = o.id_deudor;
```

### LEFT JOIN
```sql
-- All from left table (deudores)
SELECT d.nombre, ISNULL(o.importe, 0) as importe
FROM deudores d
LEFT JOIN operaciones o ON d.id = o.id_deudor;
```

## 3. Table Relationships (The Map)

### Core NPL Tables
```
deudores (Main table)
    │
    ├── operaciones (Loans/Debts)
    │     └── pagos (Payments)
    │
    ├── contactos (Contact Info)
    │     └── telefonos (Phone Numbers)
    │
    └── direcciones (Addresses)
```

### Key Fields to Remember
```sql
deudores.id           -- Primary key
operaciones.id_deudor -- Links to deudores
contactos.id_deudor   -- Links to deudores
direcciones.id_deudor -- Links to deudores
```

## 4. Business Logic (The Rules)

### Debt Status
```sql
-- Active debts
WHERE estado = 'IMPAGADO'
  AND fecha_vencimiento < GETDATE()

-- High risk cases
WHERE importe > 50000
  OR dias_retraso > 90
```

### Contact Rules
```sql
-- Priority contacts (high debt)
WHERE importe > 10000
  AND telefono IS NOT NULL
ORDER BY importe DESC

-- No contact info
WHERE telefono IS NULL 
  AND email IS NULL
```

### Payment Tracking
```sql
-- Last payment date
SELECT MAX(fecha_pago) 
FROM pagos 
WHERE id_operacion = @id

-- Payment history
SELECT fecha_pago, importe
FROM pagos
ORDER BY fecha_pago DESC
```

## 5. Common Patterns (The Templates)

### Finding a Debtor
```sql
SELECT d.dni_nif, d.nombre, o.importe
FROM deudores d
LEFT JOIN operaciones o ON d.id = o.id_deudor
WHERE d.dni_nif = '12345678A';
```

### Debt Summary
```sql
SELECT 
    d.tipo_entidad,
    COUNT(*) as total_deudores,
    SUM(o.importe) as deuda_total
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
GROUP BY d.tipo_entidad;
```

### Contact List
```sql
SELECT 
    d.dni_nif,
    d.nombre,
    c.telefono,
    dir.ciudad
FROM deudores d
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor;
```

## Tips for Memorizing 🎯

1. **Practice Daily**
   - Write queries every day
   - Start simple, add complexity
   - Use real scenarios

2. **Understand Don't Memorize**
   - Know WHY you use each JOIN
   - Understand table relationships
   - Learn business rules

3. **Common Patterns**
   - Most queries follow patterns
   - Learn the patterns, not exact syntax
   - Adapt patterns to needs

4. **Key Questions to Ask**
   - What data do I need? (SELECT)
   - Where is it stored? (FROM/JOIN)
   - How to filter it? (WHERE)
   - How to group it? (GROUP BY) 
