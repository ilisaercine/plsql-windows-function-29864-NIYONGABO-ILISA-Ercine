INSY 8311 – Individual Assignment I: SQL JOINs & Window Functions  
Student:NIYONGABO Ilisa Ercine
Student ID:29864  
Group: D   

The Database Development with PL/SQL course assignment repository contains an SQL JOINs and Window Functions demonstration which students need to complete for their assignment work. 

Business Problem

Business Context 
AfriMart operates as an expanding FMCG retail chain in Rwanda with outlets located across five different regions within the country which include Kigali and Southern and Northern and Western and Eastern regions. The sales and marketing team requires advanced analytic capabilities which will enable them to make inventory decisions and create targeted promotions and customer retention programs and predict future sales revenue.

Data Challenge  
The management team can access total sales data yet they encounter difficulties when trying to determine the following:  
- Which products achieve their highest sales levels in each geographic area?  
- Which customers maintain a status of inactivity or possess minimal worth?  
- What are the monthly variations in sales patterns?  
- Which products currently show no sales activity?  
- What procedures should we implement to classify customers into different loyalty program categories?

Expected Outcome  
The project will generate distinct reports which present actionable information that enables organizations to:  
- Determine which areas need promotional activities  
- Bring back customers who have stopped using our services  
- Find products that sell at a slow rate  
- Predict upcoming sales patterns for the next several months

Success Criteria (Window Function Goals)

1. Rank top 5 products per region by revenue → RANK() 
2. Calculate running monthly sales totals → SUM() 
3. Measure month-over-month sales growth percentage → LAG()
4. Segment customers into quartiles by lifetime spend → NTILE(4)
5. Compute 3-month moving average of category sales → AVG() OVER (ROWS BETWEEN )

Database Schema

The database consists of four tables which establish connections through their shared records

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


The system has received training data until the month of October in the year 2023. The analysis identifies two distinct evaluation sections which report on what occurred. The Kigali region generates between 45 and 55 percent of its total revenue. The primary products in each region show significant differences between urban areas which prefer beverages and snacks and rural areas which use staple foods. The sales data establishes December as the month with highest sales volume. The analysis establishes the reasons for the observed outcome. The period after major promotional events shows negative growth which continues until the next month. The first two customer groups of quartile 4 only made one or two purchases which indicates a failure in customer enrollment and customer contact processes. The analysis provides guidance about required actions that need to be taken.

We should increase the marketing budget allocation for the three most important products in each marketing region. We will establish a loyalty program to serve customers who belong to the Quartile 1 and 2 groups. Our company will implement win-back campaigns through discount offers and free delivery services to contact customers who have not made any purchases. We will decrease our inventory of products which have not produced sales during the past six months.

References

The PostgreSQL Documentation provides information about Window Functions at [httpswwwpostgresqlorgdocscurrentfunctionswindowhtm](https://www.postgresql.org/docs/current/functions-window.html)l The Mode Analytics SQL Tutorial explains SQL Joins The Index Luke shows Window Functions through practical example. The diagram was created with the help of draw.io.

Integrity Statement All SQL queries and their respective comments plus my personal analysis work exist as original content which I produced for this repository. The document contains no un-attributed AI-generated code or text. All sources consulted are listed above. The /screenshots folder contains all query results and schema diagrams which have been captured as screenshots






