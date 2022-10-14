# Sample-Superstore-Sales-Data -- Data Exploration in SQL

In this project I carried out an **Exploratory data analysis on the sample superstore** to draw insights on the state of the business between 2013-2015, 
to see how well or poorly the business has performed and to see where improvements can be made and improved to increase revenue and reach of the business.


Problem statement 

This project takes real world data from the internet from top companies and ansers the following questions:

What was the best month for sales? How much was earned that month?
What city sold the most product?
What time should we display advertisemens to maximize the likelihood of customer’s buying product?
What products are most often sold together?
What product sold the most? Why do you think it sold the most?

**Insights and Recommendations**
Sample superstore performed business for 36 months in the 3 years of data collected. I 2014, they were in operation for only 6 months and in that 6 months revenue
generated was low as compared to other years. From the data collected, there is usully a boom in salse sfrom July down to Novemeber being the month with the highest sales yearly.

In Novemeber 2003 classic cars had the most sales of 114, in 2004 Classic cars had the most sales of 104 and in 2005 thouh they operated for 5 Months only, they sold over 41 classic cars
Sample Superstore should ensure that eneough staff are employed between October, November and December to ensure that enough hands are available to handle the bulk of sales.





--inspecting data 
select *
from sales_data_sample
![sales  data](https://user-images.githubusercontent.com/108904370/191129642-c5d3f419-ee55-4989-9234-39b86f993e6b.PNG)


--checking for unique values in each KPI columns
select distinct status from sales_data_sample
select distinct YEAR_ID from sales_data_sample
select distinct PRODUCTLINE from sales_data_sample
select distinct COUNTRY from sales_data_sample
select distinct DEALSIZE from sales_data_sample
select distinct TERRITORY from sales_data_sample
![distinct](https://user-images.githubusercontent.com/108904370/191129838-dc906719-4cf7-4fdc-a60a-beb51bae6933.PNG)


--Analysis 
--Grouping sales by productline
SELECT PRODUCTLINE, sum(sales) as Revenue
FROM sales_data_sample
Group by PRODUCTLINE
ORDER BY Revenue desc

![grouping](https://user-images.githubusercontent.com/108904370/191129932-d359f132-0a68-4957-b949-14184dc34d13.PNG)


--sales across the years 
SELECT YEAR_ID, sum(sales) as Revenue
FROM sales_data_sample
GROUP BY YEAR_ID
ORDER BY Revenue DESC
![sales across years](https://user-images.githubusercontent.com/108904370/191130032-367a0319-08b3-4063-b284-69dd51e128d5.PNG)


--Analyzing why there was less sales in 2005
SELECT distinct MONTH_ID
FROM sales_data_sample
WHERE YEAR_ID = 2005

![2005 sales month](https://user-images.githubusercontent.com/108904370/191131603-d5c9b487-842b-4613-a969-1c9e635bdccf.PNG)


--why was there more sales in 2004 and 2003?

SELECT distinct MONTH_ID
FROM sales_data_sample
WHERE YEAR_ID = 2004

![2004 sales months](https://user-images.githubusercontent.com/108904370/191131628-d19f6a05-b763-4787-8040-a23e4e5a5563.PNG)

 
 SELECT MONTH_ID, sum(sales), count(ORDERNUMBER) Frequency
FROM sales_data_sample
WHERE YEAR_ID = 2003
Group by MONTH_ID
![2005 sales month](https://user-images.githubusercontent.com/108904370/191131657-a4728f7b-9ea4-4f2c-9db4-4e03e274a52c.PNG)



--Analyzing Dealsize Revenue
SELECT DEALSIZE, sum(sales) AS Revenue
FROM sales_data_sample
GROUP BY DEALSIZE
ORDER BY Revenue

![dealsize revenue](https://user-images.githubusercontent.com/108904370/191131782-815a2fe0-caab-48de-acb2-2ba4b08e7793.PNG)


--Best Month for sales in a specific year?


SELECT MONTH_ID, sum(sales) AS Revenue, COUNT(ORDERNUMBER) Frequency
FROM sales_data_sample
WHERE YEAR_ID = 2003 
GROUP BY MONTH_ID
ORDER BY Revenue desc

![2003 best sales month](https://user-images.githubusercontent.com/108904370/191132132-9eef9cf8-8a27-4e0a-baf0-9b95d2e3be25.PNG)


SELECT MONTH_ID, sum(sales) AS Revenue, COUNT(ORDERNUMBER) Frequency
FROM sales_data_sample
WHERE YEAR_ID = 2004
GROUP BY MONTH_ID
ORDER BY Revenue desc

![2004  best sales month](https://user-images.githubusercontent.com/108904370/191132157-b5d60e7f-48d0-4305-b9dd-64b55b4317e0.PNG)


SELECT MONTH_ID, sum(sales) AS Revenue, COUNT(ORDERNUMBER) Frequency, PRODUCTLINE
FROM sales_data_sample
WHERE YEAR_ID = 2005 AND MONTH_ID = 5
GROUP BY MONTH_ID, PRODUCTLINE
ORDER BY Revenue desc

![2005 best sales month](https://user-images.githubusercontent.com/108904370/191132202-9bf13f33-ddad-430d-afc1-8fef0b85ca39.PNG)


-- The best months fro sales seems to be November, December and October
--Lets Find out what sells more


SELECT MONTH_ID, PRODUCTLINE, sum(sales) as Revenue, Count(ORDERNUMBER) Frequency
FROM sales_data_sample
WHERE YEAR_ID = 2004 AND MONTH_ID = 11
GROUP BY MONTH_ID, PRODUCTLINE
ORDER BY Revenue desc
-- in 2003 classic cars had the most sales of 114
--in 2004 Classic cars had the most sales of 104
-- in 2005 thouh they operated for 5 Months only, they sold over 41 classic cars

-- Who are our best customers (using RFM analysis)
DROP TABLE IF EXISTS #rfm
;with rfm as 
(
	select 
		CUSTOMERNAME, 
		sum(sales) MonetaryValue,
		avg(sales) AvgMonetaryValue,
		count(ORDERNUMBER) Frequency,
		max(ORDERDATE) last_order_date,
		(select max(ORDERDATE) from sales_data_sample) max_order_date,
		DATEDIFF(DD, max(ORDERDATE), (select max(ORDERDATE) from sales_data_sample)) Recency
	from sales_data_sample
	group by CUSTOMERNAME
),
rfm_calc as
(

	select r.*,
		NTILE(4) OVER (order by Recency desc) rfm_recency,
		NTILE(4) OVER (order by Frequency) rfm_frequency,
		NTILE(4) OVER (order by MonetaryValue) rfm_monetary
	from rfm r
)
select 
	c.*, rfm_recency+ rfm_frequency+ rfm_monetary as rfm_cell,
	cast(rfm_recency as varchar) + cast(rfm_frequency as varchar) + cast(rfm_monetary  as varchar)rfm_cell_string
into #rfm
from rfm_calc c

select CUSTOMERNAME , rfm_recency, rfm_frequency, rfm_monetary,
	case 
		when rfm_cell_string in (111, 112 , 121, 122, 123, 132, 211, 212, 114, 141) then 'lost_customers'  --lost customers
		when rfm_cell_string in (133, 134, 143, 244, 334, 343, 344, 144) then 'slipping away, cannot lose' -- (Big spenders who haven’t purchased lately) slipping away
		when rfm_cell_string in (311, 411, 331) then 'new customers'
		when rfm_cell_string in (222, 223, 233, 322) then 'potential churners'
		when rfm_cell_string in (323, 333,321, 422, 332, 432) then 'active' --(Customers who buy often & recently, but at low price points)
		when rfm_cell_string in (433, 434, 443, 444) then 'loyal'
	end rfm_segment

from #rfm


![sales items together](https://user-images.githubusercontent.com/108904370/191132345-a2a210d3-a345-4e3e-8252-7b9f4c26a3b6.PNG)



--what products are mostly sold together


select distinct OrderNumber, stuff(

	(select ',' + PRODUCTCODE
	from sales_data_sample p
	where ORDERNUMBER in 
		(

			select ORDERNUMBER
			from (
				select ORDERNUMBER, count(*) rn
				FROM sales_data_sample
				where STATUS = 'Shipped'
				group by ORDERNUMBER
			)m
			where rn = 3
		)
		and p.ORDERNUMBER = s.ORDERNUMBER
		for xml path (''))

		, 1, 1, '') ProductCodes

from sales_data_sample s
order by 2 desc

--What city has the highest number of sales in a specific country
select city, sum (sales) Revenue
from sales_data_sample
where country = 'UK'
group by city
order by 2 desc


![sold together](https://user-images.githubusercontent.com/108904370/191132451-38ee5fcc-ab89-423f-9411-c5b732eb535b.PNG)



---What is the best product in United States?


select country, YEAR_ID, PRODUCTLINE, sum(sales) Revenue
from sales_data_sample
where country = 'USA'
group by  country, YEAR_ID, PRODUCTLINE
order by 4 desc


![countries](https://user-images.githubusercontent.com/108904370/191132527-34883ee2-8298-4f4a-8417-3d27bcfcc465.PNG)



