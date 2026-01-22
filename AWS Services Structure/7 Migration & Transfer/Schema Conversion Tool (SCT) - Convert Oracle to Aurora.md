- **What is it? (1-Sentence Pitch):** AWS Schema Conversion Tool (SCT) is a desktop application that helps you convert your existing database schema from one database engine to another (e.g., Oracle to Aurora PostgreSQL, SQL Server to Aurora MySQL).

- **Core Use Cases:**
    - Migrating databases from commercial engines (Oracle, SQL Server) to AWS (Aurora, RDS)
    - Converting database schemas as part of database migration projects
    - Assessing migration feasibility and complexity
    - Converting data warehouse schemas (Teradata, Oracle, Netezza to Redshift)
    - Migrating SQL code, stored procedures, and views

- **Key Features & Concepts:**
    - **Schema Conversion:** Converts database schema objects (tables, views, stored procedures, functions)
    - **SQL Code Conversion:** Converts SQL queries and application code
    - **Migration Assessment Report:** Analyzes schema and identifies items that cannot be automatically converted
    - **Target Platforms:**
        - **Relational:** Oracle/SQL Server → Aurora/RDS (MySQL, PostgreSQL, SQL Server)
        - **Data Warehouse:** Teradata/Oracle/Netezza → Redshift
    - **Supports Multiple Engines:** Oracle, SQL Server, MySQL, PostgreSQL, DB2, Sybase, Teradata, Netezza, Vertica
    - **Desktop Application:** Download and run on Windows, macOS, or Linux

- **How SCT Works:**
    1. **Connect to Source Database:** Point SCT to your existing database (Oracle, SQL Server, etc.)
    2. **Analyze Schema:** SCT analyzes database schema and generates migration assessment report
    3. **Review Report:** Identifies items that can/cannot be automatically converted
    4. **Convert Schema:** SCT converts schema to target database format (Aurora, RDS, Redshift)
    5. **Apply to Target:** Apply converted schema to target database
    6. **Manual Fixes:** Manually fix items that couldn't be automatically converted

- **Migration Assessment Report:**
    - **Conversion Complexity:** Easy, Medium, Complex (how difficult to migrate)
    - **Action Items:** Lists database objects that require manual intervention
    - **Recommendations:** Suggests how to handle non-convertible items
    - **Effort Estimate:** Estimates manual effort required for migration

- **SCT vs DMS (Database Migration Service):**
    | **Aspect** | **SCT** | **DMS** |
    |---|---|---|
    | **Purpose** | **Schema conversion** (structure, code) | **Data migration** (copying data) |
    | **What It Migrates** | Schema, SQL code, stored procedures | **Table data** (rows) |
    | **When to Use** | **Before migration** (convert schema first) | **During migration** (copy data) |
    | **Typical Workflow** | **SCT → convert schema → DMS → migrate data** | After SCT completes schema conversion |

- **Common Migration Paths:**
    | **Source** | **Target (AWS)** |
    |---|---|
    | Oracle | Aurora PostgreSQL, Aurora MySQL, RDS PostgreSQL |
    | SQL Server | Aurora MySQL, RDS SQL Server, RDS MySQL |
    | MySQL | Aurora MySQL, RDS MySQL |
    | PostgreSQL | Aurora PostgreSQL, RDS PostgreSQL |
    | Teradata/Netezza | Redshift |

- **Integration:**
    - **DMS:** Use SCT to convert schema, then DMS to migrate data
    - **Aurora/RDS:** Target databases for migration
    - **Redshift:** Target for data warehouse migrations
    - **S3:** Can export/import data via S3 during migration

- **Exam Tips / What to Remember:**
    - **SCT = Schema Conversion** (structure, SQL code), **DMS = Data Migration** (actual data)
    - Exam scenario: **"Migrate Oracle database to Aurora"** = Use **SCT first** (convert schema), then **DMS** (migrate data)
    - **Migration Assessment Report** shows what can/cannot be automatically converted
    - **Desktop application** (not a cloud service - download and install locally)
    - **Common path:** Commercial DB (Oracle/SQL Server) → AWS (Aurora/RDS/Redshift)
    - **Data Warehouse migration:** Teradata/Oracle/Netezza → Redshift
    - **Free tool** (no license cost)
    - SCT can convert **stored procedures, views, triggers, functions** (not just tables)
    - **Manual intervention** may be required for complex SQL code or proprietary features
