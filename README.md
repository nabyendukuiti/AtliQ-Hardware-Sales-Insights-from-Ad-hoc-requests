## SQL Project #1: AtliQ Hardware Sales Insights from Ad-hoc requests
![github cover](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/9ed44cc1-f2fe-4962-857d-88aa073f1605)

## Table of Contents

- [Introduction](#Introduction)
- [Question and Solution](#question-and-solution)
- [Power Bi Report](https://www.novypro.com/project/atliq-hardware-sales-insights-from-ad-hoc-requests)

***

## Introduction
Atliq Hardware, a key player in the Indian computer hardware industry with a global reach, is actively pursuing data insights to inform and enhance its product sales strategy. Focused on data-driven decision-making, the company is poised to unlock valuable insights for strategic business growth.

Task:
1.    Check ‘ad-hoc-requests.pdf’ - there are 10 ad hoc requests for which the business needs insights.
2.    You need to run a SQL query to answer these requests. 
3.    The target audience of this dashboard is top-level management - hence you need to create a presentation to show the insights.


## Question and Solution
> **1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.**
```
SELECT 
	DISTINCT c.market AS Market,
	ROUND((SUM(g.gross_price * s.sold_quantity))) AS Gross_sales_Amount
FROM dim_customer c
INNER JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
INNER JOIN fact_gross_price g ON g.product_code = s.product_code
WHERE customer = 'Atliq Exclusive' AND region = 'APAC'
GROUP BY 1
ORDER BY 2 DESC;
```
#### Answer 1
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/8d7cabf4-8a84-4fe6-aabb-6ab5c5d2ac4d)


> **2. What is the percentage of unique product increase in 2021 vs. 2020? 
The final output contains these fields: unique_products_2020,unique_products_2021,percentage_chg**
```
SELECT 
	a.p20 as unique_products_2020,
    	b.p21 as unique_products_2021,
	round((p21-p20)*100/p20,2) as percentage_chg
from 
	(
	(SELECT COUNT(DISTINCT product_code) as p20
		FROM fact_manufacturing_cost
		WHERE cost_year = 2020) a,
	(SELECT COUNT(DISTINCT product_code) as p21
		FROM fact_manufacturing_cost
		WHERE cost_year = 2021) b
	 );
```
#### Answer 2
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/89ac0575-f98c-4415-bf00-f3ec745f0f94)
	

> **3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. 
The final output contains 2 fields, segment & product_count**
```
SELECT 
	segment,
    	COUNT(DISTINCT product_code) as product_count
FROM dim_product
GROUP BY segment
ORDER BY 2 DESC;
```
#### Answer 3
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/1241fceb-45da-4fea-a6ee-66126832f39c)


> **4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? 
The final output contains these fields: segment, product_count_2020, product_count_2021, difference**
```
WITH CTE1 AS(
SELECT 
	DISTINCT d.segment, 
    	COUNT(DISTINCT m.product_code) as pc20
FROM dim_product d 
INNER JOIN fact_manufacturing_cost m USING(product_code)
WHERE cost_year = 2020
GROUP BY d.segment
),
CTE2 AS(
SELECT 
	DISTINCT d.segment, 
    	COUNT(DISTINCT m.product_code) as pc21
FROM dim_product d 
INNER JOIN fact_manufacturing_cost m USING(product_code)
WHERE cost_year = 2021
GROUP BY d.segment
)
SELECT 
	CTE1.segment,
	CTE1.pc20 as product_count_2020, 
    	CTE2.pc21 as product_count_2021,
    	(CTE2.pc21-CTE1.pc20) AS difference 
from CTE1,CTE2
WHERE CTE1.segment=CTE2.segment
ORDER BY 4 DESC
LIMIT 1;
```
#### Answer 4
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/2f59434e-cd8e-47ed-8b7b-51641f0d0fba)
	

> **5. Get the products that have the highest and lowest manufacturing costs. 
The final output should contain these fields: product_code,product,manufacturing_cost **
```
SELECT 
	m.product_code,
    	p.product,
    	m.manufacturing_cost
FROM dim_product p
INNER JOIN fact_manufacturing_cost m USING(product_code)
WHERE manufacturing_cost = (SELECT MAX(manufacturing_cost) from fact_manufacturing_cost) 
	  OR
      manufacturing_cost = (SELECT MIN(manufacturing_cost) from fact_manufacturing_cost)
ORDER BY 3 DESC;
```
#### Answer 5
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/09646a54-0e3e-4d3f-ba1c-c7dfaf9f1e1f)
	

