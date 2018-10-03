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


#### Some more

Let's create unique index on newspaper table

CREATE UNIQUE INDEX UNQ_Name ON Newspaper(Name)

Try to insert duplicate value


	INSERT INTO Newspaper VALUES ('The Hindustan')

Result

	Msg 2601, Level 14, State 1, Line 47
	Cannot insert duplicate key row in object 'dbo.NewsPaper' with unique index 'UNQ_Name'. 
	The duplicate key value is (The Hindustan).
	The statement has been terminated.
	
Again execute the

	SELECT @@IDENTITY As [@@IDENTITY]
	GO
	SELECT SCOPE_IDENTITY() As [SCOPE_IDENTITY]
	GO
	SELECT IDENT_CURRENT('Newspaper') [IDENT_SELECT]
	
Result

	@@IDENTITY
	NULL
	
	SCOPE_IDENTITY
	3
	
	IDENT_SELECT
	4


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


ErrorNumber | ErrorSeverity | ErrorState | ErrorProcedure | ErrorLine | ErrorMessage
------------|---------------|------------|----------------|-----------|-------------
8134	    | 16	    |1	         |NULL	          |4	      |Divide by zero error encountered.   
    

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

# 11. Difference between WHERE and HAVING clause?

HAVING specifies a search condition for a group or an aggregate function used in SELECT statement.
HAVING can be used only with the SELECT statement. 
HAVING is typically used in a GROUP BY clause. When GROUP BY is not used, HAVING behaves like a WHERE clause.
HAVING clause is like a WHERE clause, but applies only to groups as a whole, whereas the WHERE clause applies to individual rows. 
A query can contain both a WHERE clause and a HAVING clause. 

WHERE clause is applied first to the individual rows in the tables . 
Only the rows that meet the conditions in the WHERE clause are grouped. 
The HAVING clause is then applied to the rows in the result set. Only the groups that meet the HAVING conditions appear in the query output. 
This is very imp point that You can apply a HAVING clause:
1. only to columns that also appear in the GROUP BY clause Or 
2. 2. in an aggregate function.The above line proves the significance of the definition of having clause given above.

Note that all the columns that appear in select statement must also appear in group by clause also

# 12. What is the use of Set NOCOUNT ON;

By Default When we execute any command it return us the number of record affected. if we don't want to return the number of records affected then we can use Set NOCOUNT ON;

# 13. What is the difference between UNION ALL Statement and UNION

- With UNION, only distinct values are selected.
- The difference between Union and Union all is that Union all will not eliminate duplicate rows, instead it just pulls all rows from all tables fitting your query specifics and combines them into a table.
- A UNION statement effectively does a SELECT DISTINCT on the results set. If you know that all the records returned are unique from your union, use UNION ALL instead, it gives faster results.

Example  

Declare First Table 
    
        DECLARE @Table1 TABLE (ColDetail VARCHAR(10))
        INSERT INTO @Table1 SELECT 'First' UNION ALL SELECT 'Second' UNION ALL
        SELECT 'Third' UNION ALL SELECT 'Fourth' UNION ALL SELECT 'Fifth'

Declare Second Table
    
        DECLARE @Table2 TABLE (ColDetail VARCHAR(10))
        INSERT INTO @Table2
        SELECT 'First' UNION ALL SELECT 'Third' UNION ALL SELECT 'Fifth'

Check the data using SELECT

        SELECT * FROM @Table1
        SELECT * FROM @Table2

UNION ALL

        SELECT * FROM @Table1
        UNION ALL
        SELECT * FROM @Table2
        
Result

	ColDetail

	First		
        Second
        Third
        Fourth
        Fifth
        First
        Third
        Fifth
        

UNION

        SELECT * FROM @Table1
        UNION
        SELECT * FROM @Table2
Result

        ColDetail
        
        Fifth
        First
        Fourth
        Second
        Third

# 14. Difference between VARCHAR and NVARCHAR

#### VARCHAR: 

- 1.Storage: 8 bit 
- 2.Abbreviation: Variable -Length Character String 
- 3.Accepts only English character 
- 4.Doesn't supports other language symbols 
- 5.Runs faster than NVARCHAR as consumes less memory 

#### NVARCHAR:

- 1.Storage: 16 bit 
- 2.Abbreviation: uNicode 
- 3.Accepts both English character and non-English symbols 

# 15. What are magic tables?

Sometimes we need to know about the data which is being inserted/deleted by triggers in database. With the insertion and deletion of data, tables named “INSERTED” and “DELETED” gets created in SQL Server which contains modified/deleted data. 
Here, “INSERTED” and “DELETED” tables are called magic tables.

# 16. What is the difference between a Local and a Global temporary table?

A local temporary table exists only for the duration of a connection and accessible only in the session they have been created or, if defined inside a compound statement, for the duration of the compound statement. 

