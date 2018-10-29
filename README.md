# SQL-Important-Questions

# 1. Difference between RANK(), DENSE_RANK(), NTILE() and RowNumber()

	CREATE TABLE StudentResult(StudentName VARCHAR(50),Subject VARCHAR(20),Marks INT)

	INSERT INTO StudentResult VALUES('Rachel','Maths',70)
	INSERT INTO StudentResult VALUES ('Rachel','Science',80)
	INSERT INTO StudentResult VALUES ('Rachel','Social',60)

	INSERT INTO StudentResult VALUES('Bruce','Maths',60)
	INSERT INTO StudentResult VALUES ('Bruce','Science',50)
	INSERT INTO StudentResult VALUES ('Bruce','Social',70)

	INSERT INTO StudentResult VALUES('Harvey','Maths',90)
	INSERT INTO StudentResult VALUES ('Harvey','Science',90)
	INSERT INTO StudentResult VALUES ('Harvey','Social',80)


RANK()

	SELECT StudentName, Subject, Marks, RANK() OVER(PARTITION BY StudentName ORDER BY Marks DESC) RANK
	FROM StudentResult

|NAME  |Subject|Marks|Rank|
|------|-------|-----|----|
|Bruce |Social |70   |1   |
|Bruce |Maths  |60   |2   |
|Bruce |Science|50   |3   |
|Harvey|Maths  |90   |1   |
|Harvey|Science|90   |1   |
|Harvey|Social |80   |3   |
|Rachel|Science|80   |1   |
|Rachel|Maths  |70   |2   |
|Rachel|Social |60   |3   |

DENSE_RANK()

	SELECT StudentName,Subject,Marks, DENSE_RANK() OVER(PARTITION BY StudentName ORDER BY Marks DESC) 
	DENSERank FROM StudentResult

|Name   |Subject|Marks  |DENSERank|
|-------|-------|-------|---------|
|Bruce	|Social	|70	|1|
|Bruce	|Maths	|60	|2|
|Bruce	|Science|50	|3|
|Harvey	|Maths	|90	|1|
|Harvey	|Science|90	|1|
|Harvey	|Social	|80	|2|
|Rachel	|Science|80	|1|
|Rachel	|Maths	|70	|2|
|Rachel	|Social	|60	|3|


NTILE()

	SELECT StudentName,Subject,Marks, NTILE(2) OVER(PARTITION BY StudentName ORDER BY Marks DESC) 
	NTILE_ FROM StudentResult
	
|StudentName|Subjects|Marks|NTILE_|
|-------|-------|-------|-|
|Bruce	|Social	|70	|1|
|Bruce	|Maths	|60	|1|
|Bruce	|Science|50	|2|
|Harvey	|Maths	|90	|1|
|Harvey	|Science|90	|1|
|Harvey	|Social	|80	|2|
|Rachel	|Science|80	|1|
|Rachel	|Maths	|70	|1|
|Rachel	|Social	|60	|2|


Row_Number

SELECT StudentName, Subject, Marks, ROW_NUMBER() OVER(ORDER BY StudentName) RowNumber 
FROM StudentResult 

|StudentName|Subject|Marks|RowNumber|
|-----------|-------|-----|---------|
|Bruce	|Maths	|60	|1|
|Bruce	|Science|50	|2|
|Bruce	|Social	|70	|3|
|Harvey	|Maths	|90	|4|
|Harvey	|Science|90	|5|
|Harvey	|Social	|80	|6|
|Rachel	|Maths	|70	|7|
|Rachel	|Science|80	|8|
|Rachel	|Social	|60	|9|

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
	
# 22. @@ERROR
Returns the error number for the last Transact-SQL statement executed

	DECLARE @ErrorNumber INT
	RAISERROR('This is custom error',16,1);
	SELECT @ErrorNumber = @@Error
	PRINT @ErrorNumber
	IF @ErrorNumber = 50000
	BEGIN
		PRINT 'ERROR raise by RAISERROR function.'
	END
	ELSE
	BEGIN
		PRINT 'UNKNOW ERROR.'
	END

# 23. @@TRANCOUNT
Its returns the no. of BEGIN TRANSACTION statements in current connection.

	PRINT @@TRANCOUNT  
	--  The BEGIN TRAN statement will increment the  
	--  transaction count by 1.  
	BEGIN TRAN  
	    PRINT @@TRANCOUNT  
	    BEGIN TRAN  
		PRINT @@TRANCOUNT  
	--  The COMMIT statement will decrement the transaction count by 1.  
	    COMMIT  
	    PRINT @@TRANCOUNT  
	COMMIT  
	PRINT @@TRANCOUNT  
	
	--Results  
	--0  
	--1  
	--2  
	--1  
	--0 

