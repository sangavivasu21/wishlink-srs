-- 1. Create and Select Database
CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- 2. Create Database Roles
CREATE ROLE IF NOT EXISTS 'ecommerce_readonly';
CREATE ROLE IF NOT EXISTS 'ecommerce_data_entry';

-- 3. Grant Read-Only Permissions
GRANT SELECT ON ecommerce_db.Customer
TO 'ecommerce_readonly';

GRANT SELECT ON ecommerce_db.Product
TO 'ecommerce_readonly';

GRANT SELECT ON ecommerce_db.Orders
TO 'ecommerce_readonly';

GRANT SELECT ON ecommerce_db.Order_Details
TO 'ecommerce_readonly';

GRANT SELECT ON ecommerce_db.Payment
TO 'ecommerce_readonly';

-- 4. Grant Data Entry Permissions
GRANT SELECT, INSERT, UPDATE
ON ecommerce_db.Customer
TO 'ecommerce_data_entry';

GRANT SELECT, INSERT, UPDATE
ON ecommerce_db.Product
TO 'ecommerce_data_entry';

GRANT SELECT, INSERT, UPDATE
ON ecommerce_db.Orders
TO 'ecommerce_data_entry';

GRANT SELECT, INSERT, UPDATE
ON ecommerce_db.Order_Details
TO 'ecommerce_data_entry';

GRANT SELECT, INSERT, UPDATE
ON ecommerce_db.Payment
TO 'ecommerce_data_entry';

-- 5. Create Database Users
CREATE USER IF NOT EXISTS
'ecommerce_viewer'@'localhost'
IDENTIFIED BY 'ChangeThisViewer_123!';

CREATE USER IF NOT EXISTS
'ecommerce_operator'@'localhost'
IDENTIFIED BY 'ChangeThisOperator_123!';

-- 6. Assign Roles to Users
GRANT 'ecommerce_readonly'
TO 'ecommerce_viewer'@'localhost';

GRANT 'ecommerce_data_entry'
TO 'ecommerce_operator'@'localhost';

-- 7. Set Default Roles
SET DEFAULT ROLE 'ecommerce_readonly'
TO 'ecommerce_viewer'@'localhost';

SET DEFAULT ROLE 'ecommerce_data_entry'
TO 'ecommerce_operator'@'localhost';

-- 8. Revoke Payment Insert Permission
REVOKE INSERT ON ecommerce_db.Payment
FROM 'ecommerce_data_entry';

-- 9. Create Indexes
-- Using columns available in Task XI tables

CREATE INDEX idx_customer_name
ON Customer(customer_name);

CREATE INDEX idx_product_name
ON Product(product_name);

CREATE INDEX idx_product_name_price
ON Product(product_name, price);

CREATE INDEX idx_orders_customer_date
ON Orders(customer_id, order_date);

CREATE INDEX idx_orders_status
ON Orders(order_status);

CREATE INDEX idx_order_details_product
ON Order_Details(product_id);

CREATE INDEX idx_payment_status_date
ON Payment(payment_status, payment_date);

-- 10. Analyze Product Search Performance
EXPLAIN
SELECT product_id, product_name, price
FROM Product
WHERE product_name = 'Wireless Headphones';

-- 11. Analyze Product Price Search
EXPLAIN
SELECT product_id, product_name, price
FROM Product
WHERE product_name = 'Laptop'
AND price < 50000;

-- 12. Analyze Customer Order Search
EXPLAIN
SELECT order_id, order_date, total_amount, order_status
FROM Orders
WHERE customer_id = 1
ORDER BY order_date DESC;

-- 13. Analyze Payment Search
EXPLAIN
SELECT payment_id, order_id, amount, payment_status
FROM Payment
WHERE payment_status = 'SUCCESS'
ORDER BY payment_date DESC;

-- 14. Display Customer Indexes
SHOW INDEX FROM Customer;

-- 15. Display Product Indexes
SHOW INDEX FROM Product;

-- 16. Display Orders Indexes
SHOW INDEX FROM Orders;

-- 17. Display Order Details Indexes
SHOW INDEX FROM Order_Details;

-- 18. Display Payment Indexes
SHOW INDEX FROM Payment;

-- 19. Display User Permissions
SHOW GRANTS FOR 'ecommerce_viewer'@'localhost';

SHOW GRANTS FOR 'ecommerce_operator'@'localhost';

-- 20. Display Role Permissions
SHOW GRANTS FOR 'ecommerce_readonly';

SHOW GRANTS FOR 'ecommerce_data_entry';
