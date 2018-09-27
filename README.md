# SQL-Important-Questions

# 1. Difference between RANK(), DENSE_RANK() and RowNumber()

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


# 2. TOP 3rd Highest Salary

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
    WHERE Salary IN(SELECT DISTINCT TOP 3 Salary FROM Employee ORDER BY Salary DESC)


# 3. Interchange the gender in a single query

		UPDATE Employee SET Gender = (case when gender = 'Male' then 'Female' ELSE 'Male' END)

# 4. Update the salary of Female by 5000 and male with 500

		UPDATE Employee SET Salary = (case when gender = 'Male' then salary + 500 ELSE Salary + 5000 END)

# 5. Diffrence between @@IDENTITY and IDENTITY and SCOPE_IDENTITY and IDENT_CURRENT

  - The @@identity function returns the last identity created in the same session.
  - The scope_identity() function returns the last identity created in the same session and the same scope.
  - The ident_current(name) returns the last identity created for a specific table or view in any session.
  - The identity() function is not used to get an identity, it's used to create an identity in a select...into query.
        

#### Example

    CREATE TABLE Newspaper
    (
        ID INT IDENTITY(1,1) PRIMARY KEY,
        Name NVARCHAR(100)
    )

	INSERT INTO Newspaper VALUES ('The Hindu'), ('Times of India')
    
    SELECT * FROM Newspaper
    
    
    1	The Hindu
    2	Times of India
    3	The Hindustan


	CREATE TABLE LocalNewspaper
    (
        ID INT IDENTITY(100,5) PRIMARY KEY,
        Name NVARCHAR(100)
    )
    
    INSERT INTO LocalNewspaper VALUES ('Amur Ujala'), ('Danika Vichar')
	
    SELECT * FROM LocalNewspaper
    
    100	 Amur Ujala
    105	 Danika Vichar
    110	 The Hindustan
    
    
    CREATE TRIGGER insertNewspaper ON Newspaper
    FOR INSERT AS
    BEGIN
    DECLARE @name NVARCHAR(100)
    SELECT @name=name FROM INSERTED
        INSERT INTO LocalNewspaper VALUES (@name)
    END


	INSERT INTO Newspaper VALUES ('The Hindustan')
    
    SELECT @@IDENTITY As [@@IDENTITY]
    GO
    SELECT SCOPE_IDENTITY() As [SCOPE_IDENTITY]
    GO
    SELECT IDENT_CURRENT('Newspaper') [IDENT_SELECT]
    
    @@IDENTITY
	110

	SCOPE_IDENTITY
	3
    
    IDENT_SELECT
	3

#### Explaintation

The session is the database connection. The scope is the current query or the current stored procedure.

A situation where the scope_identity() and the @@identity functions differ, is if you have a trigger on the table. If you have a query that inserts a record, causing the trigger to insert another record somewhere, the scope_identity() function will return the identity created by the query, while the @@identity function will return the identity created by the trigger.

So, normally you would use the scope_identity() function.


# 6. How do you determine the maximum nested-level of Stored Procedure ?
       

Sol : 1. Creating stored procedure : 

    CREATE PROC PROC_SAMPLE1
        AS
    BEGIN
        PRINT @@NESTLEVEL

        EXEC PROC_SAMPLE1
    END

2. Executing stored procedure : 

		EXEC PROC_SAMPLE1

3. Result : 

    1 <br/>
    2 <br/>
    3 <br/>
    .. <br/>
    .. <br/>
    32 <br/>
    Msg 217, Level 16, State 1, Procedure PROC_SAMPLE1, Line 5 
    Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32).
    
# 7. How many Foreign key can i have in my MS sql table ?
       
A Maximum of 253 Foreign Keys we can have in for a single table.

# 8. Are the two statements same 
	
	select ProductName from Products;

	select All ProductName from Products;

Answer:

	YES

# 9. Write a sample syntax of table variable and show its use?


	DECLARE @TableVariableEmployee table (ID int IDENTITY(1,1),Name VARCHAR(150) NOT NULL)
	INSERT INTO @TableVariableEmployee VALUES ('James')
	SELECT * FROM @TableVariableEmployee

# 10. Can we have TRY…CATCH in Sqlserver?

Yes we can have Try catch statements in Sqlserver from SQLServer 2005

Example

    CREATE PROC AddEmployee
    @Name NVARCHAR(5)
    AS
    BEGIN
    BEGIN TRY 
        INSERT INTO EMPLOYEE(Name) VALUES (@Name + 1) -- Generating error
    END TRY 
    BEGIN CATCH 
        IF @@ERROR <> 0  
            BEGIN
                Print  'an error occurred'
            END
    END CATCH
    END

Executing

	EXEC AddEmployee 'Jame Parker'
    
Result

	(0 row(s) affected)
	an error occurred
    
#### TRY…CATCH with error information

    BEGIN TRY  
        -- Generate a divide-by-zero error.  
         DECLARE @result INT;
         SET @result = 1/0;  
    END TRY  
    BEGIN CATCH  
        SELECT  
            ERROR_NUMBER() AS ErrorNumber  
            ,ERROR_SEVERITY() AS ErrorSeverity  
            ,ERROR_STATE() AS ErrorState  
            ,ERROR_PROCEDURE() AS ErrorProcedure  
            ,ERROR_LINE() AS ErrorLine  
            ,ERROR_MESSAGE() AS ErrorMessage;  
    END CATCH;

Result
	ErrorNumber	ErrorSeverity	ErrorState	ErrorProcedure	ErrorLine	ErrorMessage
	8134	16	1	NULL	4	Divide by zero error encountered.    
    

#### Errors Unaffected by a TRY…CATCH Construct

	BEGIN TRY  
	    -- Table does not exist; object name resolution  
	    -- error not caught.  
	    SELECT * FROM NonexistentTable;  
	END TRY  
	BEGIN CATCH  
	    SELECT   
		ERROR_NUMBER() AS ErrorNumber  
	       ,ERROR_MESSAGE() AS ErrorMessage;  
	END CATCH  




