# SQL-Important-Questions

#### 1. Difference between RANK(), DENSE_RANK() and RowNumber()

	# Rown Number

SELECT ROW_NUMBER() OVER (ORDER BY TOTALAmount) AS RowNumber, TotalAmount FROM orders

RowNumber 	| TotalAmount
------		|-----------
1			|	0.00
2			|	5.00
3			|	5.00
4			|	5.00
5			|	5.00
6			|	5.00
7			|	14.00
8			|	76.00
9			|	90.00
10			|	90.00



	# RANK

SELECT RANK() OVER (ORDER BY TOTALAmount) AS [RANK], TotalAmount FROM orders

RANK		|TotalAmount
------------|------
1			|0.00
2			|5.00
2			|5.00
2			|5.00
2			|5.00
2			|5.00
7			|14.00
8			|76.00
9			|90.00
9			|90.00
9			|90.00


	# DENSE_RANK 

SELECT DENSE_RANK() OVER (ORDER BY TOTALAmount) AS [DENSE_RANK], TotalAmount FROM orders

DENSERANK	|TotalAmount
------------|------
1			|0.00
2			|5.00
2			|5.00
2			|5.00
2			|5.00
2			|5.00
3			|14.00
4			|76.00
5			|90.00
5			|90.00


2. #### TOP 3rd Highest Salary

#### Method 1

    SELECT *
    FROM
      (
       SELECT *,RANK() OVER(ORDER BY Salary) As RowNum
       FROM EMPLOYEE
       ) As A
    WHERE A.RowNum IN (3)

#### Method 2

    ;WITH CTE AS
    (
        SELECT RankId = RANK() OVER (Order BY Salary DESC), Salary FROM Employee 
    )
    SELECT DISTINCT * FROM CTE WHERE RankId=3

#### Method 3

    SELECT MAX(Salary)
    FROM Employee
    WHERE Salary IN(SELECT TOP 3 Salary FROM Employee ORDER BY Salary DESC)


3. #### Interchange the gender in a single query

		UPDATE Employee SET Gender = (case when gender = 'Male' then 'Female' ELSE 'Male' END)

4. #### Update the salary of Female by 5000 and male with 500

		UPDATE Employee SET Salary = (case when gender = 'Male' then salary + 500 ELSE Salary + 5000 END)









