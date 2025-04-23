
   <img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*T-P1WsT8iDNMz6Io4mVxLg.png" width="700">

## 📊 Project Title: AdventureWorks Dataset Exploring

🤵 Author: Loc Huynh.

📆 Date: April. 01, 2025.

💻 Tools Used: SQL .

## 📑 Table of Contents

1.📌 Background & Overview

2.📂 Dataset Description & Data Structure

3,🧠 Problem Solving Process

4.📊 Explore the Dataset & Generate Insights

5.🔎 Final Conclusion & Recommendations

## 📌 Background & Overview.

### Objective:

### 📖 What is this project about?

This project utilizes advanced SQL techniques to analyze the AdventureWorks dataset, revealing trends, enhancing data visualization, and supporting decision-making within a business environment.

### 👤 Who is this project for?

➡️ Sales manager who want to understand the Sales revenue components and Sales trend.

### ❓ Business Questions:
- Analyze data on items, sales, order quantities, growth rates, top categories, top territories, and total discount costs by subcategories.
- Assess customer retention rates, stock level trends, month-over-month differences, and stock-to-sales ratios.
- Evaluate the number and value of pending orders in 2014.

## 🎯Project Outcome:
- Sales and Growth Trends: Bike Racks and Road Frames achieved the highest sales revenue and quantity, respectively. Mountain Frames and Socks showed notable year-over-year growth.
- Discounts and Customer Retention: Helmets had continuous discount promotions with increased discount costs from 2012 to 2013, while customer retention dropped significantly after the first purchase, indicating the need for better retention strategies.
- Stock and Order Management: Quarterly stock levels consistently declined, indicating robust sales. A substantial number of pending orders highlight the need to assess vendor performance for efficiency and suggest improvements to their order processes.

## 📂 Dataset Description & Data Structure
### 📌 Data Source
- Source: The AdventureWorks2019 dataset by Microsoft is a comprehensive sample database that simulates a manufacturing company's operations.

- Size: Over 121000 rows

- Format:

To access the dataset, follow these step
## 📊 Data Structure & Relationships
### 1️⃣ Tables Used:
There're 6 tables were used in this project
### 2️⃣ Table Schema & Data Snapshot
Table 1: Sales.SalesOrderHeader
Table 2: Sales.SalesOrderDetail
Table 3: Production.Product
Table 4: Production.ProductCategory
Table 5: Production.ProductSubcategory
Table 6: Production.WorkOrder
Table 7: Purchasing.PurchaseOrderHeader
### 3️⃣ Data Relationships:

- Field SalesOrderDetai.SalesOrderID is foreign key to SalesOrderHeader.SalesOrderID
- Field SalesOrderDetai.ProductID is foreign key to Product.ProductID
- Field Product.ProductID is foreign key to WorkOrder.ProductID
- Field Product.ProductSubcategoryID is foreign key to ProductSubCategory.ProductSubcategoryID


