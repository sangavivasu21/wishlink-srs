-- 1. Create and Select Database
CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- 2. Create Customer Table
CREATE TABLE IF NOT EXISTS Customer (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_name VARCHAR(100) NOT NULL
);

-- 3. Create Product Table
CREATE TABLE IF NOT EXISTS Product (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL
);

-- 4. Create Orders Table
CREATE TABLE IF NOT EXISTS Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    order_status VARCHAR(30) DEFAULT 'PLACED',
    total_amount DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id)
);

-- 5. Create Order_Details Table
CREATE TABLE IF NOT EXISTS Order_Details (
    detail_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id)
);

-- 6. Create Payment Table
CREATE TABLE IF NOT EXISTS Payment (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    payment_mode VARCHAR(50),
    payment_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    amount DECIMAL(10,2),
    payment_status VARCHAR(30),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);

-- 7. Insert Sample Customers
INSERT IGNORE INTO Customer (customer_id, customer_name)
VALUES
(1, 'Sangavi'),
(2, 'Arun'),
(3, 'Priya');

-- 8. Insert Sample Products
INSERT IGNORE INTO Product
(product_id, product_name, price, stock)
VALUES
(1, 'Laptop', 45000, 20),
(2, 'Mobile', 15000, 30),
(3, 'Headphones', 1500, 50);

-- 9. Create Order Summary View
CREATE OR REPLACE VIEW Order_Summary AS
SELECT
    o.order_id,
    o.customer_id,
    c.customer_name,
    o.order_date,
    o.order_status,
    o.total_amount
FROM Orders o
JOIN Customer c
ON o.customer_id = c.customer_id;

-- 10. Create Product Sales Report View
CREATE OR REPLACE VIEW Product_Sales_Report AS
SELECT
    p.product_id,
    p.product_name,
    COALESCE(SUM(od.quantity), 0) AS total_quantity_sold,
    COALESCE(SUM(od.total_price), 0.00) AS total_sales
FROM Product p
LEFT JOIN Order_Details od
ON p.product_id = od.product_id
GROUP BY p.product_id, p.product_name;

-- 11. Create Payment Report View
CREATE OR REPLACE VIEW Payment_Report AS
SELECT
    py.payment_id,
    py.order_id,
    c.customer_name,
    py.payment_mode,
    py.payment_date,
    py.amount,
    py.payment_status
FROM Payment py
JOIN Orders o
ON py.order_id = o.order_id
JOIN Customer c
ON o.customer_id = c.customer_id;

-- 12. Drop Existing Function and Procedure
DROP FUNCTION IF EXISTS Calculate_Order_Sales;
DROP PROCEDURE IF EXISTS Process_Order;

DELIMITER $$

-- 13. Create Function for Sales Calculation
CREATE FUNCTION Calculate_Order_Sales(p_order_id INT)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN
    DECLARE v_sales DECIMAL(10,2);

    SELECT COALESCE(SUM(total_price), 0.00)
    INTO v_sales
    FROM Order_Details
    WHERE order_id = p_order_id;

    RETURN v_sales;
END$$

-- 14. Create Stored Procedure for Order Processing
CREATE PROCEDURE Process_Order(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT
)
BEGIN
    DECLARE v_price DECIMAL(10,2);
    DECLARE v_stock INT DEFAULT NULL;
    DECLARE v_order_id INT;
    DECLARE v_total DECIMAL(10,2);

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    IF p_quantity <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Quantity must be greater than zero';
    END IF;

    START TRANSACTION;

    IF NOT EXISTS (
        SELECT 1
        FROM Customer
        WHERE customer_id = p_customer_id
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Customer does not exist';
    END IF;

    SELECT price, stock
    INTO v_price, v_stock
    FROM Product
    WHERE product_id = p_product_id
    FOR UPDATE;

    IF v_stock IS NULL THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Product does not exist';
    END IF;

    IF v_stock < p_quantity THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Insufficient product stock';
    END IF;

    SET v_total = v_price * p_quantity;

    INSERT INTO Orders
    (customer_id, total_amount, order_status)
    VALUES
    (p_customer_id, v_total, 'PLACED');

    SET v_order_id = LAST_INSERT_ID();

    INSERT INTO Order_Details
    (order_id, product_id, quantity, unit_price, total_price)
    VALUES
    (v_order_id, p_product_id, p_quantity, v_price, v_total);

    UPDATE Product
    SET stock = stock - p_quantity
    WHERE product_id = p_product_id;

    COMMIT;
END$$

DELIMITER ;

-- 15. Test Order Processing Procedure
CALL Process_Order(1, 1, 2);

-- 16. Test Order Summary View
SELECT * FROM Order_Summary;

-- 17. Test Product Sales Report View
SELECT * FROM Product_Sales_Report;

-- 18. Test Payment Report View
SELECT * FROM Payment_Report;

-- 19. Test Sales Calculation Function
SELECT Calculate_Order_Sales(1) AS Calculated_Order_Sales;

-- 20. Check Updated Product Stock
SELECT * FROM Product;
