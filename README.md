# Project: Finance Dashboard (Management)

## Table of Content

## 📑 Table of Contents

1. [Introduction](#introduction)  
2. [Project Objectives](#project-objectives)  
3. [Research Questions](#research-questions)  
4. [About the Dataset](#about-the-dataset)  
5. [Languages, Utilities, and Environments Used](#languages-utilities-and-environments-used)  
6. [Importing the Datasets into Power BI](#importing-the-datasets-into-power-bi)  
7. [Data Automation: Cleaning and Transformation](#data-automation-cleaning-and-transformation)  
   - [Unpivot Columns](#unpivot-columns)  
   - [Rename Columns](#rename-columns)  
   - [Add Custom Column](#add-custom-column)  
   - [Create New Tables](#create-new-tables)  
   - [Create Calendar Table](#create-calendar-table)  
8. [Data Modelling](#data-modelling)  
9. [Data Analysis using Power BI DAX and Visualizations](#data-analysis-using-power-bi-dax-and-visualizations)  
   - [DAX Measures](#dax-measures)  
   - [Data Visualizations](#data-visualizations)  
10. [Insights from the Data Analysis](#insights-from-the-data-analysis)  
11. [Recommendations from the Data Analysis](#recommendations-from-the-data-analysis)  
12. [Conclusion](#conclusion)  
13. [Glossary of Terms](#glossary-of-terms)  



## Introduction
Home Store (hypthetical), is a growing grocery delivery service across Nigeria, and has witnessed year-on-year fluctuations in profitability across its regional outlets. Leadership has rolled out varying strategies across procurement and operations between 2019 and 2020 to respond to regional performance disparities and rising operational costs.

This dashboard was created using Power BI to track the month-on-month and year-over-year performance of key Profit & Loss (P&L) indicators. With granular tracking at both regional and monthly levels, the company aims to uncover insights to guide pricing adjustments, cost reduction plans, and growth opportunities. The dashboard is best suited for senior management executives, saving the need to navigate and drill through sophisticated dashboards.

## Project Objectives
1. To evaluate the impact of operational efficiency and cost structures across regions.
2. To determine which months and regions contribute most to profitability.
3. To assess how shifts in COGS, finance costs, and tax expenses are affecting Net Income.
4. To inform investment strategies for the coming fiscal year


## Research Questions
The project aims to answer the following research questions:
1. How did cost of goods sold (COGS) trends affect gross margins across months?
2. Are expense increases in 2020 justified by proportionate increases in EBIT?
3. What is the relationship between tax reductions and improvements in net income?
4. Which division(s) should receive more investment based on performance trends?

## About the Dataset
The datasets was downloaded from youtube and tweaked using Microsoft Excel to meet the purpose of this research. It contains of 2 tables - namely Journal and Chart of Account (COA).
[(Link to the dataset)]([https://docs.google.com/spreadsheets/d/1i778j9FUP08VgS25v64mK8oGUDGDGCxYe9vUziXigrg/edit?gid=0#gid=0](https://docs.google.com/spreadsheets/d/13ePx7OOsPBk7ocTlroBT7clDB-kSIqSr/edit?gid=676727392#gid=676727392))

The dataset 3223 rows rows 12 columns with transactions and accounts information. The columns are described as follows:
### Journal
* *Date*: Date a transaction was carried out
* *Division*: Regions (East, North, West, and South) where the company is operational
* *Description*: Transaction Detail
* *Dr*: Debit Column
* *Cr*: Credit Column
* *Amount*: Monetary value of transaction
### Chart of Account (COA)
* *Account Code*: Unique ID for each account type
* *Account*: Account type
* *IS or BS*: Contains validation to identify entries categorized as either Income Statements (IS) or Balance Sheets (BS)
* *Category*: Type of Account e.g. Revenue, Finance Costs, etc.
* *Debit or Credit*: Agreed discount per SKU purchase
* *Debit or Credit*: Contains validations to identify a Debit (D) or Credit account (C)
* *Account Group*: Classifies various accounts into groups.

## Languages, Utilities, and Environments Used
* Microsoft Excel: Data Simulation
* Power Query: Data Automation: Cleaning, and Transformation
* Power BI: Data Analysis and Exploration [(link to the Power BI file)]()

## Importing the Datasets into Power BI
To import the dataset into Power BI, I proceeded as follows:  
* Launched the Microsoft Power BI app
* Created a blank report > Add data to your report > Import Data from Excel
* Clicked on Browse > selected the file from my computer > Open
* In the new window that appears, I clicked on the data to preview it then clicked on Transform Data 
* This launched into the Power Query editor > Next
* Validated column properties such as Data Type, Column Quality, Column Profile, and Column Distribution
* Then I proceeded to apply a couple of transformations and cleaning to the datasets as outlined in the next section.
The above steps successfully imported the dataset into my Power Query editor and ready for transformation.

## Data Automation: Cleaning and Transformation

1. **Unpivot Columns**
*  Unpivoted the Transaction types Credit (Cr), and Debit (Dr) columns to transform into rows instead of columns for useability 
2. **Rename Columns**
* Rename the newly created columns from the step above.
3. **Add Custom Column** 
* Add a custom column to negate all the debit entries and change data type
```
let
    Source = Excel.Workbook(File.Contents("C:\Users\David Micheal\Downloads\Creating an Income Statement Dashboard_Start.xlsx"), null, true),
    Journal_Sheet = Source{[Item="Journal",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(Journal_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Date", type date}, {"Division", type text}, {"Description", type text}, {"Dr", type text}, {"Cr", type text}, {"Amount", type number}}),
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Changed Type", {"Date", "Division", "Description", "Amount"}, "Attribute", "Value"),
    #"Renamed Columns" = Table.RenameColumns(#"Unpivoted Columns",{{"Attribute", "Type"}, {"Value", "Account"}}),
    #"Added Custom" = Table.AddColumn(#"Renamed Columns", "TB Amount", each if[Type] = "Dr" then [Amount]*-1 else [Amount]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",{{"TB Amount", type number}})
in
    #"Changed Type1"
```
### Create new tables
1. **Create New Tables to be used as a visual slicer using the Division**
* Create a new table by referencing the journal table, then take out all other columns except the Division column, take out duplicates.
* Close and apply changes
```
let
    Source = Journal,
    #"Removed Other Columns" = Table.SelectColumns(Source,{"Division"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Other Columns")
in
    #"Removed Duplicates"
```

2.  **Create Calendar table**
*  In the desktop view, create Calendar table, and mark as date table
```
Calendar = ADDCOLUMNS(
                    CALENDAR(MIN(Journal[Date]),max(Journal[Date])),
                    "Year", year([Date]),
                    "Month", month([Date]),
                    "Month Name", format([Date], "mmm")
                      )
```
### Data Modelling
* I created a star schema data model by connecting all tables to the Journal table in a one-to-many relationship.
See snippet below: 
![Data Modelling](https://github.com/davidutibe/finance_management_dashboard/blob/main/Data%20Modelling.JPG)

## Data Analysis using Power BI DAX and visualizations

### Dax Measures
* To analyze the data, I created 2 groups of DAX measures:
1. **Base Measures: Containing all Financial Metrics**
```
1. Revenue = CALCULATE([Report Value],COA[Category] = "Revenue")

2. Cost of Goods Sold = CALCULATE([Report Value], COA[Category] = "Cost of Goods Sold")

3. Gross Profit = [Revenue]+[Cost of Goods Sold]

4. Tax = if( [Net Income Before Tax]>0, -[Net Income Before Tax]*0.3, 0)

5. Net Income = [Net Income Before Tax]+[Tax]

6. Net Income Before Tax = [EBIT]+[Finance Costs]

7. EBIT = [Gross Profit]+[Expenses]

8. Expenses = CALCULATE([Report Value],COA[Category] = "Expenses")

9. Finance Costs = Calculate( [Report Value], COA[Category] = "Finance Costs")

10. Current = 
var currentcategory = SELECTEDVALUE(Layout[Category])
var Amount = switch(true(),
                currentcategory = "Revenue", [Revenue],
                currentcategory = "Cost of Goods Sold", [Cost of Goods Sold],
                currentcategory = "Gross Profit", [Gross Profit],
                currentcategory = "Expenses", [Expenses],
                currentcategory = "EBIT", [EBIT],
                currentcategory = "Finance Costs", [Finance Costs],
                currentcategory = "Net Income Before Tax", [Net Income Before Tax],
                currentcategory = "Tax", [Tax],
                currentcategory = "Net Income", [Net Income],
                0)
RETURN
    Amount

11. Previous = CALCULATE([Current], SAMEPERIODLASTYEAR('Calendar'[Date]))
```
2. **KPI Measures: measures to compare metrics across periods.** 
```
1. Finance Costs Current = abs(CALCULATE([Current], Layout[Category]= "Finance Costs"))

2. Expenses Previous = abs(CALCULATE([Previous], Layout[Category] = "Expenses"))

3. Expenses % Change = CALCULATE([% Change], Layout[Category] = "Expenses")

4. EBIT Previous = abs(CALCULATE([Previous], Layout[Category] = "EBIT"))

6. Cost of Goods Sold Previous = abs(CALCULATE([Previous], Layout[Category] = "Cost of Goods Sold"))

7. Revenue Previous = abs(CALCULATE([Previous], Layout[Category] = "Cost of Goods Sold"))
 ```
## Data Visualizations
To visualize the data, I used the following native power BI visuals: Matrix, Cards, Doughnut chart, Slicer, Sparkline, and conditional formatting tools. See dashboard snippet below:

![Dashboard snippet](https://github.com/davidutibe/finance_management_dashboard/blob/main/Finance%20Dashboard%20(Management).JPG?raw=true)

## Insights from the Data Analysis
1. **How did cost of goods sold (COGS) trends affect gross margins across months?**  
   * *Observation*: From the data, months with lower Costs of Goods sold, then to show higher gross margins. For months like June 2020 where the gross profit dropped as high as 105%, we observed a corresponding increase in gross margins by as high as 44%.
   * *Insights*: Cost of goods sold directly impacts gross margins, a few outlier months such as July and Oct 2020 , however, showed relatively lower gross profit despite reduced Costs of Goods sold, possibly due to lower volume or increased operating expenses.

2. **Are expense increases in 2020 justified by proportionate increases in EBIT?**  
   * *Observation*: In 2020, while expenses increased by 104% EBIT increased by 74%, This is not a proportionate increase.
   * *Insights*: While both increased, the percentage increase in expenses (104%) is significantly higher than the percentage increase in EBIT (74%), the fact that expenses increased by a much larger percentage (104%) than EBIT (74%) raises concerns. It suggests that the business became less efficient in controlling its operating costs relative to its operating profit generation.

3. **What is the relationship between tax reductions and improvements in net income?**  
   * *Observation*: For most of the months, a decrease in tax tends to an increase in the net profit for most months. For instance, in months like February, where we witnessed a significant drop in tax, we also saw a corresponding increase in net income, and in months like October where we saw significant increase in taxes, net income also dropped significantly.
   * *Insights*: 
        * there is a general inverse relationship between tax reductions and improvements in net income. Lower taxes tend to lead to higher net income (or reduced net losses), and higher taxes tend to lead to lower net income (or increased net losses). However, the actual impact on net income is also dependent on changes in other financial factors such as costs.

4. **Which division(s) should receive more investment based on performance trends?**  
   * *Observation*:  
     * West division consistently shows higher EBIT and positive net income margins, especially in 2020. February 2020 (West): EBIT = 49,955.4, Net Income = 25.45%
December 2020 (West): EBIT = 30,589.5, strong revenue-to-expense ratio. 
   * *Insights*:  
     * The East division performed well during some months, but has higher volatility. South and North divisions underperformed with high expenses and negative or inconsistent EBIT.
      The West was the most performing division showing high revenue-expense ratio across several months.

## Recommendations from the Data Analysis
1. **COGS Management and Gross Margin Stabilization**
   * Enhance gross margin stability by reducing cost of goods sold (COGS) variability and improving cost predictability across divisions, by integrating COGS variance KPIs into operational reviews to promote accountability among sourcing and production teams.
2. **EBIT-Driven Expense Optimization**    
    * Align expense growth with EBIT (Earnings Before Interest and Taxes) to ensure every dollar spent delivers measurable financial return by introducing performance-based budgeting, tying expense approvals to historical ROI metrics and EBIT contribution.  
3. **Tax Planning Integration into Profitability**  
* Divisions with high net income also tended to have more stable COGS and lower expenses, which amplified the effect of tax cuts, to optimize net-profit-to tax ratio, the company should consider integrating tax planning with operations by identifying forecasted quarters with taxable spikes, enabling timing of deductions.
4. **Division Investment Prioritization**
* The company should double down on the West division with data-backed expansion plans—use clustering to identify high-performing months and replicate conditions. 
* For the East, pilot a performance stabilization program with a focus on forecast accuracy and expense pacing.
* Conduct a deep diagnostic review of South and North divisions, using benchmarking and root cause analysis. Consider resource reallocation if improvement KPIs aren’t met in 2–3 quarters.
revenue while maintaining price integrity.

## Conclusion
This project has demonstrated how financial data, when transformed and visualized effectively using Power BI, can offer powerful insights into operational performance, cost dynamics, and profitability trends across regions. By combining rigorous data transformation, custom DAX measures, and intuitive visuals, we identified key drivers of net income—most notably COGS variability, expense inefficiency, and tax implications.

<br/>
   
**Thank you for taking the time to read through this project!**

**For inquiries, collaboration opportunities, or to engage my services, feel free to reach out via email: mdavidutibe@gmail.com.**

### Author
[David Utibe Michael](https://github.com/davidutibe)