# 24. XACT_STATE
Return the user transaction state
Its return three values 0, 1, or -1

If 1, the transaction is committable.  
If -1, the transaction is uncommittable and should be rolled back.  
If 0, means there is no transaction and a commit or rollback operation would generate an error.  

	BEGIN TRY
	SET XACT_ABORT ON
	BEGIN TRAN
		RAISERROR('Hello this is error',16,1);
	END TRY
	BEGIN CATCH
	SELECT
	ERROR_NUMBER() as ErrorNumber,
	ERROR_MESSAGE() as ErrorMessage;

	-- Test XACT_STATE for 1 or -1.
	-- XACT_STATE = 0 means there is no transaction and
	-- a commit or rollback operation would generate an error.
	-- Test whether the transaction is uncommittable.
	IF (XACT_STATE()) = -1
	BEGIN
	PRINT
		N'The transaction is in an uncommittable state. ' + 'Rolling back transaction.'
	ROLLBACK TRANSACTION;
	END;

	-- Test whether the transaction is active and valid.
	IF (XACT_STATE()) = 1
	BEGIN
	PRINT
		N'The transaction is committable. ' + 'Committing transaction.'  
	SELECT XACT_STATE()
	COMMIT TRANSACTION; 
	END;
	SELECT XACT_STATE()
	END CATCH; 

# 25. @@ROWCOUNT
It returns the number of rows affected by the last T-SQL statement. If no. of rows are more than 2 billion, use ROWCOUNT_BIG

	UPDATE Emp SET JobTitle = N'Executive' WHERE ID = 123456789  

	IF @@ROWCOUNT = 0  
		PRINT 'No rows were updated';  
		
# 26. ROWCOUNT_BIG
It returns no. of rows affected by the last statement executed. it's the return type of ROWCOUNT_BIG is bigint.

# 27. Isolation Level

Types of Isolation Level

1. READ COMMITTED
2. READ UNCOMMITTED
3. REPEATABLE READ
4. SERIALIZABLE
5. SNAPSHOT

|ID|	Name	|Salary |
|--|------------|-------|
|1 |	Rachel	|1000   |
|2 |	Bruce	|2000   |
|3 |	Harvey	|3000   |

READ COMMITTED

Session 1:

	BEGIN TRAN

	UPDATE EMPLOYEE SET SALARY=99 WHERE ID=1
	WAITFOR DELAY '00:00:15'
	COMMIT

Session 2:

	SET TRANSACTION ISOLATION LEVEL READ COMMITTED
	SELECT SALARY FROM EMPLOYEE WHERE ID=1

Result: -

99

In second session, will return the result only after complete execution transaction in first session, Session 1 will lock on Employee table for 15 second becuase transaction is not committed. 

Session 1
	
	BEGIN TRAN
	SELECT * FROM EMPLOYEE
	WAITFOR DELAY '00:00:15'
	COMMIT

Session 2

	SET TRANSACTION ISOLATION LEVEL READ COMMITTED
	SELECT SALARY FROM EMPLOYEE 

Result

|SALARY|
|------|
|99|
|2000|
|3000|

No delay, because there is not update and delete statement in session 1's transaction.

READ UNCOMMITTED

Session 1:

	BEGIN TRAN
	UPDATE EMPLOYEE SET SALARY=999 WHERE ID=1
	WAITFOR DELAY '00:00:15'
	ROLLBACK
	
Session 2:

	SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
	SELECT SALARY FROM EMPLOYEE WHERE ID=1
	
	
Result:
999

No delay in session 2 for fetching result because uncommitted can do dirty read.

Dirty Read - Those rows whcih are uncommitted or pending committed. 


REPEATABLE READ
Data can not be modified from any other sessions till transcation is completed.

Example 1

Session 1:

	SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
	BEGIN TRAN
	SELECT * FROM Employee WHERE ID IN(1,2)
	WAITFOR DELAY '00:00:15'
	SELECT * FROM Employee WHERE ID IN (1,2)
	ROLLBACK

Session 2:

	UPDATE Employee SET SALARY=99 WHERE ID=1

Result:
Data can not be modified by session 2 update statement untill session 1 is completed.

Example 2

Session 1:

	SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
	BEGIN TRAN
	SELECT * FROM EMPLOYEE
	WAITFOR DELAY '00:00:15'
	SELECT * FROM EMPLOYEE
	ROLLBACK

