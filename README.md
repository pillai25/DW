**Practical 1

1. Implementation of Star and Snowflake Schemas
Objective: Create a physical database structure based on a conceptual model.
Practical: Write SQL DDL to create Fact and Dimension tables. Establish primary and foreign key relationships
that enforce referential integrity within a Star or Snowflake schema.

SQL DDL for Star Schema
Dimension Tables:
CREATE TABLE Dim_Product (
    Product_ID INT PRIMARY KEY,
    Product_Name VARCHAR(100),
    Category VARCHAR(50),
    Brand VARCHAR(50)
);

CREATE TABLE Dim_Customer (
    Customer_ID INT PRIMARY KEY,
    Customer_Name VARCHAR(100),
    Gender VARCHAR(10),
    City VARCHAR(50),
    State VARCHAR(50)
);

CREATE TABLE Dim_Time (
    Time_ID INT PRIMARY KEY,
    Day INT,
    Month INT,
    Quarter INT,
    Year INT
);

CREATE TABLE Dim_Store (
    Store_ID INT PRIMARY KEY,
    Store_Name VARCHAR(100),
    City VARCHAR(50),
    State VARCHAR(50)
);

2. Fact Table: 
CREATE TABLE Fact_Sales303 (
    Sales_ID INT PRIMARY KEY,
    Product_ID INT,
    Customer_ID INT,
    Time_ID INT,
    Store_ID INT,
    Quantity_Sold INT,
    Sales_Amount DECIMAL(10,2),

    CONSTRAINT FK_FS_PRODUCT
        FOREIGN KEY (Product_ID)
        REFERENCES Dim_Product(Product_ID),

    CONSTRAINT FK_FS_CUSTOMER
        FOREIGN KEY (Customer_ID)
        REFERENCES Dim_Customer(Customer_ID)
);


**practical no 2
**
2. Complex Joins for De-normalization
Objective: Flatten normalized data into a warehouse-ready format.
Practical: Use Self-Joins, Multiple Inner/Outer Joins, and Cross Joins to combine data 
from 5+ normalized tables into a single wide "denormalized" view for reporting.

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
Practical: Use the GROUP BY ROLLUP and GROUP BY CUBE clauses to generate hierarchical subtotals 
(e.g., Sales by Day > Month > Year) in a single query result set.

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
Practical: Implement RANK(), DENSE_RANK(), and ROW_NUMBER() to find "Top N" products per category
or identify the highest-earning employees in each department.

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



**Practical no 5
**
Compare current performance against previous periods.
Practical: Use LAG() and LEAD() window functions to calculate Month-over-Month (MoM) 
growth or identify trends in historical sales data.

CREATE TABLE MonthlySales (
    MonthID INT PRIMARY KEY,
    MonthName VARCHAR(20),
    Sales DECIMAL(10,2)
);

INSERT INTO MonthlySales VALUES (1,'January',50000);
INSERT INTO MonthlySales VALUES (2,'February',55000);
INSERT INTO MonthlySales VALUES (3,'March',60000);
INSERT INTO MonthlySales VALUES (4,'April',58000);
INSERT INTO MonthlySales VALUES (5,'May',65000);
INSERT INTO MonthlySales VALUES (6,'June',70000);

COMMIT;
SELECT * FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LAG(Sales) OVER (ORDER BY MonthID) AS Previous_Month_Sales
FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LEAD(Sales) OVER (ORDER BY MonthID) AS Next_Month_Sales
FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LAG(Sales) OVER (ORDER BY MonthID) AS Previous_Sales,
    ROUND(
        ((Sales - LAG(Sales) OVER (ORDER BY MonthID))
        / LAG(Sales) OVER (ORDER BY MonthID)) * 100,
        2
    ) AS MoM_Growth_Percentage

FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LAG(Sales) OVER (ORDER BY MonthID) AS Previous_Sales,
    CASE
        WHEN LAG(Sales) OVER (ORDER BY MonthID) IS NULL
            THEN 'No Previous Data'
        WHEN Sales > LAG(Sales) OVER (ORDER BY MonthID)
            THEN 'Increasing'
        WHEN Sales < LAG(Sales) OVER (ORDER BY MonthID)
            THEN 'Decreasing'
        ELSE 'No Change'
    END AS Sales_Trend

FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LEAD(Sales) OVER (ORDER BY MonthID) AS Next_Month_Sales,
    LEAD(Sales) OVER (ORDER BY MonthID) - Sales
    AS Difference

FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LAG(Sales) OVER (ORDER BY MonthID) AS Previous_Sales,
    LEAD(Sales) OVER (ORDER BY MonthID) AS Next_Sales

FROM MonthlySales;

SELECT
    MonthID,
    MonthName,
    Sales,
    LAG(Sales) OVER (ORDER BY MonthID) AS Previous_Sales,
    Sales - LAG(Sales) OVER (ORDER BY MonthID)
    AS Sales_Difference

