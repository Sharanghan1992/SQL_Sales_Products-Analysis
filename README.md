# SQL_Sales_Products-Analysis
This project contains a realistic 4-table sales dataset (Customers, Products, Stores, Sales) with 500+ rows in the fact table. The dataset was generated with realistic customer names, store names, and product details to simulate a retail sales environment.

I created a collection of SQL practice questions and solutions for MS SQL Server, ranging from beginner to advanced:

🔹 Beginner: Basic filtering, sorting, and aggregations

🔹 Intermediate: Joins, groupings, top-N analysis

🔹 Advanced: Common Table Expressions (CTEs), CASE statements, window functions (RANK, LAG, running totals), and analytical queries

Key examples include:

- Top customers by spending
- Monthly sales trends with percentage growth
- Store performance classification (Above/Below Average)
- Top-selling product per store using window functions
- Customer segmentation with CASE expressions

This project is designed as a SQL practice ground for analysts and developers preparing for real-world analytics or technical interviews.

```SQL
--Q1) List all products in the ‘Electronics’ category.
SELECT * 
FROM
    products
WHERE
    Category = 'Electronics'

--Q2) Show all sales that occurred in January 2024
SELECT *
FROM
    sales
WHERE
    MONTH(SaleDate) = 1
    AND
    YEAR(SaleDate) = 2024

--Q3) Retrieve the first 10 customers ordered by last name.
SELECT TOP 10
    CustomerID,
    FirstName,
    LastName
FROM
    customers
ORDER BY
    LastName

--Q4) Find the total sales amount per store.
SELECT
    stores.StoreName,
    ROUND(SUM(sales.TotalAmount),2) AS total_sales
FROM
    stores
LEFT JOIN
    sales ON sales.StoreID = stores.StoreID
GROUP BY
    stores.StoreName
ORDER BY
    SUM(sales.TotalAmount) DESC

--Q5) Find the total quantity sold for each category.
SELECT
    products.Category,
    SUM(sales.Quantity) AS total_units_sold
FROM
    products
LEFT JOIN
    sales ON sales.ProductID = products.ProductID
GROUP BY
    products.Category
ORDER BY
    SUM(sales.Quantity) DESC

--Q6) Get the top 5 customers by total spending.
SELECT TOP 5
    c.FirstName,
    c.LastName,
    ROUND(SUM(s.TotalAmount),2) AS total_spent
FROM
    customers c
JOIN
    sales s ON s.CustomerID = c.CustomerID
GROUP BY
    c.FirstName,
    c.LastName
ORDER BY
   SUM(s.TotalAmount) DESC 

--Q7) Calculate the month-over-month percentage change in total sales.
WITH monthly_sales AS
(
    SELECT
        YEAR(SaleDate) AS year,
        MONTH(SaleDate) AS month,
        SUM(TotalAmount) AS total_sales
    FROM
        sales
    GROUP BY
        YEAR(SaleDate),
        MONTH(SaleDate)
)
SELECT
    year,
    month,
    total_sales,
    LAG(total_sales) OVER(ORDER BY month) AS prev_month_sale,
    ROUND((total_sales-LAG(total_sales) OVER(ORDER BY year, month))/LAG(total_sales) OVER(ORDER BY year, month),2)*100 AS percentage_change
FROM
    monthly_sales

--Q8) Find the top-selling product in each store by total sales amount.
SELECT * FROM products
SELECT * FROM sales
SELECT * FROM stores

WITH product_total_sales AS
(
    SELECT
        st.StoreID,
        st.StoreName,
        p.ProductName,
        SUM(s.TotalAmount) AS total_sales
    FROM
        sales s
    JOIN
        stores st ON s.StoreID = st.StoreID
    JOIN
        products p ON p.ProductID = s.ProductID
    GROUP BY
        st.StoreID,
        st.StoreName,
        p.ProductName
),
Rank_products AS
(
SELECT
    *,
    RANK() OVER(PARTITION BY StoreID ORDER BY total_sales DESC ) AS RANK
FROM
    product_total_sales
)
SELECT
    StoreID,
    StoreName,
    ProductName,
    total_sales,
    RANK
FROM
    Rank_products
WHERE
   RANK = 1
ORDER BY
    StoreName

--Q9) Find customers whose total spending is above the overall average spending 
-- per customer.
WITH customer_spending AS
(
    SELECT
        c.CustomerID,
        c.Firstname,
        c.LastName,
        SUM(s.TotalAmount) AS total_spending
    FROM
        customers c
    JOIN
        sales s ON s.CustomerID = c.CustomerID
    GROUP BY
        c.CustomerID,
        c.Firstname,
        c.LastName
)
SELECT
    CustomerID,
    FirstName,
    LastName,
    total_spending
FROM
    customer_spending
WHERE
    total_spending > (SELECT
                        AVG(total_spending) FROM customer_spending)
ORDER BY
    total_spending DESC

--Q10) Find the top 3 categories by sales in each year using a CTE.
WITH category_sales AS
(
    SELECT
        YEAR(s.SaleDate) AS year,
        p.category,
        SUM(s.TotalAmount) As total_sales
    FROM
        products p
    JOIN
        sales s ON s.ProductID = p.ProductID
    GROUP BY
        YEAR(s.SaleDate),
        p.category
),
rank_category AS
(
    SELECT
        year,
        category,
        total_sales,
        RANK() OVER(PARTITION BY year ORDER by total_sales DESC) as RANK
    FROM
        category_sales
)
SELECT
    year,
    category,
    total_sales,
    RANK
FROM
    rank_category
WHERE
    RANK <= 3

--Q11) Calculate average monthly spending per customer and filter customers spending above the average.
SELECT * FROM customers
SELECT * FROM products
SELECT * FROM sales
SELECT * FROM stores

WITH customer_total_spending AS
(
    SELECT
        c.CustomerID,
        c.FirstName,
        c.LastName,
        YEAR(SaleDate) as year,
        MONTH(s.SaleDate) as month,
        SUM(s.TotalAmount) AS monthly_total
    FROM
        sales s
    JOIN
        customers c ON c.CustomerID = s.CustomerID
    GROUP BY
        c.CustomerID,
        c.FirstName,
        c.LastName,
        YEAR(SaleDate),
        MONTH(s.SaleDate)
),
customer_avg_spending AS
(
    SELECT
        CustomerID,
        FirstName,
        LastName,
        AVG(monthly_total) AS avg_spending
    FROM
        customer_total_spending
    GROUP BY
        CustomerID,
        FirstName,
        LastName
)
SELECT
    *
FROM
    customer_avg_spending
WHERE
    avg_spending > (SELECT 
                        AVG(avg_spending) FROM customer_avg_spending)
ORDER BY
    avg_spending DESC
