
CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- ==========================================================
-- 1. RESET PREVIOUS CAPSTONE OBJECTS
-- WARNING: This removes existing tables and their data.
-- ==========================================================

DROP VIEW IF EXISTS Order_Summary;
DROP VIEW IF EXISTS Product_Sales_Report;
DROP VIEW IF EXISTS Payment_Report;
DROP VIEW IF EXISTS Customer_Segmentation;

DROP PROCEDURE IF EXISTS Process_Capstone_Order;
DROP FUNCTION IF EXISTS Calculate_Order_Sales;

SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS Rating;
DROP TABLE IF EXISTS Review;
DROP TABLE IF EXISTS Payment;
DROP TABLE IF EXISTS Order_Details;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS Inventory;
DROP TABLE IF EXISTS Product;
DROP TABLE IF EXISTS Seller;
DROP TABLE IF EXISTS Category;
DROP TABLE IF EXISTS Customer;

SET FOREIGN_KEY_CHECKS = 1;

-- ==========================================================
-- 2. CREATE NORMALIZED DATABASE TABLES
-- ==========================================================

CREATE TABLE Customer (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    phone VARCHAR(20),
    address VARCHAR(255),
    city VARCHAR(80),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE Category (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(255)
) ENGINE=InnoDB;

CREATE TABLE Seller (
    seller_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_name VARCHAR(120) NOT NULL,
    email VARCHAR(150) UNIQUE,
    phone VARCHAR(20),
    address VARCHAR(255)
) ENGINE=InnoDB;

CREATE TABLE Product (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(150) NOT NULL,
    category_id INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    description VARCHAR(255),
    FOREIGN KEY (category_id) REFERENCES Category(category_id),
    CHECK (price >= 0),
    CHECK (stock >= 0)
) ENGINE=InnoDB;

CREATE TABLE Inventory (
    inventory_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_id INT NOT NULL,
    product_id INT NOT NULL,
    available_stock INT NOT NULL DEFAULT 0,
    unavailable_stock INT NOT NULL DEFAULT 0,
    last_updated DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (seller_id, product_id),
    FOREIGN KEY (seller_id) REFERENCES Seller(seller_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    CHECK (available_stock >= 0),
    CHECK (unavailable_stock >= 0)
) ENGINE=InnoDB;

CREATE TABLE Orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    order_status VARCHAR(30) NOT NULL DEFAULT 'PLACED',
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id),
    CHECK (total_amount >= 0),
    CHECK (
        order_status IN (
            'PLACED', 'CONFIRMED', 'SHIPPED',
            'DELIVERED', 'CANCELLED'
        )
    )
) ENGINE=InnoDB;

CREATE TABLE Order_Details (
    order_detail_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    CHECK (quantity > 0),
    CHECK (unit_price >= 0),
    CHECK (total_price >= 0)
) ENGINE=InnoDB;

CREATE TABLE Payment (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    payment_mode VARCHAR(30) NOT NULL,
    payment_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    amount DECIMAL(10,2) NOT NULL,
    payment_status VARCHAR(30) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    CHECK (amount >= 0),
    CHECK (
        payment_mode IN (
            'UPI', 'CARD', 'NET_BANKING',
            'CASH_ON_DELIVERY', 'WALLET'
        )
    ),
    CHECK (payment_status IN ('SUCCESS', 'FAILED', 'PENDING'))
) ENGINE=InnoDB;

CREATE TABLE Review (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    review_text VARCHAR(500) NOT NULL,
    review_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id),
    FOREIGN KEY (product_id) REFERENCES Product(product_id),
    UNIQUE (customer_id, product_id)
) ENGINE=InnoDB;

-- Rating is linked to Review to avoid storing
-- duplicate customer and product information.

CREATE TABLE Rating (
    review_id INT PRIMARY KEY,
    rating_value INT NOT NULL,
    rating_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (review_id) REFERENCES Review(review_id),
    CHECK (rating_value BETWEEN 1 AND 5)
) ENGINE=InnoDB;

-- ==========================================================
-- 3. INSERT REALISTIC BUSINESS DATA
-- ==========================================================

