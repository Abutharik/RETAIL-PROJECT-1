-- 1. Remove NULL or missing data from sales and inventory tables
DELETE FROM sales
WHERE sales IS NULL OR cost IS NULL OR profit IS NULL;

DELETE FROM inventory
WHERE inventory_days IS NULL OR stock_units IS NULL;

-- 2. Total Sales, Cost, and Profit
SELECT 
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND(SUM(cost), 2) AS total_cost,
    ROUND(SUM(profit), 2) AS total_profit
FROM sales;

-- 3. Profit Margin % by Category
SELECT 
    category,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND((SUM(profit) / NULLIF(SUM(sales), 0)) * 100, 2) AS profit_margin_percentage
FROM sales
GROUP BY category
ORDER BY profit_margin_percentage ASC;

-- 4. Profit Margin % by Sub-Category
SELECT 
    category,
    sub_category,
    ROUND(SUM(profit), 2) AS total_profit,
    ROUND(SUM(sales), 2) AS total_sales,
    ROUND((SUM(profit) / NULLIF(SUM(sales), 0)) * 100, 2) AS profit_margin_percentage
FROM sales
GROUP BY category, sub_category
ORDER BY profit_margin_percentage ASC;

-- 5. Inventory Turnover = Total Sales / Average Stock Units
SELECT 
    s.product_id,
    p.category,
    ROUND(SUM(s.sales), 2) AS total_sales,
    ROUND(AVG(i.stock_units), 2) AS avg_inventory,
    ROUND(SUM(s.sales) / NULLIF(AVG(i.stock_units), 0), 2) AS inventory_turnover_ratio
FROM sales s
JOIN inventory i ON s.product_id = i.product_id
JOIN products p ON s.product_id = p.product_id
GROUP BY s.product_id, p.category
ORDER BY inventory_turnover_ratio ASC;

-- 6. Slow-moving products = High inventory_days + Low sales
SELECT 
    s.product_id,
    p.category,
    i.inventory_days,
    ROUND(SUM(s.sales), 2) AS total_sales
FROM sales s
JOIN inventory i ON s.product_id = i.product_id
JOIN products p ON s.product_id = p.product_id
GROUP BY s.product_id, p.category, i.inventory_days
HAVING i.inventory_days > 60 AND SUM(s.sales) < 1000
ORDER BY i.inventory_days DESC;

-- 7. Overstocked products = High stock + Low profit
SELECT 
    s.product_id,
    p.category,
    SUM(i.stock_units) AS total_stock,
    ROUND(SUM(s.profit), 2) AS total_profit
FROM sales s
JOIN inventory i ON s.product_id = i.product_id
JOIN products p ON s.product_id = p.product_id
GROUP BY s.product_id, p.category
HAVING SUM(i.stock_units) > 500 AND SUM(s.profit) < 500
ORDER BY total_stock DESC;

