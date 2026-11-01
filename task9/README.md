-- 1. Create Database
CREATE DATABASE IF NOT EXISTS ecommerce_db;

-- 2. Select Database
USE ecommerce_db;

-- 3. Create Sales Table
CREATE TABLE IF NOT EXISTS Sales (
    Sale_ID INT PRIMARY KEY AUTO_INCREMENT,
    Customer_Name VARCHAR(100),
    Product_Name VARCHAR(100),
    Category VARCHAR(50),
    Quantity INT,
    Price DECIMAL(10,2)
);

-- 4. Insert Sample Records
INSERT INTO Sales (Customer_Name, Product_Name, Category, Quantity, Price)
VALUES
('Sangavi', 'Laptop', 'Electronics', 2, 45000),
('Arun', 'Mobile', 'Electronics', 3, 15000),
('Priya', 'Chair', 'Furniture', 4, 2000),
('Sangavi', 'Headphones', 'Electronics', 2, 1500),
('Rahul', 'Table', 'Furniture', 2, 5000),
('Priya', 'Laptop', 'Electronics', 1, 45000),
('Arun', 'Chair', 'Furniture', 3, 2000),
('Rahul', 'Mobile', 'Electronics', 2, 15000),
('Sangavi', 'Table', 'Furniture', 1, 5000),
('Priya', 'Headphones', 'Electronics', 3, 1500);

-- 5. Display Sales Records
SELECT * FROM Sales;

-- 6. COUNT() - Total Transactions
SELECT COUNT(*) AS Total_Transactions
FROM Sales;

-- 7. SUM() - Total Sales Amount
SELECT SUM(Quantity * Price) AS Total_Sales_Amount
FROM Sales;

-- 8. AVG() - Average Sales
SELECT AVG(Quantity * Price) AS Average_Sales
FROM Sales;

-- 9. MIN() - Minimum Sale
SELECT MIN(Quantity * Price) AS Minimum_Sale
FROM Sales;

-- 10. MAX() - Maximum Sale
SELECT MAX(Quantity * Price) AS Maximum_Sale
FROM Sales;

-- 11. Generate Total Sales Report
SELECT
    COUNT(*) AS Total_Orders,
    SUM(Quantity * Price) AS Total_Sales,
    AVG(Quantity * Price) AS Average_Sale,
    MIN(Quantity * Price) AS Minimum_Sale,
    MAX(Quantity * Price) AS Maximum_Sale
FROM Sales;

-- 12. Find Top Customer
SELECT
    Customer_Name,
    SUM(Quantity * Price) AS Total_Purchase
FROM Sales
GROUP BY Customer_Name
ORDER BY Total_Purchase DESC
LIMIT 1;

-- 13. Identify Best-Selling Products
SELECT
    Product_Name,
    SUM(Quantity) AS Total_Quantity_Sold,
    SUM(Quantity * Price) AS Total_Revenue
FROM Sales
GROUP BY Product_Name
ORDER BY Total_Quantity_Sold DESC;

-- 14. Category-Wise Sales Analysis
SELECT
    Category,
    COUNT(*) AS Number_of_Sales,
    SUM(Quantity) AS Total_Quantity,
    SUM(Quantity * Price) AS Total_Sales,
    AVG(Quantity * Price) AS Average_Sale
FROM Sales
GROUP BY Category
ORDER BY Total_Sales DESC;
