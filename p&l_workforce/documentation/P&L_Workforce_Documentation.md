# P&L Workforce Analytics Project Documentation

## 📋 Project Overview

The **P&L Workforce** project is a comprehensive data analytics solution designed to track and analyze workforce costs, payroll expenses, and financial performance across departments and companies. The project implements a **Medallion Architecture** (Bronze → Silver → Gold) to progressively clean, transform, and aggregate data for business intelligence and decision-making.

**Key Business Objectives:**
* Track workforce compensation costs across departments and companies
* Monitor P&L (Profit & Loss) metrics through general ledger analysis
* Calculate critical KPIs for financial and workforce performance
* Support data-driven decision making for HR and Finance teams

**Data Source:** Azure Blob Storage (integrated via Fivetran)

---

## 📁 Folder Structure

```
p&l_workforce/
├── 01_silver/          # Data Cleaning & Transformation Layer
│   ├── nb_trn_employee
│   ├── nb_trn_payroll
│   ├── nb_trn_general_ledgers
│   ├── nb_trn_departments
│   ├── nb_trn_company
│   ├── nb_trn_account
│   └── nb_salary_fix
│
└── 02_gold/            # Business Analytics Layer
    ├── nb_dim          # Dimension Tables
    ├── nb_fact         # Fact Tables
    └── nb_kpis         # Key Performance Indicators
```

---

## 🏗️ Data Architecture

### Medallion Architecture Pattern

The project follows the **Medallion Architecture** with three layers:

1. **Bronze Layer** (Source):
   * Raw data from Azure Blob Storage
   * Catalog: `bronze_workforce.azure_blob_storage`
   * Contains original files with Fivetran metadata columns

2. **Silver Layer** (Cleaned & Conformed):
   * Cleaned and standardized data
   * Catalog: `silver_workforce.azure_blob_storage`
   * Removes duplicates, fixes data types, standardizes dates
   * Removes Fivetran technical columns

3. **Gold Layer** (Business-Ready):
   * Dimensional model with fact and dimension tables
   * Catalog: `gold_workforce.azure_blob_storage`
   * Ready for reporting, dashboards, and analytics

---

## 🔧 Silver Layer - Data Transformation

The Silver layer contains 7 notebooks that clean and transform raw data from the Bronze layer.

### Common Transformation Patterns

All Silver notebooks follow these standard steps:

1. **Read from Bronze**: Load raw data from `bronze_workforce.azure_blob_storage`
2. **Remove Technical Columns**: Drop Fivetran metadata (`_file`, `_line`, `_modified`, `_fivetran_synced`)
3. **Data Type Casting**: Convert string columns to appropriate types (int, date, etc.)
4. **Date Standardization**: Parse dates from "dd-MM-yyyy HH:mm" format to proper date/timestamp
5. **Deduplication**: Remove duplicate records
6. **Data Quality**: Apply business rules and transformations
7. **Write to Silver**: Save cleaned data as Delta tables to `silver_workforce.azure_blob_storage`

---

### 📓 Notebook Details

#### 1. **nb_trn_employee**
**Purpose:** Clean and transform employee master data

**Key Transformations:**
* Drops Fivetran metadata columns
* Removes duplicate employee records
* Creates `full_name` by concatenating `first_name` and `last_name`
* Casts IDs to integers: `employee_id`, `company_id`, `department_id`
* Standardizes dates:
  * `hire_date`: Converts from "dd-MM-yyyy HH:mm" to date format
  * `termination_date`: Converts from "dd-MM-yyyy HH:mm" to date format
* Reorders columns to place `full_name` after `employee_code`

**Output Table:** `silver_workforce.azure_blob_storage.employee`

---

#### 2. **nb_trn_payroll**
**Purpose:** Clean and transform payroll transaction data

**Key Transformations:**
* Drops Fivetran metadata columns
* Standardizes three date fields:
  * `pay_period_start`: Converts to date format
  * `pay_period_end`: Converts to date format
  * `pay_date`: Converts to date format
