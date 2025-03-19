# Walmart Data Warehouse with Postgres

## Introduction
As a data engineer hired by a consumer electronics retail company, you have been tasked with designing and implementing a data warehouse to analyze sales performance and inventory management. This project will help in generating crucial reports to assess total sales revenue by different dimensions, such as city, year, product category, and store. 

This document outlines the steps taken to complete the project, covering dimensional modeling, data loading, SQL queries, materialized views, and performance optimization.

## Project Objectives
- Develop a data warehouse to support analytical reporting.
- Design and create dimension and fact tables using best practices in data modeling.
- Populate tables with relevant data while ensuring data integrity.
- Write SQL queries for aggregation and analysis using advanced PostgreSQL features.
- Optimize performance using indexes, partitions, and materialized views.
- Ensure scalability and efficiency for business intelligence and reporting purposes.

## Technology Stack
- **Database**: PostgreSQL
- **Query Language**: SQL (PostgreSQL dialect)
- **ETL Tools**: Custom SQL scripts, Python (optional for automation)
- **Reporting & Analytics**: PostgreSQL queries, BI tools (optional for visualization)

## Data Model
The data warehouse follows a **star schema**, which consists of a central fact table surrounded by multiple dimension tables.

### **Dimension Tables**
1. **MyDimDate** - Stores date-related attributes for time-based analysis.
    - `dateid` (Primary Key)
    - `year`
    - `month`
    - `monthname`
    - `day`
    - `weekday`
    - `weekdayname`

2. **MyDimProduct** - Stores product details, including category and type.
    - `productid` (Primary Key)
    - `productname`

3. **MyDimCustomerSegment** - Categorizes customers based on segmentation criteria.
    - `segmentid` (Primary Key)
    - `segmentname`

### **Fact Table**
1. **MyFactSales** - Stores sales transactions with references to dimension tables.
    - `salesid` (Primary Key)
    - `productid` (Foreign Key to MyDimProduct)
    - `quantitysold`
    - `priceperunit`
    - `segmentid` (Foreign Key to MyDimCustomerSegment)
    - `dateid` (Foreign Key to MyDimDate)

## Implementation Tasks

### Create MyDimDate Table
```sql
CREATE TABLE MyDimDate (
    dateid INT PRIMARY KEY,
    year INT,
    month INT,
    monthname VARCHAR(20),
    day INT,
    weekday INT,
    weekdayname VARCHAR(20)
);
```
### Create MyDimProduct Table
```sql
CREATE TABLE MyDimProduct (
    productid INT PRIMARY KEY,
    productname VARCHAR(255)
);
```
### Create MyDimCustomerSegment Table
```sql
CREATE TABLE MyDimCustomerSegment (
    segmentid INT PRIMARY KEY,
    segmentname VARCHAR(255)
);
```
### Create MyFactSales Table
```sql
CREATE TABLE MyFactSales (
    salesid INT PRIMARY KEY,
    productid INT REFERENCES MyDimProduct(productid),
    quantitysold INT,
    priceperunit DECIMAL (10, 2),
    segmentid INT REFERENCES MyDimCustomerSegment(segmentid),
    dateid INT REFERENCES MyDimDate(dateid)
);
```

### Load Data
- Use SQL `INSERT` statements or `COPY` command for bulk loading.
- Maintain referential integrity.

### Create Aggregation Queries
- **Grouping Sets Query**:
```sql
SELECT year, SUM(priceperunit * quantitysold) AS TotalRevenue
FROM MyFactSales
JOIN MyDimDate ON MyFactSales.dateid = MyDimDate.dateid
GROUP BY GROUPING SETS ((year), (month), (year, month));
```
- **Rollup Query**:
```sql
SELECT year, month, SUM(priceperunit * quantitysold) AS TotalRevenue
FROM MyFactSales
JOIN MyDimDate ON MyFactSales.dateid = MyDimDate.dateid
GROUP BY ROLLUP (year, month);
```
- **Cube Query**:
```sql
SELECT year, month, productid, AVG(priceperunit * quantitysold) AS AvgRevenue
FROM MyFactSales
JOIN MyDimDate ON MyFactSales.dateid = MyDimDate.dateid
GROUP BY CUBE (year, month, productid);
```

### Create Materialized Views for Performance Optimization
```sql
CREATE MATERIALIZED VIEW max_sales AS
SELECT city, productid, MAX(priceperunit * quantitysold) AS MaxSales
FROM MyFactSales
GROUP BY city, productid;
```
- Refresh materialized views periodically:
```sql
REFRESH MATERIALIZED VIEW max_sales;
```

## **Performance Optimization Techniques**
1. **Indexes**:
   - Create indexes on frequently queried columns.
   - Example:
   ```sql
   CREATE INDEX idx_factsales_date ON MyFactSales(dateid);
   ```
2. **Partitioning**:
   - Partition `MyFactSales` by year for faster querying.
   ```sql
   CREATE TABLE MyFactSales_2024 PARTITION OF MyFactSales
   FOR VALUES IN (2024);
   ```
3. **Query Optimization**:
   - Use `EXPLAIN ANALYZE` to identify slow queries.
   - Optimize queries with proper joins and indexing.

## **Scalability Considerations**
- Use **parallel queries** to speed up large aggregations.
- Implement **incremental data loading** to avoid reprocessing.
- Consider **cloud-based solutions** for handling high-volume data.

## **Conclusion**
This project demonstrates the design and implementation of a data warehouse for Walmart’s sales analytics using PostgreSQL. The structured data model, optimized queries, and materialized views provide a foundation for efficient and scalable analytical reporting. 

With these design choices, the system can efficiently process large datasets, ensuring timely and accurate insights for decision-making.

