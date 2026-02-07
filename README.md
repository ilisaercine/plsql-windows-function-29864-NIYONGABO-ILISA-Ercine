INSY 8311 – Individual Assignment I: SQL JOINs & Window Functions  
Student:NIYONGABO Ilisa Ercine
Student ID:29864  
Group:    

Repository for the Database Development with PL/SQL course assignment demonstrating SQL JOINs and Window Functions.

Business Problem

Business Context 
AfriMart is a growing FMCG retail chain operating in Rwanda with stores in multiple regions (Kigali, Southern, Northern, Western, Eastern). The sales & marketing team needs deeper analytical insights to support inventory decisions, targeted promotions, customer retention programs, and revenue forecasting.

Data Challenge  
Management can see total sales, but struggles to answer:  
- Which products perform best in each region?  
- Which customers are inactive or low-value?  
- How are sales trends changing month-to-month?  
- Which products have no recent sales?  
- How can we segment customers for loyalty programs?

Expected Outcome  
Produce clear, actionable reports that help:  
- Prioritize regional promotions  
- Reactivate dormant customers  
- Identify slow-moving stock  
- Forecast short-term sales trends

Success Criteria (Window Function Goals)

1. Rank top 5 products per region by revenue → RANK() 
2. Calculate running monthly sales totals → SUM() 
3. Measure month-over-month sales growth percentage → LAG()
4. Segment customers into quartiles by lifetime spend → NTILE(4)
5. Compute 3-month moving average of category sales → AVG() OVER (ROWS BETWEEN )

Database Schema

Four related tables were created:

- regions  
- customers  
- products

- sales

ER Diagram

 <img width="676" height="383" alt="work" src="https://github.com/user-attachments/assets/ac5f21e7-830e-40f5-bb3d-8fc0df6d91fd" />

Part A sql only
 1.INNER JOIN Active sales only

 SELECT s.sale_id, s.sale_date, c.customer_name, p.product_name, s.quantity, s.amount
FROM sales s
INNER JOIN customers c ON s.customer_id = c.customer_id
INNER JOIN products p ON s.product_id = p.product_id
ORDER BY s.sale_date DESC
LIMIT 12;

<img width="764" height="257" alt="lkjhgf" src="https://github.com/user-attachments/assets/d0911986-192c-4a4a-a201-0c2a718f75bf" />

2.LEFT JOIN Inactive customers
SELECT c.customer_id, c.customer_name, r.region_name
FROM customers c
LEFT JOIN sales s ON c.customer_id = s.customer_id
JOIN regions r ON c.region_id = r.region_id
WHERE s.sale_id IS NULL;

<img width="471" height="159" alt="poi" src="https://github.com/user-attachments/assets/60cc3f04-f0f2-448a-9f74-38fd92eff1bc" />

3.RIGHT JOIN Unsold products
SELECT p.product_id, p.product_name, p.category, p.unit_price
FROM sales s
RIGHT JOIN products p ON s.product_id = p.product_id
WHERE s.sale_id IS NULL;

<img width="662" height="121" alt="wallah" src="https://github.com/user-attachments/assets/4f0218cc-8e32-4f7c-9947-92e96097cc53" />

4.FULL OUTER JOIN Data completeness check
SELECT 
    COALESCE(c.customer_name, '—') AS customer,
    COALESCE(p.product_name, '—') AS product,
    s.sale_date,
    s.amount
FROM customers c
FULL OUTER JOIN sales s ON c.customer_id = s.customer_id
FULL OUTER JOIN products p ON s.product_id = p.product_id
WHERE s.sale_id IS NULL 
   OR c.customer_id IS NULL 
   OR p.product_id IS NULL
LIMIT 20;

<img width="701" height="230" alt="full outer join" src="https://github.com/user-attachments/assets/a19f1578-9c26-4cb0-b875-82edf74fc1c3" />


5.SELF JOIN Regional customer comparison
SELECT 
    a.customer_name AS cust_a,
    b.customer_name AS cust_b,
    r.region_name,
    COUNT(DISTINCT sa.sale_id) AS purchases_a,
    COUNT(DISTINCT sb.sale_id) AS purchases_b
FROM customers a
JOIN customers b 
    ON a.region_id = b.region_id 
    AND a.customer_id < b.customer_id
