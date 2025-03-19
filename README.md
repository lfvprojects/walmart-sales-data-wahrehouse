# Walmart Data Warehouse Project - README

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

## Initial Schema Design
The initial schema followed a **star schema** with the following tables:

### **Initial Dimension Tables**
#### MyDimDate Table
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
#### MyDimProduct Table
```sql
CREATE TABLE MyDimProduct (
    productid INT PRIMARY KEY,
    productname VARCHAR(255)
);
```
#### MyDimCustomerSegment Table
```sql
CREATE TABLE MyDimCustomerSegment (
    segmentid INT PRIMARY KEY,
    segmentname VARCHAR(255)
);
```

### **Initial Fact Table**
#### MyFactSales Table
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

## Project Update: Schema Redesign
After the initial schema design, we were informed that data could not be collected in the format initially planned due to operational issues. This means that the previous tables (**MyDimDate, MyDimProduct, MyDimCustomerSegment, MyFactSales**) in the practice database and their associated attributes are no longer applicable to the current design. The company has now provided data in CSV files according to the new design. The updated schema and ETL process will now focus on integrating these CSV files into the data warehouse.

## Redesigned Data Model
The updated data warehouse schema consists of the following tables:

### **Redesigned Dimension Tables**
#### DimDate Table
```sql
CREATE TABLE DimDate (
    Dateid INT PRIMARY KEY,
    date DATE NOT NULL,
    Year INT NOT NULL,
    Quarter INT NOT NULL,
    QuarterName VARCHAR(2) NOT NULL,
    Month INT NOT NULL,
    Monthname VARCHAR(255) NOT NULL,
    Day INT NOT NULL,
    Weekday INT NOT NULL,
    WeekdayName VARCHAR(255) NOT NULL
);
```
#### DimProduct Table
```sql
CREATE TABLE DimProduct (
    Productid INT PRIMARY KEY,
    Producttype VARCHAR(255) NOT NULL
);
```
#### DimCustomerSegment Table
```sql
CREATE TABLE DimCustomerSegment (
    Segmentid INT PRIMARY KEY,
    City VARCHAR(255) NOT NULL
);
```

### **Redesigned Fact Table**
#### FactSales Table
```sql
CREATE TABLE FactSales (
    Salesid VARCHAR(255) PRIMARY KEY,
    Dateid INT NOT NULL,
    Productid INT NOT NULL,
    Segmentid INT NOT NULL,
    Price_PerUnit DECIMAL(10, 2) NOT NULL,
    QuantitySold INT NOT NULL,
    FOREIGN KEY (Dateid) REFERENCES DimDate(Dateid),
    FOREIGN KEY (Productid) REFERENCES DimProduct(Productid),
    FOREIGN KEY (Segmentid) REFERENCES DimCustomerSegment(Segmentid)
);
```

## Data Integration and Loading
- Load CSV data into the redesigned tables using the `COPY` command.
- Maintain referential integrity while inserting data.

### Example: Load CSV Data into DimDate
```sql
COPY DimDate(Dateid, date, Year, Quarter, QuarterName, Month, Monthname, Day, Weekday, WeekdayName)
FROM '/path/to/dimdate.csv'
DELIMITER ','
CSV HEADER;
```

## Create Aggregation Queries
- **Grouping Sets Query for Product Sales**:
```sql
SELECT
    p.Productid,
    p.Producttype,
    SUM(f.Price_PerUnit * f.QuantitySold) AS TotalSales
FROM
    FactSales f
INNER JOIN
    DimProduct p ON f.Productid = p.Productid
GROUP BY GROUPING SETS (
    (p.Productid, p.Producttype),
    p.Productid,
    p.Producttype,
    ()
)
ORDER BY
    p.Productid,
    p.Producttype;
```
- **Rollup Query for Year, City, and Product Sales**:
```sql
SELECT
    d.Year,
    cs.City,
    p.Productid,
    SUM(f.Price_PerUnit * f.QuantitySold) AS TotalSales 
FROM
    FactSales f
JOIN
    DimDate d ON f.Dateid = d.Dateid
JOIN
    DimProduct p ON f.Productid = p.Productid
JOIN
    DimCustomerSegment cs ON f.Segmentid = cs.Segmentid
GROUP BY ROLLUP (d.Year, cs.City, p.Productid)
ORDER BY
    d.Year DESC,
    cs.City,
    p.Productid;
```
- **Materialized View for Maximum Sales**:
```sql
CREATE MATERIALIZED VIEW max_sales AS
SELECT
    cs.City,
    p.Productid,
    p.Producttype,
    MAX(f.Price_PerUnit * f.QuantitySold) AS MaxSales
FROM
    FactSales f
JOIN
    DimProduct p ON f.Productid = p.Productid
JOIN
    DimCustomerSegment cs ON f.Segmentid = cs.Segmentid
GROUP BY
    cs.City,
    p.Productid,
    p.Producttype
WITH DATA;
```
- To refresh the materialized view with the latest data:
```sql
REFRESH MATERIALIZED VIEW max_sales;
```

## **Conclusion**
This project demonstrates the design and implementation of a data warehouse for Walmart’s sales analytics using PostgreSQL. The structured data model, optimized queries, and materialized views provide a foundation for efficient and scalable analytical reporting. 

With these design choices, the system can efficiently process large datasets, ensuring timely and accurate insights for decision-making.

