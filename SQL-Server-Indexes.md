# Pages
* Its a fundamental unit of data storage in SQL Server is the PAGE (8 KB) in size.
* 128 pages per MB
* Each page begin with 9-bytes header
* Maximum amount of data contained in a single ROW is 8060 bytes 
* It leave 36-bytes slot array that will be use 

# Extents
* Collection of 8 physically contiguous pages.

# Heap
* A heap is table without clustered index.
* The data row are not stored in any particular order
* Data pages are not linked in a linked list.

# Clustered Index

* Data rows are stored in order based on the clustered index key.
* Clustered index is implemented as a B-tree index structure.  
* Data pages in the leaf level are linked in a doubly-linked list
* Clustered indexes have one row in sys.partition, with index_id =1
		
		Index = 1  - means table has got clustered index
		Index = 0  - means you have a heap
		Index_id > 1 - means they do have non-clustered index
      
 # Non - Clustered Index
* Nonclustered indexes have a B-tree index structure similar to the the one in the clustered indexes
      The difference is that nonclustered index do not effect the order of the data rows
* Each index row contains the nonclustered key value, a row locator and any included, or nonkey columns.

# Fill Factor
* The method to pre-allocate some space for the future expansion Fill-Factor is used.
* To avoid PAGESPLIST and degrade performance.

# Module 2:

# Primary Key
* Primary key automatically creates clustered index.
	* Except - when non clustered index is specified 
	* Except - when clustered index already exists
* Primary key automatically creates unique index on column.
	* Note: non unique clustered index adds 4-byte unique identifier column  

1.If we create primary key on a table then, a clustered index will automatically get created. 

## Example

Create a table without any primary key

	CREATE TABLE CompanyRecords
	(
		Id BIGINT NOT NULL,
		Name NVARCHAR(100),
		NumberOfEmployees INT
	)

Create a primary

	ALTER TABLE CompanyRecords
	ADD CONSTRAINT [PK_ComnayRecord] PRIMARY KEY ([Id] ASC) 

Result - A clustered index will automatcially created 

Now, Drop the primary Key

	ALTER TABLE CompanyRecords DROP CONSTRAINT [PK_ComnayRecord]

Now Add a Non-Clustered index in Id with primary key column

	ALTER TABLE CompanyRecords 
	ADD CONSTRAINT [PK_ComnayRecord] PRIMARY KEY NONCLUSTERED ([Id] ASC) 

Result:A unique, non-clustered index and primary key will get created. 

Now Again drop the primary key

	ALTER TABLE CompanyRecords DROP CONSTRAINT [PK_ComnayRecord] 

Now there is no primary key on the table and we are going to create a Clustered Index on the table

	CREATE CLUSTERED INDEX [CL_ComapyRecord_Index] ON CompanyRecords 
	(
		Name ASC
	)

Result: A cluster index will get created.

Now a primary key again

	ALTER TABLE CompanyRecords
	ADD CONSTRAINT [PK_ComnayRecord] PRIMARY KEY ([Id] ASC) 


Result: A unique, non-clustered index will get created. 