* Removes duplicate payroll records
* Casts IDs to integers: `payroll_id`, `employee_id`, `company_id`, `department_id`

**Key Columns:**
* Compensation: `gross_salary`, `bonus`, `overtime_pay`, `commission`, `allowances`
* Deductions: `tax_deduction`, `social_security`, `health_insurance`, `retirement_contribution`, `other_deductions`
* Payment info: `payment_method`, `status`, `net_salary`

**Output Table:** `silver_workforce.azure_blob_storage.payroll`

---

#### 3. **nb_trn_general_ledgers**
**Purpose:** Clean and transform general ledger financial transactions

**Key Transformations:**
* Drops Fivetran metadata columns
* Casts IDs to integers: `gl_id`, `company_id`, `account_id`, `department_id`
* Casts fiscal info to integers: `fiscal_year`, `fiscal_period`, `created_by`, `approved_by`
* Standardizes dates:
  * `entry_date`: Converts to date format
  * `posting_date`: Converts to date format
* Removes duplicate GL entries

**Transaction Types:** Payment, Reversal, Accrual, Journal Entry, Adjustment, Invoice

**Output Table:** `silver_workforce.azure_blob_storage.general_ledgers`

---

#### 4. **nb_trn_departments**
**Purpose:** Clean and transform department master data

**Transformations:**
* Standard cleaning process (metadata removal, type casting, deduplication)

**Output Table:** `silver_workforce.azure_blob_storage.departments`

---

#### 5. **nb_trn_company**
**Purpose:** Clean and transform company master data

**Transformations:**
* Standard cleaning process (metadata removal, type casting, deduplication)

**Companies:**
* TechNova Industries (Technology, USA)
* Nova AI (Technology, USA)

**Output Table:** `silver_workforce.azure_blob_storage.company`

---

#### 6. **nb_trn_account**
**Purpose:** Clean and transform chart of accounts data

**Transformations:**
* Standard cleaning process (metadata removal, type casting, deduplication)

**Output Table:** `silver_workforce.azure_blob_storage.accounts`

---

#### 7. **nb_salary_fix**
**Purpose:** Apply corrections or fixes to salary data

**Note:** This appears to be a utility notebook for one-time data corrections

---

## 💎 Gold Layer - Dimensional Model

The Gold layer implements a **Star Schema** with dimension and fact tables optimized for analytics.

---

### 📊 Dimension Tables

Dimension tables are created in **nb_dim** notebook. Each dimension table includes:
* All attributes from Silver layer
* A surrogate key (generated using `monotonically_increasing_id()`)

#### **dim_employee**
* Source: `silver_workforce.azure_blob_storage.employee`
* Surrogate Key: `employee_key`
* Contains: Employee details, hire date, termination date, position, full name, status

#### **dim_company**
* Source: `silver_workforce.azure_blob_storage.company`
* Surrogate Key: `company_key`
* Contains: `company_id`, `company_name`, `industry`, `country`
* Companies: TechNova Industries, Nova AI

#### **dim_departments**
* Source: `silver_workforce.azure_blob_storage.departments`
* Surrogate Key: `department_key`
* Contains: `department_id`, `department_name`
* 20 Departments: SALES, MARKETING, FINANCE, HR, IT, OPERATIONS, CUSTOMER SERVICE, LEGAL, R&D, PROCUREMENT, LOGISTICS, QUALITY ASSURANCE, PRODUCTION, WAREHOUSE, ADMINISTRATION, BUSINESS DEVELOPMENT, COMPLIANCE, INTERNAL AUDIT, STRATEGY, CORPORATE COMMUNICATIONS

#### **dim_accounts**
* Source: `silver_workforce.azure_blob_storage.accounts`
* Surrogate Key: `account_key`
* Contains: Chart of accounts with account codes, names, types, categories, and status

**All dimension tables stored in:** `gold_workforce.azure_blob_storage.dim_*`

---

### 📈 Fact Tables

