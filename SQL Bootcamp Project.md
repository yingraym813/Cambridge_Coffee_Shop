**_This file contains some SQL queries I wrote when I was doing SQL Bootcamp Project._**
1. Report the products that have not been sold.
```javascript
SELECT * FROM products
WHERE NOT EXISTS (SELECT * FROM orderdetails
                  WHERE products.productCode = orderdetails.productCode );

```
2. Return orderNumber, orderDate, orderLineNumber, productName, quantityOrdered, priceEach after Sep 13, 2003 based on The order of orderNumber and orderLineNumber
```javascript
SELECT 
  OD.orderNumber, O.orderDate,
  OD.orderLineNumber, P.productName,
  OD.quantityOrdered, OD.priceEach
FROM orderdetails AS OD
  INNER JOIN products AS P 
    ON OD.productCode = P.productCode
  INNER JOIN orders AS O 
    ON OD.orderNumber = O.orderNumber
WHERE O.orderDate > '2003-09-13'
ORDER BY OD.orderNumber, OD.orderLineNumber;
```
3. Keep all employees and return the employee’s name, customerName, checkNumber, amount based on the order of last name, which have checknumber.
```javascript
SELECT 
  EE.lastName, EE.firstName,
  ctm.customerName, pmt.checkNumber, pmt.amount
FROM employees AS EE 
  LEFT JOIN (customers AS ctm
             INNER JOIN payments AS pmt 
                ON pmt.customerNumber = ctm.customerNumber)
    ON ctm.salesRepEmployeeNumber = EE.employeeNumber
WHERE pmt.checkNumber IS NOT NULL
ORDER BY EE.lastName;
```

4.	List the amount paid by each customer.
```javascript
SELECT Orders.customerNumber, customerName, 
  ROUND(SUM(Detail.quantityOrdered * Detail.priceEach), 2) AS 'Amount Paid'
FROM Customers
  INNER JOIN Orders
    ON Customers.customerNumber = Orders.customerNumber
  INNER JOIN OrderDetails Detail
    ON Orders.orderNumber = Detail.orderNumber
GROUP BY Orders.customerNumber,customerName
ORDER BY SUM(Detail.quantityOrdered * Detail.priceEach) DESC;
```
5. Report the number of orders 'On Hold' for each customer.
```javascript
SELECT customerName , count(*) As 'Orders on Hold'
FROM customers
  INNER JOIN orders
    ON customers.customerNumber = orders.customerNumber
WHERE orders.status = 'On Hold'
GROUP BY customerName;
```

6. Report the names of executives with VP or Manager in their title.
```javascript
SELECT CONCAT(firstName, ' ',lastName) As 'Full Name'
FROM employees
WHERE 
  (jobTitle LIKE '%VP%') 
  OR 
  (jobTitle LIKE '%Manager%');
```

\
Following queries use the **sales_detail** table from _**sales**_ database with the following fields and data types.
Field | Type 
------------ | -------------
sales_detail_id | varchar(50)
sales_id | smallint
sales_date | date
product_id | varchar(50)
price | smallint
Quantity | smallint
Total_price | smallint

7. Report running average total sales for every 2 days.
```javascript
SELECT sales_date, SUM(total_price) AS total_sales,
  AVG(SUM(total_price)) OVER (ORDER BY sales_date rows 
                              BETWEEN CURRENT ROW AND 1 FOLLOWING) AS avg_sales_every_2_day
FROM sales_detail
GROUP BY sales_date
ORDER BY sales_date;
```
8. Report complete sales details with total amount of sales for each salesperson.
```javascript
SELECT *, 
  SUM(Total_price) OVER (PARTITION BY sales_id 
                         ORDER BY sales_id) AS Total_sales
FROM sales_detail;
```

9. Rank total sales for each sale_id.
```javescript
SELECT sales_id, SUM(total_price) AS total, 
		RANK() OVER (ORDER BY SUM(total_price)) AS RANKING
FROM sales_detail
GROUP BY sales_id;
```

10. Report the date which total sales over 230 and the number of order larger than 5.
```javescript
SELECT sales_date 
FROM sales_detail
GROUP BY sales_date
HAVING COUNT(sales_detail_id) > 5 AND SUM(total_price) > 230;
```

11. Report the date which has max total sales.
```javascript
SELECT sales_date
FROM sales_detail
GROUP BY sales_date
ORDER BY SUM(total_price) DESC
limit 1;
```
