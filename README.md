# SQL Data Warehouse Project

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Project Structure](#project-structure)
4. [Data Layers](#data-layers)
5. [Source Systems](#source-systems)
6. [Data Flow](#data-flow)
7. [Database Objects](#database-objects)
8. [Installation & Setup](#installation--setup)
9. [ETL Procedures](#etl-procedures)
10. [Data Quality](#data-quality)
11. [Naming Conventions](#naming-conventions)
12. [Getting Started](#getting-started)

---

## Project Overview

This is a comprehensive **SQL Server Data Warehouse Project** designed using a **Medallion Architecture** (Bronze-Silver-Gold layers). The project demonstrates professional data engineering practices including data ingestion, transformation, cleansing, standardization, and business-ready analytics preparation.

**Key Features:**
- Multi-layered data warehouse architecture
- Automated ETL processes using SQL Server stored procedures
- Data quality validation and monitoring
- Star schema modeling for analytics
- Comprehensive documentation and naming conventions
- Support for multiple data sources (CRM & ERP systems)

---

## Architecture

The data warehouse follows the **Medallion Architecture** pattern with three distinct layers:

### Layer Architecture Overview

![Layer Architecture Diagram](docs/layer-architecture.png)

The architecture consists of:

```
┌─────────────────────────────────────────────────────────────────┐
│                     SOURCE SYSTEMS                               │
│            (CSV Files from CRM & ERP Systems)                    │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ 🟠 BRONZE LAYER - Raw Data Repository                            │
│    Definition: Raw, unprocessed data as-is from sources          │
│    Objective: Traceability & Debugging                           │
│    Object Type: Tables | Load Method: Full Load (Truncate)       │
│    Data Transformation: None (as-is)                             │
│    Target Audience: Data Engineers                               │
└────────────────────────┬────────────────────────────────────────┘
                         │ (Transformation & Cleaning)
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ ⚪ SILVER LAYER - Cleaned & Standardized Data                    │
│    Definition: Clean, standardized, normalized data              │
│    Objective: Prepare Data for Analysis (Intermediate Layer)    │
│    Object Type: Tables | Load Method: Full Load (Truncate)       │
│    Data Transformation:                                          │
│      • Data Cleansing & Validation                               │
│      • Data Standardization & Normalization                      │
│      • Derived Columns Creation                                  │
│      • Data Enrichment                                           │
│    Target Audience: Data Analysts, Data Engineers                │
└────────────────────────┬────────────────────────────────────────┘
                         │ (Business Logic & Aggregation)
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ 🟡 GOLD LAYER - Business-Ready Data                              │
│    Definition: Business-ready data modeled as Star Schema        │
│    Objective: Provide data for Analytics & Reporting             │
│    Object Type: Views | Load Method: None (Real-time Views)      │
│    Data Modeling:                                                │
│      • Star Schema (Dimensions + Facts)                          │
│      • Data Integration & Aggregation                            │
│      • Business Logic & Rules                                    │
│    Includes:                                                      │
│      • Dimension Tables (Customers, Products)                    │
│      • Fact Tables (Sales Transactions)                          │
│    Target Audience: Business Users, Analysts, BI Tools           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
sql-data-warehouse-project-main/
├── datasets/                           # Source Data Files
│   ├── source_crm/                     # CRM System Data
│   │   ├── cust_info.csv              # Customer Information
│   │   ├── prd_info.csv               # Product Information
│   │   └── sales_details.csv          # Sales Transactions
│   └── source_erp/                     # ERP System Data
│       ├── CUST_AZ12.csv              # Customer Demographics (Birthdate, Gender)
│       ├── LOC_A101.csv               # Customer Location (Country)
│       └── PX_CAT_G1V2.csv            # Product Categories & Maintenance Info
│
├── scripts/                            # SQL Scripts
│   ├── init_database.sql              # Database & Schema Initialization
│   ├── bronze/
│   │   ├── ddl_bronze.sql             # Bronze Layer Table Definitions
│   │   └── proc_load_bronze.sql       # Stored Procedure: Load Bronze
│   ├── silver/
│   │   ├── ddl_silver.sql             # Silver Layer Table Definitions
│   │   └── proc_load_silver.sql       # Stored Procedure: Load Silver
│   └── gold/
│       └── ddl_gold.sql               # Gold Layer View Definitions
│
├── tests/                              # Data Quality Tests
│   ├── quality_checks_silver.sql      # Silver Layer Data Quality Tests
│   └── quality_checks_gold.sql        # Gold Layer Data Quality Tests
│
├── docs/                               # Documentation & Diagrams
│   ├── data_architecture.png          # Overall Architecture Diagram
│   ├── data_flow.png                  # Data Flow Visualization
│   ├── data_integration.png           # Data Integration Design
│   ├── data_model.png                 # Star Schema Model
│   ├── data_layers.pdf                # Layer-wise Documentation
│   ├── ETL.png                        # ETL Process Workflow
│   ├── data_catalog.md                # Data Dictionary
│   ├── naming_conventions.md          # Naming Standards
│   └── Project_Notes_Sketches.pdf     # Project Design Notes
│
└── README.md                           # This File
```

---

## Data Layers

### 🟠 BRONZE LAYER: Raw Data Repository

**Purpose:** Store raw, unprocessed data as-is from source systems for traceability and debugging.

**Characteristics:**
- Exact replica of source data
- No transformations applied
- Serves as audit trail
- Full load strategy (truncate & insert)

**Tables in Bronze Layer:**

#### 1. `bronze.crm_cust_info` - Customer Information (CRM)
| Column | Data Type | Description |
|--------|-----------|-------------|
| cst_id | INT | Unique customer identifier |
| cst_key | NVARCHAR(50) | Customer number/code |
| cst_firstname | NVARCHAR(50) | Customer's first name |
| cst_lastname | NVARCHAR(50) | Customer's last name |
| cst_marital_status | NVARCHAR(50) | Marital status (S/M - coded) |
| cst_gndr | NVARCHAR(50) | Gender (F/M - coded) |
| cst_create_date | DATE | Record creation date |

#### 2. `bronze.crm_prd_info` - Product Information (CRM)
| Column | Data Type | Description |
|--------|-----------|-------------|
| prd_id | INT | Product identifier |
| prd_key | NVARCHAR(50) | Product code (contains category info) |
| prd_nm | NVARCHAR(50) | Product name |
| prd_cost | INT | Product cost |
| prd_line | NVARCHAR(50) | Product line (M/R/S/T - coded) |
| prd_start_dt | DATETIME | Product availability start date |
| prd_end_dt | DATETIME | Product availability end date |

#### 3. `bronze.crm_sales_details` - Sales Transactions (CRM)
| Column | Data Type | Description |
|--------|-----------|-------------|
| sls_ord_num | NVARCHAR(50) | Sales order number |
| sls_prd_key | NVARCHAR(50) | Product key |
| sls_cust_id | INT | Customer ID |
| sls_order_dt | INT | Order date (numeric format: YYYYMMDD) |
| sls_ship_dt | INT | Shipping date (numeric format: YYYYMMDD) |
| sls_due_dt | INT | Due date (numeric format: YYYYMMDD) |
| sls_sales | INT | Total sales amount |
| sls_quantity | INT | Quantity ordered |
| sls_price | INT | Unit price |

#### 4. `bronze.erp_cust_az12` - Customer Demographics (ERP)
| Column | Data Type | Description |
|--------|-----------|-------------|
| cid | NVARCHAR(50) | Customer ID |
| bdate | DATE | Customer birthdate |
| gen | NVARCHAR(50) | Gender (F/M - coded) |

#### 5. `bronze.erp_loc_a101` - Customer Location (ERP)
| Column | Data Type | Description |
|--------|-----------|-------------|
| cid | NVARCHAR(50) | Customer ID |
| cntry | NVARCHAR(50) | Country code or name |

#### 6. `bronze.erp_px_cat_g1v2` - Product Categories (ERP)
| Column | Data Type | Description |
|--------|-----------|-------------|
| id | NVARCHAR(50) | Product category ID |
| cat | NVARCHAR(50) | Category name (e.g., Bikes, Components) |
| subcat | NVARCHAR(50) | Subcategory |
| maintenance | NVARCHAR(50) | Maintenance requirement flag |

---

### ⚪ SILVER LAYER: Cleaned & Standardized Data

**Purpose:** Provide intermediate layer with cleaned, standardized, and normalized data prepared for analysis.

**Characteristics:**
- Data cleansing applied (trim whitespace, handle nulls)
- Standardization of coded values (S→Single, M→Married, F→Female, M→Male)
- Data type conversions (numeric dates to DATE format)
- Deduplication using row_number()
- Data enrichment (derived columns)
- Technical columns added (dwh_create_date)
- Full load strategy (truncate & insert)

**Transformations Applied:**

#### CRM Customer Information Cleaning:
```sql
• Trim whitespace from names
• Normalize marital status: S→Single, M→Married, else→n/a
• Normalize gender: F→Female, M→Male, else→n/a
• Remove duplicate records (keep most recent)
• Add technical metadata column (dwh_create_date)
```

#### CRM Product Information Transformation:
```sql
• Extract category ID from product key
• Extract product key from composite field
• Handle null costs (default to 0)
• Normalize product line codes: M→Mountain, R→Road, S→Other Sales, T→Touring
• Convert dates to DATE type
• Calculate product end dates using window functions
```

#### CRM Sales Details Cleansing:
```sql
• Convert numeric date format (YYYYMMDD) to DATE type
• Handle invalid dates (0 or incorrect length)
• Recalculate sales amount (quantity × price) if invalid
• Derive unit price if missing
• Validate data integrity
```

#### ERP Customer Demographics Cleaning:
```sql
• Remove 'NAS' prefix from customer IDs
• Filter out future birthdates (set to NULL)
• Normalize gender values to standardized format
```

#### ERP Location Standardization:
```sql
• Remove hyphen separators from customer IDs
• Map country codes: DE→Germany, US/USA→United States
• Handle blank/null countries as 'n/a'
```

#### ERP Product Categories:
```sql
• Copy as-is (already clean from source)
```

---

### 🟡 GOLD LAYER: Business-Ready Analytics Data (Star Schema)

**Purpose:** Provide business-ready, aggregated data modeled as a star schema for reporting and analytics.

**Characteristics:**
- Dimension and Fact table design
- Surrogate keys for referential integrity
- Denormalized for query performance
- Real-time views (no materialization)
- Business-aligned naming
- Multiple source data integration

**Dimension & Fact Tables:**

#### Dimension: `gold.dim_customers` - Customer Dimension
| Column | Type | Description |
|--------|------|-------------|
| customer_key | INT | **Surrogate key** - unique identifier for dimension |
| customer_id | INT | Business key from CRM system |
| customer_number | NVARCHAR(50) | Customer code/number |
| first_name | NVARCHAR(50) | Customer's first name |
| last_name | NVARCHAR(50) | Customer's last name |
| country | NVARCHAR(50) | Country of residence (from ERP) |
| marital_status | NVARCHAR(50) | Customer marital status |
| gender | NVARCHAR(50) | Customer gender (CRM priority, ERP fallback) |
| birthdate | DATE | Customer birthdate (from ERP) |
| create_date | DATE | Record creation date (from CRM) |

**Data Sources Integration:**
- Primary: `silver.crm_cust_info`
- Enriched with: `silver.erp_cust_az12` (birthdate, gender)
- Enriched with: `silver.erp_loc_a101` (country)

**Key Logic:**
- Surrogate key generated using ROW_NUMBER()
- Gender prioritizes CRM data; falls back to ERP if CRM is 'n/a'
- Left joins ensure all CRM customers included even without ERP data

---

#### Dimension: `gold.dim_products` - Product Dimension
| Column | Type | Description |
|--------|------|-------------|
| product_key | INT | **Surrogate key** - unique identifier for dimension |
| product_id | INT | Business key from CRM system |
| product_number | NVARCHAR(50) | Product code |
| product_name | NVARCHAR(50) | Product descriptive name |
| category_id | NVARCHAR(50) | Product category identifier |
| category | NVARCHAR(50) | Category name (Bikes, Components, etc.) |
| subcategory | NVARCHAR(50) | Product subcategory |
| maintenance | NVARCHAR(50) | Maintenance requirement indicator |
| cost | INT | Product cost |
| product_line | NVARCHAR(50) | Product line (Mountain, Road, Touring, etc.) |
| start_date | DATE | Product availability start date |

**Data Sources Integration:**
- Primary: `silver.crm_prd_info`
- Enriched with: `silver.erp_px_cat_g1v2` (category, subcategory, maintenance)

**Key Logic:**
- Surrogate key generated using ROW_NUMBER() ordered by start_date
- Filters out historical/ended products (WHERE prd_end_dt IS NULL)
- Left join ensures all CRM products included

---

#### Fact Table: `gold.fact_sales` - Sales Transactions
| Column | Type | Description |
|--------|------|-------------|
| order_number | NVARCHAR(50) | Unique sales order identifier |
| product_key | INT | **Foreign key** to dim_products |
| customer_key | INT | **Foreign key** to dim_customers |
| order_date | DATE | Date order was placed |
| shipping_date | DATE | Date order was shipped |
| due_date | DATE | Payment due date |
| sales_amount | INT | Total transaction value |
| quantity | INT | Units ordered |
| price | INT | Unit price |

**Data Sources Integration:**
- Primary: `silver.crm_sales_details`
- Linked with: `gold.dim_products` (via product_key)
- Linked with: `gold.dim_customers` (via customer_key)

**Key Logic:**
- Joins Silver sales data with Gold dimension tables
- Uses surrogate keys for efficient joins
- Left joins handle cases where dimension records don't exist

---

## Source Systems

### 1. **CRM System (Customer Relationship Management)**
   - **Tables:** Customer Info, Product Info, Sales Details
   - **Format:** CSV Files
   - **Data Characteristics:** Operational, transactional data
   - **Update Frequency:** Regular (full load strategy)

   **File Locations:**
   ```
   datasets/source_crm/cust_info.csv       - Customer master data
   datasets/source_crm/prd_info.csv        - Product catalog
   datasets/source_crm/sales_details.csv   - Sales transactions
   ```

### 2. **ERP System (Enterprise Resource Planning)**
   - **Tables:** Customer Demographics, Location, Product Categories
   - **Format:** CSV Files
   - **Data Characteristics:** Reference/master data
   - **Update Frequency:** Regular (full load strategy)

   **File Locations:**
   ```
   datasets/source_erp/CUST_AZ12.csv       - Customer demographics
   datasets/source_erp/LOC_A101.csv        - Customer locations
   datasets/source_erp/PX_CAT_G1V2.csv     - Product categories
   ```

---

## Data Flow

### ETL Process Workflow

![ETL Workflow Diagram](docs/etl-workflow.png)

### ETL Pipeline Overview:

```
STEP 1: DATA INGESTION (Source → Bronze)
├─ Read CSV files from CRM system
├─ Read CSV files from ERP system
├─ Bulk Insert into Bronze tables (BULK INSERT with FIRSTROW=2, FIELDTERMINATOR=',')
└─ No transformation, data loaded as-is

STEP 2: DATA TRANSFORMATION (Bronze → Silver)
├─ Cleanse: Trim whitespace, handle nulls, validate data types
├─ Standardize: Convert coded values to readable formats
│  ├─ Marital Status: S→Single, M→Married
│  ├─ Gender: F→Female, M→Male
│  ├─ Product Line: M→Mountain, R→Road, S→Other Sales, T→Touring
│  └─ Country: DE→Germany, US→United States
├─ Normalize: Extract category IDs, derive columns
├─ Deduplicate: Keep most recent records using ROW_NUMBER()
├─ Enrich: Add technical metadata (dwh_create_date)
└─ Load into Silver tables (Truncate & Insert)

STEP 3: DATA MODELING (Silver → Gold)
├─ Design Star Schema:
│  ├─ Dimension Tables (dim_customers, dim_products)
│  └─ Fact Table (fact_sales)
├─ Integrate multiple sources:
│  ├─ Customers: CRM + ERP (demographics + location)
│  ├─ Products: CRM + ERP (catalog + categories)
│  └─ Sales: CRM only
├─ Create Surrogate Keys using ROW_NUMBER()
├─ Implement business logic:
│  ├─ Gender priority (CRM > ERP)
│  ├─ Product lifecycle (current only, filter ended products)
│  └─ Sales hierarchy (customer → product relationships)
└─ Create Views in Gold schema (real-time, materialized views)

STEP 4: DATA QUALITY VALIDATION
├─ Silver Layer Tests:
│  ├─ Nullability checks
│  ├─ Data type validation
│  ├─ Deduplication verification
│  └─ Referential integrity checks
└─ Gold Layer Tests:
   ├─ Surrogate key uniqueness
   ├─ Foreign key relationships
   └─ Business rule validation
```

### Data Integration Diagram:

```
CRM System                  ERP System
    │                           │
    ├─ cust_info.csv           ├─ CUST_AZ12.csv
    ├─ prd_info.csv            ├─ LOC_A101.csv
    └─ sales_details.csv       └─ PX_CAT_G1V2.csv
         │                           │
         └─────────────┬─────────────┘
                       │
                    (LOAD)
                       ↓
            BRONZE LAYER (Raw)
         ┌──────────────────────┐
         │  As-is Data Storage  │
         │  (Audit Trail)       │
         └──────────────────────┘
                       │
                   (TRANSFORM)
                       ↓
           SILVER LAYER (Cleansed)
         ┌──────────────────────┐
         │ Standardized & Clean │
         │ (Ready for Analysis) │
         └──────────────────────┘
                       │
                  (MODEL + INTEGRATE)
                       ↓
            GOLD LAYER (Business Ready)
         ┌──────────────────────────────┐
         │ Star Schema (Dimensions +     │
         │ Facts) - Analytics & BI       │
         └──────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ↓              ↓              ↓
    Reports      Dashboards      Analytics
```

---

## Database Objects

### Visual Database Structure

The following image shows the complete database structure as viewed in SQL Server Object Explorer:

![Database Explorer View](docs/database-explorer.png)

### Schemas
```sql
bronze   -- Raw data layer
silver   -- Cleaned & standardized data layer
gold     -- Business-ready analytics layer
```

### Bronze Layer Tables (7 Tables)
```
✓ bronze.crm_cust_info       - Raw customer data
✓ bronze.crm_prd_info        - Raw product data
✓ bronze.crm_sales_details   - Raw sales transactions
✓ bronze.erp_cust_az12       - Raw customer demographics
✓ bronze.erp_loc_a101        - Raw customer locations
✓ bronze.erp_px_cat_g1v2     - Raw product categories
```

### Silver Layer Tables (6 Tables)
```
✓ silver.crm_cust_info       - Cleaned customer data + metadata
✓ silver.crm_prd_info        - Standardized product data + metadata
✓ silver.crm_sales_details   - Cleansed sales data + metadata
✓ silver.erp_cust_az12       - Standardized demographics + metadata
✓ silver.erp_loc_a101        - Normalized locations + metadata
✓ silver.erp_px_cat_g1v2     - Clean categories + metadata
```

### Gold Layer Views (3 Views)
```
✓ gold.dim_customers  - Customer dimension (integrated from 3 sources)
✓ gold.dim_products   - Product dimension (integrated from 2 sources)
✓ gold.fact_sales     - Sales fact table (from cleaned source)
```

### Stored Procedures (2 Procedures)
```
✓ bronze.load_bronze   - Load data from CSV → Bronze
✓ silver.load_silver   - Transform Bronze → Silver
```

---

## Installation & Setup

### Prerequisites
- SQL Server 2016 or later
- SQL Server Management Studio (SSMS)
- Read/Write access to data warehouse database
- Access to source CSV files at specified paths

### Step 1: Create Database & Schemas
```sql
-- Execute this script to initialize the database structure
EXEC sp_executesql N'
USE master;
GO
-- Creates DataWarehouse database with bronze, silver, gold schemas
'

-- File: scripts/init_database.sql
-- Action: Creates database and three schemas (bronze, silver, gold)
```

**Expected Output:**
```
Database 'DataWarehouse' created successfully
Schema 'bronze' created
Schema 'silver' created
Schema 'gold' created
```

### Step 2: Create Table Structures
```sql
-- Execute Bronze layer DDL
-- File: scripts/bronze/ddl_bronze.sql
-- Creates 6 bronze tables for raw data

-- Execute Silver layer DDL
-- File: scripts/silver/ddl_silver.sql
-- Creates 6 silver tables with metadata columns

-- Execute Gold layer DDL
-- File: scripts/gold/ddl_gold.sql
-- Creates 3 views with business logic and integrations
```

### Step 3: Configure Data Paths (IMPORTANT)
Before running the load procedures, ensure CSV file paths are correct:

**File locations to update in `proc_load_bronze.sql`:**
```
D:\Pavan\DataWarehouse\sql-data-warehouse-project-main\sql-data-warehouse-project-main\datasets\source_crm\cust_info.csv
D:\Pavan\DataWarehouse\sql-data-warehouse-project-main\sql-data-warehouse-project-main\datasets\source_crm\prd_info.csv
D:\Pavan\DataWarehouse\sql-data-warehouse-project-main\sql-data-warehouse-project-main\datasets\source_crm\sales_details.csv
D:\Pavan\DataWarehouse\sql-data-warehouse-project-main\sql-data-warehouse-project-main\datasets\source_erp\CUST_AZ12.csv
D:\Pavan\DataWarehouse\sql-data-warehouse-project-main\sql-data-warehouse-project-main\datasets\source_erp\LOC_A101.csv
D:\Pavan\DataWarehouse\sql-data-warehouse-project-main\sql-data-warehouse-project-main\datasets\source_erp\PX_CAT_G1V2.csv
```

---

## ETL Procedures

### Stored Procedure 1: `bronze.load_bronze`

**Purpose:** Load raw data from CSV files into Bronze layer tables.

**Execution:**
```sql
EXEC bronze.load_bronze;
```

**What It Does:**
1. Truncates all Bronze tables (fresh load each time)
2. Uses BULK INSERT to load CSV files
3. Configuration: FIRSTROW=2 (skip header), FIELDTERMINATOR=','
4. Provides detailed logging with load duration for each table
5. Includes error handling with try-catch

**Expected Output:**
```
================================================
Loading Bronze Layer
================================================
>> Truncating Table: bronze.crm_cust_info
>> Inserting Data Into: bronze.crm_cust_info
>> Load Duration: X seconds
...
Loading Bronze Layer is Completed
   - Total Load Duration: Y seconds
==================================================
```

**Performance Considerations:**
- Uses TABLOCK hint for parallel processing
- Full load strategy (truncate & insert)
- No index creation during bulk insert (add post-load)

---

### Stored Procedure 2: `silver.load_silver`

**Purpose:** Transform and load cleansed data from Bronze to Silver layer.

**Execution:**
```sql
EXEC silver.load_silver;
```

**What It Does:**
1. **CRM Customer Info Transformation:**
   - Trims whitespace from names
   - Normalizes marital status (S→Single, M→Married)
   - Normalizes gender (F→Female, M→Male)
   - Removes duplicates (keeps most recent by date)
   - Adds dwh_create_date metadata

2. **CRM Product Info Transformation:**
   - Extracts category ID from composite product key
   - Normalizes product line codes
   - Converts dates from DATETIME to DATE
   - Calculates end dates using window functions
   - Handles null costs (default to 0)

3. **CRM Sales Details Transformation:**
   - Converts numeric dates (YYYYMMDD format) to DATE type
   - Validates and recalculates sales amounts
   - Derives missing prices from quantity/sales
   - Handles invalid/null date scenarios

4. **ERP Customer Demographics Transformation:**
   - Removes 'NAS' prefix from customer IDs
   - Filters out future birthdates
   - Normalizes gender values
   - Adds dwh_create_date metadata

5. **ERP Location Standardization:**
   - Removes hyphen separators from IDs
   - Maps country codes (DE→Germany, US→United States)
   - Handles null/blank countries as 'n/a'

6. **ERP Product Categories:**
   - Loads as-is with metadata

**Expected Output:**
```
================================================
Loading Silver Layer
================================================
>> Truncating Table: silver.crm_cust_info
>> Inserting Data Into: silver.crm_cust_info
>> Load Duration: X seconds
...
Loading Silver Layer is Completed
   - Total Load Duration: Y seconds
==================================================
```

**Advanced Features:**
- Window functions for deduplication (ROW_NUMBER with PARTITION BY)
- Case statements for value standardization
- LEAD/LAG for derived column calculations
- Comprehensive data validation logic

---

### Gold Layer View Creation

**Purpose:** Create business-ready analytics views by integrating cleaned data from Silver.

**Views Created:**

#### View 1: `gold.dim_customers`
```sql
-- Integrates 3 sources: CRM customers + ERP demographics + ERP location
-- Generates surrogate key using ROW_NUMBER()
-- Uses left joins to preserve all CRM customers
-- Prioritizes CRM gender data, falls back to ERP
```

#### View 2: `gold.dim_products`
```sql
-- Integrates 2 sources: CRM products + ERP categories
-- Generates surrogate key with order by start_date for consistency
-- Filters current products only (prd_end_dt IS NULL)
-- Provides business-friendly naming and structure
```

#### View 3: `gold.fact_sales`
```sql
-- Links sales transactions to dimension tables
-- Uses surrogate keys for referential integrity
-- Left joins handle missing dimension records
-- Real-time view (no materialization)
```

---

## Data Quality

### Quality Checks - Silver Layer

The `tests/quality_checks_silver.sql` script includes validations for:

1. **Nullability Checks**
   - Identifies null values in critical fields
   - Validates business rule compliance

2. **Data Type Validation**
   - Ensures correct data types
   - Verifies data conversions

3. **Deduplication Verification**
   - Confirms no duplicate records
   - Validates one record per business key

4. **Referential Integrity**
   - Checks relationships between tables
   - Validates foreign key concepts

### Quality Checks - Gold Layer

The `tests/quality_checks_gold.sql` script includes:

1. **Surrogate Key Validation**
   - Checks uniqueness of surrogate keys
   - Ensures no gaps in key sequence

2. **Foreign Key Relationships**
   - Validates dimension-to-fact relationships
   - Identifies orphaned records

3. **Business Rule Validation**
   - Confirms business logic implementation
   - Validates aggregations and calculations

### Running Quality Checks

```sql
-- Execute Silver layer quality checks
EXEC sp_executesql N'SELECT * FROM quality_checks_silver';

-- Execute Gold layer quality checks
EXEC sp_executesql N'SELECT * FROM quality_checks_gold';
```

---

## Naming Conventions

### Table Naming

#### Bronze Layer: `<source_system>_<entity>`
```
bronze.crm_cust_info          -- CRM system customer data
bronze.crm_prd_info           -- CRM system product data
bronze.crm_sales_details      -- CRM system sales data
bronze.erp_loc_a101           -- ERP system location data
bronze.erp_cust_az12          -- ERP system customer data
bronze.erp_px_cat_g1v2        -- ERP system product categories
```

#### Silver Layer: `<source_system>_<entity>`
```
silver.crm_cust_info          -- Cleaned CRM customer data
silver.crm_prd_info           -- Standardized CRM product data
silver.crm_sales_details      -- Validated CRM sales data
silver.erp_loc_a101           -- Normalized ERP location data
silver.erp_cust_az12          -- Standardized ERP customer data
silver.erp_px_cat_g1v2        -- Clean ERP product categories
```

#### Gold Layer: `<category>_<entity>`
```
gold.dim_customers            -- Dimension: Customers
gold.dim_products             -- Dimension: Products
gold.fact_sales               -- Fact: Sales Transactions
```

### Column Naming

#### Surrogate Keys: `<table_name>_key`
```
customer_key                  -- Primary key in dim_customers
product_key                   -- Primary key in dim_products
```

#### Technical Columns: `dwh_<column_name>`
```
dwh_create_date              -- When record was loaded
dwh_update_date              -- When record was last updated
```

#### Source System Abbreviations:
```
crm_  -- CRM System prefixes (bronze.crm_*, silver.crm_*)
erp_  -- ERP System prefixes (bronze.erp_*, silver.erp_*)
```

---

## Getting Started

### Quick Start Guide

**1. Initialize Database (One-time setup)**
```sql
-- Open SQL Server Management Studio
-- Open: scripts/init_database.sql
-- Execute script
```

**2. Create Table Structures (One-time setup)**
```sql
-- Open: scripts/bronze/ddl_bronze.sql → Execute
-- Open: scripts/silver/ddl_silver.sql → Execute
-- Open: scripts/gold/ddl_gold.sql → Execute
```

**3. Load Data (Repeatable)**
```sql
-- Load Bronze Layer (from CSV files)
EXEC bronze.load_bronze;

-- Load Silver Layer (transform Bronze)
EXEC silver.load_silver;

-- Gold Layer views are automatically populated
-- Query them:
SELECT * FROM gold.dim_customers;
SELECT * FROM gold.dim_products;
SELECT * FROM gold.fact_sales;
```

### Recommended Queries

**View Sample Customer Data:**
```sql
SELECT TOP 10
    customer_key,
    first_name,
    last_name,
    country,
    marital_status,
    gender,
    birthdate
FROM gold.dim_customers
ORDER BY customer_key;
```

**View Sample Product Data:**
```sql
SELECT TOP 10
    product_key,
    product_name,
    category,
    subcategory,
    cost,
    product_line
FROM gold.dim_products
ORDER BY product_key;
```

**View Sample Sales Data:**
```sql
SELECT TOP 10
    f.order_number,
    c.first_name + ' ' + c.last_name AS customer_name,
    p.product_name,
    f.order_date,
    f.quantity,
    f.sales_amount
FROM gold.fact_sales f
LEFT JOIN gold.dim_customers c ON f.customer_key = c.customer_key
LEFT JOIN gold.dim_products p ON f.product_key = p.product_key
ORDER BY f.order_date DESC;
```

**Basic Analytics - Sales by Country:**
```sql
SELECT
    c.country,
    COUNT(DISTINCT f.order_number) AS order_count,
    SUM(f.quantity) AS total_quantity,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
LEFT JOIN gold.dim_customers c ON f.customer_key = c.customer_key
GROUP BY c.country
ORDER BY total_sales DESC;
```

---

## Troubleshooting

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| BULK INSERT fails | CSV path incorrect | Update file paths in proc_load_bronze.sql |
| Load procedure errors | Missing database/schema | Run init_database.sql first |
| Gold views return empty | Silver tables empty | Execute silver.load_silver first |
| Surrogate key conflicts | View definition issue | Recreate Gold views with ddl_gold.sql |
| Data quality failures | Data format issues | Check Bronze data before loading Silver |

---

**Note:** Files in bold are the key architectural diagrams referenced throughout this README.

---

## Summary

This data warehouse project demonstrates a professional, production-grade implementation of a medallion architecture data warehouse with:

✅ **Multi-layer approach** for scalability and maintainability  
✅ **Comprehensive transformations** for data quality and business readiness  
✅ **Star schema design** for efficient analytics  
✅ **Multi-source integration** combining CRM and ERP data  
✅ **Automated ETL processes** using SQL Server stored procedures  
✅ **Quality validation** for data integrity  
✅ **Professional documentation** and naming conventions  

The three-layer architecture ensures clean separation of concerns while enabling agile development and maintenance of the data warehouse.

---

**Created:** September 2026  
**Author:** Pavan (Apollo247 Team)  
**Database:** SQL Server 2016+  
**Version:** 1.0
