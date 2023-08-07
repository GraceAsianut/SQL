# 📚 AamzingWorks With MySQL Queries 


## Business Task
These queries were carried out as part of my practice of MySQL using Azure Data Studio on the Adventure Works database during my Business Intelligence and Data Analysis Course with Corporate Finance Institute. We are going to explore different types of operations on SQL. 

***

## 📚 Table of Contents
- 1. TOP N
- 2. OFFSET FETCH
- 3. AGGREGATE FUNCTIONS
- 4. NUMERIC FUNCTIONS
- 5. DATE FUNCTIONS
- 6. STRING FUNCTIONS
- 7. LOGICAL OPERATORS
- 8. CONDITIONAL COLUMNS: IF CASE AND NULL
- 9. LEFT JOIN
- 10. UNIONS
- 11. CREATING A VIEW

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***


**1. This is a basic SQL query using TOP N filtre to select data**

````sql
 SELECT TOP(10) PERCENT
SalesOrderNumber AS InvoiceNumber,
OrderDate,
SUM(SalesAmount) AS InvoiceSubTotal,
SUM(TaxAmt) AS TaxAmount,
SUM(OrderQuantity) AS TotalQuantity,
SUM(SalesAmount) + SUM(TaxAmt) AS InvoiceTotal

FROM FactInternetSales
WHERE SalesTerritoryKey = 6

GROUP BY SalesOrderNumber, OrderDate
HAVING SUM(SalesAmount) > 1000

ORDER BY InvoiceSubTotal DESC
````

#### Steps:
- Use **SELECT** to query from `FactInternetSales` to get `SalesOrderNumber` as `InvoiceNumber`, `OrderDate`, `SUM(SalesAmount)` as `InvoiceSubTotal`, `SUM(TaxAmt)` AS `TaxAmount`, `SUM(OrderQuantity)` AS `TotalQuantity`, and `SUM(SalesAmount)` + `SUM(TaxAmt)` AS `InvoiceTotal` .
- Use **SUM** to calculate the new metrics in the columns.
- Group the aggregated results by `SalesOrderNumber` and `OrderDate`. 
- Order by `InvoiceSubTotal` in descending order.


***

**2. Query using OFFSET FETCH**

````sql
SELECT
SalesOrderNumber AS InvoiceNumber,
OrderDate,
SUM(SalesAmount) AS InvoiceSubTotal,
SUM(TaxAmt) AS TaxAmount,
SUM(OrderQuantity) AS TotalQuantity,
SUM(SalesAmount) + SUM(TaxAmt) AS InvoiceTotal

FROM FactInternetSales
WHERE SalesTerritoryKey = 6

GROUP BY SalesOrderNumber, OrderDate
HAVING SUM(SalesAmount) > 1000

ORDER BY InvoiceSubTotal DESC

OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY
````


***

**3. Manipulating Values - a demonstration of aggregating functions, including COUNT to count rows**

````sql
SELECT

COUNT(*) AS TotalCustomers,
AVG(YearlyIncome) AS AverageIncome,

FROM DimCustomer
````

***

**4. A Query demonstrating Numeric functions**

````sql
SELECT TOP(10) PERCENT
SalesOrderNumber AS InvoiceNumber,
OrderDate,

SUM(SalesAmount) AS InvoiceSubTotal,
ROUND(SUM(SalesAmount),1) AS InvoiceSubTotalRounded,

SUM(TaxAmt) AS TaxAmount,
FLOOR(SUM(TaxAmt)) AS TaxAmountFloor,

SUM(OrderQuantity) AS TotalQuantity,
SUM(SalesAmount) + SUM(TaxAmt) AS InvoiceTotal

FROM FactInternetSales
WHERE SalesTerritoryKey = 6

GROUP BY SalesOrderNumber, OrderDate
HAVING SUM(SalesAmount) > 1000

ORDER BY InvoiceSubTotal DESC
````

***

**5. A query manipulating values using date functions**

````sql
SELECT