INSERT INTO Customer
(customer_name, email, phone, address, city)
VALUES
('Arun Kumar', 'arun.kumar@example.com', '9000000001', 'Anna Nagar', 'Chennai'),
('Priya S', 'priya.s@example.com', '9000000002', 'Velachery', 'Chennai'),
('Karthik R', 'karthik.r@example.com', '9000000003', 'Tambaram', 'Chennai'),
('Meena Devi', 'meena.devi@example.com', '9000000004', 'RS Puram', 'Coimbatore'),
('Rahul M', 'rahul.m@example.com', '9000000005', 'Gandhipuram', 'Coimbatore'),
('Nisha P', 'nisha.p@example.com', '9000000006', 'KK Nagar', 'Madurai');

INSERT INTO Category (category_name, description)
VALUES
('Electronics', 'Electronic accessories and devices'),
('Clothing', 'Everyday clothing'),
('Books', 'Education and programming books'),
('Home Appliances', 'Useful home appliances');

INSERT INTO Seller (seller_name, email, phone, address)
VALUES
('Tech World', 'techworld@example.com', '9100000001', 'Chennai'),
('Fashion Store', 'fashion@example.com', '9100000002', 'Coimbatore'),
('Book House', 'bookhouse@example.com', '9100000003', 'Madurai'),
('Home Needs', 'homeneeds@example.com', '9100000004', 'Chennai');

INSERT INTO Product
(product_name, category_id, price, stock, description)
VALUES
('Wireless Headphones', 1, 1499.00, 50, 'Bluetooth headphones'),
('Smart Watch', 1, 2499.00, 30, 'Fitness and notification watch'),
('USB Keyboard', 1, 699.00, 65, 'Wired USB keyboard'),
('Cotton T-Shirt', 2, 599.00, 100, 'Cotton casual t-shirt'),
('Running Shoes', 2, 1899.00, 40, 'Everyday running shoes'),
('Java Programming Book', 3, 799.00, 40, 'Java learning guide'),
('SQL Fundamentals', 3, 649.00, 35, 'Database learning book'),
('Electric Kettle', 4, 1299.00, 25, 'Stainless steel kettle');

INSERT INTO Inventory
(seller_id, product_id, available_stock, unavailable_stock)
VALUES
(1, 1, 50, 0),
(1, 2, 30, 0),
(1, 3, 65, 0),
(2, 4, 100, 0),
(2, 5, 40, 0),
(3, 6, 40, 0),
(3, 7, 35, 0),
(4, 8, 25, 0);

INSERT INTO Orders
(customer_id, order_date, total_amount, order_status)
VALUES
(1, '2026-09-02', 2098.00, 'DELIVERED'),
(2, '2026-09-05', 2499.00, 'DELIVERED'),
(3, '2026-09-12', 1897.00, 'SHIPPED'),
(1, '2026-09-18', 1899.00, 'DELIVERED'),
(4, '2026-09-22', 1948.00, 'CONFIRMED'),
(5, '2026-09-25', 1299.00, 'PLACED'),
(6, '2026-09-28', 3148.00, 'DELIVERED');

INSERT INTO Order_Details
(order_id, product_id, quantity, unit_price, total_price)
VALUES
(1, 1, 1, 1499.00, 1499.00),
(1, 6, 1, 599.00, 599.00),
(2, 2, 1, 2499.00, 2499.00),
(3, 3, 1, 699.00, 699.00),
(3, 4, 2, 599.00, 1198.00),
(4, 5, 1, 1899.00, 1899.00),
(5, 6, 1, 799.00, 799.00),
(5, 7, 1, 649.00, 649.00),
(5, 4, 1, 500.00, 500.00),
(6, 8, 1, 1299.00, 1299.00),
(7, 1, 1, 1499.00, 1499.00),
(7, 7, 1, 649.00, 649.00),
(7, 4, 1, 1000.00, 1000.00);

INSERT INTO Payment
(order_id, payment_mode, payment_date, amount, payment_status)
VALUES
(1, 'UPI', '2026-09-02', 2098.00, 'SUCCESS'),
(2, 'CARD', '2026-09-05', 2499.00, 'SUCCESS'),
(3, 'UPI', '2026-09-12', 1897.00, 'SUCCESS'),
(4, 'WALLET', '2026-09-18', 1899.00, 'SUCCESS'),
(5, 'NET_BANKING', '2026-09-22', 1948.00, 'PENDING'),
(6, 'CASH_ON_DELIVERY', '2026-09-25', 1299.00, 'PENDING'),
(7, 'CARD', '2026-09-28', 3148.00, 'SUCCESS');