Fact tables are created in **nb_fact** notebook by joining Silver layer transactional data with Gold layer dimensions.

#### **fact_payroll**
**Purpose:** Contains payroll transactions with all compensation and deduction details

**Source Tables:**
* `silver_workforce.azure_blob_storage.payroll`
* `gold_workforce.azure_blob_storage.dim_employee`
* `gold_workforce.azure_blob_storage.dim_departments`
* `gold_workforce.azure_blob_storage.dim_company`

**Join Type:** INNER JOIN on employee_id, department_id, company_id

**Measures:**
* Compensation: `base_salary`, `bonus`, `overtime_pay`, `commission`, `allowances`
* Deductions: `tax_deduction`, `social_security`, `health_insurance`, `retirement_contribution`, `other_deductions`
* Net: `net_salary`

**Dimensions (Surrogate Keys):**
* `employee_key`
* `department_key`
* `company_key`

**Other Attributes:**
* Date info: `pay_period_start`, `pay_period_end`, `pay_date`
* Employee info: `termination_date`, `is_active`

**Output Table:** `gold_workforce.azure_blob_storage.fact_payroll`

---

#### **fact_general_ledgers**
**Purpose:** Contains general ledger transactions for P&L analysis

**Source Tables:**
* `silver_workforce.azure_blob_storage.general_ledgers`
* `gold_workforce.azure_blob_storage.dim_departments`
* `gold_workforce.azure_blob_storage.dim_accounts`
* `gold_workforce.azure_blob_storage.dim_company`

**Join Type:** INNER JOIN on department_id, account_id, company_id

**Measures:**
* `debit_amount`
* `credit_amount`
* `total_amount` (calculated as `credit_amount + debit_amount`)

**Dimensions (Surrogate Keys):**
* `department_key`
* `account_key`
* `company_key`

**Other Attributes:**
* GL info: `gl_id`, `fiscal_year`, `fiscal_period`
* Dates: `entry_date`, `posting_date`
* Audit: `created_by`, `approved_by`

**Output Table:** `gold_workforce.azure_blob_storage.fact_general_ledgers`

---

## 📊 KPIs and Analytics

The **nb_kpis** notebook contains SQL queries to calculate critical business metrics.

### Key Performance Indicators

#### **KPI 1: Monthly Revenue with Month-over-Month Growth**
**Metric:** Total revenue by month with growth percentage

**Calculation:**
```sql
SELECT
  date_trunc('month', posting_date) AS month,
  SUM(total_amount) AS total_revenue,
  ROUND(100 * (SUM(total_amount) - LAG(SUM(total_amount)) OVER (ORDER BY month)) 
    / NULLIF(LAG(SUM(total_amount)) OVER (ORDER BY month), 0), 2) AS mom_growth_pct
FROM fact_general_ledgers
GROUP BY month
```

**Business Value:** Tracks revenue trends and identifies growth patterns month-by-month

---

#### **KPI 2: Cost of Sales by Month and Company**
**Metric:** Monthly cost of sales segmented by company

**Calculation:**
```sql
SELECT
  date_trunc('month', gl.posting_date) AS month,
  gl.company_id,
  SUM(gl.total_amount) AS cost_of_sales
FROM fact_general_ledgers gl
JOIN dim_accounts da ON gl.account_id = da.account_id
WHERE da.category = 'Cost of Sales'
GROUP BY month, company_id
```

**Business Value:** Monitors direct costs related to revenue generation

---

#### **KPI 5: Average Total Compensation by Position and Company**
**Metric:** Average compensation (salary + bonus + overtime + commission) by position level

**Calculation:**
```sql
SELECT
  e.position,
  c.company_id,
  AVG(p.base_salary + p.bonus + p.overtime_pay + p.commission) AS avg_total_compensation
FROM dim_employee e
JOIN fact_payroll p ON e.employee_id = p.employee_id
JOIN dim_company c ON p.company_id = c.company_id
GROUP BY e.position, c.company_id
```