> **6. Generate a report which contains the top 5 customers who received an 
average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. 
The final output contains these fields: customer_code, customer,average_discount_percentage**
```
SELECT 
	i.customer_code,
	c.customer,
    	ROUND(AVG(i.pre_invoice_discount_pct)*100,2) as average_discount_percentage
FROM fact_pre_invoice_deductions i
INNER JOIN  dim_customer c USING(customer_code)
WHERE fiscal_year = 2021 AND market = 'India'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;
```
#### Answer 6
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/fce3c8f1-c731-4c63-9674-7fd8f3f3e036)

 
> **7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month . 
This analysis helps to get an idea of low and high-performing months and take strategic decisions. 
The final report contains these columns: Month, Year,Gross sales Amount**
```
SELECT  
	MONTHNAME(s.date) as `month`,
    	YEAR(s.date) as `year`,
    	CONCAT('$',(ROUND((SUM(g.gross_price * s.sold_quantity))/1000000,2))) AS Gross_sales_Amount
FROM dim_customer c
INNER JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
INNER JOIN fact_gross_price g ON g.product_code = s.product_code
WHERE c.customer='Atliq Exclusive'
GROUP BY s.date
ORDER BY s.date,3;

```
#### Answer 7
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/469294c1-a509-4151-999d-d78e9214f751)


 > **8. In which quarter of 2020, got the maximum total_sold_quantity? 
The final output contains these fields sorted by the Quarter, total_sold_quantity **
```
SELECT 
	CASE
		WHEN MONTH(date) IN (9,10,11) then 'Q1'
		WHEN MONTH(date) IN (12,1,2) then 'Q2'
		WHEN MONTH(date) IN (3,4,5) then 'Q3'
		ELSE 'Q4'
    	END AS Quarters,
    	SUM(sold_quantity) AS total_sold_quantity
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY 1
ORDER BY 2 DESC;
```
#### Answer 8
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/502794c1-2228-487f-8bb8-435f96dbb61a)
	

> **9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? 
The final output contains these fields:channel,gross_sales_mln,percentage **
```
WITH CTE AS (
SELECT 
	c.channel,
	ROUND((SUM(g.gross_price * s.sold_quantity))/1000000,2) AS gross_sales_mln
FROM dim_customer c
INNER JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
INNER JOIN fact_gross_price g ON g.product_code = s.product_code
WHERE s.fiscal_year = 2021
GROUP BY 1
ORDER BY 2 DESC
)
SELECT 
	channel,
	gross_sales_mln,
	round(gross_sales_mln*100/sum(gross_sales_mln) over(),2) AS percentage
FROM cte
GROUP BY 1,2;
```
#### Answer 9
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/f8502715-9a7a-408d-b9e4-f9ec9f1fe442)

 
> **10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? 
The final output contains these fields: division, product_code, product, total_sold_quantity, rank_order**
```
WITH CTE AS(
SELECT 
	p.division,
    	s.product_code,
    	p.product,
    	SUM(s.sold_quantity) AS total_sold_quantity,
    	DENSE_RANK() OVER(PARTITION BY division ORDER BY SUM(s.sold_quantity) DESC) AS rank_order 
FROM dim_product p
INNER JOIN fact_sales_monthly s USING(product_code)
WHERE fiscal_year = 2021 
GROUP BY 1,2,3 
ORDER BY 4 DESC
)
SELECT 
    	division,
    	product_code,
    	product,
    	total_sold_quantity,
	rank_order 
FROM CTE
WHERE rank_order IN (1,2,3);
```
#### Answer 10
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/1efac098-7497-4f11-a5e5-383eabc3cca8)
	

## Bonus Question
> - Top 5 customer based on Gross sales
```
SELECT 
	c.customer,
	c.channel,
	ROUND((SUM(g.gross_price * s.sold_quantity))/1000000,2) AS gross_sales_mln
FROM dim_customer c
INNER JOIN fact_sales_monthly s ON c.customer_code = s.customer_code
INNER JOIN fact_gross_price g ON g.product_code = s.product_code
GROUP BY 1,2
ORDER BY 3 DESC
limit 5;
```
#### Answer Bonus
![image](https://github.com/nabyendukuiti/AtliQ-Hardware-Sales-Insights-from-Ad-hoc-requests/assets/140970847/00341c17-f82c-4260-9eb5-e9db8628d681)

