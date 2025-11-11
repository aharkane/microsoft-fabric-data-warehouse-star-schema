# Sales Data Warehouse Implementation and Customer-Product Reporting in Microsoft Fabric using SQL & Data Factory

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Project Architecture](#project-architecture)
3. [Featured SQL Examples](#featured-sql-examples)
   1. [Stored Procedure: Automated Data Loading](#stored-procedure-automated-data-loading)
   2. [Top Customers by Revenue Analysis](#top-customers-by-revenue-analysis)
   3. [Top Products by Revenue](#top-products-by-revenue)
   4. [Advanced Analytics: Top Customer per Category (Window Functions)](#advanced-analytics-top-customer-per-category-window-functions)



## Executive Summary

**Overview**  

Dimensional data warehouse on Microsoft Fabric, implementing dimensional modeling with staging layer, stored procedure-based ETL automation, and incremental loading for sales analytics. Processes 121,317 sales transactions into optimized star schema.

**Tasks and Goals**

- Build a scalable, production-grade data warehouse using Microsoft Fabric
- Design and implement star schema dimensional model for analytical queries
- Develop ETL processes with stored procedures for automated data loading
- Enable multi-dimensional business analytics across customers, products, and categories
- Demonstrate data governance and referential integrity patterns

**Key Achievements & Skills**

✓ Designed and implemented a star schema with fact and dimension tables, including primary/foreign key relationships
✓ Built an automated ETL pipeline using stored procedures, handling data loading, deduplication, and type consistency
✓ Loaded 28,000+ transactions from staging (Lakehouse) to the fact table while maintaining clean data separation
✓ Applied SQL analytics: complex CASE statements, window functions (ROW_NUMBER, PARTITION BY), multi-table joins, aggregations, and CTEs
✓ Executed multi-dimensional business reporting for insights on customers, products, and sales performance
✓ Managed Microsoft Fabric environment: lakehouse staging, data warehouse creation, schema organization, and integrated analytics

**Repository Structure**

```
├── data/
│   └── sales.csv
├── MSFabric_Sales_DWH_Implementation.ipynb
└── README.md
```

## Project Architecture

**Technology Stack**

| Component | Technology |
|-----------|-----------|
| **Platform** | Microsoft Fabric |
| **Data Storage** | Fabric Lakehouse + Data Warehouse |
| **Query Language** | T-SQL |
| **ETL Framework** | Stored Procedures & Views |
| **Data Format** | CSV (staging) → Dimensional Tables |
| **Analytics** | SQL queries with window functions |


**Data Flow**

```
Sales CSV → Lakehouse (Staging) → ETL Pipeline → Data Warehouse
                                   ↓
                           Dimensional Star Schema
                           ├── DimCustomer
                           ├── DimItem
                           └── FactSales

┌─────────────┐     ┌──────────────────┐     ┌──────────────────┐
│             │     │                  │     │                  │
│  sales.csv  │────>│  Lakehouse       │────>│  Data Warehouse  │
│             │     │  (sales_dlh)     │     │  (sales_dwh)     │
└─────────────┘     │                  │     │                  │
                    │  ┌─────────────┐ │     │  ┌────────────┐  │
                    │  │ sales_stg   │ │     │  │ Dim_       │  │
                    │  │ (staging)   │ │     │  │ Customer   │  │
                    │  └─────────────┘ │     │  └────────────┘  │
                    │                  │     │                  │
                    └──────────────────┘     │  ┌────────────┐  │
                                             │  │ Dim_Item   │  │
                                             │  └────────────┘  │
                                             │                  │
                                             │  ┌────────────┐  │
                                             │  │ Fact_Sales │  │
                                             │  └────────────┘  │
                                             └──────────────────┘

```







## Featured SQL Examples

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