**Positions Analyzed:** Director, Executive, Junior, Manager, Senior, VP

**Business Value:** Helps benchmark compensation across roles and companies

---

#### **KPI 6: Monthly Net Profit**
**Metric:** Net profit calculated from all P&L categories

**Calculation:**
```sql
SELECT
  date_trunc('month', gl.posting_date) AS month,
  SUM(gl.total_amount) AS net_profit
FROM fact_general_ledgers gl
JOIN dim_accounts da ON gl.account_id = da.account_id
WHERE da.category IN ('Revenue', 'Cost of Sales', 'Operating Expenses', 
                       'Other Income', 'Other Expenses')
GROUP BY month
```

**Business Value:** Tracks overall profitability trends

---

#### **KPI 7: Variable Compensation Analysis by Department**
**Metric:** Overtime pay, bonuses, and their ratio to base salary by department

**Calculation:**
```sql
SELECT
  d.department_name,
  SUM(p.overtime_pay) AS total_overtime_pay,
  SUM(p.bonus) AS total_bonus,
  SUM(p.overtime_pay + p.bonus) AS total_variable_compensation,
  SUM(p.overtime_pay) / NULLIF(SUM(p.base_salary), 0) AS overtime_to_base_salary_ratio
FROM dim_departments d
JOIN fact_payroll p ON d.department_id = p.department_id
GROUP BY d.department_name
```

**Business Value:** Identifies departments with high variable compensation costs

---

#### **KPI 8: Total Payroll Expenses by Department**
**Metric:** Total compensation costs per department

**Calculation:**
```sql
SELECT
  d.department_name,
  SUM(p.base_salary + p.bonus + p.overtime_pay + p.commission) AS total_payroll_expenses
FROM dim_departments d
JOIN fact_payroll p ON d.department_id = p.department_id
GROUP BY d.department_name
```

**Business Value:** Shows workforce cost allocation across organizational units

---

## 🔄 Data Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│           BRONZE LAYER (Raw Data)                       │
│         bronze_workforce.azure_blob_storage             │
├─────────────────────────────────────────────────────────┤
│  • employee           • general_ledgers                 │
│  • payroll            • departments                     │
│  • company            • accounts                        │
└────────────────┬────────────────────────────────────────┘
                 │ Fivetran Sync from Azure Blob Storage
                 ↓
┌─────────────────────────────────────────────────────────┐
│          SILVER LAYER (Cleaned Data)                    │
│        silver_workforce.azure_blob_storage              │
├─────────────────────────────────────────────────────────┤
│  Transformations:                                       │
│  • Remove Fivetran columns                              │
│  • Cast data types                                      │
│  • Standardize dates                                    │
│  • Remove duplicates                                    │
│  • Apply business rules                                 │
├─────────────────────────────────────────────────────────┤
│  Notebooks:                                             │
│  nb_trn_employee | nb_trn_payroll                       │
│  nb_trn_general_ledgers | nb_trn_departments            │
│  nb_trn_company | nb_trn_account                        │
└────────────────┬────────────────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────┐
│           GOLD LAYER (Analytics Ready)                  │
│         gold_workforce.azure_blob_storage               │
├─────────────────────────────────────────────────────────┤
│  DIMENSIONS (nb_dim):                                   │
│  • dim_employee        • dim_company                    │
│  • dim_departments     • dim_accounts                   │
│                                                         │
│  FACTS (nb_fact):                                       │
│  • fact_payroll                                         │
│  • fact_general_ledgers                                 │
│                                                         │
│  ANALYTICS (nb_kpis):                                   │
│  • Revenue & Growth Analysis                            │
│  • Cost Analysis                                        │
│  • Compensation Benchmarking                            │
│  • Profitability Metrics                                │
│  • Departmental Cost Analysis                           │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 Key Tables Summary

