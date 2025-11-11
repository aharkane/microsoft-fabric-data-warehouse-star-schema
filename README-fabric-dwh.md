# 1. Project Overview

### Business Problem
As our business grows, we need a structured way to store, process, and analyze sales data. Currently, we lack a centralized data solution, making it difficult to track key business metrics, optimize performance, and generate insightful reports.

### Solution
We need a **data warehouse** that integrates with **Microsoft Fabric** and allows us to:  
- Ingest sales data from external sources  
- Organize and structure data efficiently using dimensional modeling  
- Ensure fast and accurate reporting on sales performance  
- Provide clear visibility into customer behavior, product performance, and revenue trends  

### Implementation Summary

#### Data Lakehouse Setup
1. Created workspace: `sales_ws`  
2. Created lakehouse: `sales_dlh` for raw data storage  
3. Uploaded `sales.csv` and created staging table: `sales_stg`  

#### Data Warehouse Setup
1. Created data warehouse: `sales_dwh`  
2. Created dedicated schema: `sales_schema`  
3. Built star schema with dimension and fact tables  
4. Implemented stored procedure for automated ETL  
5. Executed analytical queries and created visualizations  

---

# 2. Architecture

```text
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

# Microsoft Fabric Sales Data Warehouse

Dimensional data warehouse on Microsoft Fabric with star schema design and automated ETL processes.

## Overview

End-to-end data warehouse implementing dimensional modeling with staging layer, stored procedure-based ETL automation, and incremental loading for sales analytics. Processes 121,317 sales transactions into optimized star schema.

## Architecture

**Three-Layer Design:**
- **Lakehouse**: Raw CSV data storage
- **Staging**: Data validation and transformation layer
- **Production**: Star schema (5 dimensions + 1 fact table)

**Star Schema:**
- DimCustomer (10,492 records), DimProduct (117), DimSalesperson, DimAddress
- FactSales (28,784 transactions)

## Key Features

- Parametric stored procedures with year-based filtering
- NOT EXISTS patterns for deduplication
- Incremental loading with cutoff dates
- Type casting and data validation
- Advanced analytics with window functions

## Technology Stack

| Component | Technology |
|-----------|------------|
| **Platform** | Microsoft Fabric |
| **Storage** | Lakehouse + Data Warehouse |
| **ETL** | T-SQL Stored Procedures |
| **Analytics** | SQL with window functions |

## Results

- 28,784 transactions loaded into FactSales
- 10,492 customers and 117 products processed
- Sub-second query performance on star schema
- Automated deduplication and change detection

## Skills Demonstrated

<table>
<tr>
<td width="55%" valign="top">

**Data Warehousing**
- Star schema dimensional modeling
- Staging layer architecture
- Surrogate key management
- Referential integrity

**SQL Development**
- T-SQL programming
- Window functions and CTEs
- Set-based operations
- Complex join patterns

</td>
<td width="45%" valign="top">

**ETL Development**
- Stored procedure automation
- Incremental loading
- Change detection
- Data validation

**Microsoft Fabric**
- Lakehouse integration
- Data Warehouse setup
- Workspace management
- Multi-component orchestration

</td>
</tr>
</table>
