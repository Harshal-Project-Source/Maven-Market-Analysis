# Title : Maven Market Dashboard

### Dashboard Link : 

## Overview :

Maven Market, a multi-national grocery chain with the location in Canada, Mexico and United States. The report includes various metrics such as product brands, total transaction, total profits, profit margins and return rate.
KPIs (Total Profits, Total Returns, Current Month Transactions), a gauge chart showing revenue vs target, weekly revenue trends, a slicer for regions, and a treemap categorizing transaction by country, state and city.

## Problem Statement :

The management team wants to analyse the performance of products brands across different regions (USA, Mexico, Canada).
Specifically they want to:

1.Identify which product brands are underperforming in terms of profit margin and return rate.

2.Analyze regional trends in terms of total transaction and profit margins for each country, state and city.

3.Assess whether current sales performance is on track to meet targets for the month.

## Steps to Create Report :

- Step 1 : Load data into Power BI Desktop, dataset is a csv file, csv file includes (Maven Market Customers, Products, Stores, Regions, Calendar, Transactions that is all 6 tables).
- Step 2 : Rename the tables, check data types (ex. customer_id & product_id should be whole numbers, product_retail_price & product_cost should be decimal numbers, etc)
- Step 3 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.
- Step 4 : It was observed that in none of the columns contains error or empty values.
- Step 5 : Also since by default, profile will be opened only for 1000 rows so you need to select "column profiling based on entire dataset".
- Step 6 :  In a customer table create new column named "birth_year" to extract the year from the "birthdate column" and format as text. (Select "birthdate column", in Add column tab click on extract option and using text 			after delimiter option extract the year from "birthdate column")
- Step 7 : Select "full_name" and "last_name" column in Customer table. In Add column tab select merge columns option, merge the both columns and separated by space, named new column "full_name".
- Step 8 : Using a conditional columns in Add column tab create "has_children" which equals "N" if "total_children" = 0, otherwise "Y".
- Step 9 : Use the data tools in the query editor to add the following columns: Start of week, Name of Day, Start of Month, Name of Month, Quarter of Year, Year.

- Step 10 : Confirm that all tables are now accessible within both Table View & the Model View. 
		
#### Link of solutions screenshot : 

- Step 11 : In the Model View, arrange tables with the dimensions tables above the fact tables.

        Connect Transaction_Data (Fact Table) to Customers, Products and Stores using valid Primary/Foreign Keys
        Connect Transaction_Data (Fact Table) to Calendar using both "date" field, with an inactive "stock_date" relationship
        Connect Return_Data to Products, Calendar and Stores using valid Primary/Foreign Keys
        Connect Stores to Regions as a "Snowflake" schema 

