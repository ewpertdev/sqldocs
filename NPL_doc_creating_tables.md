# NPL Database: Table Creation Guide 📋

## Understanding Keys

### Primary Key
- Unique identifier for each row
- Cannot be NULL
- Usually an ID column
```sql
-- Example 1: Simple ID as primary key
CREATE TABLE deudores (
    id INT IDENTITY(1,1) PRIMARY KEY,  -- Auto-incrementing ID
    dni_nif NVARCHAR(9),
    nombre NVARCHAR(100)
);

-- Example 2: Composite primary key
CREATE TABLE pagos (
    id_deudor INT,
    fecha_pago DATE,
    importe MONEY,
    PRIMARY KEY (id_deudor, fecha_pago)  -- Two columns form the key
);
```

### Foreign Key
- References primary key in another table
- Creates relationships between tables
```sql
-- Example: prestamos references deudores
CREATE TABLE prestamos (
    id INT IDENTITY(1,1) PRIMARY KEY,
    importe MONEY,
    id_deudor INT,                              -- This will reference deudores
    FOREIGN KEY (id_deudor) REFERENCES deudores(id)  -- This creates the link
);
```

## Checking If Tables Exist

### Method 1: Using IF EXISTS
```sql
-- Drop if exists, then create
IF OBJECT_ID('deudores', 'U') IS NOT NULL
    DROP TABLE deudores;

CREATE TABLE deudores (
    id INT IDENTITY(1,1) PRIMARY KEY,
    dni_nif NVARCHAR(9)
);
```

### Method 2: Using Stored Procedure
```sql
CREATE PROCEDURE sp_CrearTablaDeudores
AS
BEGIN
    -- Check if exists
    IF OBJECT_ID('deudores', 'U') IS NOT NULL
    BEGIN
        PRINT 'Table already exists. Dropping...';
        DROP TABLE deudores;
    END

    -- Create table
    CREATE TABLE deudores (
        id INT IDENTITY(1,1) PRIMARY KEY,
        dni_nif NVARCHAR(9)
    );
    
    PRINT 'Table created successfully';
END;
```

## Creating Related Tables

### Step 1: Parent Table (Referenced)
```sql
CREATE TABLE deudores (
    id INT IDENTITY(1,1) PRIMARY KEY,
    dni_nif NVARCHAR(9),
    nombre NVARCHAR(100)
);
```

### Step 2: Child Tables (Referencing)
```sql
-- Loans reference debtors
CREATE TABLE prestamos (
    id INT IDENTITY(1,1) PRIMARY KEY,
    numero_prestamo NVARCHAR(50),
    importe MONEY,
    id_deudor INT,
    FOREIGN KEY (id_deudor) REFERENCES deudores(id)
);

-- Contacts reference debtors
CREATE TABLE contactos (
    id INT IDENTITY(1,1) PRIMARY KEY,
    telefono NVARCHAR(15),
    email NVARCHAR(100),
    id_deudor INT,
    FOREIGN KEY (id_deudor) REFERENCES deudores(id)
);
```

## Complete Example with Stored Procedures

### Creating Database Structure
```sql
CREATE PROCEDURE sp_CrearEstructuraDB
AS
BEGIN
    BEGIN TRY
        -- 1. Check and create deudores (must be first - referenced by others)
        IF OBJECT_ID('deudores', 'U') IS NOT NULL
            DROP TABLE deudores;
            
        CREATE TABLE deudores (
            id INT IDENTITY(1,1) PRIMARY KEY,
            dni_nif NVARCHAR(9),
            nombre NVARCHAR(100)
        );
        
        -- 2. Check and create prestamos
        IF OBJECT_ID('prestamos', 'U') IS NOT NULL
            DROP TABLE prestamos;
            
        CREATE TABLE prestamos (
            id INT IDENTITY(1,1) PRIMARY KEY,
            numero_prestamo NVARCHAR(50),
            importe MONEY,
            id_deudor INT,
            FOREIGN KEY (id_deudor) REFERENCES deudores(id)
        );
        
        PRINT 'Database structure created successfully';
    END TRY
    BEGIN CATCH
        PRINT 'Error creating database structure:';
        PRINT ERROR_MESSAGE();
    END CATCH
END;
```

### Important Points to Remember:
1. **Order Matters**
   - Create parent tables before child tables
   - Drop child tables before parent tables

2. **Primary Keys**
   - Usually INT IDENTITY(1,1)
   - Must be unique
   - Cannot be NULL

3. **Foreign Keys**
   - Reference existing tables
   - Can be NULL (unless specified)
   - Create relationships

4. **Best Practices**
   - Always check if tables exist
   - Use error handling
   - Document relationships
   - Use meaningful names

### Common Mistakes to Avoid:
1. Creating child tables before parent tables
2. Forgetting to check if tables exist
3. Not handling errors in procedures
4. Missing foreign key constraints 
