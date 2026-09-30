DROP DATABASE IF EXISTS ECommercePaymentDB;

CREATE DATABASE ECommercePaymentDB;

USE ECommercePaymentDB;

CREATE TABLE Customers (
    Customer_ID INT PRIMARY KEY AUTO_INCREMENT,
    Customer_Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Phone VARCHAR(15)
);

CREATE TABLE Orders (
    Order_ID INT PRIMARY KEY AUTO_INCREMENT,
    Customer_ID INT NOT NULL,
    Order_Date DATE NOT NULL,
    Total_Amount DECIMAL(10,2) NOT NULL,
    Order_Status VARCHAR(20) DEFAULT 'Placed',
    FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID)
);

CREATE TABLE Payment (
    Payment_ID INT PRIMARY KEY AUTO_INCREMENT,
    Order_ID INT NOT NULL,
    Customer_ID INT NOT NULL,
    Payment_Mode VARCHAR(30) NOT NULL,
    Payment_Date DATE NOT NULL,
    Amount DECIMAL(10,2) NOT NULL,
    Payment_Status VARCHAR(20) NOT NULL,
    FOREIGN KEY (Order_ID) REFERENCES Orders(Order_ID),
    FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID)
);

INSERT INTO Customers
(Customer_Name, Email, Phone)
VALUES
('Sangavi', 'sangavi@gmail.com', '9876543210'),
('Rahul', 'rahul@gmail.com', '9876543211'),
('Priya', 'priya@gmail.com', '9876543212'),
('Arun', 'arun@gmail.com', '9876543213');

INSERT INTO Orders
(Customer_ID, Order_Date, Total_Amount, Order_Status)
VALUES
(1, '2026-09-01', 52000.00, 'Delivered'),
(2, '2026-09-01', 3500.00, 'Shipped'),
(1, '2026-09-02', 25800.00, 'Placed'),
(3, '2026-09-02', 2000.00, 'Placed'),
(4, '2026-09-02', 15000.00, 'Placed');

INSERT INTO Payment
(Order_ID, Customer_ID, Payment_Mode, Payment_Date, Amount, Payment_Status)
VALUES
(1, 1, 'Credit Card', '2026-09-01', 52000.00, 'Successful'),
(2, 2, 'UPI', '2026-09-01', 3500.00, 'Successful'),
(3, 1, 'Debit Card', '2026-09-02', 25800.00, 'Failed'),
(4, 3, 'Cash on Delivery', '2026-09-02', 2000.00, 'Pending'),
(5, 4, 'UPI', '2026-09-02', 15000.00, 'Successful');

SELECT * FROM Customers;

SELECT * FROM Orders;

SELECT * FROM Payment;

UPDATE Payment
SET Payment_Status = 'Successful'
WHERE Payment_ID = 3;

UPDATE Payment
SET Payment_Mode = 'UPI'
WHERE Payment_ID = 3;

SELECT
    Payment_ID,
    Order_ID,
    Customer_ID,
    Payment_Mode,
    Payment_Date,
    Amount,
    Payment_Status
FROM Payment
WHERE Payment_Status = 'Successful';

SELECT
    Payment_ID,
    Order_ID,
    Customer_ID,
    Payment_Mode,
    Payment_Date,
    Amount,
    Payment_Status
FROM Payment
WHERE Payment_Status = 'Failed';

SELECT
    Payment_Mode,
    COUNT(*) AS Number_of_Transactions
FROM Payment
GROUP BY Payment_Mode;

SELECT
    Payment_Mode,
    SUM(Amount) AS Total_Amount
FROM Payment
WHERE Payment_Status = 'Successful'
GROUP BY Payment_Mode;

SELECT
    Payment_Status,
    COUNT(*) AS Number_of_Transactions
FROM Payment
GROUP BY Payment_Status;

SELECT
    c.Customer_ID,
    c.Customer_Name,
    p.Payment_ID,
    p.Order_ID,
    p.Payment_Mode,
    p.Payment_Date,
    p.Amount,
    p.Payment_Status
FROM Customers c
JOIN Payment p
ON c.Customer_ID = p.Customer_ID
ORDER BY p.Payment_Date DESC;

SELECT
    p.Payment_ID,
    c.Customer_Name,
    o.Order_ID,
    o.Order_Date,
    o.Total_Amount,
    p.Payment_Mode,
    p.Payment_Date,
    p.Amount,
    p.Payment_Status
FROM Payment p
JOIN Customers c
ON p.Customer_ID = c.Customer_ID
JOIN Orders o
ON p.Order_ID = o.Order_ID
ORDER BY p.Payment_Date DESC;

SELECT
    COUNT(*) AS Total_Transactions,
    SUM(Amount) AS Total_Transaction_Amount
FROM Payment;

SELECT
    COUNT(*) AS Successful_Transactions,
    SUM(Amount) AS Successful_Amount
FROM Payment
WHERE Payment_Status = 'Successful';

SELECT
    COUNT(*) AS Failed_Transactions,
    SUM(Amount) AS Failed_Amount
FROM Payment
WHERE Payment_Status = 'Failed';

SELECT
    Payment_Mode,
    COUNT(*) AS Transactions,
    SUM(Amount) AS Total_Amount
FROM Payment
GROUP BY Payment_Mode
ORDER BY Transactions DESC;
