---
title: Tabular expression statements - Azure Data Explorer
description: Learn how to use tabular expression statements to produce tabular data sets.
ms.reviewer: alexans
ms.topic: reference
ms.date: 03/12/2023
---
# Tabular expression statements

The tabular expression statement is what people usually have in mind when they talk about queries. This statement usually appears last in the statement list, and both its input and its output consists of tables or tabular data sets.
Any two statements must be separated by a semicolon.

Azure Data Explorer uses a data flow model for the tabular expression statement. A tabular expression statement is generally composed of *tabular data sources*
such as Azure Data Explorer tables, *tabular data operators* such as filters and projections, and optional *rendering operators*. The composition is represented by the pipe character (`|`), giving the statement a very regular form that visually represents the flow of tabular data from left to right.
Each operator accepts a tabular data set "from the pipe", and other inputs including more tabular data sets from the body of the operator, then emits a tabular data set to the next operator that follows.

## Syntax

*Source* `|` *Operator1* `|` *Operator2* `|` *RenderInstruction*

## Parameters

|Name|Type|Required|Description|
|--|--|--|--|
|*Source*|string|&check;|A tabular data source, such as an Azure Data Explorer table.|
|*Operator*|string|&check;|Tabular data operators, such as filters and projections.|
|*RenderInstruction*|string||Rendering operators or instructions.|

## Tabular data sources

A tabular data source produces sets of records, to be further processed by tabular data operators. Azure Data Explorer supports several of these sources:

* Table references (which refer to an Azure Data Explorer table, in the context database or some other cluster/database.)
* The tabular [range operator](rangeoperator.md).
* The [print operator](printoperator.md).
* An invocation of a function that returns a table.
* A [table literal](datatableoperator.md) ("datatable").

## Example

In the following more complex example, the `join` operator is used to combine records from two input data sets, one that is a filter on the `Logs` table, and another that is a filter on the `Events` table.

```kusto
Logs 
| where Timestamp > ago(1d) 
| join 
(
    Events 
    | where continent == 'Europe'
) on RequestId 
```
