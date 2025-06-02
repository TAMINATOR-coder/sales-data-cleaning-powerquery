# Sales Data Cleaning & Analysis 


## Table of Contents

- [Sales Data Cleaning & Analysis](#sales-data-cleaning--analysis)
- [Project Overview](#project-overview)
- [Objective](#objective)
- [Data Source](#data-source)
- [Tools Used](#tools-used)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Key Steps](#key-steps)
  - [1. Data Import & Type Transformation](#1-data-import--type-transformation)
  - [2. Standardizing Location Names with Replace Values and Currency Conversion Using Exchange Rates](#2-standardizing-location-names-with-replace-values-and-currency-conversion-using-exchange-rates)
    - [i. Replace Inconsistent Location Names](#i-corrected-inconsistent-store-location-names-using-power-querys-replace-value-transformations)
    - [ii. Convert Foreign Currency Transactions](#ii-converted-all-foreign-currency-transactions-eur-usd-gbp-to-naira)
  - [3. Filling Missing Store IDs with XLOOKUP](#3-filling-missing-store-ids-with-xlookup)
  - [4. Ranking Stores by Sales](#4-ranking-stores-by-sales)
- [Results & Findings](#results--findings)
  - [Overall Sales Performance](#overall-sales-performance)
  - [Store Ranking by Sales Volume](#store-ranking-by-sales-volume)
  - [Growth Trend](#growth-trend)
  - [Underperforming Location](#underperforming-location)
- [Recommendations](#recommendations)


## Project Overview

This project showcases how I used **Excel** and **Power Query (M Language)** to clean, transform, and analyze a messy sales dataset. The goal was to prepare the data for accurate reporting, unify inconsistent location names, and rank store performance.

---

## Objective

To clean and standardize raw sales data, enrich it using lookup logic, and create a pivot table that identifies top-performing store locations using a ranking system.

## Data Source

The data used in this analysis is the "Sales_data_Cleaned" file containing detailed information about the companies sales transactions

---

##  Tools Used

- Microsoft Excel
  -  Power Query (M)
  -  Pivot Tables 
  -  Excel Formulas (`XLOOKUP`, `RANK`)


---

## Exploratory Data Analysis
  - Which store location recorded the highest total sales, and how does it compare to others?
  - Is there a correlation between the number of transactions and total sales per store?
  - Are there seasonal or monthly trends in sales performance across different locations?

##  Key Steps


### 1. Data Import & Type Transformation
Loaded raw data from Excel and changed column types to appropriate formats (dates, integers, text, etc.) using Power Query:

```
let
    Source = Excel.Workbook(File.Contents("C:\Users\Tamen\Documents\Sales Data.xlsx"), null, true),
    Sales_Sheet = Source{[Item="Sales_With_NGN_Rate_2",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(Sales_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"RowID", Int64.Type}, {"TransactionID", type text}, {"Transaction Date", type date},
        {"Customer Name", type text}, {"Product", type text}, {"Quantity", Int64.Type},
        {"UnitPrice", Int64.Type}, {"Currency", type text}, {"TotalAmount", Int64.Type},
        {"SalesRep", type text}, {"Store Location", type text}, {"Standardised Sales", Int64.Type},
        {"StoreID", type text}
    })
in
    #"Changed Type"
```


### 2. **Standardizing Location Names with Replace Values and Currency Conversion Using Exchange Rates:**


#### i. Corrected inconsistent store location names using Power Query’s `Replace Value` transformations to ensure consistency for accurate analysis.

```
let
    Step1 = Table.ReplaceValue(#"Changed Type", "Lagoss", "Lagos", Replacer.ReplaceText, {"Store Location"}),
    Step2 = Table.ReplaceValue(Step1, "Lago", "Lagos", Replacer.ReplaceText, {"Store Location"}),
    Step3 = Table.ReplaceValue(Step2, "Portharcourt", "Port Harcourt", Replacer.ReplaceText, {"Store Location"}),
    Step4 = Table.ReplaceValue(Step3, "Abj", "Abuja", Replacer.ReplaceText, {"Store Location"})
in
    Step4
```

#### ii. Converted all foreign currency transactions (EUR, USD, GBP) to Naira 
```
 // Load exchange rate table
    ExchangeRateSource = Excel.CurrentWorkbook(){[Name="Exchange_rate"]}[Content],
    #"Changed Exchange Types" = Table.TransformColumnTypes(ExchangeRateSource,{{"Currency", type text}, {"RateToNGN", Int64.Type}}),

    // Merge sales table with exchange rates
    #"Merged Queries" = Table.NestedJoin(#"Replaced Abj", {"Currency"}, #"Changed Exchange Types", {"Currency"}, "ExchangeRate", JoinKind.LeftOuter),
    #"Expanded Exchange Rate" = Table.ExpandTableColumn(#"Merged Queries", "ExchangeRate", {"RateToNGN"}),

    // Calculate standardised sales in NGN
    #"Added Standardised Sales" = Table.AddColumn(#"Expanded Exchange Rate", "Standardised Sales", each [TotalAmount] * [RateToNGN], Int64.Type)
in
    #"Added Standardised Sales"
    
```

 ### 3. **Filling Missing Store IDs with XLOOKUP**

Used Excel’s `XLOOKUP` function to populate missing Store IDs by matching store locations to a lookup table.

```
=XLOOKUP([@Location], Map_Lookup_Table[Location], Map_Lookup_Table[StoreID], "Not Found")
```


### 4. **Ranking Stores by Sales**

Created a PivotTable to summarize sales by store location and used the `RANK` formula to rank the stores from highest to lowest sales.

```
=RANK(E11, $E$11:$E$15, 0)
```

## Results & Findings

1. Overall Sales Performance

  - The total standardised sales across all store locations from January to April amounted to ₦185,347,860.

  - April recorded the highest sales (₦55,448,591), while February had the lowest (₦32,849,454).

  - Store Ranking by Sales Volume

 | Rank | Store Location   | Standardised Sales (₦) |
|------|------------------|-------------------------|
| 1    | Abuja            | 48,268,508              |
| 2    | Port Harcourt    | 40,537,341              |
| 3    | Kaduna           | 36,546,836              |
| 4    | Lagos            | 35,845,011              |
| 5    | Enugu            | 24,150,164              |


2. Growth Trend

  - Sales followed an upward trajectory overall:

    - From February to March, sales increased by approximately 67%.

    - From March to April, sales rose by around 20%.

3. Underperforming Location

  - Enugu had the lowest cumulative sales, contributing just about 13% to the grand total.


## Recommendations

1. Replicate Abuja’s Performance

  - Analyze the success factors behind Abuja’s strong performance (e.g., demand, marketing strategy, team productivity).

  - Apply those insights to improve outcomes in lower-performing locations.

2. Investigate Enugu’s Low Sales

  - Conduct a root-cause analysis into Enugu's performance.

  - Consider sales campaigns to improve customer engagement, and surveys to enquire issues faced by both staff and customers.

3. Review Lagos Operations

  - Despite being a commercial hub, Lagos ranked 4th.

  - Assess for inefficiencies, market saturation, customer dissatisfaction or misaligned pricing strategies.



