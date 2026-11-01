
-- TASK XII: TRANSACTION MANAGEMENT SYSTEM

-- 1. Select Database
CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- 2. Ensure Tables Use InnoDB
ALTER TABLE Customer ENGINE = InnoDB;
ALTER TABLE Product ENGINE = InnoDB;
ALTER TABLE Orders ENGINE = InnoDB;
ALTER TABLE Order_Details ENGINE = InnoDB;
ALTER TABLE Payment ENGINE = InnoDB;

-- 3. Drop Existing Procedures
DROP PROCEDURE IF EXISTS Place_Order_Transaction;
DROP PROCEDURE IF EXISTS Demonstrate_Rollback;

DELIMITER $$

-- 4. Create Order Placement Transaction Procedure
CREATE PROCEDURE Place_Order_Transaction(
    IN p_customer_id INT,
    IN p_product_id INT,
    IN p_quantity INT,
    IN p_payment_mode VARCHAR(30)
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

    IF p_quantity IS NULL OR p_quantity <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Quantity must be greater than zero';
    END IF;

    IF p_payment_mode IS NULL OR
       p_payment_mode NOT IN
       ('UPI', 'CARD', 'NET_BANKING',
        'CASH_ON_DELIVERY', 'WALLET') THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid payment mode';
    END IF;

    START TRANSACTION;

    -- Check Customer
    IF NOT EXISTS (
        SELECT 1
        FROM Customer
        WHERE customer_id = p_customer_id
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Customer does not exist';
    END IF;

    -- Check Product and Lock Stock
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

    -- Calculate Total Amount
    SET v_total = v_price * p_quantity;

    -- Insert Order
    INSERT INTO Orders
    (customer_id, total_amount, order_status)
    VALUES
    (p_customer_id, v_total, 'PLACED');

    SET v_order_id = LAST_INSERT_ID();

    -- Insert Order Details
    INSERT INTO Order_Details
    (order_id, product_id, quantity, unit_price, total_price)
    VALUES
    (v_order_id, p_product_id, p_quantity, v_price, v_total);

    -- Update Product Stock
    UPDATE Product
    SET stock = stock - p_quantity
    WHERE product_id = p_product_id;

    -- Insert Payment
    INSERT INTO Payment
    (order_id, payment_mode, payment_date,
     amount, payment_status)
    VALUES
    (v_order_id, p_payment_mode, CURRENT_TIMESTAMP,
     v_total, 'SUCCESS');

    -- Save Transaction
    COMMIT;

    SELECT v_order_id AS order_id,
           v_total AS total_amount,
           'SUCCESS' AS transaction_status;
END$$

-- 5. Create Rollback Demonstration Procedure
CREATE PROCEDURE Demonstrate_Rollback(
    IN p_product_id INT,
    IN p_quantity INT
)
BEGIN
    DECLARE v_stock INT DEFAULT NULL;

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    IF p_quantity IS NULL OR p_quantity <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Quantity must be greater than zero';
    END IF;

    START TRANSACTION;

    -- Lock Product Stock
    SELECT stock
    INTO v_stock
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

    -- Temporary Stock Update
    UPDATE Product
    SET stock = stock - p_quantity
    WHERE product_id = p_product_id;

    -- Cancel Transaction
    ROLLBACK;

    SELECT product_id, product_name, stock,
           'ROLLBACK completed; stock change cancelled'
           AS transaction_status
    FROM Product
    WHERE product_id = p_product_id;
END$$

DELIMITER ;

-- 6. Test COMMIT Transaction
CALL Place_Order_Transaction(1, 1, 1, 'UPI');

-- 7. Test ROLLBACK Transaction
CALL Demonstrate_Rollback(1, 1);

-- 8. Display Orders
SELECT * FROM Orders
ORDER BY order_id DESC;

-- 9. Display Order Details
SELECT * FROM Order_Details
ORDER BY detail_id DESC;

-- 10. Display Payment Details
SELECT * FROM Payment
ORDER BY payment_id DESC;

-- 11. Check Updated Product Stock
SELECT product_id, product_name, stock
FROM Product
ORDER BY product_id;

-- 12. Check Database Consistency
SELECT
    o.order_id,
    o.total_amount,
    COALESCE(SUM(od.total_price), 0.00)
        AS details_total,
    CASE
        WHEN o.total_amount =
             COALESCE(SUM(od.total_price), 0.00)
        THEN 'CONSISTENT'
        ELSE 'CHECK REQUIRED'
    END AS consistency_status
FROM Orders o
LEFT JOIN Order_Details od
ON o.order_id = od.order_id
GROUP BY o.order_id, o.total_amount
ORDER BY o.order_id DESC;