Global temporary tables (created with a double “##”) are visible to all sessions. You should always check for existence of the global temporary table before creating it… if it already exists, then you will get a duplicate object error. 

Global temporary tables are dropped when the session that created it ends, and all other sessions have stopped referencing it.

# 17. What is #temp table and @table variable in SQL Server?

It's a kind of normal table but it is created and populated on disk, in the system database tempdb — with a session-specific identifier packed onto the name, to differentiate between similarly-named #temp tables created from other sessions. 

The data in this #temp table (in fact, the table itself) is visible only to the current scope. Generally, the table gets cleared up automatically when the current procedure goes out of scope, however, we should manually clean up the data when we are done with it. 

#### Syntax: 

#### create temporary table

		CREATE TABLE #myTempTable (AutoID int,MyName char(50) )

#### populate temporary table

		INSERT INTO #myTempTable (AutoID, MyName )

		SELECT 	AutoID, MyName FROM 	myOriginalTable WHERE 	AutoID <= 50000

#### Drop temporary table

		drop table #myTempTable 

#### @table variable 

Table variable is similar to temporary table It is not physically stored in the hard disk, it is stored in the memory. 

#### Syntax:

		DECLARE @myTable TABLE (AutoID int,myName char(50) )
		INSERT INTO @myTable (AutoID, myName )

		SELECT 	YakID, YakName FROM 	myTable  WHERE 	AutoID <= 50

We don't need to drop the @temp variable as this is created inside the memory and automatically disposed when scope finishes.

# 18. Is table variables are Transaction neutral?

YES, Table variables are Transaction neutral. They are variables and thus aren't bound to a transaction.
Temp tables behave same as normal tables and are bound by transactions.

#### Example

A simple example shows this difference quite nicely:

    BEGIN TRAN
    declare @var table (id int, data varchar(20) )
    create table #temp (id int, data varchar(20) )

    insert into @var
    select 1, 'data 1' union all
    select 2, 'data 2' union all
    select 3, 'data 3'

    insert into #temp
    select 1, 'data 1' union all
    select 2, 'data 2' union all
    select 3, 'data 3'

    select * from #temp
    select * from @var
    ROLLBACK
    select * from @var
    if object_id('tempdb..#temp') is null
        select '#temp does not exist outside the transaction' 

#### Result

We see that the table variable still exists and has all it's data unlike the temporary table that doesn't exists when the 	transaction rollbacked.

# 19. Difference between Primary Key and unique key?

- Primary Key Restrict duplicate values and null values each table can have only one primary key,default clustered index is the primary key. 

- Unique key restrict duplicate values and allow only one null value.

# 20. Store procedure 

Store Procedure 
- Without parameter
- With single parameter
- With multiple parameters
- With output parameter
- Default parameter 

#### Without parameter

    CREATE PROCEDURE uspGetProducts
    AS
    BEGIN

    SELECT * 
    FROM Products

    END

    EXEC uspGetProducts
    
    
#### With single parameter

    CREATE PROCEDURE uspGetProductsById @ID int 
    AS
    BEGIN

        SELECT * 
        FROM Products WHERE ProductID=@ID

    END

    EXEC uspGetProductsById 1
    
    
#### With multiple parameters
 
    CREATE PROCEDURE uspGetProductsByIdName 
    @ID int,
    @Name nvarchar(100)
    AS
    BEGIN

        SELECT * 
        FROM Products WHERE ProductID=@ID AND ProductName=@Name

    END

    EXEC uspGetProductsByIdName 1, 'chai'
    
#### With output parameter

    CREATE PROC GetProductNameById
    @Id int,
    @name nvarchar(100) out
    AS
    BEGIN
    SELECT @name=ProductName FROM Products where ProductID=@Id
    END


    Declare @name nvarchar(100)
    exec GetProductNameById 1,@name output
    select @name
    
#### Default Parameter

    CREATE PROCEDURE uspGetAddress @City nvarchar(30) = NULL
    AS
    SELECT * FROM AdventureWorks.Person.Address
    WHERE City = @City
    
#### How to DROP SP

    DROP PROCEDURE uspGetAddress 
    GO
    -- or
    DROP PROC uspGetAddress 
    GO
    -- or 
    DROP PROC dbo.uspGetAddress -- also specify the schema

#### Dropping Multiple Stored Procedures

    DROP PROCEDURE uspGetAddress, uspInsertAddress, uspDeleteAddress
    GO
    
    -- or
    
    DROP PROC uspGetAddress, uspInsertAddress, uspDeleteAddress
    
#### Encrypting and Decrypting SQL Server Stored Procedures, Views and User-Defined Functions
    
    CREATE PROC test_encrp WITH ENCRYPTION
    AS
    BEGIN

    SELECT 'This is ecrypt text.'
    END


	EXEC test_encrp
    
#### Result 

	This is ecrypt text.

#### To see store procedure
	SP_Helptext test_encrp

#### Result

The text for object 'test_encrp' is encrypt

# 21. How to INSERT N number of records using single INSERT query?

	INSERT INTO Indexing VALUES('Demo Name','Demo company 1',1500)
	GO 100

