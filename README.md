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
