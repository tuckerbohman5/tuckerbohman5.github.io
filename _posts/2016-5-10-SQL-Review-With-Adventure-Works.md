---
layout: post
title: SQL Review With Adventure Works
---

Create an instance of SQL Server Express 2012 on a cloud provider (AWS, Azure, etc.). Attach the AdventureWorksLT2012 database (which can be found on CodePlex). 
The first step I took was to create an Azure account with Microsoft. I then searched the marketplace on Azure for SQL Server Express 2012. I then deployed the Virtual Machine running SQL Server Express 2012. I then used the SQL Database tool built into Azure to connect a sample database containing the AdventureWorks2012 data. I followed along the tutorials in Microsoft’s documentation to figure out exactly how to do this. Now that I have my server up and running it is time to connect to the database and run queries. One issue I was running into was finding a good tool to run SQL queries from my Macbook. I tried RazorSQL but after spending an hour trying to troubleshoot the connection with no help from the error messages. I decided to use a different tool called SQLPro after downloading and launching SQLPro I knew what my problem was… I was doing everything right I was just using the wrong password to connect. Now that I have everything connected I can start running queries.
 Author some SQL queries against this database, demonstrating knowledge of several of the following: joins, grouping, window functions, common table expressions, etc.  Also, author some queries against the system tables to bring back some information about the SQL Server instance or the AdventureWorksLT2012 database.
 
Please provide the queries and results for the below:

1. Please show the title and last name of customers, Company Name, and Address (Address lines 1 and 2, City, State, and Postal Code). If there is no address line 2 please show it as blank.

```sql
SELECT title, lastName, companyName, addressLine1, ISNULL(addressLine2,'') as AddressLine2, city, stateProvince, postalCode
FROM SalesLT.customer
INNER JOIN SalesLT.customerAddress ON SalesLT.customer.CustomerID = salesLT.CustomerAddress.CustomerID
INNER JOIN SalesLT.address on salesLT.CustomerAddress.AddressID = salesLT.Address.AddressID
```

I used the ISNULL function to replace all null values for addressline2 with a blank space per the instructions.

2. Please provide the count of products with a standard cost greater than 500, both the number of products and number of prices.

```sql
SELECT COUNT(name) as ProductCount, COUNT(DISTINCT StandardCost) as DistinctStandardCost, COUNT(DISTINCT ListPrice) as DistinctPrices
FROM SalesLT.Product
WHERE StandardCost > 500
```

This query returns the total number of products, the number of distinct standard costs and the number of distinct list prices. I was unsure if number of prices referred to standard cost or list price so I included both. 

3. Please provide the Product name, total quantity ordered, product list price, and the total price for all ordered products

```sql
SELECT p.name as ProductName, SUM(so.orderqty) as TotalQuantityOrdered, p.ListPrice as ProductListPrice, SUM(so.orderQty) * p.ListPrice as SalesTotal
FROM SalesLT.SalesOrderDetail so
INNER JOIN SalesLT.Product p ON p.ProductID = so.ProductID
GROUP BY p.name, p.ListPrice
```

This query was a little tricky because I needed to use the group by keyword and the SUM function to calculate the total order quantity for each product that may have been included on multiple different orders. However, after breaking it down into small pieces and slowly building up the functionality I was able to get it to return the data I was looking for. 

```sql
WITH product_summary as ( 
  SELECT so.productID, SUM(so.orderqty) as TotalQuantityOrdered 
  FROM SalesLT.SalesOrderDetail so
  GROUP BY so.ProductID
)
SELECT p.Name as ProductName, ps.TotalQuantityOrdered, p.ListPrice, p.ListPrice * ps.TotalQuantityOrdered as SalesTotal
FROM product_summary ps
INNER JOIN salesLT.Product p on p.ProductID = ps.ProductID
```

I decided to write the query using a common table expression for some additional practice above. Below I created the same query but instead of using a common table expression I just used a similar sub query: 

```sql
 SELECT p.Name as ProductName, ps.TotalQuantityOrdered, p.ListPrice, p.ListPrice * ps.TotalQuantityOrdered as SalesTotal
FROM (SELECT so.productID, SUM(so.orderqty) as TotalQuantityOrdered 
  FROM SalesLT.SalesOrderDetail so
  GROUP BY so.ProductID) as ps
INNER JOIN salesLT.Product  p on p.ProductID = ps.ProductID
```

4. Check that Common Language Runtime is enabled. If it is not turned on turn Common Language Runtime on.

While trying to run the command to enable CLR I ran into the following error: Could not find stored procedure 'sp_configure'. From the documentation I have read on Microsoft’s website as well as several articles on stack overflow there is no support for CLR in Azure SQL database which is what I am using. If I was not using Azure SQL I would be able to run the following command: sp_configure ‘clr enabled’, 1; and then RECONFIGURE;

5. Please provide the number of rows and index size for the Product and Address tables

```sql
DECLARE  @temp_table TABLE (     
        tablename sysname 
,       row_count INT 
,       reserved VARCHAR(50) collate database_default 
,       data VARCHAR(50) collate database_default 
,       index_size VARCHAR(50) collate database_default 
,       unused VARCHAR(50) collate database_default  
); 
INSERT INTO @temp_table
EXEC sp_spaceused 'SalesLT.Product';

INSERT INTO @temp_table
EXEC sp_spaceused 'SalesLT.Address';

SELECT tablename as TableName, row_count as NumOfRows, index_size as IndexSize
FROM @temp_table;
```

This was another tricky query. I first was able to get the results in two separate queries by just running the stored procedure sp_spaceused for each table but I wanted to get the specific information required from each table combined into one table. I was able to do this by first declaring a temp_table with the same schema as the results of sp_spaceused. I was then able to insert the results of sp_spaceused for each table (Product and Address) into the temporary table. I could then query the needed columns from that temp_table.


