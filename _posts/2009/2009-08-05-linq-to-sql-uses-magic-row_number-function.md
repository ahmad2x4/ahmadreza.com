---
layout: post
title: "LINQ-To-SQL Uses magic ROW_NUMBER() function"
date: 2009-08-05 23:39
author: ahmadrezaa
comments: true
categories: [Blog]
tags: [LINQ to SQL, SQL Server]
---


Recently I founded that LINQ-To-SQL Uses magic ROW_NUMBER() function. ROW_NUMBER() function is a magic function which was added in SQL Server 2005. Microsoft put this function in version 2005 so that developers will not take it for granted and appreciate it. ROW_NUMBER " returns the sequential number of a row within a partition of a result set, starting at 1 for the first row in each partition". 


ROW_NUMBER() Syntax:
  

ROW_NUMBER </span><span style="color:black;font-weight:bold;">(</span><span style="color:black;"> </span><span style="color:black;font-weight:bold;">)</span><span style="color:black;">&#160;&#160;&#160;&#160; OVER </span><span style="color:black;font-weight:bold;">(</span><span style="color:black;"> [ ] &lt;</span>[order_by_clause](//MS.SQLCC.v9/MS.SQLSVR.v9.en/tsqlref9/html/bb394abe-cae6-4905-b5c6-8daaded77742.htm)<span style="color:black;">&gt; </span><span style="color:black;font-weight:bold;">)</span>
  

<span style="color:black;">For example following query add a number field which is partitioned by ProductID (reset on new ProductID) in descending order on </span>UnitPrice * OrderQty.
  <div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:18bbd99c-da4b-4a8d-91ed-efd816d340f2" class="wlWriterEditableSmartContent">


``` SQL
SELECT ROW_NUMBER()
OVER (Partition by ProductID ORDER BY UnitPrice * OrderQty DESC) AS ROWNUM,*
FROM Sales.SalesOrderDetail
```


    
ROW_NUMBER() helps programmer to select specified amount of rows within a select command. This feature commonly used in paging. Following example returns @P1<sup>th</sup> to @P2<sup>th rows which list is ordered by ProductId.
    

<div style="display:inline;float:none;margin:0;padding:0;" id="scid:887EC618-8FBE-49a5-A908-2339AF2EC720:04ef15f3-ee11-467f-b1d4-0052225b52f0" class="wlWriterEditableSmartContent"><pre>

``` SQL 
SELECT * FROM
SELECT *, ROW_NUMBER() OVER (ORDER BY ProductID) AS ROWNUMBER
FROM Sales.SalesOrderDetail) AS ALLDATA
WHERE ALLDATA.ROWNUMBER  BETWEEN @P1 and @P2
```
    
Skip() and Take()LINQ-To-SQL functions generate ROW_NUMBER syntax in query result. I created a LINQ-TO-SQL dbml file on AdventureWorks. I selected SalesOrderDetail as Data Class.
    

``` CSharp 
AdventureWorksDataContext advDC = new AdventureWorksDataContext();

IQueryable orderDetails = advDC.SalesOrderDetails.OrderBy(f => f.SalesOrderDetailID)
                                                 .Skip(20).Take(10);

foreach (SalesOrderDetail orderDetail in orderDetails)
    Console.WriteLine(orderDetail.ProductID);
Console.ReadLine();
```

Above code generates following SQL for orderDetails:
    

``` SQL 

SELECT [t1].[SalesOrderID], [t1].[SalesOrderDetailID], [t1].[CarrierTrackingNumber], [t1].[OrderQty], 
    [t1].[ProductID], [t1].[SpecialOfferID], [t1].[UnitPrice], [t1].[UnitPriceDiscount], 
    [t1].[LineTotal], [t1].[rowguid], [t1].[ModifiedDate]
FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY [t0].[SalesOrderDetailID]) AS [ROW_NUMBER], 
            [t0].[SalesOrderID], [t0].[SalesOrderDetailID], [t0].[CarrierTrackingNumber], 
            [t0].[OrderQty], [t0].[ProductID], [t0].[SpecialOfferID], [t0].[UnitPrice], 
            [t0].[UnitPriceDiscount], [t0].[LineTotal], [t0].[rowguid], [t0].[ModifiedDate]
    FROM [Sales].[SalesOrderDetail] AS [t0]
    ) AS [t1]
WHERE [t1].[ROW_NUMBER] BETWEEN @p0 + 1 AND @p0 + @p1
ORDER BY [t1].[ROW_NUMBER]
}

```

As you see, Skip and Take functions interpreted as ROW_NUMBER() function.

