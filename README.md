# Power BI Module End Evaluation

**Total Marks: 15**

## Project Files

| File Name | Description |
|---|---|
| `SalesData_1000Rows_WithIssues_copy.csv` | Original dataset containing missing values and duplicate records. |
| `SalesData_Cleaned.csv` | Cleaned dataset used in Power BI. |
| `PowerBI_Module_End_Evaluation_Solution.docx` | Documentation of the project and results. |
| `PowerBI_Module_End_Evaluation.pbip` | Power BI project file. |

---

## Q1. Data Cleaning and Loading (5 Marks)

### Data Cleaning Operations

The original dataset was imported into Power Query Editor and the following operations were performed:

1. **Duplicate Removal**
   - Identified and removed 20 duplicate records.
   - Removed duplicates based on the `OrderID` column.
   - Obtained 1,000 unique orders.

2. **Handling Missing Numerical Values**
   - Missing values in `UnitPrice`, `Sales`, `Cost` and `Profit` were calculated using mathematical formulas.

   Formulas used:
   - `Sales = Quantity × UnitPrice`
   - `Profit = Sales - Cost`
   - `UnitPrice = Sales / Quantity`
   - `Cost = Sales - Profit`

3. **Date Format Correction**
   - Converted `OrderDate` into Date format.
   - Used English (United Kingdom) locale to maintain the DD-MM-YYYY format.

4. **Handling Missing Text Values**
   - Replaced missing values in `CustomerName`, `Region` and `Product` with `Unknown` or `Unknown Product`.

### Power Query M-Code

```m
let
    Source = Csv.Document(File.Contents("SalesData_1000Rows_WithIssues_copy.csv"), [Delimiter=",", Columns=11, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    PromoteHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),

    RemoveDuplicates = Table.Distinct(PromoteHeaders, {"OrderID"}),

    ParseDates = Table.TransformColumnTypes(RemoveDuplicates, {{"OrderDate", type date}}, "en-GB"),

    SetInitialTypes = Table.TransformColumnTypes(ParseDates, {
        {"Quantity", Int64.Type},
        {"UnitPrice", type number},
        {"Sales", type number},
        {"Cost", type number},
        {"Profit", type number}
    }),

    ImputeUnitPrice = Table.ReplaceValue(SetInitialTypes, null, each if [UnitPrice] = null and [Sales] <> null and [Quantity] <> null then [Sales] / [Quantity] else [UnitPrice], Replacer.ReplaceValue, {"UnitPrice"}),

    ImputeSales = Table.ReplaceValue(ImputeUnitPrice, null, each if [Sales] = null and [Quantity] <> null and [UnitPrice] <> null then [Quantity] * [UnitPrice] else [Sales], Replacer.ReplaceValue, {"Sales"}),

    ImputeCost = Table.ReplaceValue(ImputeSales, null, each if [Cost] = null and [Sales] <> null and [Profit] <> null then [Sales] - [Profit] else [Cost], Replacer.ReplaceValue, {"Cost"}),

    ImputeProfit = Table.ReplaceValue(ImputeCost, null, each if [Profit] = null and [Sales] <> null and [Cost] <> null then [Sales] - [Cost] else [Profit], Replacer.ReplaceValue, {"Profit"}),

    FillCust = Table.ReplaceValue(ImputeProfit, null, "Unknown", Replacer.ReplaceValue, {"CustomerName"}),

    FillRegion = Table.ReplaceValue(FillCust, null, "Unknown", Replacer.ReplaceValue, {"Region"}),

    FillProduct = Table.ReplaceValue(FillRegion, null, "Unknown Product", Replacer.ReplaceValue, {"Product"})
in
    FillProduct
```

---

## Q2. Visualizations and Insights (5 Marks)

### 1. Regional Order Distribution

**Chart Type:** Pie Chart

**Fields:**
- Legend: Region
- Values: Count of OrderID

| Region | Orders | Percentage |
|---|---:|---:|
| East | 263 | 26.3% |
| South | 255 | 25.5% |
| West | 230 | 23.0% |
| North | 228 | 22.8% |
| Unknown | 24 | 2.4% |

**Observation:** East has the highest number of orders, accounting for 26.3% of the total.

### 2. Top 5 Trending Products

**Chart Type:** Clustered Column Chart

**Fields:**
- X-axis: Product
- Y-axis: Count of OrderID
- Filter: Top 5 products

| Product | Orders |
|---|---:|
| Monitor | 132 |
| Laptop | 131 |
| Bookshelf | 129 |
| Cabinet | 125 |
| Headphones | 122 |

**Observation:** Monitor is the most frequently ordered product, followed by Laptop.

### 3. Profit Trend Over Time

**Chart Type:** Line Chart

**Fields:**
- X-axis: OrderDate (Year)
- Y-axis: Sum of Profit

**Observations:**
- Highest annual profit was recorded in 2015: $136,253.
- Profit in 1980 was $134,228.
- Profit in 2021 was $117,749.
- Profit in 2024 was $113,481.

**Observation:** The highest annual profit was recorded in 2015.

---

## Q3. DAX Calculations (5 Marks)

### 1. Calculated Table

**Table Name:** `EastRegionOrders`

```dax
EastRegionOrders =
FILTER(
    SalesData,
    SalesData[Region] = "East"
)
```

**Result:** Created a table containing 263 orders from the East region.

### 2. Calculated Column

**Column Name:** `ProfitCostDifference`

```dax
ProfitCostDifference =
SalesData[Profit] - SalesData[Cost]
```

**Result:** Calculates the difference between Profit and Cost for each order.

### 3. Calculated Measure

**Measure Name:** `TotalProfit`

```dax
TotalProfit =
SUM(SalesData[Profit])
```

**Result:** Total Profit = **$5,142,654.00**
