---
title: series_fit_lowess_fl() - Azure Data Explorer
description: This article describes the series_fit_lowess_fl() user-defined function in Azure Data Explorer.
ms.reviewer: adieldar
ms.topic: reference
ms.date: 03/13/2023
no-loc: LOWESS
---
# series_fit_lowess_fl()

The function `series_fit_lowess_fl()` is a [user-defined function (UDF)](../query/functions/user-defined-functions.md) that applies a [LOWESS regression](https://www.wikipedia.org/wiki/Local_regression) on a series. This function takes a table with multiple series (dynamic numerical arrays) and generates a *LOWESS Curve*, which is a smoothed version of the original series.

## Prerequisites

* The Python plugin must be [enabled on the cluster](../query/pythonplugin.md#enable-the-plugin). This is required for the inline Python used in the function.

## Syntax

`T | invoke series_fit_lowess_fl(`*y_series*`,` *y_fit_series*`,` [ *fit_size* ]`,` [ *x_series* ]`,` [ *x_istime* ]`)`

## Parameters

|Name|Type|Required|Description|
|--|--|--|--|
|*y_series*| string | &check; | The name of the input table column containing the [dependent variable](https://www.wikipedia.org/wiki/Dependent_and_independent_variables). This column is the series to fit.|
|*y_fit_series*| string| &check; | The name of the column to store the fitted series.|
|*fit_size*| int | | For each point, the local regression is applied on its respective *fit_size* closest points. The default is 5.|
|*x_series*| string | | The name of the column containing the [independent variable](https://www.wikipedia.org/wiki/Dependent_and_independent_variables), that is, the x or time axis. This parameter is optional, and is needed only for [unevenly spaced series](https://www.wikipedia.org/wiki/Unevenly_spaced_time_series). The default value is an empty string, as x is redundant for the regression of an evenly spaced series.|
|*x_istime*| bool | | This boolean parameter is needed only if *x_series* is specified and it's a vector of datetime. The default is `false`.|

## Function definition

You can define the function by either embedding its code as a query-defined function, or creating it as a stored function in your database, as follows:

### [Query-defined](#tab/query-defined)

Define the function using the following [let statement](../query/letstatement.md). No permissions are required.

> [!IMPORTANT]
> A [let statement](../query/letstatement.md) can't run on its own. It must be followed by a [tabular expression statement](../query/tabularexpressionstatements.md). To run a working example of `series_fit_lowess_fl()`, see [Examples](#examples).

```kusto
let series_fit_lowess_fl=(tbl:(*), y_series:string, y_fit_series:string, fit_size:int=5, x_series:string='', x_istime:bool=False)
{
    let kwargs = bag_pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_size', fit_size, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_size = kargs["fit_size"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'import statsmodels.api as sm\n'
        'def lowess_fit(ts_row, x_col, y_col, fsize):\n'
        '    y = ts_row[y_col]\n'
        '    fraction = fsize/len(y)\n'
        '    if x_col == "": # If there is no x column creates sequential range [1, len(y)]\n'
        '       x = np.arange(len(y)) + 1\n'
        '    else: # if x column exists check whether its a time column. If so, normalize it to the [1, len(y)] range, else take it as is.\n'
        '       if x_istime: \n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))\n'
        '           x = x - x.min()\n'
        '           x = x / x.max()\n'
        '           x = x * (len(x) - 1) + 1\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    lowess = sm.nonparametric.lowess\n'
        '    z = lowess(y, x, return_sorted=False, frac=fraction)\n'
        '    return list(z)\n'
        '\n'
        'result = df\n'
        'result[y_fit_series] = df.apply(lowess_fit, axis=1, args=(x_series, y_series, fit_size))\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
};
// Write your query to use the function here.
```

### [Stored](#tab/stored)

Define the stored function once using the following [`.create function`](../management/create-function.md). [Database User permissions](../management/access-control/role-based-access-control.md) are required.

> [!IMPORTANT]
> You must run this code to create the function before you can use the function as shown in the [Examples](#examples).

```kusto
.create-or-alter function with (folder = "Packages\\Series", docstring = "Fits a local polynomial using LOWESS method to a series")
series_fit_lowess_fl(tbl:(*), y_series:string, y_fit_series:string, fit_size:int=5, x_series:string='', x_istime:bool=False)
{
    let kwargs = bag_pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_size', fit_size, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_size = kargs["fit_size"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'import statsmodels.api as sm\n'
        'def lowess_fit(ts_row, x_col, y_col, fsize):\n'
        '    y = ts_row[y_col]\n'
        '    fraction = fsize/len(y)\n'
        '    if x_col == "": # If there is no x column creates sequential range [1, len(y)]\n'
        '       x = np.arange(len(y)) + 1\n'
        '    else: # if x column exists check whether its a time column. If so, normalize it to the [1, len(y)] range, else take it as is.\n'
        '       if x_istime: \n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))\n'
        '           x = x - x.min()\n'
        '           x = x / x.max()\n'
        '           x = x * (len(x) - 1) + 1\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    lowess = sm.nonparametric.lowess\n'
        '    z = lowess(y, x, return_sorted=False, frac=fraction)\n'
        '    return list(z)\n'
        '\n'
        'result = df\n'
        'result[y_fit_series] = df.apply(lowess_fit, axis=1, args=(x_series, y_series, fit_size))\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
}
```

---

## Examples

The following examples use the [invoke operator](../query/invokeoperator.md) to run the function.

### LOWESS regression on regular time series

### [Query-defined](#tab/query-defined)

To use a query-defined function, invoke it after the embedded function definition.

```kusto
let series_fit_lowess_fl=(tbl:(*), y_series:string, y_fit_series:string, fit_size:int=5, x_series:string='', x_istime:bool=False)
{
    let kwargs = bag_pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_size', fit_size, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_size = kargs["fit_size"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'import statsmodels.api as sm\n'
        'def lowess_fit(ts_row, x_col, y_col, fsize):\n'
        '    y = ts_row[y_col]\n'
        '    fraction = fsize/len(y)\n'
        '    if x_col == "": # If there is no x column creates sequential range [1, len(y)]\n'
        '       x = np.arange(len(y)) + 1\n'
        '    else: # if x column exists check whether its a time column. If so, normalize it to the [1, len(y)] range, else take it as is.\n'
        '       if x_istime: \n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))\n'
        '           x = x - x.min()\n'
        '           x = x / x.max()\n'
        '           x = x * (len(x) - 1) + 1\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    lowess = sm.nonparametric.lowess\n'
        '    z = lowess(y, x, return_sorted=False, frac=fraction)\n'
        '    return list(z)\n'
        '\n'
        'result = df\n'
        'result[y_fit_series] = df.apply(lowess_fit, axis=1, args=(x_series, y_series, fit_size))\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
};
//
// Apply 9 points LOWESS regression on regular time series
//
let max_t = datetime(2016-09-03);
demo_make_series1
| make-series num=count() on TimeStamp from max_t-1d to max_t step 5m by OsVer
| extend fnum = dynamic(null)
| invoke series_fit_lowess_fl('num', 'fnum', 9)
| render timechart
```

### [Stored](#tab/stored)

> [!IMPORTANT]
> For this example to run successfully, you must first run the [Function definition](#function-definition) code to store the function.

```kusto
//
// Apply 9 points LOWESS regression on regular time series
//
let max_t = datetime(2016-09-03);
demo_make_series1
| make-series num=count() on TimeStamp from max_t-1d to max_t step 5m by OsVer
| extend fnum = dynamic(null)
| invoke series_fit_lowess_fl('num', 'fnum', 9)
| render timechart
```

---

**Output**

:::image type="content" source="images/series-fit-lowess-fl/lowess-regular-time-series.png" alt-text="Graph showing nine points LOWESS fit to a regular time series." border="false":::

### Test irregular time series

### [Query-defined](#tab/query-defined)

To use a query-defined function, invoke it after the embedded function definition.

```kusto
let series_fit_lowess_fl=(tbl:(*), y_series:string, y_fit_series:string, fit_size:int=5, x_series:string='', x_istime:bool=False)
{
    let kwargs = bag_pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_size', fit_size, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_size = kargs["fit_size"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'import statsmodels.api as sm\n'
        'def lowess_fit(ts_row, x_col, y_col, fsize):\n'
        '    y = ts_row[y_col]\n'
        '    fraction = fsize/len(y)\n'
        '    if x_col == "": # If there is no x column creates sequential range [1, len(y)]\n'
        '       x = np.arange(len(y)) + 1\n'
        '    else: # if x column exists check whether its a time column. If so, normalize it to the [1, len(y)] range, else take it as is.\n'
        '       if x_istime: \n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))\n'
        '           x = x - x.min()\n'
        '           x = x / x.max()\n'
        '           x = x * (len(x) - 1) + 1\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    lowess = sm.nonparametric.lowess\n'
        '    z = lowess(y, x, return_sorted=False, frac=fraction)\n'
        '    return list(z)\n'
        '\n'
        'result = df\n'
        'result[y_fit_series] = df.apply(lowess_fit, axis=1, args=(x_series, y_series, fit_size))\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
};
let max_t = datetime(2016-09-03);
demo_make_series1
| where TimeStamp between ((max_t-1d)..max_t)
| summarize num=count() by bin(TimeStamp, 5m), OsVer
| order by TimeStamp asc
| where hourofday(TimeStamp) % 6 != 0   //  delete every 6th hour to create irregular time series
| summarize TimeStamp=make_list(TimeStamp), num=make_list(num) by OsVer
| extend fnum = dynamic(null)
| invoke series_fit_lowess_fl('num', 'fnum', 9, 'TimeStamp', True)
| render timechart 
```

### [Stored](#tab/stored)

> [!IMPORTANT]
> For this example to run successfully, you must first run the [Function definition](#function-definition) code to store the function.

```kusto
let max_t = datetime(2016-09-03);
demo_make_series1
| where TimeStamp between ((max_t-1d)..max_t)
| summarize num=count() by bin(TimeStamp, 5m), OsVer
| order by TimeStamp asc
| where hourofday(TimeStamp) % 6 != 0   //  delete every 6th hour to create irregular time series
| summarize TimeStamp=make_list(TimeStamp), num=make_list(num) by OsVer
| extend fnum = dynamic(null)
| invoke series_fit_lowess_fl('num', 'fnum', 9, 'TimeStamp', True)
| render timechart 
```

---

**Output**

:::image type="content" source="images/series-fit-lowess-fl/lowess-irregular-time-series.png" alt-text="Graph showing nine points LOWESS fit to an irregular time series." border="false":::

### Compare LOWESS versus polynomial fit

### [Query-defined](#tab/query-defined)

To use a query-defined function, invoke it after the embedded function definition.

```kusto
let series_fit_lowess_fl=(tbl:(*), y_series:string, y_fit_series:string, fit_size:int=5, x_series:string='', x_istime:bool=False)
{
    let kwargs = bag_pack('y_series', y_series, 'y_fit_series', y_fit_series, 'fit_size', fit_size, 'x_series', x_series, 'x_istime', x_istime);
    let code=
        '\n'
        'y_series = kargs["y_series"]\n'
        'y_fit_series = kargs["y_fit_series"]\n'
        'fit_size = kargs["fit_size"]\n'
        'x_series = kargs["x_series"]\n'
        'x_istime = kargs["x_istime"]\n'
        '\n'
        'import statsmodels.api as sm\n'
        'def lowess_fit(ts_row, x_col, y_col, fsize):\n'
        '    y = ts_row[y_col]\n'
        '    fraction = fsize/len(y)\n'
        '    if x_col == "": # If there is no x column creates sequential range [1, len(y)]\n'
        '       x = np.arange(len(y)) + 1\n'
        '    else: # if x column exists check whether its a time column. If so, normalize it to the [1, len(y)] range, else take it as is.\n'
        '       if x_istime: \n'
        '           x = pd.to_numeric(pd.to_datetime(ts_row[x_col]))\n'
        '           x = x - x.min()\n'
        '           x = x / x.max()\n'
        '           x = x * (len(x) - 1) + 1\n'
        '       else:\n'
        '           x = ts_row[x_col]\n'
        '    lowess = sm.nonparametric.lowess\n'
        '    z = lowess(y, x, return_sorted=False, frac=fraction)\n'
        '    return list(z)\n'
        '\n'
        'result = df\n'
        'result[y_fit_series] = df.apply(lowess_fit, axis=1, args=(x_series, y_series, fit_size))\n'
    ;
    tbl
     | evaluate python(typeof(*), code, kwargs)
};
range x from 1 to 200 step 1
| project x = rand()*5 - 2.3
| extend y = pow(x, 5)-8*pow(x, 3)+10*x+6
| extend y = y + (rand() - 0.5)*0.5*y
| summarize x=make_list(x), y=make_list(y)
| extend y_lowess = dynamic(null)
| invoke series_fit_lowess_fl('y', 'y_lowess', 15, 'x')
| extend series_fit_poly(y, x, 5)
| project x, y, y_lowess, y_polynomial=series_fit_poly_y_poly_fit
| render linechart
```

### [Stored](#tab/stored)

> [!IMPORTANT]
> For this example to run successfully, you must first run the [Function definition](#function-definition) code to store the function.

```kusto
range x from 1 to 200 step 1
| project x = rand()*5 - 2.3
| extend y = pow(x, 5)-8*pow(x, 3)+10*x+6
| extend y = y + (rand() - 0.5)*0.5*y
| summarize x=make_list(x), y=make_list(y)
| extend y_lowess = dynamic(null)
| invoke series_fit_lowess_fl('y', 'y_lowess', 15, 'x')
| extend series_fit_poly(y, x, 5)
| project x, y, y_lowess, y_polynomial=series_fit_poly_y_poly_fit
| render linechart
```

---

**Output**

:::image type="content" source="images/series-fit-lowess-fl/lowess-vs-poly-fifth-order-noise.png" alt-text="Graphs of LOWESS vs polynomial fit for a fifth order polynomial with noise on x & y axes":::