FROM MonthlySales;


**Practical no 6
**
Display Hierarchy, Display Reporting Levels

CREATE TABLE Employeee (
    EmpID INT PRIMARY KEY,
    EmpName VARCHAR (50),
    Department VARCHAR (30),
    Salary DECIMAL (10,2)
);

INSERT INTO Employeee VALUES (101, 'Amit', 'HR', 45000);
INSERT INTO Employeee VALUES (102, 'Neha', 'HR', 52000);
INSERT INTO Employeee VALUES (103, 'Rahul', 'IT', 70000);
INSERT INTO Employeee VALUES (104, 'Priya', 'IT', 65000);
INSERT INTO Employeee VALUES (105, 'Kiran', 'Sales', 48000);
INSERT INTO Employeee VALUES (106, 'Anita', 'Sales', 55000);
INSERT INTO Employeee VALUES (107, 'Vikas', 'IT', 90000);


SELECT EmpName, Salary
FROM Employeee
WHERE Salary >
(
    SELECT AVG(Salary)
    FROM Employeee
);


WITH AvgSalary AS
(
    SELECT AVG(Salary) AS AvgSal
    FROM Employeee
)

SELECT EmpName, Salary
FROM Employeee
JOIN AvgSalary
ON Employeee.Salary > AvgSalary.AvgSal;


SELECT EmpName, Department, Salary
FROM Employeee E
WHERE Salary = (
    SELECT MAX(Salary)
    FROM Employeee
    WHERE Department = E.Department
);

WITH DeptMax AS
(
    SELECT Department,
           MAX(Salary) AS MaxSalary
    FROM Employeee
    GROUP BY Department
)

SELECT E.EmpName,
       E.Department,
       E.Salary
FROM Employeee E
JOIN DeptMax d
ON E.Department=D.Department
AND E.Salary=D.MaxSalary;


CREATE TABLE Employees (
    EmpID NUMBER PRIMARY KEY,
    EmpName VARCHAR2(50),
    ManagerID NUMBER
);

INSERT INTO Employeees VALUES (1, 'CEO', NULL);
INSERT INTO Employeees VALUES (2, 'Manager A', 1);
INSERT INTO Employeees VALUES (3, 'Manager B', 1);
INSERT INTO Employeees VALUES (4, 'Team Lead A', 2);
INSERT INTO Employeees VALUES (5, 'Developer A', 4);
INSERT INTO Employeees VALUES (6, 'Developer B', 4);
INSERT INTO Employeees VALUES (7, 'HR Executive', 2);
INSERT INTO Employeees VALUES (8, 'Sales Executive', 3);


Display Hierarchy with Manager Names
SELECT
    EmpID,
    EmpName,
    ManagerID,
    SYS_CONNECT_BY_PATH(EmpName, ' -> ') AS Hierarchy
FROM Employees
START WITH ManagerID IS NULL
CONNECT BY PRIOR EmpID = ManagerID;

Display Reporting Levels:
SELECT
    EmpName,
    LEVEL - 1 AS ReportingLevel
FROM Employees
START WITH ManagerID IS NULL
CONNECT BY PRIOR EmpID = ManagerID;

**Practical no 7
**
Use the PIVOT operator (or CASE WHEN logic) to turn monthly sales rows 
into columns for a "Side-by-Side" yearly comparison report.

CREATE TABLE Sales2 (
    SalesYear NUMBER,
    Month VARCHAR2(10),
    SalesAmount NUMBER
);

INSERT INTO Sales2 VALUES (2024,'Jan',5000);
INSERT INTO Sales2 VALUES (2024,'Feb',7000);
INSERT INTO Sales2 VALUES (2024,'Mar',6000);

INSERT INTO Sales2 VALUES (2025,'Jan',6500);
INSERT INTO Sales2 VALUES (2025,'Feb',8000);
INSERT INTO Sales2 VALUES (2025,'Mar',7500);

COMMIT;

SELECT
    Month,
    SUM(CASE WHEN SalesYear = 2024 THEN SalesAmount ELSE 0 END) AS Sales_2024,
    SUM(CASE WHEN SalesYear = 2025 THEN SalesAmount ELSE 0 END) AS Sales_2025
FROM Sales2
GROUP BY Month
ORDER BY Month;


**Practical no 8**
Write an UPDATE/INSERT script to implement SCD Type 2. This involves using SQL to expire old records
(setting an end_date) and inserting new versions of a record to keep history.

CREATE TABLE Employee_Dim (
    EmpID NUMBER,
    EmpName VARCHAR2(50),
    Department VARCHAR2(30),
    Start_Date DATE,
    End_Date DATE,
    Is_Current CHAR(1)
);

