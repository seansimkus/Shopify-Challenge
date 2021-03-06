# Shopify Data Science Challenge

Author: Sean Simkus

Date: Jan 05, 2022

This repository is my submission for the Summer 2022 Data Science Intern Challenge.

Code for question 1 is located in [DS-Challenge-Question1.ipynb](https://github.com/seansimkus/Shopify-Challange/blob/main/DS-Challenge-Question1.ipynb)

Code for question 2 is located in [DS-Challenge-Question2.sql](https://github.com/seansimkus/Shopify-Challange/blob/main/DS-Challenge-Question2.sql)

- [Question 1](#question-1)
  - [A) Think about what could be going wrong with our calculation](#a-think-about-what-could-be-going-wrong-with-our-calculation)
  - [B) What metric would you report for this dataset?](#b-what-metric-would-you-report-for-this-dataset)
  - [C) What is its value?](#c-what-is-its-value)
- [Question 2](#question-2)
  - [A) How many orders were shipped by Speedy Express in total?](#a-how-many-orders-were-shipped-by-speedy-express-in-total)
  - [B) What is the last name of the employee with the most orders?](#b-what-is-the-last-name-of-the-employee-with-the-most-orders)
  - [C) What product was ordered the most by customers in Germany?](#c-what-product-was-ordered-the-most-by-customers-in-germany)

## Question 1

### A) Think about what could be going wrong with our calculation

Due to the average order value (AOV) being so high, our data is most likely affected by outliers, assuming that the previous user completed the calculations correctly. The high AOV is a strong indicator that the average was derived using the mean, thus heavily influenced by outliers.

```python
# Get the mean of order_amount
mean_order = store_df['order_amount'].mean()
# Get the mean of total_items
mean_items = store_df['total_items'].mean()

print(f'The mean average of the order is ${mean_order} with an mean of {mean_items} items sold')

""" 
returns:
The mean average of the order is $3145.128 with a mean of 8.7872 items sold
"""
```

  After performing some calculations on the dataset, we can see that the mean of the `order_amount` is the same as the reported unreasonably high AOV.

When taking a closer look, we can see that at the distribution of `order_amount` and `total_items`, there is a concerning standard deviation and a considerable max number for each of the variables.

  ```python
        order_amount  total_items
count    5000.000000   5000.00000
mean     3145.128000      8.78720
std     41282.539349    116.32032
min        90.000000      1.00000
25%       163.000000      1.00000
50%       284.000000      2.00000
75%       390.000000      3.00000
max    704000.000000   2000.00000
```

#### Scatter Plot of order_amount and total_items

  ![Scatter Plot](/scatterplot.jpeg "Scatter Plot")

  As mentioned previously, a weakness of using a mean calculation is that outliers can strongly influence it. As we can see from the graph above, an outlier is so significant that it dramatically skews the results. A better way to evaluate this data would be to take the median of the `order_amount` as it is not as affected nearly as much by the outliers.

  ```python
  # Get the median of order_amount
median_order = store_df['order_amount'].median()
# Get the median of total_items
median_items = store_df['total_items'].median()

print(f'The median average of the order is ${median_order} with an median of {median_items} items sold')

"""
Returns:
The median average of the order is $284.0 with a median of 2.0 items sold
"""
```

When using this method, the value comes out to $284, which seems more reasonable given the context.

---

### B) What metric would you report for this dataset?

The metric I would report for this dataset would be the average amount of shoes sold per day. The purpose of this metric is to identify trends in shoe sales and give an idea of the day-to-day performance of the stores. This result could be adapted to focus on each store's average daily sales if the dollar amount is more important than sales volume.

Given the outliers in our data, certain stores perform far better than the others and will need to be investigated to understand the reasoning.

---

### C) What is its value?

```python
# Splt the date and time components into own columns
store_df[['date_created','time_created']] = store_df['created_at'].str.split(' ',1,expand = True)

# Split date compnents into individual columns
store_df[['year_created','month_created','day_created']] = store_df['date_created'].str.split('-', expand=True)

# Group values by day of the month
grouped_df = store_df.groupby(['day_created'], as_index=False).mean()

# Calculate the median shoes sold per day
avg_shoes_per_day = grouped_df['total_items'].median()
print(f'The median amount of shoes sold in a day is {avg_shoes_per_day} shoes')

"""
returns:
The median amount of shoes sold in a day is 2.096788976328182 shoes
"""
```

According to the code above, the median number of shoes sold per day is two. However, this metric is not perfect as we know many days sell far more than two pairs of shoes. I have created a visual below to demonstrate the discrepancy

#### Median Shoes Sold Per Day

![Shoes Per Day](ShoesPerDay.jpeg "Shoes Per Day Bar Chart")

As we can see, there is a drastic swing in the number of shoes sold per day, with most days being around 2 but some days reaching 35 shoes sold. Due to the limited dataset, we cannot determine the exact reason for such a dramatic difference. The data science team will investigate further to understand what effect is causing these discrepancies.

---

## Question 2

### A) How many orders were shipped by Speedy Express in total?

```SQL
-- Query
-- How many orders were shipped by Speedy Express in total?
SELECT COUNT(OrderID) AS Num_Orders FROM Orders INNER JOIN (
    SELECT * FROM Shippers
    WHERE ShipperName = 'Speedy Express') AS SpeedyExpress
ON Orders.ShipperID = SpeedyExpress.ShipperID;

/*
Returns:
Num_Orders: 54
*/
```

Speedy Express shipped 54 orders.

---

### B) What is the last name of the employee with the most orders?

```SQL
-- What is the last name of the employee with the most orders?
-- Query
SELECT LastName FROM Employees INNER JOIN (
    SELECT EmployeeID, COUNT(EmployeeID) AS num_orders FROM [Orders]
    GROUP BY EmployeeID
    ORDER BY num_orders DESC
    LIMIT 1) AS TopEmployee
ON Employees.EmployeeID = TopEmployee.EmployeeID;

/*
Returns:
LastName: Peacock
*/
```

The last name of the employee with the most orders is Peacock.

---

### C) What product was ordered the most by customers in Germany?

```SQL
-- What product was ordered the most by customers in Germany?
-- Query
Select ProductName 
FROM Products INNER JOIN (
  -- Get the ProductID of the most purchased item
  Select ProductID, SUM(Quantity) AS AmountOrdered 
  FROM OrderDetails INNER JOIN (
    -- Select OrderID from German customers
    SELECT OrderID FROM Orders 
    WHERE CustomerID IN (
      -- Select German customers
      SELECT CustomerID FROM Customers
      WHERE Country = 'Germany')) AS GermanOrders
  ON OrderDetails.OrderID = GermanOrders.OrderID
  GROUP BY ProductID
  ORDER BY AmountOrdered DESC
  LIMIT 1) AS TopGermanOrder
ON Products.ProductID = TopGermanOrder.ProductID;
/*
Returns:
ProductName: Boston Crab Meat
*/
```

The most ordered product by customers in Germany was Boston Crab Meat.

---
