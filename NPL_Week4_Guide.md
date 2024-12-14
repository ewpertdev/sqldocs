# NPL SQL Week 4: Performance and Best Practices 🚀

## Day 1-2: Index Management

### Creating Indexes
```sql
-- Index for DNI searches (most common)
CREATE INDEX IX_Deudores_DNI 
ON deudores(dni_nif);

-- Composite index for loan status and date
CREATE INDEX IX_Prestamos_Estado_Fecha 
ON prestamos(estado, fecha_concesion)
INCLUDE (importe_inicial);

-- Check index usage
SELECT 
    OBJECT_NAME(s.object_id) as TableName,
    i.name as IndexName,
    s.user_seeks + s.user_scans as total_usage
FROM sys.dm_db_index_usage_stats s
JOIN sys.indexes i ON s.object_id = i.object_id 
WHERE database_id = DB_ID('YourNPLDatabase');
```

## Day 3-4: Query Optimization

### Before and After
```sql
-- Before: Slow query
SELECT d.*, p.*, c.*
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
JOIN contactos c ON d.id = c.id_deudor;

-- After: Optimized
SELECT 
    d.dni_nif,
    d.nombre,
    p.numero_prestamo,
    p.importe_inicial,
    c.telefono1
FROM deudores d
JOIN prestamos p ON d.id = p.id_deudor
JOIN contactos c ON d.id = c.id_deudor
WHERE p.estado = 'IMPAGADO';
```

### Performance Monitoring
```sql
-- Find slow queries
SELECT TOP 10
    qs.total_elapsed_time / qs.execution_count as avg_time,
    qs.execution_count,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((qs.statement_end_offset - qs.statement_start_offset)/2)+1) as query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY avg_time DESC;
```

## Day 5: Maintenance and Backup

### Regular Maintenance
```sql
-- Update statistics
UPDATE STATISTICS deudores;
UPDATE STATISTICS prestamos;

-- Rebuild fragmented indexes
ALTER INDEX ALL ON deudores REBUILD;
ALTER INDEX ALL ON prestamos REBUILD;

-- Clean old data (archive if needed)
BEGIN TRANSACTION;
    INSERT INTO prestamos_historico
    SELECT * FROM prestamos 
    WHERE estado = 'PAGADO' 
        AND fecha_concesion < DATEADD(YEAR, -5, GETDATE());
    
    DELETE FROM prestamos 
    WHERE estado = 'PAGADO' 
        AND fecha_concesion < DATEADD(YEAR, -5, GETDATE());
COMMIT;
```

### Backup Strategy
```sql
-- Daily full backup
BACKUP DATABASE NPLDatabase
TO DISK = 'C:\Backups\NPL_Full.bak'
WITH INIT;

-- Transaction log backup (every 4 hours)
BACKUP LOG NPLDatabase
TO DISK = 'C:\Backups\NPL_Log.trn';
```

### Key Concepts:
1. Index design and maintenance
2. Query optimization
3. Performance monitoring
4. Data archiving
5. Backup strategies

### Best Practices:
1. Regular index maintenance
2. Monitor query performance
3. Archive old data
4. Regular backups
5. Document all changes 