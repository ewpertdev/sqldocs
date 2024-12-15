# Understanding Multiple JOINs in NPL Context 🔄

## How JOINs Work: Step by Step

### 1. Basic Concept
Think of JOINs like connecting puzzle pieces:
```
deudores (Debtors)     operaciones (Loans)
┌─────────┐            ┌──────────┐
│ id: 1   │──────────▶│id_deudor:1│
│ dni: A  │            │importe:100│
└─────────┘            └──────────┘
```

### 2. Adding More Tables
Each new JOIN adds another piece:
```
deudores         operaciones        contactos
┌─────────┐      ┌──────────┐      ┌──────────┐
│ id: 1   │─────▶│id_deudor:1│     │id_deudor:1│
│ dni: A  │      │importe:100│     │tel: 123  │
└─────────┘      └──────────┘      └──────────┘
```

### 3. How Multiple JOINs Execute
SQL processes JOINs from left to right:
1. First JOIN: `deudores + operaciones`
2. Second JOIN: `(result above) + contactos`
3. Third JOIN: `(result above) + direcciones`

### Example with Data:
```sql
-- Starting with deudores
id  | dni_nif   | nombre
1   | 12345678A | Juan
2   | 87654321B | Maria

-- Adding operaciones (First JOIN)
dni_nif   | nombre | importe
12345678A | Juan   | 1000
87654321B | Maria  | 2000

-- Adding contactos (Second JOIN)
dni_nif   | nombre | importe | telefono
12345678A | Juan   | 1000    | 666555444
87654321B | Maria  | 2000    | 999888777

-- Final result with direcciones (Third JOIN)
dni_nif   | nombre | importe | telefono  | ciudad
12345678A | Juan   | 1000    | 666555444 | Madrid
87654321B | Maria  | 2000    | 999888777 | Barcelona
```

### Understanding JOIN Types in Sequence

#### 1. First JOIN (INNER)
```sql
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
-- Shows only debtors WITH loans
```

#### 2. Second JOIN (LEFT)
```sql
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor
-- Keeps all debtors with loans, even if no contact info
```

#### 3. Third JOIN (LEFT)
```sql
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor
-- Keeps all previous results, even if no address
```

### Why Use Different JOIN Types?
- INNER JOIN: When you MUST have data from both tables
- LEFT JOIN: When you want ALL records from the left table
Example:
```sql
-- You want ALL debtors, even those without phones
SELECT d.nombre, ISNULL(c.telefono, 'No Phone')
FROM deudores d
LEFT JOIN contactos c ON d.id = c.id_deudor;
```

## Basic Structure
```sql
SELECT [columns]
FROM table1
JOIN table2 ON condition
JOIN table3 ON condition
...
```

## Common NPL Tables Relationship
```sql
deudores (Debtors)
    ↓
    ├── operaciones (Loans/Operations)
    ├── contactos (Contact Info)
    └── direcciones (Addresses)
```

## Simple to Complex Examples

### 1. Basic Two-Table JOIN
```sql
-- Get debtor with their loans
SELECT 
    d.dni_nif,
    d.nombre,
    o.numero_operacion,
    o.importe
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor;
```

### 2. Three-Table JOIN
```sql
-- Get debtor, loans, and contact info
SELECT 
    d.dni_nif,
    d.nombre,
    o.numero_operacion,
    c.telefono
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor;
```

### 3. Four-Table JOIN
```sql
-- Complete debtor profile
SELECT 
    d.dni_nif,
    d.nombre,
    o.numero_operacion,
    o.importe,
    c.telefono,
    dir.ciudad
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor;
```

## Understanding JOIN Types

### INNER JOIN
```sql
-- Only shows matches (debtors WITH loans)
SELECT d.nombre, o.importe
FROM deudores d
INNER JOIN operaciones o ON d.id = o.id_deudor;
```

### LEFT JOIN
```sql
-- Shows ALL debtors (even those WITHOUT loans)
SELECT 
    d.nombre, 
    ISNULL(o.importe, 0) as importe
FROM deudores d
LEFT JOIN operaciones o ON d.id = o.id_deudor;
```

## Common Use Cases

### 1. Portfolio Analysis
```sql
-- Debt by city and type
SELECT 
    dir.ciudad,
    d.tipo_entidad,
    COUNT(*) as total_deudores,
    SUM(o.importe) as deuda_total
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor
GROUP BY dir.ciudad, d.tipo_entidad;
```

### 2. Contact Campaign
```sql
-- Find debtors with missing contact info
SELECT 
    d.dni_nif,
    d.nombre,
    c.telefono,
    c.email,
    dir.ciudad
FROM deudores d
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor
WHERE c.telefono IS NULL OR c.email IS NULL;
```

### 3. High Risk Analysis
```sql
-- Find high-debt cases with contact info
SELECT 
    d.dni_nif,
    d.nombre,
    SUM(o.importe) as deuda_total,
    c.telefono,
    dir.ciudad
FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor
LEFT JOIN contactos c ON d.id = c.id_deudor
LEFT JOIN direcciones dir ON d.id = dir.id_deudor
GROUP BY d.dni_nif, d.nombre, c.telefono, dir.ciudad
HAVING SUM(o.importe) > 50000;
```

## Common Mistakes to Avoid

### 1. Missing JOIN Conditions
```sql
-- WRONG: Cartesian product
SELECT * FROM deudores, operaciones;

-- RIGHT: Proper JOIN
SELECT * FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor;
```

### 2. Wrong JOIN Type
```sql
-- WRONG: INNER JOIN loses debtors without loans
SELECT d.nombre, o.importe
FROM deudores d
INNER JOIN operaciones o ON d.id = o.id_deudor;

-- RIGHT: LEFT JOIN keeps all debtors
SELECT d.nombre, ISNULL(o.importe, 0) as importe
FROM deudores d
LEFT JOIN operaciones o ON d.id = o.id_deudor;
```

### 3. Column Ambiguity
```sql
-- WRONG: Ambiguous 'id' column
SELECT id, nombre FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor;

-- RIGHT: Specify table
SELECT d.id, d.nombre FROM deudores d
JOIN operaciones o ON d.id = o.id_deudor;
```

## Best Practices
1. Always alias tables
2. Specify table for columns
3. Use appropriate JOIN type
4. Handle NULL values
5. Test with small datasets first

## System Query that shows how tables are connected

```sql
-- First, make sure you're in the right database
USE YourDatabaseName;
GO

-- Then run the relationships query
SELECT 
    fk.name as FK_name,
    tp.name as parent_table,
    cp.name as parent_column,
    tr.name as referenced_table,
    cr.name as referenced_column
FROM sys.foreign_keys fk
INNER JOIN sys.tables tp ON fk.parent_object_id = tp.object_id
INNER JOIN sys.tables tr ON fk.referenced_object_id = tr.object_id
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
INNER JOIN sys.columns cp ON fkc.parent_object_id = cp.object_id AND fkc.parent_column_id = cp.column_id
INNER JOIN sys.columns cr ON fkc.referenced_object_id = cr.object_id AND fkc.referenced_column_id = cr.column_id;
```


