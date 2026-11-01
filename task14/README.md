-- 1. Select Database
CREATE DATABASE IF NOT EXISTS ecommerce_db;
USE ecommerce_db;

-- 2. Customer Analytical Report and Segmentation
WITH CustomerMetrics AS (
    SELECT
        c.customer_id,
        c.customer_name,
        COUNT(DISTINCT o.order_id) AS total_orders,
        COALESCE(SUM(o.total_amount), 0.00) AS total_spend,
        COALESCE(AVG(o.total_amount), 0.00) AS average_order_value,
        MAX(o.order_date) AS last_order_date,
        COALESCE(
            DATEDIFF(CURRENT_DATE, MAX(o.order_date)), -1
        ) AS days_since_last_order
    FROM Customer c
    LEFT JOIN Orders o
        ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_orders,
    total_spend,
    average_order_value,
    last_order_date,
    days_since_last_order,
    CASE
        WHEN total_spend >= 5000 THEN 'HIGH_VALUE'
        WHEN total_spend >= 2000 THEN 'MEDIUM_VALUE'
        WHEN total_spend > 0 THEN 'LOW_VALUE'
        ELSE 'NO_PURCHASE'
    END AS customer_segment
FROM CustomerMetrics
ORDER BY total_spend DESC;

-- 3. Product Sales Report Using RANK and ROW_NUMBER
WITH ProductMetrics AS (
    SELECT
        p.product_id,
        p.product_name,
        p.price,
        p.stock,
        COALESCE(SUM(od.quantity), 0) AS total_quantity_sold,
        COALESCE(SUM(od.total_price), 0.00) AS total_revenue
    FROM Product p
    LEFT JOIN Order_Details od
        ON p.product_id = od.product_id
    GROUP BY
        p.product_id,
        p.product_name,
        p.price,
        p.stock
)
SELECT
    product_id,
    product_name,
    price,
    stock,
    total_quantity_sold,
    total_revenue,
    RANK() OVER (
        ORDER BY total_revenue DESC
    ) AS revenue_rank,
    ROW_NUMBER() OVER (
        ORDER BY total_quantity_sold DESC, product_id
    ) AS quantity_row_number
FROM ProductMetrics
ORDER BY revenue_rank, product_id;

-- 4. Customer Spending Ranking
WITH CustomerMetrics AS (
    SELECT
        c.customer_id,
        c.customer_name,
        COUNT(DISTINCT o.order_id) AS total_orders,
        COALESCE(SUM(o.total_amount), 0.00) AS total_spend,
        COALESCE(AVG(o.total_amount), 0.00) AS average_order_value,
        MAX(o.order_date) AS last_order_date
    FROM Customer c
    LEFT JOIN Orders o
        ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_orders,
    total_spend,
    average_order_value,
    last_order_date,
    RANK() OVER (
        ORDER BY total_spend DESC
    ) AS spending_rank,
    ROW_NUMBER() OVER (
        ORDER BY total_spend DESC, customer_id
    ) AS customer_row_number
FROM CustomerMetrics
ORDER BY spending_rank, customer_id;

-- 5. Prepare Customer Dataset for AI Models
WITH CustomerFeatures AS (
    SELECT
        c.customer_id,
        c.customer_name,
        COUNT(DISTINCT o.order_id) AS order_count,
        COALESCE(SUM(o.total_amount), 0.00) AS total_spend,
        COALESCE(AVG(o.total_amount), 0.00) AS average_order_value,
        COALESCE(MAX(o.total_amount), 0.00) AS maximum_order_value,
        MAX(o.order_date) AS last_order_date,
        COALESCE(
            DATEDIFF(CURRENT_DATE, MAX(o.order_date)), -1
        ) AS days_since_last_order
    FROM Customer c
    LEFT JOIN Orders o
        ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT
    customer_id,
    customer_name,
    order_count,
    total_spend,
    average_order_value,
    maximum_order_value,
    last_order_date,
    days_since_last_order,
    CASE
        WHEN order_count > 0 THEN 1
        ELSE 0
    END AS has_purchased,
    CASE
        WHEN total_spend >= 5000 THEN 'HIGH_VALUE'
        WHEN total_spend >= 2000 THEN 'MEDIUM_VALUE'
        WHEN total_spend > 0 THEN 'LOW_VALUE'
        ELSE 'NO_PURCHASE'
    END AS customer_segment
FROM CustomerFeatures
ORDER BY customer_id;

-- 6. Product Feature Extraction
SELECT
    p.product_id,
    p.product_name,
    p.price,
    p.stock,
    CASE
        WHEN p.stock > 0 THEN 1
        ELSE 0
    END AS is_available,
    COALESCE(sales.total_quantity_sold, 0)
        AS total_quantity_sold,
    COALESCE(sales.total_revenue, 0.00)
        AS total_revenue,
    COALESCE(sales.average_unit_price, 0.00)
        AS average_unit_price
FROM Product p
LEFT JOIN (
    SELECT
        product_id,
        SUM(quantity) AS total_quantity_sold,
        SUM(total_price) AS total_revenue,
        AVG(unit_price) AS average_unit_price
    FROM Order_Details
    GROUP BY product_id
) sales
    ON p.product_id = sales.product_id
ORDER BY p.product_id;

-- 7. Order Feature Extraction
SELECT
    o.order_id,
    o.customer_id,
    o.order_date,
    o.total_amount,
    COUNT(od.detail_id) AS total_line_items,
    COALESCE(SUM(od.quantity), 0) AS total_items,
    COALESCE(AVG(od.unit_price), 0.00)
        AS average_item_price,
    DATEDIFF(CURRENT_DATE, o.order_date)
        AS days_since_order,
    CASE
        WHEN o.total_amount >= 2000 THEN 1
        ELSE 0
    END AS is_large_order
FROM Orders o
LEFT JOIN Order_Details od
    ON o.order_id = od.order_id
GROUP BY
    o.order_id,
    o.customer_id,
    o.order_date,
    o.total_amount
ORDER BY o.order_date DESC, o.order_id;