INSERT INTO Review
(customer_id, product_id, review_text, review_date)
VALUES
(1, 1, 'Clear sound and comfortable fit.', '2026-09-10'),
(2, 2, 'Useful features and easy to use.', '2026-09-11'),
(3, 3, 'Good keyboard for everyday work.', '2026-09-15'),
(4, 6, 'Helpful book for learning Java.', '2026-09-23'),
(5, 8, 'Heats water quickly and works well.', '2026-09-26');

INSERT INTO Rating (review_id, rating_value, rating_date)
VALUES
(1, 5, '2026-09-10'),
(2, 4, '2026-09-11'),
(3, 4, '2026-09-15'),
(4, 5, '2026-09-23'),
(5, 4, '2026-09-26');

-- ==========================================================
-- 4. CREATE INDEXES
-- ==========================================================

CREATE INDEX idx_product_category_price
ON Product(category_id, price);

CREATE INDEX idx_product_name
ON Product(product_name);

CREATE INDEX idx_orders_customer_date
ON Orders(customer_id, order_date);

CREATE INDEX idx_orders_status_date
ON Orders(order_status, order_date);

CREATE INDEX idx_order_details_product
ON Order_Details(product_id);

CREATE INDEX idx_payment_status_date
ON Payment(payment_status, payment_date);

CREATE INDEX idx_review_product_date
ON Review(product_id, review_date);

CREATE INDEX idx_rating_value
ON Rating(rating_value);

-- ==========================================================
-- 5. CREATE BUSINESS INTELLIGENCE VIEWS
-- ==========================================================

CREATE VIEW Order_Summary AS
SELECT
    o.order_id,
    o.order_date,
    o.order_status,
    c.customer_id,
    c.customer_name,
    c.city,
    o.total_amount
FROM Orders o
JOIN Customer c
ON c.customer_id = o.customer_id;

CREATE VIEW Product_Sales_Report AS
SELECT
    p.product_id,
    p.product_name,
    c.category_name,
    p.price,
    p.stock,
    COALESCE(SUM(od.quantity), 0) AS total_quantity_sold,
    COALESCE(SUM(od.total_price), 0.00) AS total_sales
FROM Product p
JOIN Category c
ON c.category_id = p.category_id
LEFT JOIN Order_Details od
ON od.product_id = p.product_id
GROUP BY
    p.product_id,
    p.product_name,
    c.category_name,
    p.price,
    p.stock;

CREATE VIEW Payment_Report AS
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
ON o.order_id = py.order_id
JOIN Customer c
ON c.customer_id = o.customer_id;

CREATE VIEW Customer_Segmentation AS
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.total_amount), 0.00) AS total_spend,
    CASE
        WHEN COALESCE(SUM(o.total_amount), 0.00) >= 5000
            THEN 'HIGH_VALUE'
        WHEN COALESCE(SUM(o.total_amount), 0.00) >= 2000
            THEN 'MEDIUM_VALUE'
        WHEN COALESCE(SUM(o.total_amount), 0.00) > 0
            THEN 'LOW_VALUE'
        ELSE 'NO_PURCHASE'
    END AS customer_segment
FROM Customer c
LEFT JOIN Orders o
ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name;

-- ==========================================================
-- 6. CREATE ORDER DETAIL TRIGGERS
-- These maintain product stock and order totals.
-- ==========================================================

DELIMITER $$

CREATE TRIGGER trg_order_details_before_insert
BEFORE INSERT ON Order_Details
FOR EACH ROW
BEGIN
    DECLARE v_stock INT DEFAULT NULL;
    DECLARE v_price DECIMAL(10,2);

    SELECT stock, price
    INTO v_stock, v_price
    FROM Product
    WHERE product_id = NEW.product_id;

    IF v_stock IS NULL THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Product does not exist';
    END IF;

    IF NEW.quantity IS NULL OR NEW.quantity <= 0
       OR v_stock < NEW.quantity THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid quantity or insufficient stock';
    END IF;

    SET NEW.unit_price = v_price;
    SET NEW.total_price = v_price * NEW.quantity;
END$$

