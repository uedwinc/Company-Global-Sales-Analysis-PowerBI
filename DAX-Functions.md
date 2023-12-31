## DAX Functions

### Calendar Table

```
Calendar Table = 
ADDCOLUMNS(
    CALENDARAUTO(), 
        "Year", YEAR([Date]),
        "Month", MONTH([Date]),
        "Month Name", FORMAT([Date], "MMMM"),
        "Month Year", FORMAT([Date], "MM/YYYY"))
```

### Measures

**% Currency Conversion**
```
% Currency Conversion = 
VAR salesAllCountries =
    CALCULATE ( [Currency Conversion], ALL ( dimSalesTerritory ) )
RETURN
    DIVIDE ( [Currency Conversion], salesAllCountries )
```

**% Indicator YoY Sales Growth**
```
% Indicator YoY Sales Growth = 
SWITCH (
    TRUE (),
    [% Sales YoY Growth] < 0.1, UNICHAR ( 9660 ),
    [% Sales YoY Growth] > 0.1, UNICHAR ( 9650 ),
    BLANK ()
)
    & FORMAT ( [% Sales YoY Growth], "Percent" )
```

**% Sales Amount Selected Age Buckets**
```
% Sales Amount Selected Age Buckets = 
DIVIDE (
    [Iterator SUMX],
    CALCULATE ( [Total Sales Amount], ALL ( dimCustomer[Age Buckets] ) )
)
```

**% Sales YoY Growth**
```
% Sales YoY Growth = DIVIDE (
    [Sales Amount Current Year],
    [Total Sales Amount All Except Latest Year]
)

--We can also calculate this by just entering the formulas of each measure directly here instead of creating 3 different measure. But this will look very clumsy and confusing.

--Another thing we can do is use variables. Here's how:
/*
var salesAmtCurrYear = [Total Sales Amount] - CALCULATE([Total Sales Amount], PARALLELPERIOD('Calendar Table'[Date], -12, MONTH))
var salesAmtAllExceptCurrYear = CALCULATE([Total Sales Amount], PARALLELPERIOD('Calendar Table'[Date], -12, MONTH))
RETURN
DIVIDE(salesAmtCurrYear, salesAmtAllExceptCurrYear)
*/
--var and DIVIDE are functions we called.
```

**Accumulative Sales**
```
Accumulative Sales = 
CALCULATE ( [Currency Conversion], DATESYTD ( 'Calendar Table'[Date] ) )
```

**Accumulative Sales LY**
```
Accumulative Sales LY = 
CALCULATE ( [Sales Amount LY], DATESYTD ( 'Calendar Table'[Date] ) )
```

**Color**
```
Color = 
SWITCH(
    TRUE(),
        [% Sales YoY Growth] > 0.01, "Green", "Red")
```

**Concat Filter Selection**
```
Concat Filter Selection = 
IF (
    ISFILTERED ( dimCustomer[CustomerName] ),
    CONCATENATEX (
        VALUES ( dimCustomer[CustomerName] ),
        dimCustomer[CustomerName],
        ", ",
        dimCustomer[CustomerName], ASC
    )
)
```

**Currency Conversion**
```
Currency Conversion = 
/*getting the total sales amount in bases currency EURO*/
VAR salesEUR =
    SUM ( factInternetSales[Sales Amount EUR] )
RETURN
    IF (
        ISFILTERED ( dimCurrency[CurrencyAlternateKey] ),
        salesEUR
            * CALCULATE (
                MAX ( dimCurrency[Exchange Rate] ),
                TREATAS (
                    VALUES ( dimCurrency[CurrencyAlternateKey] ),
                    factInternetSales[Currency Code]
                )
            ),
        salesEUR
    )
```

**Currency Table Filtered**
```
Currency Table Filtered = 
IF (
    ISFILTERED ( dimCurrency ),
    SELECTEDVALUE ( dimCurrency[CurrencyName] ),
    "No Currency Selected"
)
```

**Current Selection**
```
Current Selection = 
VAR selectedCustomer =
    IF (
        ISFILTERED ( dimCustomer[CustomerName] ),
        CONCATENATEX (
            VALUES ( dimCustomer[CustomerName] ),
            dimCustomer[CustomerName],
            ", ",
            dimCustomer[CustomerName], ASC
        )
    )
VAR selectedCountry =
    SELECTEDVALUE ( dimSalesTerritory[SalesTerritoryCountry] )
VAR selectedProduct =
    SELECTEDVALUE ( dimProduct[EnglishProductName] )
RETURN
    IF (
        ISFILTERED ( dimCustomer[CustomerName] ),
        CONCATENATEX (
            VALUES ( dimCustomer[CustomerName] ),
            dimCustomer[CustomerName],
            ", ",
            dimCustomer[CustomerName], ASC
        )
    )
        & IF ( ISBLANK ( selectedCustomer ), "", " / " ) & selectedCountry
        & IF ( ISBLANK ( selectedCountry ), "", " / " ) & selectedProduct
        & IF ( ISBLANK ( selectedProduct ), "", " / " )
```

