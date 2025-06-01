# Sales Data Cleaning & Analysis (Excel + Power Query)

This project showcases how I used **Excel** and **Power Query (M Language)** to clean, transform, and analyze a messy sales dataset. The goal was to prepare the data for accurate reporting, unify inconsistent location names, and rank store performance.

---

# Objective

To clean and standardize raw sales data, enrich it using lookup logic, and create a pivot table that identifies top-performing store locations using a ranking system.

---

#  Tools Used

- Microsoft Excel
- Power Query (M)
- Pivot Tables
- Excel Formulas (`XLOOKUP`, `RANK`)
- Data Cleaning Techniques

---

#  Key Steps


 1. **Data Import & Type Transformation**
Loaded raw data from Excel and changed column types to appropriate formats (dates, integers, text, etc.) using Power Query:

```m
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

 2. **Standardizing Location Names with Replace Values**

Corrected inconsistent store location names using Power Query’s `Replace Value` transformations to ensure consistency for accurate analysis.

```m
let
    Step1 = Table.ReplaceValue(#"Changed Type", "Lagoss", "Lagos", Replacer.ReplaceText, {"Store Location"}),
    Step2 = Table.ReplaceValue(Step1, "Lago", "Lagos", Replacer.ReplaceText, {"Store Location"}),
    Step3 = Table.ReplaceValue(Step2, "Portharcourt", "Port Harcourt", Replacer.ReplaceText, {"Store Location"}),
    Step4 = Table.ReplaceValue(Step3, "Abj", "Abuja", Replacer.ReplaceText, {"Store Location"})
in
    Step4


 3. **Filling Missing Store IDs with XLOOKUP**

Used Excel’s `XLOOKUP` function to populate missing Store IDs by matching store locations to a lookup table.

```excel
=XLOOKUP([@Location], Map_Lookup_Table[Location], Map_Lookup_Table[StoreID], "Not Found")


 4. 🏆 **Ranking Stores by Sales**

Created a PivotTable to summarize sales by store location and used the `RANK` formula to rank the stores from highest to lowest sales.

```excel
=RANK(E11, $E$11:$E$15, 0)