![image](https://github.com/user-attachments/assets/45757004-76fe-4bdf-b00f-d13db27bc258)

### 🧠 Problem Solving Process

| 1️⃣ Understand Problem	 | 2️⃣ Break it down into smaller pieces | 3️⃣ Ideate | 4️⃣ Implement and Review |
| --------               | --------                              | --------  | -------------------------|
| Which to be calculated (sum, count, ratio, etc.) and grouped? Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? Which columns have data corresponded to the problem? Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? Is it optimized? | We'll go through this step in the order of each query listed below |
⬇️

## 📊 Explore the Dataset & Generate Insights

Query 1️⃣: Calc Quantity of items, Sales value & Order quantity by each Subcategory in Last 12 Mth

```sql
SELECT
    FORMAT(sod.ModifiedDate, 'MMM yyyy') AS period,
    pps.Name AS name,
    SUM(sod.OrderQty) AS qty_item,
    ROUND(SUM(sod.LineTotal), 2) AS total_sales,
    COUNT(DISTINCT sod.SalesOrderID) AS order_cnt
FROM Sales.SalesOrderDetail AS sod
LEFT JOIN Production.Product AS pp
    ON sod.ProductID = pp.ProductID
LEFT JOIN Production.ProductSubcategory AS pps
    ON pp.ProductSubcategoryID = pps.ProductSubcategoryID
WHERE sod.ModifiedDate >= DATEADD(month, -12, (SELECT MAX(ModifiedDate) FROM Sales.SalesOrderDetail))
GROUP BY FORMAT(sod.ModifiedDate, 'MMM yyyy'), pps.Name
ORDER BY name, period DESC;
```

![image](https://github.com/user-attachments/assets/a560dafe-42df-49a7-b6f2-7b1a19cd06fb)

💡 Road Bikes have had the highest sales revenue in the past 12 months, indicating strong market demand.

Query 2️⃣: Calc total sale and percentage of category which has been saled the most in total values 

```sql
With cte as 
(
Select ppc.name 
	  
	  ,sum(sod.LineTotal) as total_sales
From Sales.SalesOrderDetail as sod 
LEFT JOIN Production.Product as pp 
ON sod.ProductID = pp.ProductID
LEFT JOIN Production.ProductSubcategory as pps 
ON pp.ProductSubcategoryID = pps.ProductCategoryID
LEFT JOIN Production.ProductCategory as ppc 
ON ppc.ProductCategoryID = pps.ProductCategoryID
Where ppc.name is not null 
Group by ppc.name 

)

, cte1 as (
Select name, total_sales , sum(total_sales) OVER (Order by total_sales) as accum_sales
From cte
)

Select name, total_sales
, (Select MAX(accum_sales) From cte1 )as accum_total_sales 
,format(total_sales*1.0/(Select MAX(accum_sales) From cte1),'p') as pct
from cte1
Order by total_sales
```
![image](https://github.com/user-attachments/assets/71865f98-97b3-4ad7-b560-27234fa84e0e)
💡 
- Highest Total Sales: The "Components" category has significantly higher total sales (614,732,125.11) compared to all other categories.
- Largest Percentage Contribution: "Components" account for a massive 73.14% of the total sales. This indicates that components are the primary revenue driver for the business.
Query 3️⃣: Usinng Time Series to identify trend line of each category by number of transaction

```sql
With cte as (
Select ppc.name as category
	  
	 --, pps.name as subcategory
	 , year(sod.ModifiedDate) as year
	 , month(sod.ModifiedDate) as month
	 , convert(nvarchar(6),sod.ModifiedDate,112) as time_calendar
	 , count(SalesOrderID) as number_trans
	 
From Sales.SalesOrderDetail as sod 
LEFT JOIN Production.Product as pp 
ON sod.ProductID = pp.ProductID
LEFT JOIN Production.ProductSubcategory as pps 
ON pp.ProductSubcategoryID = pps.ProductCategoryID
LEFT JOIN Production.ProductCategory as ppc 
ON ppc.ProductCategoryID = pps.ProductCategoryID
Where ppc.name is not null  
Group by ppc.name , year(sod.ModifiedDate) , month(sod.ModifiedDate) , convert(nvarchar(6),sod.ModifiedDate,112)

)

Select  year,month, time_calendar
, SUM(case when category = 'Bikes' then number_trans end) as Bikes
, SUM(case when category = 'Accessories' then number_trans end) as Accessories
, SUM(case when category = 'Components' then number_trans end) as Components
, SUM(case when category = 'Clothing' then number_trans end) as Clothing
From cte
Group by   year , month , time_calendar

```

![image](https://github.com/user-attachments/assets/90199364-43cd-4ffc-a4f3-cc2cdcecead4)
![image](https://github.com/user-attachments/assets/9ff16dc5-1f99-416a-8c58-8fb7ec4d4be0)

💡 
- Components Dominate: The "Components" category consistently shows the highest number of transactions throughout the entire period displayed.
- Accessories Show Volatility: The "Accessories" category experiences significant fluctuations, with peaks and troughs much more pronounced than other categories.
- Bikes Show Relative Stability: The "Bikes" category maintains a relatively low and stable number of transactions across the observed timeframe.
- Clothing Shows Growth and Fluctuation: The "Clothing" category starts with very low transaction numbers but exhibits a noticeable increase starting around mid-2013. It also shows considerable volatility after this growth..

Query 4️⃣:  Evalue number of customer in 2013 by month

```sql
Select month(orderdate) as subsequent_month 
      ,count(distinct(CustomerID)) as number_customer
From Sales.SalesOrderHeader
Where year(OrderDate) = 2013
Group by month(orderdate)
Order by subsequent_month
```

![image](https://github.com/user-attachments/assets/9b4bc43f-7703-4a08-a440-228cb2e02189)
![image](https://github.com/user-attachments/assets/764d077a-7885-44ee-a443-267f039eb43f)

💡 Significant Growth After Month 5: The number of customers remains relatively stable in the first five months, fluctuating slightly between roughly 300 and 500. However, starting around month 6, there is a sharp and substantial increase in the number of customers


Query 5️⃣:  Query 5: Retention rate of Customer in 2013 with status of Successfully Shipped (Cohort Analysis)

```sql
WITH 
t0 AS (
    SELECT *
    FROM Sales.SalesOrderHeader
    WHERE YEAR(OrderDate) = 2013 and Status = 5
)
, t1 AS (
    SELECT  TotalDue,OrderDate, CustomerID ,  FORMAT( OrderDate , 'yyyy-MM-01' ) Order_Month
    FROM t0
    )
, t2 AS (
    SELECT CustomerID, FORMAT(MIN(OrderDate), 'yyyy-MM-01' ) Cohort_Month
    FROM t0
    GROUP BY CustomerID
)
, t3 AS (
    SELECT t1.*, t2.Cohort_Month, DATEDIFF(month, CAST(t2.Cohort_Month AS date), CAST(t1.Order_Month AS date))+1 AS CohortIndex
    FROM t1
    JOIN t2 ON t1.CustomerID = t2.CustomerID
)
, t4 AS (
    SELECT Cohort_Month, Order_Month, CohortIndex, COUNT(DISTINCT CustomerID) Count_CustomerID
    FROM t3
    GROUP BY Cohort_Month, Order_Month, CohortIndex 
)
SELECT * FROM t4

```

![image](https://github.com/user-attachments/assets/817f1bf2-b4b4-49e2-afd4-fba42f761cf8)





