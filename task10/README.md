-- 1. Select Database
USE ecommerce_db;

-- 2. Find Products Above Average Price (Subquery)
SELECT Product_Name, Price
FROM Sales
WHERE Price > (
    SELECT AVG(Price)
    FROM Sales
);

-- 3. Find the Most Expensive Product (Nested Query)
SELECT Product_Name, Price
FROM Sales
WHERE Price = (
    SELECT MAX(Price)
    FROM Sales
);

-- 4. Find Customers Whose Purchases Exceed 50000
SELECT Customer_Name, Total_Purchase
FROM (
    SELECT Customer_Name,
           SUM(Quantity * Price) AS Total_Purchase
    FROM Sales
    GROUP BY Customer_Name
) AS Customer_Sales
WHERE Total_Purchase > 50000;

-- 5. Identify Customer With Maximum Purchase
SELECT Customer_Name,
       SUM(Quantity * Price) AS Total_Purchase
FROM Sales
GROUP BY Customer_Name
ORDER BY Total_Purchase DESC
LIMIT 1;

-- 6. Find Customers Using HAVING Clause
SELECT Customer_Name,
       SUM(Quantity * Price) AS Total_Purchase
FROM Sales
GROUP BY Customer_Name
HAVING SUM(Quantity * Price) > 50000;

-- 7. Identify Best-Selling Product
SELECT Product_Name,
       SUM(Quantity) AS Total_Quantity_Sold
FROM Sales
GROUP BY Product_Name
ORDER BY Total_Quantity_Sold DESC
LIMIT 1;

-- 8. Find Highest-Selling Category
SELECT Category,
       SUM(Quantity * Price) AS Total_Sales
FROM Sales
GROUP BY Category
ORDER BY Total_Sales DESC
LIMIT 1;

-- 9. Find Customers Who Purchased Laptops
SELECT DISTINCT Customer_Name
FROM Sales
WHERE Product_Name = 'Laptop';

-- 10. Generate Advanced Customer Sales Report
SELECT
    Customer_Name,
    COUNT(*) AS Total_Orders,
    SUM(Quantity) AS Total_Quantity,
    SUM(Quantity * Price) AS Total_Purchase,
    AVG(Quantity * Price) AS Average_Purchase,
    MIN(Quantity * Price) AS Minimum_Purchase,
    MAX(Quantity * Price) AS Maximum_Purchase
FROM Sales
GROUP BY Customer_Name
ORDER BY Total_Purchase DESC;
