# SQL-Important-Questions

#### Difference between RANK(), DENSE_RANK() and RowNumber()

	Rown Number

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



	RANK

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


	DENSE_RANK 

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
