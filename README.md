# HR Analytics: Attrition, Pay Equity & Workforce Insights (SQL Server + Power BI)

**A relational database + SQL + Power BI dashboard project built on a real 311-employee HR dataset, designed to demonstrate data modeling, T-SQL, and business intelligence skills for HR/HCM analytics and reporting roles.**

![Dashboard](powerbi/dashboard_final.png)

---

## 1. Data Source

**[Human Resources Data Set (HRDataset_v14.csv)](https://github.com/bisquittz/Human-Resources-Dataset-Analysis)** — a widely used, publicly available HR analytics dataset: 311 real employee records, 36 fields, covering hire/termination dates, department, position, manager, salary, performance scores, engagement survey results, and attendance data. Real, structured data — not synthetically generated.

## 2. Objective

1. What does the workforce composition look like, and where is attrition concentrated?
2. Are employees paid equitably across gender and position?
3. Which recruitment channels bring in retained, high-performing employees?
4. Is there a link between manager, engagement, and performance?
5. Can these answers be delivered as a live, interactive dashboard instead of static reports?

## 3. Tech Stack

`Microsoft SQL Server` · `T-SQL` · `SSMS` · `Power BI Desktop` · `DAX`

## 4. Methodology

### Step 1 — Import
Loaded the raw CSV directly into SQL Server as a staging table (`HRDataset_v14`) using SSMS's Import Flat File wizard.

### Step 2 — Data quality check
Found that the source file's own `DeptID` and `PositionID` columns were unreliable — the same ID was reused across two different department/position names. Rather than trust broken source IDs, rebuilt clean surrogate keys from the unique department/position **names** instead.

### Step 3 — Normalization
Split the single flat staging table into 6 related tables using `SELECT ... INTO`, then layered on primary keys and foreign key constraints:

```
Departments (DeptID PK)        Positions (PositionID PK)
Managers (ManagerID PK)        RecruitmentSources (SourceID PK)
        │                              │
        └──────────────┬───────────────┘
                        ▼
              Employees (EmpID PK, DeptID/PositionID/ManagerID/SourceID FK)
                        │
                        ▼
              PerformanceReviews (EmpID PK/FK)
```

### Step 4 — SQL business queries (`sql/hr_project_mssql.sql`, `sql/07_short_extra_queries.sql`)
Wrote T-SQL queries answering the objectives above — headcount, attrition rate by department, pay equity by gender/position, recruitment source effectiveness, diversity profile, manager-vs-performance, and an at-risk (PIP) employee list.

### Step 5 — Power BI dashboard (`powerbi/dashboard_final.png`)
Connected Power BI Desktop directly to the SQL Server database (Import mode), built the relationship model (including fixing one missing relationship — `Employees.ManagerID → Managers.ManagerID` — that had no matching foreign key in SQL, so Power BI couldn't auto-detect it), then built:
- 3 KPI cards: Total Headcount, Overall Attrition %, Average Salary
- Headcount by Department (bar)
- Attrition % by Department (bar) — calculated with a DAX measure
- Average Salary by Department (column)
- Performance Score distribution (pie)
- A Department slicer, making the whole dashboard interactive

## 5. Key Findings

| Question | Finding |
|---|---|
| Overall attrition | **33.4%** of all employees ever hired have left |
| Highest-risk departments | **Production (39.7%)** and Software Engineering (36.4%) — more than double Sales (16.1%) |
| Salary by department | Executive Office shows the highest average salary — expected, as it's a single senior role, not a broad department |
| Performance distribution | 78% of employees are rated "Fully Meets"; **4% are on a Performance Improvement Plan** and warrant retention attention |
| Recruitment source | Indeed and LinkedIn bring the highest volume of hires *and* the lowest attrition (~23-24%) |
| Data quality | Source file's ID columns for department/position were inconsistent — fixed by rebuilding surrogate keys from name values |

## 6. Conclusion

Attrition in this workforce is not evenly spread — it concentrates heavily in Production and Software Engineering, both running above one-third turnover, while Sales retains far better. Pay and performance data, once properly modeled with keys and relationships, becomes queryable for exactly the questions an HR or People Analytics function needs answered day to day: where are we losing people, are we paying fairly, and which hiring channels are actually working. Turning this into an interactive Power BI dashboard — rather than a set of static reports — means a stakeholder can filter to one department and get the same set of updated numbers instantly, which is the practical value a reporting layer like this is meant to deliver.

## 7. How to Reproduce
1. Download `data/HRDataset_v14.csv`
2. In SSMS: create a database, import the CSV as a staging table (or use `BULK INSERT`)
3. Run `sql/hr_project_mssql.sql` — builds the full normalized schema, keys, and core queries
4. Run `sql/06_short_add_missing_columns.sql` then `sql/07_short_extra_queries.sql` for the extended query set
5. Open Power BI Desktop → Get Data → SQL Server → point at your database → Import all 6 tables → build relationships (see Step 5 above) → recreate the visuals

## 8. Possible Advanced Extensions
- SQL Views and Stored Procedures wrapping the core queries
- Window functions (e.g. `RANK()` for salary ranking within department)
- A second Power BI page: recruitment source effectiveness and manager-level performance view
- Python attrition-prediction model (logistic regression / random forest) layered on top of this same schema
- Publish the dashboard to Power BI Service for a shareable live link