GETDATE() AS DateTimeStamp,
DueDate,
ShipDate,
DATEDIFF(day,ShipDate,DueDate) AS DaysBetweenShippedAndDueDate,
DATEDIFF(hour,ShipDate,DueDate) AS HoursBetweenShippedAndDueDate,
DATEADD(day,-10,DueDate ) AS DueDatePlusTenDays

FROM FactInternetSales
````

***

**6. A query using string functions **

````sql
SELECT
EnglishProductName as ProductName,
EnglishDescription AS ProductDescription,
CONCAT(EnglishProductName,'-',EnglishDescription) AS ProductNameAndDescription,
LEN(EnglishDescription) AS DescriptionLength,
UPPER(EnglishProductName) AS UpperProductName,
LOWER(EnglishProductName) AS LowerProductName,
REPLACE(EnglishProductName,'Front', 'Ultra Durable Front') AS EnglishProductNameReplaced,
LEFT(ProductAlternateKey,2) AS ProductShort,
RIGHT(ProductAlternateKey,LEN(ProductAlternateKey)-3) AS ProductAlternateKey2

FROM DimProduct
WHERE ProductKey = 555
````

***

**7. A query for fundamental logical operators**

```sql
WITH joined_as_member AS (
 SELECT

EnglishProductName,
EnglishDescription,
Color,
[Status],
Class

FROM DimProduct

WHERE (Class <> 'H' OR Class IS NULL) AND [Status] IS NOT NULL



--Advanced Logical Operators - Using IN and BETWEEN to simplyfy multiple conditions
SELECT

EnglishProductName,
EnglishDescription,
Color,
[Status],
Class,
SafetyStockLevel

FROM DimProduct

--Using BETWEEN to replace two numeric or date range conditions
WHERE (SafetyStockLevel BETWEEN 500 AND 1000) AND [Status] IS NOT NULL --BETWEEN IS INCLUSIVE OF BOTH ENDS
--WHERE (SafetyStockLevel >= 500 AND SafetyStockLevel <=1000) AND [Status] IS NOT NULL

--Using IN to replace multiple OR statements
--WHERE Color IN ('Black','Silver','White','Yellow')
--WHERE Color = 'Black' OR Color = 'Silver' OR Color = 'White' OR Color = 'Yellow'


--Advanced Logical Operators - LIKE to identify partial matches using Wildcards
SELECT

FirstName,
EmailAddress

FROM DimCustomer

WHERE FirstName LIKE '_R%'
````

***
    
**8. Query for conditional columns IF CASE and replacing NULLS **

````sql
SELECT
    FirstName,
    IIF(MiddleName IS NULL,'UNKN',MiddleName) AS MiddleName,
    ISNULL(MiddleName,'UNKN') AS MiddleName2,
    COALESCE(MiddleName,'UNKN') AS MiddleName3,
    LastName,
    YearlyIncome,
    EmailAddress,
    IIF(YearlyIncome >50000,'Above Average','Below Average') AS IncomeCategory,
    CASE
        WHEN NumberChildrenAtHome = 0 THEN '0'
        WHEN NumberChildrenAtHome = 1 THEN '1'
        WHEN NumberChildrenAtHome BETWEEN 2 AND 4 THEN '2-4'
        WHEN NumberChildrenAtHome >=5 THEN '5+'
        ELSE 'UNKN'
    END AS NumberChildrenCategory,
    NumberChildrenAtHome AS ActualChildren

FROM DimCustomer

WHERE IIF(YearlyIncome >50000,'Above Average','Below Average') = 'Above Average'

````

***
    
**9. Manipulating values using left join **

````sql
SELECT

    dp.EnglishProductName AS ProductName,
    dp.Color AS ProductColor,
    ISNULL(dp.Size,'UNKN') AS ProductSize,
    ISNULL(SUM(fs.SalesAmount),0) AS SalesAmount

/*
FROM FactInternetSales AS fs
    RIGHT JOIN DimProduct AS dp
    ON fs.ProductKey = dp.ProductKey
*/

FROM DimProduct AS dp
    LEFT JOIN FactInternetSales AS fs
    ON dp.ProductKey = fs.ProductKey

WHERE dp.Status = N'Current'

GROUP BY dp.EnglishProductName, dp.Color, dp.Size

ORDER BY SalesAmount DESC


/* Tests for number of total current products
SELECT
    EnglishProductName

FROM DimProduct

WHERE [Status] = N'Current'
*/
````

***
    
**10. A query using UNIONS **

````sql
SELECT 
    --Sales and promo summary from Internet Sales
    fs.SalesOrderNumber AS InvoiceNumber,
    fs.SalesOrderLineNumber AS InvoiceLineNumber,
    fs.OrderDate AS OrderDate,
    fs.SalesAmount AS SalesAmount,
    fs.TaxAmt AS TaxAmount,
    fs.Freight AS FreightAmount,
    fs.OrderQuantity AS Quantity,
 
    dp.EnglishProductName AS ProductName,
 
    dpsc.EnglishProductSubcategoryName AS ProductSubCategory,
 
    dpc.EnglishProductCategoryName  AS ProductCategory,
 
    dst.SalesTerritoryCountry AS Country,
    dst.SalesTerritoryRegion AS Region,
 
    dpr.EnglishPromotionName AS PromotionName,
    dpr.EnglishPromotionType AS PromotionType,
    dpr.EnglishPromotionCategory AS PromotionCategory,
 
    dcy.CurrencyName AS Currency,

    'Web' AS Source
 
FROM FactInternetSales AS fs
    INNER JOIN DimProduct AS dp
    ON fs.ProductKey=dp.ProductKey
    INNER JOIN DimProductSubcategory AS dpsc
    ON dp.ProductSubcategoryKey=dpsc.ProductSubcategoryKey
    INNER JOIN DimProductCategory AS dpc
    ON dpsc.ProductCategoryKey=dpc.ProductCategoryKey
    INNER JOIN DimSalesTerritory AS dst
    ON fs.SalesTerritoryKey=dst.SalesTerritoryKey
    INNER JOIN DimPromotion AS dpr
    ON fs.PromotionKey=dpr.PromotionKey
    INNER JOIN DimCurrency AS dcy
    On fs.CurrencyKey=dcy.CurrencyKey
 
WHERE YEAR(fs.OrderDate) IN (2012,2013)



UNION


SELECT 
    --Sales and promo summary from Reseller sales
    fs.SalesOrderNumber AS InvoiceNumber,
    fs.SalesOrderLineNumber AS InvoiceLineNumber,
    fs.OrderDate AS OrderDate,
    fs.SalesAmount AS SalesAmount,
    fs.TaxAmt AS TaxAmount,
    fs.Freight AS FreightAmount,
    fs.OrderQuantity AS Quantity,
 
    dp.EnglishProductName AS ProductName,
 
    dpsc.EnglishProductSubcategoryName AS ProductSubCategory,
 
    dpc.EnglishProductCategoryName  AS ProductCategory,
 
    dst.SalesTerritoryCountry AS Country,
    dst.SalesTerritoryRegion AS Region,
 
    dpr.EnglishPromotionName AS PromotionName,
    dpr.EnglishPromotionType AS PromotionType,
    dpr.EnglishPromotionCategory AS PromotionCategory,
 
    dcy.CurrencyName AS CurrencyKey,

    dr.ResellerName AS Source
 
FROM FactResellerSales AS fs
    INNER JOIN DimProduct AS dp
    ON fs.ProductKey=dp.ProductKey
    INNER JOIN DimProductSubcategory AS dpsc
    ON dp.ProductSubcategoryKey=dpsc.ProductSubcategoryKey
    INNER JOIN DimProductCategory AS dpc
    ON dpsc.ProductCategoryKey=dpc.ProductCategoryKey
    INNER JOIN DimSalesTerritory AS dst
    ON fs.SalesTerritoryKey=dst.SalesTerritoryKey
    INNER JOIN DimPromotion AS dpr
    ON fs.PromotionKey=dpr.PromotionKey
    INNER JOIN DimCurrency AS dcy
    On fs.CurrencyKey=dcy.CurrencyKey
    INNER JOIN DimReseller AS dr
    ON fs.ResellerKey = dr.ResellerKey
 
WHERE YEAR(fs.OrderDate) IN (2012,2013)

ORDER BY OrderDate DESC

````

***
    
**6. A query creating a view **

````sql
CREATE VIEW vwOrdersALL
AS

-- Combined summary of sales and promo from both Internet Sales and Reseller sales in 2012 and 2013.

SELECT 
    --Sales and promo summary from Internet Sales
    fs.SalesOrderNumber AS InvoiceNumber,
    fs.SalesOrderLineNumber AS InvoiceLineNumber,
    fs.OrderDate AS OrderDate,
    fs.SalesAmount AS SalesAmount,
    fs.TaxAmt AS TaxAmount,
    fs.Freight AS FreightAmount,
    fs.OrderQuantity AS Quantity,
 
    dp.EnglishProductName AS ProductName,
    dp.Status AS [Status],
 
    dpsc.EnglishProductSubcategoryName AS ProductSubCategory,
 
    dpc.EnglishProductCategoryName  AS ProductCategory,
 
    dst.SalesTerritoryCountry AS Country,
    dst.SalesTerritoryRegion AS Region,
 
    dpr.EnglishPromotionName AS PromotionName,
    dpr.EnglishPromotionType AS PromotionType,
    dpr.EnglishPromotionCategory AS PromotionCategory,
 
    dcy.CurrencyName AS Currency,

    'Web' AS Source
 
FROM FactInternetSales AS fs
    INNER JOIN DimProduct AS dp
    ON fs.ProductKey=dp.ProductKey
    INNER JOIN DimProductSubcategory AS dpsc
    ON dp.ProductSubcategoryKey=dpsc.ProductSubcategoryKey
    INNER JOIN DimProductCategory AS dpc
    ON dpsc.ProductCategoryKey=dpc.ProductCategoryKey
    INNER JOIN DimSalesTerritory AS dst
    ON fs.SalesTerritoryKey=dst.SalesTerritoryKey
    INNER JOIN DimPromotion AS dpr
    ON fs.PromotionKey=dpr.PromotionKey
    INNER JOIN DimCurrency AS dcy
    On fs.CurrencyKey=dcy.CurrencyKey
 
WHERE YEAR(fs.OrderDate) IN (2012,2013)


UNION


SELECT 
    --Sales and promo summary from Reseller sales
    fs.SalesOrderNumber AS InvoiceNumber,
    fs.SalesOrderLineNumber AS InvoiceLineNumber,
    fs.OrderDate AS OrderDate,
    fs.SalesAmount AS SalesAmount,
    fs.TaxAmt AS TaxAmount,
    fs.Freight AS FreightAmount,
    fs.OrderQuantity AS Quantity,
 
    dp.EnglishProductName AS ProductName,
    dp.Status AS [Status],
 
    dpsc.EnglishProductSubcategoryName AS ProductSubCategory,
 
    dpc.EnglishProductCategoryName  AS ProductCategory,
 
    dst.SalesTerritoryCountry AS Country,
    dst.SalesTerritoryRegion AS Region,
 
    dpr.EnglishPromotionName AS PromotionName,
    dpr.EnglishPromotionType AS PromotionType,
    dpr.EnglishPromotionCategory AS PromotionCategory,
 
    dcy.CurrencyName AS CurrencyKey,

    dr.ResellerName AS Source
 
FROM FactResellerSales AS fs
    INNER JOIN DimProduct AS dp
    ON fs.ProductKey=dp.ProductKey
    INNER JOIN DimProductSubcategory AS dpsc
    ON dp.ProductSubcategoryKey=dpsc.ProductSubcategoryKey
    INNER JOIN DimProductCategory AS dpc
    ON dpsc.ProductCategoryKey=dpc.ProductCategoryKey
    INNER JOIN DimSalesTerritory AS dst
    ON fs.SalesTerritoryKey=dst.SalesTerritoryKey
    INNER JOIN DimPromotion AS dpr
    ON fs.PromotionKey=dpr.PromotionKey
    INNER JOIN DimCurrency AS dcy
    On fs.CurrencyKey=dcy.CurrencyKey
    INNER JOIN DimReseller AS dr
    ON fs.ResellerKey = dr.ResellerKey
 
WHERE YEAR(fs.OrderDate) IN (2012,2013)

GO
````

***