JOIN regions r 
    ON a.region_id = r.region_id
LEFT JOIN sales sa 
    ON a.customer_id = sa.customer_id
LEFT JOIN sales sb 
    ON b.customer_id = sb.customer_id
GROUP BY a.customer_name, b.customer_name, r.region_name
ORDER BY r.region_name;

<img width="756" height="495" alt="dfghjk" src="https://github.com/user-attachments/assets/4536fc63-99ee-4fc9-a087-a609f4a08831" />


PART B WINDOW Fns

1.Ranking
WITH revenue AS (
    SELECT
        r.region_name,
        p.product_name,
        SUM(s.amount) AS total_revenue
    FROM sales s
    JOIN customers c ON s.customer_id = c.customer_id
    JOIN regions r ON c.region_id = r.region_id
    JOIN products p ON s.product_id = p.product_id
    GROUP BY r.region_name, p.product_name
)
SELECT
    region_name,
    product_name,
    total_revenue,
    DENSE_RANK() OVER (PARTITION BY region_name ORDER BY total_revenue DESC) AS rank
FROM revenue
WHERE rank <= 5
ORDER BY region_name, rank;

<img width="561" height="565" alt="mnbvc" src="https://github.com/user-attachments/assets/ffb9601a-255f-4588-a8af-1a50deb7aa0d" />

2.Agrregate window running total
SELECT 
    DATE_FORMAT(sale_date, '%Y-%m-01') AS sale_month,
    SUM(amount) AS monthly_total,
    SUM(SUM(amount)) OVER (ORDER BY DATE_FORMAT(sale_date, '%Y-%m-01')) AS running_total
FROM sales
GROUP BY sale_month
ORDER BY sale_month;

<img width="429" height="395" alt="kjh" src="https://github.com/user-attachments/assets/633b7d35-9611-4a7b-a3ac-1498584f16c2" />

3.Navigation Month-over-month growth
WITH monthly AS (
    SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS sales
    FROM sales
    GROUP BY month
)
SELECT
    month,
    sales,
    LAG(sales) OVER (ORDER BY month) AS prev_sales,
    ROUND(100.0 * (sales - LAG(sales) OVER (ORDER BY month)) / LAG(sales) OVER (ORDER BY month), 1) AS growth_pct
FROM monthly
ORDER BY month;

<img width="475" height="388" alt="aaaaaaaaaaa" src="https://github.com/user-attachments/assets/1b3c3035-a135-4d78-93d3-cff9f4400946" />

4.Distribution Customer quartiles
WITH monthly AS (
    SELECT DATE_TRUNC('month', sale_date) AS month, SUM(amount) AS sales
    FROM sales
    GROUP BY month
)
SELECT
    month,
    sales,
    LAG(sales) OVER (ORDER BY month) AS prev_sales,
    ROUND(100.0 * (sales - LAG(sales) OVER (ORDER BY month)) / LAG(sales) OVER (ORDER BY month), 1) AS growth_pct
FROM monthly
ORDER BY month;

<img width="455" height="392" alt="lakh" src="https://github.com/user-attachments/assets/9b280f5f-eee6-4947-8105-3124fc968980" />


Results Analysis
Descriptive : What happened?
Kigali region generates ~45–55% of revenue. Top products differ strongly by region (beverages & snacks in urban areas, staples in rural). Sales show clear December peaks.
Diagnostic – Why did it happen?
Negative month-over-month growth often follows end of major promotions. Many quartile-4 customers made only 1–2 purchases → weak onboarding / follow-up.
Prescriptive – What should we do?

Allocate more marketing budget to top-3 regional products per area
Create loyalty program for Quartile 1 & 2 customers
Launch win-back campaigns (discounts, free delivery) for inactive customers
Reduce stock of items with zero sales in last 6 months

References

PostgreSQL Documentation – Window Functions: https://www.postgresql.org/docs/current/functions-window.html
SQL Joins – Mode Analytics SQL Tutorial
Window Functions by Example – Use The Index, Luke
ER Diagram created with draw.io

Integrity Statement
All SQL queries, comments, interpretations, and analysis in this repository are my original work. No un-attributed AI-generated code or text was used. All sources consulted are listed above.
Screenshots
All query results and schema diagrams are included in the /screenshots folder.






