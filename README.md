# Microsoft Fabric Sales Data Warehouse Implementation

## Project Overview

A comprehensive **data warehouse solution** built on Microsoft Fabric demonstrating enterprise data architecture principles. This project implements a complete Extract-Transform-Load (ETL) pipeline with dimensional modeling, integrating raw sales data into a star schema data warehouse for advanced business analytics.

## Project Goals

- Build a scalable, production-grade data warehouse using Microsoft Fabric
- Design and implement star schema dimensional model for analytical queries
- Develop ETL processes with stored procedures for automated data loading
- Enable multi-dimensional business analytics across customers, products, and categories
- Demonstrate data governance and referential integrity patterns

## Technology Stack

| Component | Technology |
|-----------|-----------|
| **Platform** | Microsoft Fabric |
| **Data Storage** | Fabric Lakehouse + Data Warehouse |
| **Query Language** | T-SQL |
| **ETL Framework** | Stored Procedures & Views |
| **Data Format** | CSV (staging) → Dimensional Tables |
| **Analytics** | SQL queries with window functions |

## Architecture & Implementation

### Data Flow

```
Sales CSV → Lakehouse (Staging) → ETL Pipeline → Data Warehouse
                                   ↓
                           Dimensional Star Schema
                           ├── DimCustomer
                           ├── DimItem
                           └── FactSales
```

### Schema Design

**Staging Layer (Lakehouse)**
- `salesstg`: Raw staging table from imported CSV data
- Location: `salesdlh.dbo.salesstg` (Data Lakehouse)

**Dimensional Model (Data Warehouse)**
- `DimCustomer`: Customer master data with surrogate keys
- `DimItem`: Product catalog with item details
- `FactSales`: Transactional sales facts with foreign keys to dimensions
- Location: `salesdwh.salesschema` (Dedicated Schema)

### Key Features

**Schema & Table Management**
- Dedicated `salesschema` for organizing data warehouse objects
- Primary keys on all dimension tables (nonclustered, not enforced)
- Foreign key constraints enforcing referential integrity
- Star schema design for optimized analytical queries

**ETL Automation**
- `LoadDataFromStaging` stored procedure with parametric year filtering
- Automatic deduplication using NOT EXISTS logic
- Type casting for data consistency
- Incremental data loading capability

**Data Loading Results (2021 Dataset)**
- Loaded 10,492 unique customers into DimCustomer
- Loaded 117 unique products into DimItem
- Loaded 28,784 transactions into FactSales

## Featured SQL Implementations

### Stored Procedure: Automated Data Loading

```sql
CREATE OR ALTER PROCEDURE salesschema.LoadDataFromStaging @OrderYear INT AS
BEGIN
    -- Insert customers into DimCustomer with deduplication
    INSERT INTO salesschema.DimCustomer (CustomerID, CustomerName, EmailAddress)
    SELECT DISTINCT 
        CustomerName,
        CustomerName,
        EmailAddress
    FROM salesdlh.dbo.salesstg
    WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1 FROM salesschema.DimCustomer c
            INNER JOIN salesdlh.dbo.salesstg s
            ON c.CustomerName = s.CustomerName AND c.EmailAddress = s.EmailAddress
        )

    -- Insert products into DimItem
    INSERT INTO salesschema.DimItem (ItemID, ItemName)
    SELECT DISTINCT Item, Item
    FROM salesdlh.dbo.salesstg
    WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1 FROM salesschema.DimItem i
            INNER JOIN salesdlh.dbo.salesstg s ON i.ItemName = s.Item
        )

    -- Insert transactions into FactSales
    INSERT INTO salesschema.FactSales 
        (CustomerID, ItemID, SalesOrderNumber, SalesOrderLineNumber, 
         OrderDate, Quantity, TaxAmount, UnitPrice)
    SELECT DISTINCT 
        CAST(CustomerName AS VARCHAR(255)),
        CAST(Item AS VARCHAR(255)),
        CAST(SalesOrderNumber AS VARCHAR(30)),
        CAST(SalesOrderLineNumber AS INT),
        CAST(OrderDate AS DATE),
        CAST(Quantity AS INT),
        CAST(TaxAmount AS FLOAT),
        CAST(UnitPrice AS FLOAT)
    FROM salesdlh.dbo.salesstg
    WHERE YEAR(OrderDate) = @OrderYear
END
```

