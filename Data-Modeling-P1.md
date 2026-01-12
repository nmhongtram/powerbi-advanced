# Data Modeling in Power BI - Pragmatic Works

Table of Contents
1.  [Importance of Data Modeling](#1-importance-of-data-modeling)
2.  [The Star Schema & Table Types](#2-the-star-schema--table-types)
3.  [Data Modeling Workflow & Implementation](#3-data-modeling-workflow--implementation)
4.  [Advanced Modeling Scenarios](#4-advanced-modeling-scenarios)
5.  [Future Proofing with Microsoft Fabric](#5-future-proofing-with-microsoft-fabric)
***
### **1. Importance of Data Modeling**
*   **DAX Simplicity:** Approximately 98% of Power BI users struggle with DAX. However, a **strong data model** allows basic DAX and filter context knowledge to go much further, reducing the need for complex code.
*   **Core Benefits:** A good model optimizes **storage constraints** (better compression), improves **report performance**, and makes managing **Row Level Security (RLS)** significantly easier.
*   **Attributes of a Good Model:** It must be **easily consumed** by end users (well-named tables), **scalable** to handle growing data volumes, **predictable** in performance, and **flexible** enough to adapt to new data sources over time.

### **2. The Star Schema & Table Types**
*   **Star Schema:** This is the ideal reporting structure where a **Fact Table** (the "many" side) sits in the middle, surrounded by **Dimension Tables** (the "one" side).
*   **Fact Tables:** Represent **business processes or events** (e.g., a sale, a student taking a class, or a phone call). They contain measurable, additive items like sales amounts or counts of interactions.
*   **Dimension Tables:** Provide the **context** (Who, What, When, Where, Why, and How) for the facts. They are typically **wide**, containing many descriptive columns/attributes like product color, customer demographics, or holiday flags.
*   **Granularity:** Data should always be stored at the **lowest level of granularity** (most detail). Higher-level storage (e.g., storing data only by month instead of day) results in a loss of **dimensionality**, preventing users from slicing data effectively.

### **3. Data Modeling Workflow & Implementation**
*   **Model Types:**
    *   **Conceptual:** High-level table names and basic flow.
    *   **Logical:** Defining specific columns and attributes within those tables.
    *   **Physical:** Technical metadata like nullability and primary/foreign keys (usually handled by the database).
*   **Power Query Process:**
    *   **Duplication:** To turn a flat "Excel-style" table into a Star Schema, duplicate the source query for each dimension.
    *   **Isolating Dimensions:** Use "Choose Columns" to keep only relevant attributes and "Remove Duplicates" to ensure each row in a dimension table is unique.
    *   **Surrogate Keys:** If a natural key (like Product ID) isn't unique or available, create an **Index Column** to act as a unique surrogate key for relationships.
*   **Date Table:** A dedicated Date Table with **no gaps** is mandatory for **Time Intelligence** (e.g., Year-to-Date, Prior Year comparisons). If your relationship uses an integer key instead of a date data type, you must **"Mark as Date Table"** in Power BI.

### **4. Advanced Modeling Scenarios**
*   **Snowflake Schema:** Occurs when a dimension table filters another dimension table before filtering the Fact table (e.g., Geography → Customer → Sales). While it should be avoided to keep models simple, it is often necessary in mature environments.
*   **Multiple Fact Tables:** As models grow, they often include multiple facts (e.g., Sales and Budget). If these tables have **different granularity** (e.g., Budget is at the Category level while Sales is at the Product level), you must build a "higher-level" dimension (like Category) to link them.
*   **Role-Playing Dimensions:** Used when one table has multiple relationships to the same dimension (e.g., Order Date vs. Ship Date).
    *   **Option 1:** Duplicate the dimension table (Ship Date table and Order Date table).
    *   **Option 2 (Preferred):** Use **inactive relationships** and the DAX function `USERELATIONSHIP` to specify which date to use in a specific calculation.
*   **Context Transition:** The process of turning a Row Context into a Filter Context, often triggered by using `CALCULATE` or calling a measure within a calculated column.

### **5. Future Proofing with Microsoft Fabric**
*   **Unified Solution:** Fabric is an end-to-end analytical platform that allows data analysts, scientists, and engineers to work on the same data.
*   **One Lake:** Acts as "OneDrive for data," providing a centralized storage location that reduces **data redundancy** through **shortcuts** (referencing data without duplicating it).
*   **Direct Lake:** A new high-performance connectivity mode for semantic models in Fabric that provides the speed of **Import** mode with the real-time nature of **Direct Query**.

***

References:
*  [Pragmatic Works Data Modeling Course](https://youtu.be/air7T8wCYkU?si=gH9godOwO-nqe3WD)