Session 2:

 	INSERT INTO EMPLOYEE(ID,NAME,SALARY) VALUES( 11,'James',1000)
	
Result:

|ID|	Name|	Salary|
|--|--------|---------|
|1|	Rachel|	99|
|2|	Bruce|	2000|
|3|	Harvey|	3000|

|ID|	Name|	Salary|
|--|--------|---------|
|1|	Rachel|	99|
|2|	Bruce|	2000|
|3|	Harvey|	3000|
|4|     James|  1000| <----- This is a Phantom row to prevent this we use SERIALIZABLE isolation level.

session 2 will execute without any delay because insert query is for new record. 
REPEATABLE READ allows to insert new data but does not allow to modify data that is used in select query executed in transaction.

Example 3:

Session 1:
	 SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
	 BEGIN TRAN
	 SELECT * FROM EMPLOYEE WHERE ID IN(1,2)
	 WAITFOR DELAY '00:00:15'
	 SELECT * FROM EMPLOYEE WHERE ID IN (1,2)
	 ROLLBACK

Session 2:

	UPDATE EMPLOYEE SET SALARY=999 WHERE ID=3

Result:

Session 2 Update query will execute without any delay because ID=3 is not locked by session 1 

SERIALIZABLE

Similar to Repeatable Read Isolation but the difference is it prevents Phantom Read.

Example 1:

Session 1:

	SET TRANSACTION ISOLATION LEVEL SERIALIZABLE
	BEGIN TRAN
	SELECT * FROM EMPLOYEE
	WAITFOR DELAY '00:00:15'
	SELECT * FROM EMPLOYEE
	ROLLBACK

Session 2:

	INSERT INTO EMP(ID,NAME,SALARY)  VALUES( 11,'STEWART',11000)

Result:

Session 2, will have to wait untill session 1 will get completed, because session 1 will lock the complete table. 

Example 2:

Session 1:

	SET TRANSACTION ISOLATION LEVEL SERIALIZABLE
	BEGIN TRAN
	SELECT * FROM EMPLOYEE WHERE ID BETWEEN 1 AND 3
	WAITFOR DELAY '00:00:15'
	SELECT * FROM EMPLOYEE WHERE ID BETWEEN 1 AND 3
	ROLLBACK

Session 2:

	INSERT INTO EMPLOYEE(ID,NAME,SALARY)  VALUES( 13,'Rocko',11000)

Result : session 2 will not delay it will get executed and insert a new record. Becuase session 1 will only lock the record between 1 and 3.

SNAPSHOT

Similar to Serializable isolation. The difference is Snapshot does not hold lock on table during the transaction so table can be modified in other sessions. 
Snapshot isolation maintains versioning in Tempdb for old data in case of any data modification occurs in other sessions then existing transaction displays the old data from Tempdb.

Session 1:

	ALTER DATABASE EFDemo  
	SET ALLOW_SNAPSHOT_ISOLATION ON  

	ALTER DATABASE EFDemo  
	SET READ_COMMITTED_SNAPSHOT ON  

	SET TRANSACTION ISOLATION LEVEL SNAPSHOT
	BEGIN TRAN
	SELECT * FROM EMPLOYEE
	WAITFOR DELAY '00:00:15'
	SELECT * FROM EMPLOYEE
	ROLLBACK
	
Session 2:

 INSERT INTO EMPLOYEE(ID,NAME,SALARY) VALUES( 15,'Dave',11000)
 UPDATE EMPLOYEE SET SALARY=2000 WHERE ID=2
 SELECT * FROM EMPLOYEE
 
 
 Result"
 
 Session 1:
 
|ID	|Name	|Salary|
|-------|-------|------|
|1	|Rachel	|99|
|2	|Bruce	|2000|
|3	|Harvey	|999|
|11	|Mark	|11000|
|12	|James	|11000|
|13	|Rocko	|11000|
 
 
|ID	|Name	|Salary|
|-------|-------|------|
|1	|Rachel	|99|
|2	|Bruce	|2000|
|3	|Harvey	|999|
|11	|Mark	|11000|
|12	|James	|11000|
|13	|Rocko	|11000|
 
 
 
 Session 2:
 
|ID	|Name	|Salary|
|-------|-------|------|
|1	|Rachel	|99|
|2	|Bruce	|2000|
|3	|Harvey	|999|
|11	|Mark	|11000|
|12	|James	|11000|
|13	|Rocko	|11000|
|15	|Dave	|11000|
 
