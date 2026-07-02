**Practical 1

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


**practical no 2
**
2. Complex Joins for De-normalization
Objective: Flatten normalized data into a warehouse-ready format.
Practical: Use Self-Joins, Multiple Inner/Outer Joins, and Cross Joins to combine data from 5+ normalized tables into a single wide "denormalized" view for reporting.



   CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    City VARCHAR(50)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    EmployeeID INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE OrderDetails (
    OrderDetailID INT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(100),
    CategoryID INT
);


CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    CategoryName VARCHAR(100)
);

CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    ManagerID INT
);

CREATE VIEW Sales_Report AS
SELECT
    o.OrderID,
    o.OrderDate,

    c.CustomerID,
    c.CustomerName,
    c.City,

    p.ProductID,
    p.ProductName,

    cat.CategoryName,

    od.Quantity,

    e.EmployeeName AS Salesperson,
    m.EmployeeName AS Manager

FROM Orders o

INNER JOIN Customers c
ON o.CustomerID = c.CustomerID

INNER JOIN OrderDetails od
ON o.OrderID = od.OrderID

INNER JOIN Products p
ON od.ProductID = p.ProductID

LEFT JOIN Categories cat
ON p.CategoryID = cat.CategoryID

INNER JOIN Employees e
ON o.EmployeeID = e.EmployeeID

LEFT JOIN Employees m
ON e.ManagerID = m.EmployeeID;

SELECT
    p.ProductName,
    c.CategoryName
FROM Products p
CROSS JOIN Categories c;


**practical no 3
**Advanced Aggregations with ROLLUP and CUBE
Objective: Create multi-level summary reports.
Practical: Use the GROUP BY ROLLUP and GROUP BY CUBE clauses to generate hierarchical subtotals (e.g., Sales by Day > Month > Year) in a single query result set.

CREATE TABLE Sales (
    SaleID NUMBER PRIMARY KEY,
    SaleDate DATE,
    Product VARCHAR2(50),
    Amount NUMBER
);
INSERT INTO Sales VALUES (1, DATE '2025-01-10', 'Laptop', 50000);
INSERT INTO Sales VALUES (2, DATE '2025-01-15', 'Mouse', 1500);
INSERT INTO Sales VALUES (3, DATE '2025-02-12', 'Laptop', 55000);
INSERT INTO Sales VALUES (4, DATE '2025-02-20', 'Chair', 6000);
INSERT INTO Sales VALUES (5, DATE '2026-01-05', 'Laptop', 52000);
INSERT INTO Sales VALUES (6, DATE '2026-01-18', 'Mouse', 1800);
INSERT INTO Sales VALUES (7, DATE '2026-02-22', 'Chair', 7000);

COMMIT;

SELECT * FROM Sales;

SELECT
    EXTRACT(YEAR FROM SaleDate) AS Year,
    EXTRACT(MONTH FROM SaleDate) AS Month,
    SUM(Amount) AS TotalSales
FROM Sales
GROUP BY ROLLUP(
    EXTRACT(YEAR FROM SaleDate),
    EXTRACT(MONTH FROM SaleDate)
)
ORDER BY Year, Month;

SELECT
    EXTRACT(YEAR FROM SaleDate) AS Year,
    Product,
    SUM(Amount) AS TotalSales
FROM Sales
GROUP BY CUBE(
    EXTRACT(YEAR FROM SaleDate),
    Product
)
ORDER BY Year, Product;

SELECT
    EXTRACT(YEAR FROM SaleDate) AS Year,
    EXTRACT(MONTH FROM SaleDate) AS Month,
    EXTRACT(DAY FROM SaleDate) AS Day,
    SUM(Amount) AS TotalSales
FROM Sales
GROUP BY ROLLUP(
    EXTRACT(YEAR FROM SaleDate),
    EXTRACT(MONTH FROM SaleDate),
    EXTRACT(DAY FROM SaleDate)
)
ORDER BY Year, Month, Day;

SELECT
    EXTRACT(YEAR FROM SaleDate) AS Year,
    EXTRACT(MONTH FROM SaleDate) AS Month,
    Product,
    SUM(Amount) AS TotalSales
FROM Sales
GROUP BY CUBE(
    EXTRACT(YEAR FROM SaleDate),
    EXTRACT(MONTH FROM SaleDate),
    Product
)
ORDER BY Year, Month, Product;

**Practical no 4
**
Ranking and Window Functions
Objective: Analyze data relative to other rows without grouping.
Practical: Implement RANK(), DENSE_RANK(), and ROW_NUMBER() to find "Top N" products per category or identify the highest-earning employees in each department.




CREATE TABLE Employee2 (
    EmpID NUMBER PRIMARY KEY,
    EmpName VARCHAR2(50),
    Department VARCHAR2(30),
    Salary NUMBER
);
INSERT INTO Employee2 VALUES (101,'Amit','IT',60000);
INSERT INTO Employee2 VALUES (102,'Rahul','IT',75000);
INSERT INTO Employee2 VALUES (103,'Priya','IT',75000);
INSERT INTO Employee2 VALUES (104,'Sneha','HR',50000);
INSERT INTO Employee2 VALUES (105,'Rohan','HR',65000);
INSERT INTO Employee2 VALUES (106,'Kiran','Sales',55000);
INSERT INTO Employee2 VALUES (107,'Neha','Sales',70000);

COMMIT;
SELECT * FROM Employee2;

SELECT
    EmpID,
    EmpName,
    Department,
    Salary,
    RANK() OVER
    (PARTITION BY Department
    ORDER BY Salary DESC) AS Rank_No
FROM Employee2;

SELECT
    EmpID,
    EmpName,
    Department,
    Salary,
    DENSE_RANK() OVER
    (PARTITION BY Department
    ORDER BY Salary DESC) AS Dense_Rank
FROM Employee2;

SELECT
    EmpID,
    EmpName,
    Department,
    Salary,
    ROW_NUMBER() OVER
    (PARTITION BY Department
    ORDER BY Salary DESC) AS Row_No
FROM Employee2;

SELECT *
FROM
(
    SELECT
        EmpID,
        EmpName,
        Department,
        Salary,
        ROW_NUMBER() OVER
        (
            PARTITION BY Department
            ORDER BY Salary DESC
        ) AS Row_No
    FROM Employee2
)
WHERE Row_No <= 2;

CREATE TABLE ProductSales (
    ProductID NUMBER PRIMARY KEY,
    ProductName VARCHAR2(50),
    Category VARCHAR2(30),
    Sales NUMBER
);

INSERT INTO ProductSales VALUES (1,'Laptop','Electronics',90000);
INSERT INTO ProductSales VALUES (2,'Mouse','Electronics',25000);
INSERT INTO ProductSales VALUES (3,'Keyboard','Electronics',35000);
INSERT INTO ProductSales VALUES (4,'Chair','Furniture',45000);
INSERT INTO ProductSales VALUES (5,'Table','Furniture',60000);

COMMIT;

SELECT *
FROM
(
    SELECT
        ProductID,
        ProductName,
        Category,
        Sales,
        RANK() OVER
        (
            PARTITION BY Category
            ORDER BY Sales DESC
        ) AS Rank_No
    FROM ProductSales
)
WHERE Rank_No = 1;





