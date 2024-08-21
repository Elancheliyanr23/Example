# Example
WITH ProductCost AS (
    SELECT
        name,
        price,
        quantity,
        -- Calculate the maximum number of items that can be bought given the price and quantity
        CASE
            WHEN price <= 100 THEN
                CASE
                    WHEN quantity <= 100 / price THEN quantity
                    ELSE 100 / price
                END
            ELSE 0
        END AS max_items
    FROM products
),
ItemOptions AS (
    SELECT
        name,
        price,
        max_items,
        -- Calculate the total cost and total items for each option
        max_items * price AS total_cost,
        max_items AS total_items
    FROM ProductCost
),
-- Use a self-join to consider all combinations of products
Combination AS (
    SELECT
i1.name AS name1,
        i1.price AS price1,
        i1.total_items AS items1,
        i1.total_cost AS cost1,
i2.name AS name2,
        i2.price AS price2,
        i2.total_items AS items2,
        i2.total_cost AS cost2,
        -- Calculate total cost and total items for each combination
        COALESCE(i1.total_items, 0) + COALESCE(i2.total_items, 0) AS total_items,
        COALESCE(i1.total_cost, 0) + COALESCE(i2.total_cost, 0) AS total_cost
    FROM ItemOptions i1
LEFT JOIN ItemOptions i2 ON i1.name < i2.name
    UNION ALL
    SELECT
        name AS name1,
        price AS price1,
        total_items AS items1,
        total_cost AS cost1,
        NULL AS name2,
        NULL AS price2,
        NULL AS items2,
        NULL AS cost2,
        total_items AS total_items,
        total_cost AS total_cost
    FROM ItemOptions
)
SELECT MAX(total_items) AS quantity
FROM Combination
WHERE total_cost <= 100;
