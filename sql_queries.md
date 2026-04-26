# 10 SQL Queries

---

## 1. Find the second highest salary

```sql id="c6q5j5"
SELECT MAX(Salary) AS SecondHighestSalary
FROM Employee
WHERE Salary < (
    SELECT MAX(Salary)
    FROM Employee
);
```

---

## 2. Find duplicate records

Example: duplicate emails

```sql id="ihp8lz"
SELECT Email, COUNT(*) AS Count
FROM Users
GROUP BY Email
HAVING COUNT(*) > 1;
```

---

## 3. Remove duplicate records while keeping one

Assume `Id` is unique

```sql id="y46cf5"
DELETE u1
FROM Users u1
JOIN Users u2
ON u1.Email = u2.Email
AND u1.Id > u2.Id;
```

This keeps the smallest `Id`.

---

## 4. Employees joined in last 30 days

```sql id="rw4mfh"
SELECT *
FROM Employees
WHERE JoiningDate >= CURDATE() - INTERVAL 30 DAY;
```

---

## 5. Top 3 highest salaries

```sql id="d6y1oe"
SELECT *
FROM Employee
ORDER BY Salary DESC
LIMIT 3;
```

---

## 6. Customers who never placed an order

```sql id="t2zthm"
SELECT c.*
FROM Customers c
LEFT JOIN Orders o
ON c.CustomerId = o.CustomerId
WHERE o.CustomerId IS NULL;
```

---

## 7. Display manager name for each employee

(Self Join)

```sql id="t57u3c"
SELECT e.EmployeeName,
       m.EmployeeName AS ManagerName
FROM Employees e
LEFT JOIN Employees m
ON e.ManagerId = m.EmployeeId;
```

---

## 8. Latest order for each customer

```sql id="gwyv9n"
SELECT o.*
FROM Orders o
JOIN (
    SELECT CustomerId, MAX(OrderDate) AS LatestOrder
    FROM Orders
    GROUP BY CustomerId
) latest
ON o.CustomerId = latest.CustomerId
AND o.OrderDate = latest.LatestOrder;
```

---

## 9. Department-wise highest salary employee

```sql id="6icnqb"
SELECT e.*
FROM Employees e
JOIN (
    SELECT DepartmentId, MAX(Salary) AS MaxSalary
    FROM Employees
    GROUP BY DepartmentId
) dept
ON e.DepartmentId = dept.DepartmentId
AND e.Salary = dept.MaxSalary;
```

---

## 10. Pagination for employee records

Example: page 3, 10 records per page

```sql id="xw8mwu"
SELECT *
FROM Employees
ORDER BY EmployeeId
LIMIT 10 OFFSET 20;
```

Formula:

```text
OFFSET = (PageNumber - 1) * PageSize
```

---

These 10 are extremely common in interviews and cover most SQL patterns.