**Iterator SUMX**
```
Iterator SUMX = SUMX (
    FILTER (
        factInternetSales,
        RELATED ( dimCustomer[Age Buckets] ) IN VALUES ( dimCustomer[Age Buckets] )
    ),
    factInternetSales[Sales Amount EUR]
)
```

**Late Orders Value**
```
Late Orders Value = 
CALCULATE (
    [Total Sales Amount],
    KEEPFILTERS ( factInternetSales[Late Shipment] = "Late Shipment" )
)
```

**LY vs TY Difference**
```
LY vs TY Difference = 
VAR ty =
    CALCULATE (
        [Currency Conversion],
        'Calendar Table'[Year] = SELECTEDVALUE ( 'Calendar Table'[Year] )
    )
RETURN
    ty - [Selected Year Minus One]
```

**Number of Late Orders**
```
Number of Late Orders = 
CALCULATE (
    [Number of Sales],
    KEEPFILTERS ( factInternetSales[Late Shipment] = "Late Shipment" )
)
```

**Number of Sales**
```
Number of Sales = COUNTROWS(factInternetSales)
--You can also use COUUNT function for this. Only you will have to specify the column you want to count.
/*And this is how you leave comments on DAX*/
```

**Orders greater than 100**
```
Orders greater than 100 = 
CALCULATE (
    [Number of Sales],
    FILTER ( factInternetSales, '#Measures'[Total Sales Amount] > 100 )
)
```

**Sales Amount Current Year**
```
Sales Amount Current Year = [Total Sales Amount]
    - CALCULATE (
        [Total Sales Amount],
        PARALLELPERIOD ( 'Calendar Table'[Date], -12, MONTH )
    )
```

**Sales Amount LM**
```
Sales Amount LM = 
CALCULATE (
    [Currency Conversion],
    DATEADD ( 'Calendar Table'[Date], -1, MONTH )
)
```

**Sales Amount LY**
```
Sales Amount LY = 
CALCULATE (
    [Currency Conversion],
    DATEADD ( 'Calendar Table'[Date], -1, YEAR )
)
```

**Selected Year Minus One**
```
Selected Year Minus One = 
VAR selectedYearMinus1 =
    IF (
        ISFILTERED ( 'Calendar Table'[Year] ),
        SELECTEDVALUE ( 'Calendar Table'[Year] ) - 1,
        BLANK ()
    )
RETURN
    CALCULATE ( [Currency Conversion], 'Calendar Table'[Year] = selectedYearMinus1 )
```

**Switched**
```
Switched = 
SWITCH (
    SELECTEDVALUE ( 'Data Type'[Type] ),
    "Percentage", [% Currency Conversion],
    "Number", [Currency Conversion]
)
```

**Switched IF**
```
Switched IF = 
IF (
    SELECTEDVALUE ( 'Data Type'[Type] ) = "Percentage",
    [% Currency Conversion],
    IF ( SELECTEDVALUE ( 'Data Type'[Type] ) = "Number", [Currency Conversion] )
)
```

**Target Sales**
```
Target Sales = [Number of Sales] * 0.8
```

**Title**
```
Title = 
VAR ty =
    SELECTEDVALUE ( 'Calendar Table'[Year] )
VAR ly =
    SELECTEDVALUE ( 'Calendar Table'[Year] ) - 1
VAR result = ty & " vs " & ly & " Difference"
VAR isSelected =
    IF ( ty = BLANK (), "Please select a year from the slicer", result )
RETURN
    isSelected
```

**Total Sales Amount**
```
Total Sales Amount = SUM(factInternetSales[Sales Amount EUR])
```

**Total Sales Amount All Except Latest Year**
```
Total Sales Amount All Except Latest Year = CALCULATE (
    [Total Sales Amount],
    PARALLELPERIOD ( 'Calendar Table'[Date], -12, MONTH )
)
```

**Total Sales Amount CALCULATE**
```
Total Sales Amount CALCULATE = CALCULATE ( [Total Sales Amount], dimProduct[Color] = "Blue" )
```

**Total Sales Amount FILTER**
```
Total Sales Amount FILTER = CALCULATE (
    [Total Sales Amount],
    FILTER ( dimProduct, dimProduct[Color] = "Blue" )
)
```

**Total Sales Amount KEEPFILTERS**
```
Total Sales Amount KEEPFILTERS = CALCULATE ( [Total Sales Amount], KEEPFILTERS ( dimProduct[Color] = "Blue" ) )
```