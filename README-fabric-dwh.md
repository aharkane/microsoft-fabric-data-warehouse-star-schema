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