CREATE TRIGGER trg_order_details_before_update
BEFORE UPDATE ON Order_Details
FOR EACH ROW
BEGIN
    DECLARE v_stock INT DEFAULT NULL;
    DECLARE v_price DECIMAL(10,2);

    SELECT stock, price
    INTO v_stock, v_price
    FROM Product
    WHERE product_id = NEW.product_id;

    IF v_stock IS NULL THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Product does not exist';
    END IF;

    IF NEW.quantity IS NULL OR NEW.quantity <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid quantity';
    END IF;

    IF OLD.product_id = NEW.product_id THEN
        IF v_stock + OLD.quantity < NEW.quantity THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Insufficient stock';
        END IF;
    ELSE
        IF v_stock < NEW.quantity THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Insufficient stock';
        END IF;
    END IF;

    SET NEW.unit_price = v_price;
    SET NEW.total_price = v_price * NEW.quantity;
END$$

CREATE TRIGGER trg_order_details_after_insert
AFTER INSERT ON Order_Details
FOR EACH ROW
BEGIN
    UPDATE Product
    SET stock = stock - NEW.quantity
    WHERE product_id = NEW.product_id;

    UPDATE Orders
    SET total_amount = (
        SELECT COALESCE(SUM(total_price), 0.00)
        FROM Order_Details
        WHERE order_id = NEW.order_id
    )
    WHERE order_id = NEW.order_id;
END$$

CREATE TRIGGER trg_order_details_after_update
AFTER UPDATE ON Order_Details
FOR EACH ROW
BEGIN
    IF OLD.product_id = NEW.product_id THEN
        UPDATE Product
        SET stock = stock + OLD.quantity - NEW.quantity
        WHERE product_id = NEW.product_id;
    ELSE
        UPDATE Product
        SET stock = stock + OLD.quantity
        WHERE product_id = OLD.product_id;

        UPDATE Product
        SET stock = stock - NEW.quantity
        WHERE product_id = NEW.product_id;
    END IF;

    UPDATE Orders
    SET total_amount = (
        SELECT COALESCE(SUM(total_price), 0.00)
        FROM Order_Details
        WHERE order_id = NEW.order_id
    )
    WHERE order_id = NEW.order_id;

    IF OLD.order_id <> NEW.order_id THEN
        UPDATE Orders
        SET total_amount = (
            SELECT COALESCE(SUM(total_price), 0.00)
            FROM Order_Details
            WHERE order_id = OLD.order_id
        )
        WHERE order_id = OLD.order_id;
    END IF;
END$$

CREATE TRIGGER trg_order_details_after_delete
AFTER DELETE ON Order_Details
FOR EACH ROW
BEGIN
    UPDATE Product
    SET stock = stock + OLD.quantity
    WHERE product_id = OLD.product_id;

    UPDATE Orders
    SET total_amount = (
        SELECT COALESCE(SUM(total_price), 0.00)
        FROM Order_Details
        WHERE order_id = OLD.order_id
    )
    WHERE order_id = OLD.order_id;
END$$

DELIMITER ;

-- ==========================================================
-- 7. CREATE STORED PROCEDURE
-- ==========================================================

DELIMITER $$

