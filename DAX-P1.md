# DAX Notes - Pragmatic Works

Table of Contents
- [Intro to DAX](#intro-to-dax-0000)
- [Navigation Functions & Building Relationships](#navigation-functions-1000--building-relationships-2400)
- [Row Context & Filter Limitations](#row-context-1400--filter-limitations-2626)
- [Conditional Logic & The First Pass Rule](#conditional-logic-1700--the-first-pass-rule-2000)
- [Many-to-Many Relationships & Bridge Tables](#many-to-many-relationships--bridge-tables-3011)
- [RELATED vs. RELATEDTABLE](#related-vs-relatedtable-3300)
- [The SWITCH Function](#the-switch-function-3620)
- [X Functions / Iterators](#x-functions--iterators-3935)
- [Calculated Measures](#calculated-measures-4615)
- [Filter Context](#filter-context-4800)
- [Overriding Filter Context](#overriding-filter-context-4925)
- [Semi-Additive Measures](#semi-additive-measures-5130)
- [Time Intelligence & Importance of a Date Table](#time-intelligence--importance-of-a-date-table-5314)
- [Calculated Measures in Report View](#calculated-measures-in-report-view-5627)
- [Why Measures vs. Columns?](#why-measures-vs-columns-13340)
- [Modifying Filter Context with CALCULATE](#modifying-filter-context-with-calculate-14056)
- [Removing Filters](#removing-filters-14537)
- [Using OR Logic and IN for OR](#using-or-logic-15814-and-in-for-or-20100)
- [Time Series Analysis: Year-to-Date](#time-series-analysis-year-to-date-20919)
- [Comparing Current and Previous Years](#comparing-current-and-previous-years-21000)
- [Semi-Additive Measures](#semi-additive-measures-22600)
- [Opening Balance Month](#opening-balance-month-23800)
- [Context Transition](#context-transition-24500)
***
### **Intro to DAX (00:00)**
*   **DAX** stands for **Data Analysis Expressions** and is the language used in Power BI Desktop, SSAS Tabular, and Power Pivot for Excel. 
*   It was designed to be similar to Excel functions to help data analysts get up to speed quickly. 
*   DAX is primarily used to create **Calculated Columns** (to describe data better or create slicers) and **Calculated Measures** (for dynamic aggregations like total sales, ratios, and time intelligence). 
*   **Best Practice:** Whenever possible, create columns upstream (in SQL, Excel, or Power Query) rather than using DAX to allow for **better data compression**.

### **Navigation Functions (10:00) & Building Relationships (24:00)**
*   Navigation functions allow you to go to another table, grab a column, and add it to your current table, acting similarly to a `VLOOKUP` or a SQL `JOIN`. 
*   A **relationship must already exist** in your data model to use these functions. 
*   In some cases, you may need to build a relationship by creating a **key column**, which might involve combining multiple columns to ensure uniqueness on the "one" side of the relationship.

### **Row Context (14:00) & Filter Limitations (26:26)**
*   **Row Context** is created whenever you create a calculated column or use certain functions; it allows DAX to perform operations **one row at a time** (iterating). 
*   Crucially, when a row context exists, the **active relationships and filters in the data model are deactivated**. 
*   Because the data model does not automatically "see" the relationship during row iteration, you must use navigation functions to bring those relationships back to life.

### **Conditional Logic (17:00) & The First Pass Rule (20:00)**
*   DAX uses the **IF** function for conditional logic, requiring a logical test, a value if true, and a value if false. 
*   DAX follows the **"First Pass Rule"**: in a nested IF statement, any row that satisfies the first condition is eliminated from the result set for subsequent passes. 
*   This rule makes code more efficient because you do not need to define upper thresholds for every subsequent condition.

### **Many-to-Many Relationships & Bridge Tables (30:11)**
*   While some relationships are simple, complex **many-to-many relationships** may require the use of a **bridge table** to connect the data correctly.

### **RELATED vs. RELATEDTABLE (33:00)**
*   These functions are used to navigate relationships when the automatic filter is disabled in a row context.
*   **RELATED:** Used when you are on the **"many" side** (transaction table) and want to grab a single value from the **"one" side** (descriptive/dimension table).
*   **RELATEDTABLE:** Used when you are on the **"one" side** and want to retrieve all associated rows from the **"many" side**. It returns an entire table of filtered records.

### **The SWITCH Function (36:20)**
*   The **SWITCH** statement is a cleaner, more manageable alternative to nested IF statements. 
*   A common "diabolical" pattern is using `SWITCH(TRUE(), ...)` to evaluate multiple conditions, similar to a `CASE` statement in SQL.

### **X Functions / Iterators (39:35)**
*   **X Functions** (like `SUMX`, `MAXX`, `MINX`) are **iterators** that work in two parts. 
*   The **"X" part** iterates through a table row by row to perform an expression, creating a row context. 
*   The second part performs an **aggregation** (like finding the Max or Sum) on the results of those row-by-row calculations.

***

### **Calculated Measures (46:15)**
*   **Dynamic Aggregations:** Calculated measures are primarily used for metrics that need to change based on user interaction, such as **total sales, profit, margin, and ratios**.
*   **End-User Tool:** They are considered a powerful tool that makes Power BI highly dynamic, allowing users to modify reports without needing to know complex languages like SQL.
*   **Reusability:** A best practice is to **reuse existing measures** inside other measures (e.g., using `[Total Sales]` and `[Total Cost]` to calculate `[Profit]`).
*   **Table Independence:** Unlike columns, measures are not strictly tied to the table they are created in; they can be moved between tables and will still function correctly because they must have **unique names** across the entire model.

### **Filter Context (48:00)**
*   **Definition:** Filter context consists of the **active relationships and filters** currently applied to a report page.
*   **Components:** This context is shaped by attributes on visual rows/columns, slicers, the filters pane, and filters defined directly within DAX expressions.
*   **Dynamic Calculation:** A measure calculates "within the current filter context," meaning it only "sees" the subset of data allowed by the report's filters. If a relationship is deleted or inactive, the filter context for those related tables is lost.

### **Overriding Filter Context (49:25)**
*   **The CALCULATE Function:** This is the **single most important function** in DAX. It allows you to evaluate an expression within a **modified filter context**.
*   **Common Uses:** It is used to create ratios (like percent of total), compare values across time, or force a calculation for a specific category (e.g., total sales for only the "United States").
*   **Logic:** `CALCULATE` takes an expression (usually an aggregation) and applies one or more filters that override or add to the existing report filters.

### **Semi-Additive Measures (51:30)**
*   **Behavior:** These measures are **additive across some dimensions** (like customers or geography) but **not across time**.
*   **Examples:** Common scenarios include **bank account balances** or **inventory levels**. You cannot sum a bank balance over 30 days, as that would incorrectly inflate the total; instead, you usually want the balance from the **last date** in the period.
*   **Handling Blanks:** Functions like `LASTNONBLANK` are used to find the most recent period with data, ensuring that holidays or weekends don't result in blank values.

### **Time Intelligence & Importance of a Date Table (53:14)**
*   **Requirements:** To use time intelligence functions (like `TOTALYTD` or `SAMEPERIODLASTYEAR`), you **must have a dedicated Date Table** in your model.
*   **Date Table Rules:** The table must have **no gaps** (including weekends/holidays) and must cover the **full range of years** present in your data.
*   **Ease of Use:** DAX makes complex time calculations (e.g., Year-to-Date) extremely simple, often requiring only a few words of code compared to dozens of lines in SQL.

### **Calculated Measures in Report View (56:27)**
*   **Validation:** Measures should be created and validated in the **Report View** because they do not produce a static value in a table like calculated columns do.
*   **Visual Interaction:** Since measures are dynamic, you must place them into a visual (like a table or matrix) and use slicers to verify that the numbers change correctly according to the filter context.

### **Why Measures vs. Columns? (1:33:40)**
*   **Rule of Thumb:** If you are performing an **aggregation** (Sum, Count, Average), it should be a **Calculated Measure** because it is dynamic and respects all report filters.
*   **Use Cases for Columns:** Calculated columns should only be used when you need **descriptive attributes** to use in slicers or as categories on a visual axis.
*   **Performance:** Creating columns "upstream" (in SQL or Power Query) is preferred for better data compression, whereas measures are calculated on the fly and don't increase the file size as much.

***

### **Modifying Filter Context with CALCULATE (1:40:56)**
*   **The Most Important Function:** `CALCULATE` is the single most powerful function in DAX, used in approximately 80% of all measures.
*   **Functionality:** It evaluates an expression within a **modified filter context**, allowing you to override or add to the filters currently active in a report.
*   **Standard Pattern:** The syntax typically follows: `CALCULATE([Existing Measure], Filter Condition)`.
*   **Example:** To find sales specifically for the United States regardless of other filters, you would use: `CALCULATE([Total Sales], SalesTerritory[Country] = "United States")`.

### **Removing Filters (1:45:37)**
*   **ALL Function:** The `ALL` function returns all rows in a table or values in a column, **ignoring any filters** that have been applied. 
*   **REMOVEFILTERS:** This is a newer, more descriptive function that performs the same task as `ALL` when used inside `CALCULATE` to clear filters.
*   **Application (Percent of Total):** To calculate a "Percent of Total," you use `ALL` or `REMOVEFILTERS` to create a denominator that ignores specific slicers or row attributes (e.g., ignoring Country to see how one country compares to the world).

### **Using OR Logic (1:58:14) and IN for OR (2:01:00)**
*   **OR Operators:** You can use the `OR` function or the **double pipe delimiter (`||`)** to check multiple conditions.
*   **Limitations of OR:** The `OR` function itself only accepts **two parameters**, making it difficult to use for long lists of conditions.
*   **The IN Clause:** For better management of multiple conditions, the **`IN` operator** is preferred. 
    *   **Syntax:** `Column IN { "Value1", "Value2", ... }`.
    *   This is easier to read and maintain than nested `OR` statements.

### **Time Series Analysis: Year-to-Date (2:09:19)**
*   **TOTALYTD Function:** A simple way to calculate cumulative totals from the start of the year.
*   **Requirements:** Must be used with a **Date Table** that has no gaps.
*   **Fiscal Year Support:** You can define a custom fiscal year-end by using the optional **`Year_End_Date`** parameter (e.g., "06-30" for a year ending June 30th).
*   **Extra Filtering:** `TOTALYTD` also has an optional filter parameter, allowing for specialized calculations like "Year-to-Date sales for weekdays only".

### **Comparing Current and Previous Years (2:10:00)**
*   **SAMEPERIODLASTYEAR:** This function shifts the current filter context back exactly one year.
*   **Reusable Pattern:** It is used within `CALCULATE` to compare any metric (Sales, Profit, etc.) across time: `CALCULATE([Measure], SAMEPERIODLASTYEAR('Date'[Date]))`.
*   **Advanced Nesting:** You can wrap an existing YTD measure inside this pattern to create a **"Prior Year YTD"** calculation for trend analysis.

### **Semi-Additive Measures (2:26:00)**
*   **Definition:** Metrics that are additive across most dimensions (like products or geography) but **not across time** (like bank balances or inventory). 
*   **Standard Approach:** Use `LASTDATE` to find the value on the final day of a period (e.g., the balance on December 31st for a yearly report).
*   **Handling Blanks:** If the last day of a period is a holiday or weekend with no data, `LASTDATE` returns blank. In these cases, use **`LASTNONBLANK`** to find the most recent date that actually contains a value.

### **Opening Balance Month (2:38:00)**
*   **Concept:** The opening balance for the current month is technically the **closing balance of the previous month**.
*   **OPENINGBALANCEMONTH:** A dedicated function that automatically looks back at the prior month's final day.
*   **Gap Handling:** Similar to closing balances, if the last day of the prior month is blank, this function fails. A more robust solution uses **`PARALLELPERIOD`** and **`LASTNONBLANK`** to manually navigate back and find the last available data point.

### **Context Transition (2:45:00)**
*   **Definition:** The process where an existing **Row Context** is transformed into a **Filter Context**.
*   **How it is Triggered:** 
    1.  Calling a **measure** inside a calculated column (DAX implicitly wraps all measures in `CALCULATE`).
    2.  Explicitly using the **`CALCULATE`** function.
*   **Effect:** It forces the row-level data to filter the entire data model. While it can simplify code (avoiding complex `MAXX` or `RELATEDTABLE` logic), it is considered one of the most advanced and potentially confusing topics in DAX.

***

References:
*   [Pragmatic Works DAX Course](https://www.youtube.com/live/QJw4HkagVWc?si=2_vvUm9VjA8h2IuD)