### Top Customers by Revenue Analysis

```sql
SELECT TOP 5
    c.CustomerName,
    ROUND(SUM(s.Quantity * s.UnitPrice), 2) AS TotalSales
FROM salesschema.DimCustomer c
LEFT JOIN salesschema.FactSales s ON c.CustomerID = s.CustomerID
WHERE YEAR(s.OrderDate) = 2021
GROUP BY c.CustomerName
ORDER BY TotalSales DESC
```

**Results:**
- Jordan Turner: $14,686.70
- Nicole Blue: $11,494.93
- Maurice Shan: $10,525.60
- Janet Munoz: $10,070.11
- Alexandra Hall: $9,710.76

### Top Products by Revenue

```sql
SELECT TOP 5
    i.ItemName,
    ROUND(SUM(s.Quantity * s.UnitPrice), 2) AS TotalSales
FROM salesschema.DimItem i
LEFT JOIN salesschema.FactSales s ON i.ItemID = s.ItemID
GROUP BY i.ItemName
ORDER BY TotalSales DESC
```

**Results:**
- Mountain-200 Black, 46: $718,987.58
- Mountain-200 Silver, 46: $687,794.17
- Mountain-200 Black, 38: $668,006.02
- Mountain-200 Black, 42: $647,760.93
- Mountain-200 Silver, 38: $641,145.80

### Advanced Analytics: Top Customer per Category (Window Functions)

```sql
WITH CategorizedSales AS (
    SELECT
        CASE
            WHEN i.ItemName LIKE '%Helmet%' THEN 'Helmet'
            WHEN i.ItemName LIKE '%Bike%' THEN 'Bike'
            WHEN i.ItemName LIKE '%Gloves%' THEN 'Gloves'
            ELSE 'Other'
        END AS Category,
        c.CustomerName,
        s.UnitPrice * s.Quantity AS Sales
    FROM salesschema.FactSales s
    JOIN salesschema.DimCustomer c ON s.CustomerID = c.CustomerID
    JOIN salesschema.DimItem i ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
),
RankedSales AS (
    SELECT
        Category,
        CustomerName,
        SUM(Sales) AS TotalSales,
        ROW_NUMBER() OVER (PARTITION BY Category ORDER BY SUM(Sales) DESC) AS SalesRank
    FROM CategorizedSales
    WHERE Category IN ('Helmet', 'Bike', 'Gloves')
    GROUP BY Category, CustomerName
)
SELECT Category, CustomerName, TotalSales
FROM RankedSales
WHERE SalesRank = 1
ORDER BY TotalSales DESC
```

**Results:**
- Bike: Carson Butler - $318.00
- Helmet: Hailey Patterson - $209.94
- Gloves: Joan Coleman - $97.96

## Repository Structure

```
├── data/
│   └── sales.csv
├── MSFabric_Sales_DWH_Implementation.ipynb
└── README.md
```

## Key Accomplishments

✓ Designed and implemented complete star schema dimensional model  
✓ Built automated ETL pipeline with stored procedures  
✓ Loaded 28,784+ transactions from staging to fact table  
✓ Implemented referential integrity with foreign key constraints  
✓ Executed advanced analytics with window functions (ROW_NUMBER)  
✓ Demonstrated multi-dimensional business reporting  
✓ Achieved clean data separation (Lakehouse → Warehouse)  

## Skills Demonstrated

**Data Warehouse Design:**
- Star schema dimensional modeling
- Fact and dimension table architecture
- Primary/foreign key relationships
- Schema organization best practices

**ETL & Data Integration:**
- Stored procedure development
- Parametric data loading
- Deduplication logic (NOT EXISTS)
- Type casting and data consistency

**SQL Analytics:**
- Complex CASE statements for categorization
- Window functions (ROW_NUMBER, PARTITION BY)
- Common Table Expressions (CTEs)
- Multi-table joins and aggregations
- Data grouping and ordering

**Microsoft Fabric:**
- Lakehouse staging layer setup
- Data Warehouse creation and management
- Schema and security implementation
- Integrated analytics environment

---

*This project demonstrates enterprise data warehouse architecture and advanced SQL analytics capabilities in a modern cloud data platform.*