CREATE PROCEDURE Process_Capstone_Order(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    IN p_payment_mode VARCHAR(30)
)
BEGIN
    DECLARE v_price DECIMAL(10,2) DEFAULT NULL;
    DECLARE v_stock INT DEFAULT NULL;
    DECLARE v_order_id INT;
    DECLARE v_total DECIMAL(10,2);

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    IF p_quantity IS NULL OR p_quantity <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Quantity must be greater than zero';
    END IF;

    IF p_payment_mode IS NULL OR
       p_payment_mode NOT IN (
           'UPI', 'CARD', 'NET_BANKING',
           'CASH_ON_DELIVERY', 'WALLET'
       ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid payment mode';
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
        SET MESSAGE_TEXT = 'Insufficient stock';
    END IF;

    SET v_total = v_price * p_quantity;

    INSERT INTO Orders
        (customer_id, total_amount, order_status)
    VALUES
        (p_customer_id, 0.00, 'PLACED');

    SET v_order_id = LAST_INSERT_ID();

    INSERT INTO Order_Details
        (order_id, product_id, quantity, unit_price, total_price)
    VALUES
        (v_order_id, p_product_id, p_quantity, v_price, v_total);

    INSERT INTO Payment
        (order_id, payment_mode, amount, payment_status)
    VALUES
        (v_order_id, p_payment_mode, v_total, 'PENDING');

    COMMIT;

    SELECT
        v_order_id AS order_id,
        v_total AS total_amount,
        'PENDING' AS payment_status;
END$$

DELIMITER ;

-- ==========================================================
-- 8. CREATE FUNCTION
-- ==========================================================

DELIMITER $$

CREATE FUNCTION Calculate_Order_Sales(p_order_id INT)
RETURNS DECIMAL(10,2)
READS SQL DATA
BEGIN
    DECLARE v_total DECIMAL(10,2);

    SELECT COALESCE(SUM(total_price), 0.00)
    INTO v_total
    FROM Order_Details
    WHERE order_id = p_order_id;

    RETURN v_total;
END$$

DELIMITER ;

-- ==========================================================
-- 9. BASIC BUSINESS REPORTS
-- ==========================================================

SELECT * FROM Customer;
SELECT * FROM Category;
SELECT * FROM Seller;
SELECT * FROM Product;
SELECT * FROM Inventory;
SELECT * FROM Orders;
SELECT * FROM Order_Details;
SELECT * FROM Payment;
SELECT * FROM Review;
SELECT * FROM Rating;

-- Order summary
SELECT *
FROM Order_Summary
ORDER BY order_date DESC;

-- Product sales report
SELECT *
FROM Product_Sales_Report
ORDER BY total_sales DESC;

-- Payment report
SELECT *
FROM Payment_Report
ORDER BY payment_date DESC;

-- Customer segmentation
SELECT *
FROM Customer_Segmentation
ORDER BY total_spend DESC;

-- ==========================================================
-- 10. ADVANCED SQL AND BUSINESS INSIGHTS
-- ==========================================================

-- Insight 1: Category-wise sales
SELECT
    category_name,
    SUM(total_sales) AS category_sales
FROM Product_Sales_Report
GROUP BY category_name
ORDER BY category_sales DESC;

-- Insight 2: Product sales ranking
SELECT
    product_id,
    product_name,
    total_quantity_sold,
    total_sales,
    RANK() OVER (
        ORDER BY total_sales DESC
    ) AS sales_rank,
    ROW_NUMBER() OVER (
        ORDER BY total_quantity_sold DESC, product_id
    ) AS quantity_rank
FROM Product_Sales_Report
ORDER BY sales_rank, product_id;

-- Insight 3: Payment status analysis
SELECT
    payment_status,
    COUNT(*) AS transaction_count,
    SUM(amount) AS total_payment_amount
FROM Payment
GROUP BY payment_status
ORDER BY transaction_count DESC;

-- Insight 4: Product rating analysis
SELECT
    p.product_name,
    ROUND(AVG(r.rating_value), 2) AS average_rating,
    COUNT(r.review_id) AS rating_count
FROM Product p
LEFT JOIN Review rv
ON rv.product_id = p.product_id
LEFT JOIN Rating r
ON r.review_id = rv.review_id
GROUP BY p.product_id, p.product_name
ORDER BY average_rating DESC;

-- Insight 5: Customer purchase analysis
SELECT
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    COALESCE(SUM(o.total_amount), 0.00) AS total_spend
FROM Customer c
LEFT JOIN Orders o
ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_spend DESC;

-- Insight 6: Low stock products
SELECT
    product_name,
    stock
FROM Product
WHERE stock <= 10
ORDER BY stock;

-- Insight 7: City-wise customer analysis
SELECT
    city,
    COUNT(*) AS customer_count
FROM Customer
GROUP BY city
ORDER BY customer_count DESC;

-- Insight 8: Order sales using function
SELECT
    o.order_id,
    o.total_amount,
    Calculate_Order_Sales(o.order_id) AS calculated_detail_total
FROM Orders o
ORDER BY o.order_id;

-- ==========================================================
-- 11. TEST STORED PROCEDURE
-- Uncomment the next line to place a sample order.
-- Example: customer 1 buys product 3, quantity 1.
-- ==========================================================

-- CALL Process_Capstone_Order(1, 3, 1, 'UPI');

-- ==========================================================
-- 12. INDEX PERFORMANCE CHECK
-- ==========================================================

EXPLAIN
SELECT
    order_id,
    order_date,
    total_amount
FROM Orders
WHERE customer_id = 1
ORDER BY order_date DESC;

-- ==========================================================
-- 13. FINAL VERIFICATION
-- ==========================================================

SHOW TABLES;

SHOW FULL TABLES
WHERE Table_type = 'VIEW';