### Bronze Layer
| Table | Description | Source |
|-------|-------------|--------|
| `employee` | Raw employee master data | Azure Blob Storage (via Fivetran) |
| `payroll` | Raw payroll transactions | Azure Blob Storage (via Fivetran) |
| `general_ledgers` | Raw GL transactions | Azure Blob Storage (via Fivetran) |
| `departments` | Raw department data | Azure Blob Storage (via Fivetran) |
| `company` | Raw company data | Azure Blob Storage (via Fivetran) |
| `accounts` | Raw chart of accounts | Azure Blob Storage (via Fivetran) |

### Silver Layer
| Table | Description | Notebook |
|-------|-------------|----------|
| `employee` | Cleaned employee data with full names | nb_trn_employee |
| `payroll` | Cleaned payroll with standardized dates | nb_trn_payroll |
| `general_ledgers` | Cleaned GL with proper types | nb_trn_general_ledgers |
| `departments` | Cleaned department data | nb_trn_departments |
| `company` | Cleaned company data | nb_trn_company |
| `accounts` | Cleaned account data | nb_trn_account |

### Gold Layer - Dimensions
| Table | Description | Surrogate Key | Notebook |
|-------|-------------|---------------|----------|
| `dim_employee` | Employee dimension | employee_key | nb_dim |
| `dim_company` | Company dimension | company_key | nb_dim |
| `dim_departments` | Department dimension | department_key | nb_dim |
| `dim_accounts` | Account dimension | account_key | nb_dim |

### Gold Layer - Facts
| Table | Description | Grain | Notebook |
|-------|-------------|-------|----------|
| `fact_payroll` | Payroll transactions | One row per payroll record | nb_fact |
| `fact_general_ledgers` | GL transactions | One row per GL entry | nb_fact |

---

## 🚀 How to Use This Project

### For Data Engineers:
1. **Silver Layer**: Run notebooks in `01_silver/` folder in any order (they're independent)
2. **Gold Layer**: 
   * First, run `nb_dim` to create dimension tables
   * Then, run `nb_fact` to create fact tables
   * Finally, run `nb_kpis` to calculate metrics

### For Analysts:
* Query the Gold layer tables directly:
  * `gold_workforce.azure_blob_storage.fact_payroll`
  * `gold_workforce.azure_blob_storage.fact_general_ledgers`
  * Use dimension tables for filtering and grouping

### For Business Users:
* Review KPI queries in `nb_kpis` notebook
* Build dashboards on top of Gold layer tables
* Focus on fact tables joined with relevant dimensions

---

## 📝 Notes and Best Practices

1. **Data Refresh**: Silver and Gold layers should be refreshed after Bronze layer updates
2. **Delta Lake**: All tables use Delta format for ACID transactions and time travel
3. **Incremental Processing**: Currently uses full refresh (overwrite mode)
4. **Date Formats**: All dates standardized to "dd-MM-yyyy" format from source
5. **Deduplication**: Applied at Silver layer to ensure data quality
6. **Surrogate Keys**: Generated using `monotonically_increasing_id()` for dimension tables

---

## 🔍 Common Queries

### Find all active employees:
```sql
SELECT * FROM gold_workforce.azure_blob_storage.dim_employee
WHERE is_active = true
```

### Calculate total payroll by department:
```sql
SELECT 
  d.department_name,
  SUM(f.net_salary) as total_payroll
FROM gold_workforce.azure_blob_storage.fact_payroll f
JOIN gold_workforce.azure_blob_storage.dim_departments d 
  ON f.department_id = d.department_id
GROUP BY d.department_name
```

### View P&L by account category:
```sql
SELECT 
  a.category,
  SUM(f.total_amount) as total
FROM gold_workforce.azure_blob_storage.fact_general_ledgers f
JOIN gold_workforce.azure_blob_storage.dim_accounts a 
  ON f.account_id = a.account_id
GROUP BY a.category
```

---

## 📧 Contact & Support

For questions or issues with this project, contact the data engineering team.

---

**Last Updated:** 2026-03-27  
**Version:** 1.0  
**Maintained By:** Data Engineering Team