- Step 12 : Confirms :

        All relationship follow one-to-many cardinality, with primary keys (1) on the dimension table side and foreign keys (*) on the fact table side
        Filters are all one-way (no two-way filters)
        Filters Context flows "downstream" from dimension tables to fact table
        Fact tables are connected via shared lookup tables (not directly to each others)

	![data_model](https://github.com/user-attachments/assets/a7f985b7-3977-4303-b102-8ebc054c7169)


- Step 13 : For creating new columns following DAX expression was written :

		Weekend = IF ( 'Calendar' [Day Name] = "Saturday" || 'Calendar' [Day Name] = "Sunday",  "Y",  "N")

		End of Month = EOMONTH ( 'Calendar' [Start of Month], 'Calendar' [Month] )

		Priority =  IF ( Customers [homeowner] = "Y"  && Customers [member_card ] = "Golden",
					    "High",
 					   "Standard" )
		
		Year_Since_Remodel = DATEDIFF (Stores [last_remodel_date], TODAY(), YEAR)
		
- Step 14 : For creating new measures following DAX expression was written :

Create new measures named Total Transaction and Total Returns to calculate returns row each data table

		Total Transaction = COUNT(Transaction_Data [transaction_date])
		Toatl Returns = COUNT(Returns_Data [return_date])

Create measure Total Cost based on transaction quantity and product cost

		Total Cost = SUMX(Transaction_Data,Transaction_Data[quantity] * RELATED(Products [product_cost]))

Create measures total profit and profit margins

		Total Profit = [Total Revenue] - [Total Cost]
		Profit Margin = [Total Profit] / [Total Revenue]

Calculate last month transaction, profit and returns

		Last Month Transaction = CALCULATE([Total Transaction], DATEADD('Calendar'[date],-1,MONTH))
		Last Month Profit = CALCULATE([Total Profit], DATEADD('Calendar'[date],-1,MONTH))
		Last Month Returns = CALCULATE([Toatl Returns], DATEADD('Calendar'[date],-1,MONTH))

Create a new measure named revenue Target based on 5% lift over the previous month revenue

		Revenue Target = ([Last Month Revenue] * 1.05)

Create Total Revenue measures

		Total Revenue = SUMX(Transaction_Data,Transaction_Data[quantity] * RELATED(Products[product_retail_price]))


- Step 15 : Creating visuals :
		
	Insert a Matrix visual to show Total Transactions, Total Profits, Profit Margin and Return Rate by Product_Brands (on rows)

	![matrix_visual](https://github.com/user-attachments/assets/24cfd827-ed10-414b-915d-8e32103c24ef)


	Add KPI Card to show Total Transaction with Start of Month as the trend axis and Last Month Transaction as the target goal

	![kpis](https://github.com/user-attachments/assets/b43d567b-649e-4588-aa48-2f05a7fb647a)


	Add Map visual to show Total Transaction by store_city

	![map](https://github.com/user-attachments/assets/99bf28aa-31c0-43bb-b320-e0d7fcb1ae72)


	Next to the map, add Treemap visual to break down Total Transaction by store_country
	
	![treemap](https://github.com/user-attachments/assets/deabc1a4-3c48-4f2b-b5e2-3833ff84b9d1)

	Add a Column Chart to show Total Revenue by week, and format it
		
	![weekly_revenue_trending](https://github.com/user-attachments/assets/ede0a527-ac2c-4b29-a84b-277cd9a67744)


	In the lower right, add a Gauge Chart to show Total Revenue against Revenue Target (as "target value")

	![gauge_chart](https://github.com/user-attachments/assets/9861c244-75a2-4202-80ae-a563c2012a21)
	
	Add slicer for store country
	
	![slicers](https://github.com/user-attachments/assets/b3dce818-f088-46a6-bf27-92eb99ea4ede)

## Snapshot of Dashboard (Power BI)

![final_dashboard](https://github.com/user-attachments/assets/28cd06de-5d5d-4948-af83-9a1458c2d29f)

## Solution :

1.Problem : Identify which product brands are underperforming in terms of profit margin and return rate.

Solution :

Create a filter on Matrix Table : 

Filter Type : Visual Level TOP N filter,  Show Item : Bottom 30, By Value : Total Transactions . 
We obtain bottom 30 products which are underperforming in terms of Total Transaction.

Focus on profit margin and return rates :

Use the matrix table to sort by profit margin in descending order. This will allow to quickly identify the product brands with lowest profit margin.
Sort the matrix table by return rates in descending order. Product brands with high returns rate have issues with customer satisfaction and profitability.

Combine Insights :

Products brands with both low profit margins and high returns rates. These are the product brands that are likely underperforming.

Recommendation : 

- Focus on improving product quality and marketing strategies for specific products.
- Offering discounts and promotions to improve satisfaction.

2.Problem : Analyze regional trends in terms of total transaction and profit margins for each country, state and city.

Solution :

Use the Treemap :

Select the slicer for region (USA, Mexico, Canada) and analyse the performance of different categories (country, state and city).

Filter by Total Transaction :

Look for the countries, states or cities with the highest transaction volumes.

Looks for regional patterns :

Identify any particular states or cities in each region that consistently perform better or worse.

Recommendation :

- For region with high transaction but lower profit margins, investigate operational costs, shipping fees and pricing strategies.

- For underperforming  regions, consider targeted marketing campaigns or sales promotions to increase profitability.
			
3.Problem : Assess whether current sales performance is on track to meet targets for the month.

Solution : 	

Gauge Chart Analysis :

Review current revenue against the target for the current month. If the needle is below the target , check the trend in improving or declining.

Current Month Transaction :

Use the KPI for current month transaction verify the number of transactions is meet the expected levels or not.

Weekly Revenue Trend :

Use the column chart to see weekly revenue trends and identify there are any dips or inconsistencies.

Recommendation :

- If the revenue is falling, evaluate whether the drop is seasonal or caused by specific issues on one region or product brands.

- Increasing marketing efforts or adjusting pricing to boost sales in short term.

## Final Recommendation :

	1. Quality Control : Address product quality issues that may be leading to higher returns rate.	

	2. Customer Feedback : Customer Feedback very important to identify specific issues causing returns and dissatisfaction.

	3. Promotions : Run target promotion or campaigns for regions or products brands that are underperforming in terms of sales.

	4. Pricing Strategy : Reviewing the pricing strategy for underperforming products. Offer discounts to drive higher transaction volumes.

## Conclusion :
By analyzing the Power BI report with the above problem statements, able to identify underperforming products brands, compare the regional trends and assess the company's performance against revenue targets.			This will allow to provide actionable insights to management on how to improve product brands performance, increase transaction volumes and meet revenue targets.
			
This problem and solution approach covers various analytical techniques and Power BI functionalities like filtering, sorting, add calculated columns and measures using DAX, visual interpretation to know real world Power Bi analytics.		  
