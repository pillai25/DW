Practical 1
1. Implementation of Star and Snowflake Schemas
Objective: Create a physical database structure based on a conceptual model.
Practical: Write SQL DDL to create Fact and Dimension tables. Establish primary and foreign key relationships that enforce referential integrity within a Star or Snowflake schema.

SQL DDL for Star Schema
Dimension Tables:
-- Product Dimension
CREATE TABLE Dim_Product (
    Product_ID INT PRIMARY KEY,
    Product_Name VARCHAR(100),
    Category VARCHAR(50),
    Brand VARCHAR(50)
);

-- Customer Dimension
CREATE TABLE Dim_Customer (
    Customer_ID INT PRIMARY KEY,
    Customer_Name VARCHAR(100),
    Gender VARCHAR(10),
    City VARCHAR(50),
    State VARCHAR(50)
);

-- Time Dimension
CREATE TABLE Dim_Time (
    Time_ID INT PRIMARY KEY,
    Day INT,
    Month INT,
    Quarter INT,
    Year INT
);

-- Store Dimension
CREATE TABLE Dim_Store (
    Store_ID INT PRIMARY KEY,
    Store_Name VARCHAR(100),
    City VARCHAR(50),
    State VARCHAR(50)
);

2. Fact Table: 
CREATE TABLE Fact_Sales (
    Sales_ID INT PRIMARY KEY,
    Product_ID INT,
    Customer_ID INT,
    Time_ID INT,
    Store_ID INT,
    Quantity_Sold INT,
    Sales_Amount DECIMAL(10,2),