INSERT INTO Employee_Dim
VALUES (101, 'Amit', 'IT', DATE '2024-01-01', NULL, 'Y');

COMMIT;

UPDATE Employee_Dim
SET End_Date = DATE '2025-06-30',
    Is_Current = 'N'
WHERE EmpID = 101
AND Is_Current = 'Y';

INSERT INTO Employee_Dim
VALUES (101, 'Amit', 'HR', DATE '2025-07-01', NULL, 'Y');

COMMIT;

SELECT *
FROM Employee_Dim
ORDER BY EmpID, Start_Date;

**Practical no 9
**
Create Materialized Views to pre-calculate heavy aggregations. Compare the execution plan (using EXPLAIN) of a 
query before and after adding B-Tree or Bitmap indexes.

CREATE TABLE sales4 (
    sale_id       NUMBER PRIMARY KEY,
    product_id    NUMBER,
    customer_id   NUMBER,
    sale_date     DATE,
    quantity      NUMBER,
    amount        NUMBER,
    region        VARCHAR2(30)
);

CREATE TABLE products4 (
    product_id    NUMBER PRIMARY KEY,
    product_name  VARCHAR2(100),
    category      VARCHAR2(50)
);

EXPLAIN PLAN FOR
SELECT
    p.category,
    s.region,
    SUM(s.amount) AS total_sales,
    AVG(s.amount) AS avg_sales,
    COUNT(*) AS total_transactions
FROM sales4 s
JOIN products4 p
    ON s.product_id = p.product_id
GROUP BY p.category, s.region;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);


CREATE MATERIALIZED VIEW mv_sales_summary
BUILD IMMEDIATE
REFRESH COMPLETE
ON DEMAND
AS
SELECT
    p.category,
    s.region,
    SUM(s.amount) AS total_sales,
    AVG(s.amount) AS avg_sales,
    COUNT(*) AS total_transactions
FROM sales4 s
JOIN products4 p
    ON s.product_id = p.product_id
GROUP BY p.category, s.region;

EXPLAIN PLAN FOR
SELECT
    category,
    region,
    total_sales,
    avg_sales,
    total_transactions
FROM mv_sales_summary;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

CREATE INDEX idx_sales_product
ON sales4(product_id);

CREATE INDEX idx_sales_date
ON sales4(sale_date);

EXPLAIN PLAN FOR
SELECT *
FROM sales4
WHERE product_id = 100;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

CREATE BITMAP INDEX idx_sales_region_bitmap
ON sales4(region);

EXPLAIN PLAN FOR
SELECT *
FROM sales4
WHERE region = 'WEST';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

CREATE INDEX idx_sales_customer
ON sales(customer_id);

EXPLAIN PLAN FOR
SELECT *
FROM sales4
WHERE customer_id = 101;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- Bitmap
CREATE BITMAP INDEX idx_sales_region
ON sales4(region);

EXPLAIN PLAN FOR
SELECT *
FROM sales4
WHERE region = 'WEST';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

**practical no 10

DIRTY DATA IDENTIFICATION AND PREVENTION

1. Create table
CREATE TABLE customers7 (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(50),
    age NUMBER,
    email VARCHAR2(100)
);

-- 2. Insert sample data
INSERT INTO customers7 VALUES (1, 'John', 25, 'john@gmail.com');
INSERT INTO customers7 VALUES (2, 'Sam', 30, 'sam@gmail.com');
INSERT INTO customers7 VALUES (3, 'Tom', 150, 'tom@gmail.com');
INSERT INTO customers7 VALUES (4, 'Mike', 25, NULL);
INSERT INTO customers7 VALUES (5, 'Bob', 30, 'sam@gmail.com');

COMMIT;


 3. FIND NULL VALUES

SELECT *
FROM customers7
WHERE email IS NULL;

 4. FIND DUPLICATE VALUES

SELECT email, COUNT(*)
FROM customers7
GROUP BY email
HAVING COUNT(*) > 1;

 6. FIND OUTLIERS

SELECT *
FROM customers7
WHERE age > 100;


 6. FIX NULL EMAIL
UPDATE customers7
SET email = 'unknown@gmail.com'
WHERE email IS NULL;

COMMIT;

 7. CHECK CONSTRAINT FOR EMAIL
ALTER TABLE customers7
ADD CONSTRAINT chk_email
CHECK (email IS NOT NULL);


8. CHECK CONSTRAINT FOR AGE

ALTER TABLE customers7
ADD CONSTRAINT chk_age
CHECK (age BETWEEN 1 AND 200);


9. TEST CONSTRAINT
This should give an error

INSERT INTO customers7
VALUES (6, 'Alex', 150, 'alex@gmail.com');

10. TEST NULL EMAIL
 This should also give an error

INSERT INTO customers7
VALUES (7, 'David', 25, NULL);








