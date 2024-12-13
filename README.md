# Comprehensive Database Administrator Interview Guide

## 1. Batch, Script, and Transaction

### Batch
A batch in SQL Server is a group of one or more Transact-SQL statements processed together as a single unit. 

Example:
```sql
-- Batch example
BEGIN
    DECLARE @Count INT
    SET @Count = (SELECT COUNT(*) FROM Students)
    PRINT 'Total Students: ' + CAST(@Count AS VARCHAR(10))
    
    UPDATE Students 
    SET Status = 'Active' 
    WHERE EnrollmentDate < GETDATE()
END
```

### Script
A script is a collection of SQL statements saved in a file, typically used for database management, schema creation, or data manipulation.

Example:
```sql
-- Database creation script
USE master
GO

CREATE DATABASE UniversityDB
ON PRIMARY 
(
    NAME = 'UniversityDB_Primary',
    FILENAME = 'C:\Databases\UniversityDB_Primary.mdf',
    SIZE = 100MB,
    MAXSIZE = UNLIMITED,
    FILEGROWTH = 10%
)
LOG ON 
(
    NAME = 'UniversityDB_Log',
    FILENAME = 'C:\Databases\UniversityDB_Log.ldf',
    SIZE = 50MB,
    MAXSIZE = 2GB,
    FILEGROWTH = 10%
)
```

### Transaction
A transaction is a sequence of database operations that are treated as a single logical unit of work.

Example:
```sql
-- Transaction example
BEGIN TRANSACTION StudentRegistration

BEGIN TRY
    INSERT INTO Students (StudentName, Email) VALUES ('John Doe', 'john@example.com')
    INSERT INTO Enrollments (StudentID, CourseID) VALUES (SCOPE_IDENTITY(), 101)
    
    COMMIT TRANSACTION
    PRINT 'Student registered successfully'
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION
    PRINT 'Error in student registration'
END CATCH
```

## 2. ACID Properties

ACID is a set of properties that guarantee database transaction reliability:

### Atomicity
- Ensures all operations in a transaction are completed or none are
- Example: Money transfer between bank accounts must complete entirely or not at all

### Consistency
- Maintains database integrity by ensuring transactions move from one valid state to another
- Example: Enforcing referential integrity constraints

### Isolation
- Prevents interference between concurrent transactions
- Ensures transactions are processed independently

### Durability
- Guarantees that once a transaction is committed, it remains permanent
- Typically implemented through transaction logs and backup mechanisms

## 3. Cursor
A database object that allows row-by-row processing of query results.

Example:
```sql
DECLARE @StudentName NVARCHAR(100)
DECLARE StudentCursor CURSOR FOR 
    SELECT Name FROM Students

OPEN StudentCursor
FETCH NEXT FROM StudentCursor INTO @StudentName

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Processing Student: ' + @StudentName
    FETCH NEXT FROM StudentCursor INTO @StudentName
END

CLOSE StudentCursor
DEALLOCATE StudentCursor
```

## 4. Types of DB Backup

1. **Full Backup**: Complete database backup
2. **Differential Backup**: Captures changes since last full backup
3. **Transaction Log Backup**: Backs up database transaction logs
4. **Incremental Backup**: Backs up only changed data since last backup
5. **Copy-Only Backup**: Does not affect backup sequence

## 5. Identity Columns

### Writing Values
- By default, you cannot directly insert values into identity columns
- Use `SET IDENTITY_INSERT ON` to allow manual insertion

```sql
SET IDENTITY_INSERT Students ON
INSERT INTO Students (StudentID, Name) VALUES (1000, 'John Doe')
SET IDENTITY_INSERT Students OFF
```

### Resetting Identity
```sql
-- Reset identity seed
DBCC CHECKIDENT('TableName', RESEED, 0)
```

## 6. Database Job
Scheduled tasks in SQL Server for automated maintenance:
- Backup strategies
- Index maintenance
- Database integrity checks
- Reporting generation

## 7. Insert Statement Types

1. **Standard Insert**
```sql
INSERT INTO Students (Name, Email) VALUES ('John', 'john@example.com')
```

2. **Bulk Insert**
```sql
BULK INSERT Students
FROM 'C:\Students.csv'
WITH (FIELDTERMINATOR = ',', ROWTERMINATOR = '\n')
```

3. **Insert with Select**
```sql
INSERT INTO StudentArchive
SELECT * FROM Students WHERE GraduationYear < 2020
```

4. **Multiple Row Insert**
```sql
INSERT INTO Students (Name, Email)
VALUES 
    ('John', 'john@example.com'),
    ('Jane', 'jane@example.com')
```

5. **Default Insert**
```sql
INSERT INTO Students DEFAULT VALUES
```

# Database Administrator Interview Guide (Continued)

## 8. Snapshot
A snapshot is a read-only, point-in-time view of a database that captures the state of data at a specific moment.

Key Characteristics:
- Provides a static view of data
- Useful for reporting and historical analysis
- Minimal storage overhead
- Does not impact original database performance

Example Creation:
```sql
-- Create database snapshot
CREATE DATABASE StudentDB_Snapshot
ON 
(
    NAME = StudentDB_Data,
    FILENAME = 'C:\Snapshots\StudentDB_Snapshot.mdf'
)
AS SNAPSHOT OF StudentDB
```

## 9. SQLCLR (SQL Common Language Runtime)
SQLCLR enables integration of .NET Framework languages with SQL Server, allowing:
- Complex calculations
- Custom data types
- Advanced string manipulations
- External language integration

Example (C# CLR Function):
```csharp
[Microsoft.SqlServer.Server.SqlFunction]
public static SqlInt32 CalculateStudentAge(SqlDateTime birthDate)
{
    return DateTime.Now.Year - birthDate.Value.Year;
}
```

## 10. SQL Injection
A code injection technique where malicious SQL statements are inserted into application entry points.

Prevention Techniques:
- Parameterized Queries
- Input Validation
- Stored Procedures
- Least Privilege Principle

Vulnerable Code:
```sql
-- Vulnerable
string query = "SELECT * FROM Users WHERE Username = '" + username + "' AND Password = '" + password + "'";

-- Secure Parameterized Query
string query = "SELECT * FROM Users WHERE Username = @Username AND Password = @Password";
```

## 11. Red Gate
A suite of SQL Server tools for:
- Database development
- Performance monitoring
- Backup and recovery
- Schema comparison
- Data generation

## 12. Data Quality Services (DQS)
Microsoft SQL Server component for:
- Data cleansing
- Matching
- Profiling
- Knowledge discovery

Workflow:
1. Knowledge Discovery
2. Domain Management
3. Cleansing
4. Matching
5. Export Results

## 13. XML in SQL Server
XML (eXtensible Markup Language) support includes:
- Storage
- Querying
- Transformation

Examples:
```sql
-- XML Data Type
DECLARE @XMLData XML
SET @XMLData = '<Students>
    <Student>
        <Name>John Doe</Name>
        <Age>22</Age>
    </Student>
</Students>'

-- XML Query
SELECT 
    T.c.value('Name[1]', 'VARCHAR(50)') AS StudentName,
    T.c.value('Age[1]', 'INT') AS StudentAge
FROM @XMLData.nodes('/Students/Student') T(c)
```

## 14. Triggers
Database objects that automatically execute in response to specific database events.

Types:
- INSTEAD OF Trigger
- AFTER Trigger

Prevention Friday Insertion Trigger:
```sql
CREATE TRIGGER PreventFridayInsertion
ON Students
FOR INSERT
AS
BEGIN
    IF DATEPART(dw, GETDATE()) = 6  -- Friday
    BEGIN
        ROLLBACK TRANSACTION
        RAISERROR('Cannot insert records on Friday', 16, 1)
    END
END
```

Audit Trigger:
```sql
CREATE TABLE StudentAudit (
    AuditID INT IDENTITY(1,1) PRIMARY KEY,
    StudentID INT,
    OldName VARCHAR(100),
    NewName VARCHAR(100),
    UpdateDate DATETIME
)

CREATE TRIGGER StudentUpdateAudit
ON Students
AFTER UPDATE
AS
BEGIN
    INSERT INTO StudentAudit 
    (StudentID, OldName, NewName, UpdateDate)
    SELECT 
        i.StudentID, 
        d.Name AS OldName, 
        i.Name AS NewName, 
        GETDATE()
    FROM inserted i
    JOIN deleted d ON i.StudentID = d.StudentID
END
```

## 15. Query Execution Plan
Steps in Query Processing:
1. Parsing
2. Compilation
3. Optimization
4. Execution
5. Result Generation

Viewing Execution Plan:
- Use `EXPLAIN`
- SQL Server Management Studio (Actual Execution Plan)
- Dynamic Management Views (DMVs)

Example:
```sql
-- Enable Actual Execution Plan
SET STATISTICS IO, TIME ON

SELECT * FROM Students 
WHERE Department = 'Computer Science'

-- Analyze execution plan graphically
```

# Database Administrator Interview Guide (Advanced Concepts)

## 16. Stored Procedures (SP)

A stored procedure is a precompiled collection of SQL statements stored in the database as a named object. Think of it as a reusable function that can perform complex database operations.

### Key Properties:
- Improved performance through caching
- Enhanced security through parameterization
- Reduced network traffic
- Modular and maintainable code

### Types of Stored Procedures:
1. **System Stored Procedures**
```sql
-- Example of a system stored procedure
EXEC sp_helpdb  -- Provides information about databases
```

2. **User-Defined Stored Procedures**
```sql
CREATE PROCEDURE GetStudentsByDepartment
    @Department NVARCHAR(50)
AS
BEGIN
    SELECT * FROM Students 
    WHERE Department = @Department
END

-- Execution
EXEC GetStudentsByDepartment @Department = 'Computer Science'
```

3. **Temporary Stored Procedures**
```sql
-- Local temporary procedure
CREATE PROCEDURE #TempStudentReport
AS
BEGIN
    SELECT Department, COUNT(*) as StudentCount
    FROM Students
    GROUP BY Department
END
```

### Return Values
Stored Procedures can return values:
```sql
CREATE PROCEDURE CalculateStudentCount
    @DepartmentID INT,
    @StudentCount INT OUTPUT
AS
BEGIN
    SELECT @StudentCount = COUNT(*) 
    FROM Students 
    WHERE DepartmentID = @DepartmentID
END

-- Calling with output
DECLARE @Total INT
EXEC CalculateStudentCount @DepartmentID = 1, @StudentCount = @Total OUTPUT
PRINT @Total
```

## 17. Views

A view is a virtual table based on the result of a SQL statement, providing a way to simplify complex queries and add a layer of abstraction.

### Types of Views:
1. **Standard View**
```sql
CREATE VIEW ActiveStudents AS
SELECT StudentID, Name, Department
FROM Students
WHERE Status = 'Active'
```

2. **Indexed View (Materialized View)**
```sql
CREATE VIEW StudentSummary
WITH SCHEMABINDING
AS
SELECT 
    Department, 
    COUNT_BIG(*) as TotalStudents,
    AVG(Age) as AverageAge
FROM dbo.Students
GROUP BY Department
```

3. **Partitioned View**
```sql
CREATE VIEW StudentArchive
AS
SELECT * FROM Students2020
UNION ALL
SELECT * FROM Students2021
UNION ALL
SELECT * FROM Students2022
```

### Data Manipulation
```sql
-- Inserting through a view
CREATE VIEW StudentDetails AS
SELECT StudentID, Name, Email
FROM Students
WHERE Department = 'Computer Science'
WITH CHECK OPTION  -- Prevents inserting rows outside the view's filter

-- Insert operation
INSERT INTO StudentDetails (Name, Email)
VALUES ('New Student', 'student@example.com')
```

## 18. MERGE Statement
A powerful SQL command that performs INSERT, UPDATE, or DELETE operations in a single statement.

```sql
MERGE INTO TargetStudents AS target
USING SourceStudents AS source
ON (target.StudentID = source.StudentID)
WHEN MATCHED THEN
    UPDATE SET 
        target.Name = source.Name,
        target.Email = source.Email
WHEN NOT MATCHED THEN
    INSERT (StudentID, Name, Email)
    VALUES (source.StudentID, source.Name, source.Email)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

## 19. ROLLUP, CUBE, and Grouping Sets

### ROLLUP
Generates subtotal rows for hierarchical aggregations:
```sql
SELECT 
    Department, 
    Course, 
    COUNT(*) as StudentCount
FROM Enrollments
GROUP BY ROLLUP(Department, Course)
```

### CUBE
Creates subtotals for all possible combinations:
```sql
SELECT 
    Department, 
    Course, 
    COUNT(*) as StudentCount
FROM Enrollments
GROUP BY CUBE(Department, Course)
```

### Grouping Sets
Allows multiple grouping specifications in a single query:
```sql
SELECT 
    Department, 
    Course, 
    COUNT(*) as StudentCount
FROM Enrollments
GROUP BY GROUPING SETS 
    ((Department, Course), 
     (Department), 
     (Course), 
     ())
```

## 20. PIVOT and UNPIVOT

### PIVOT
Transforms rows into columns:
```sql
SELECT * FROM 
(
    SELECT Department, Semester, EnrollmentCount
    FROM Enrollments
) AS SourceTable
PIVOT
(
    SUM(EnrollmentCount)
    FOR Semester IN ([Spring], [Summer], [Fall])
) AS PivotTable
```

### UNPIVOT
Transforms columns back to rows:
```sql
SELECT Department, Semester, EnrollmentCount
FROM 
(
    SELECT Department, Spring, Summer, Fall
    FROM EnrollmentSummary
) AS SourceTable
UNPIVOT
(
    EnrollmentCount FOR Semester IN (Spring, Summer, Fall)
) AS UnpivotTable
```

# Database Administrator Interview Guide (Advanced Database Concepts)

## 21. Indexes

An index is a database object that improves the speed of data retrieval operations by creating a separate structure that allows faster searching.

### Types of Indexes:

1. **Clustered Index**
- Determines the physical order of data in a table
- Only one per table
- Typically created on the primary key

```sql
CREATE CLUSTERED INDEX IX_Students_StudentID
ON Students (StudentID)
```

2. **Non-Clustered Index**
- Creates a separate structure pointing to data
- Multiple allowed per table
- Useful for frequently queried columns

```sql
CREATE NONCLUSTERED INDEX IX_Students_LastName
ON Students (LastName)
INCLUDE (FirstName, Email)
```

3. **Unique Index**
- Ensures no duplicate values in indexed columns

```sql
CREATE UNIQUE NONCLUSTERED INDEX IX_Students_Email
ON Students (Email)
```

4. **Composite Index**
- Index on multiple columns

```sql
CREATE NONCLUSTERED INDEX IX_Students_NameDepartment
ON Students (LastName, Department)
```

## 22. SQL Profiler and SQL Tuning Advisor

### SQL Profiler
A graphical tool for monitoring and analyzing SQL Server events:
- Captures detailed performance information
- Helps identify performance bottlenecks
- Tracks query execution details

Key Trace Events:
- Performance-related events
- Error events
- Audit events

### SQL Tuning Advisor
Automated tool for:
- Analyzing query performance
- Recommending indexes
- Suggesting query optimizations

## 23. System Databases

1. **Master Database**
- Contains system-level configuration
- Stores server-wide settings
- Tracks all other databases

2. **Model Database**
- Template for creating new databases
- Default schema and settings
- Used when creating new database instances

3. **MSDB Database**
- Stores scheduling and job information
- Manages SQL Server Agent jobs
- Tracks backup and restore history

4. **Tempdb Database**
- Temporary storage database
- Handles:
  - Temporary tables
  - Table variables
  - Query sorting and joining operations
- Recreated each time SQL Server restarts

## 24. Types of Tables

1. **Permanent Tables**
```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name NVARCHAR(100),
    Department NVARCHAR(50)
)
```

2. **Temporary Tables**
- Local Temporary Table
```sql
CREATE TABLE #TempStudents (
    StudentID INT,
    Name NVARCHAR(100)
)
```

- Global Temporary Table
```sql
CREATE TABLE ##GlobalTempStudents (
    StudentID INT,
    Name NVARCHAR(100)
)
```

3. **Memory-Optimized Tables**
```sql
CREATE TABLE MemoryOptimizedStudents (
    StudentID INT PRIMARY KEY NONCLUSTERED,
    Name NVARCHAR(100)
) WITH (MEMORY_OPTIMIZED = ON)
```

## 25. Database Functions

### Types of Functions:
1. **Scalar Functions**
```sql
CREATE FUNCTION dbo.CalculateStudentAge
(
    @BirthDate DATE
)
RETURNS INT
AS
BEGIN
    RETURN DATEDIFF(YEAR, @BirthDate, GETDATE())
END
```

2. **Table-Valued Functions**
```sql
CREATE FUNCTION dbo.GetStudentsByDepartment
(
    @Department NVARCHAR(50)
)
RETURNS TABLE
AS
RETURN
(
    SELECT * FROM Students
    WHERE Department = @Department
)
```

3. **Aggregate Functions**
```sql
-- Built-in aggregate functions
SELECT 
    AVG(Age) AS AverageAge,
    COUNT(*) AS TotalStudents,
    MAX(GPA) AS HighestGPA
FROM Students
```

### Windowing Functions
```sql
-- Rank students within their department
SELECT 
    StudentName, 
    Department, 
    GPA,
    RANK() OVER (PARTITION BY Department ORDER BY GPA DESC) AS DepartmentRank
FROM Students
```
# Database Administrator Interview Guide (Advanced Database Concepts)

## 21. Indexes

An index is a database object that improves the speed of data retrieval operations by creating a separate structure that allows faster searching.

### Types of Indexes:

1. **Clustered Index**
- Determines the physical order of data in a table
- Only one per table
- Typically created on the primary key

```sql
CREATE CLUSTERED INDEX IX_Students_StudentID
ON Students (StudentID)
```

2. **Non-Clustered Index**
- Creates a separate structure pointing to data
- Multiple allowed per table
- Useful for frequently queried columns

```sql
CREATE NONCLUSTERED INDEX IX_Students_LastName
ON Students (LastName)
INCLUDE (FirstName, Email)
```

3. **Unique Index**
- Ensures no duplicate values in indexed columns

```sql
CREATE UNIQUE NONCLUSTERED INDEX IX_Students_Email
ON Students (Email)
```

4. **Composite Index**
- Index on multiple columns

```sql
CREATE NONCLUSTERED INDEX IX_Students_NameDepartment
ON Students (LastName, Department)
```

## 22. SQL Profiler and SQL Tuning Advisor

### SQL Profiler
A graphical tool for monitoring and analyzing SQL Server events:
- Captures detailed performance information
- Helps identify performance bottlenecks
- Tracks query execution details

Key Trace Events:
- Performance-related events
- Error events
- Audit events

### SQL Tuning Advisor
Automated tool for:
- Analyzing query performance
- Recommending indexes
- Suggesting query optimizations

## 23. System Databases

1. **Master Database**
- Contains system-level configuration
- Stores server-wide settings
- Tracks all other databases

2. **Model Database**
- Template for creating new databases
- Default schema and settings
- Used when creating new database instances

3. **MSDB Database**
- Stores scheduling and job information
- Manages SQL Server Agent jobs
- Tracks backup and restore history

4. **Tempdb Database**
- Temporary storage database
- Handles:
  - Temporary tables
  - Table variables
  - Query sorting and joining operations
- Recreated each time SQL Server restarts

## 24. Types of Tables

1. **Permanent Tables**
```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name NVARCHAR(100),
    Department NVARCHAR(50)
)
```

2. **Temporary Tables**
- Local Temporary Table
```sql
CREATE TABLE #TempStudents (
    StudentID INT,
    Name NVARCHAR(100)
)
```

- Global Temporary Table
```sql
CREATE TABLE ##GlobalTempStudents (
    StudentID INT,
    Name NVARCHAR(100)
)
```

3. **Memory-Optimized Tables**
```sql
CREATE TABLE MemoryOptimizedStudents (
    StudentID INT PRIMARY KEY NONCLUSTERED,
    Name NVARCHAR(100)
) WITH (MEMORY_OPTIMIZED = ON)
```

## 25. Database Functions

### Types of Functions:
1. **Scalar Functions**
```sql
CREATE FUNCTION dbo.CalculateStudentAge
(
    @BirthDate DATE
)
RETURNS INT
AS
BEGIN
    RETURN DATEDIFF(YEAR, @BirthDate, GETDATE())
END
```

2. **Table-Valued Functions**
```sql
CREATE FUNCTION dbo.GetStudentsByDepartment
(
    @Department NVARCHAR(50)
)
RETURNS TABLE
AS
RETURN
(
    SELECT * FROM Students
    WHERE Department = @Department
)
```

3. **Aggregate Functions**
```sql
-- Built-in aggregate functions
SELECT 
    AVG(Age) AS AverageAge,
    COUNT(*) AS TotalStudents,
    MAX(GPA) AS HighestGPA
FROM Students
```

### Windowing Functions
```sql
-- Rank students within their department
SELECT 
    StudentName, 
    Department, 
    GPA,
    RANK() OVER (PARTITION BY Department ORDER BY GPA DESC) AS DepartmentRank
FROM Students
```

# Database Administrator Interview Guide (Continued)

## 26. Variables and Dynamic Query

### Types of Variables

1. **Local Variables**
```sql
-- Declare and use local variables
DECLARE @StudentName NVARCHAR(100)
DECLARE @StudentAge INT

SET @StudentName = 'John Doe'
SET @StudentAge = 22

SELECT @StudentName AS Name, @StudentAge AS Age
```

2. **Table Variables**
```sql
DECLARE @StudentTemp TABLE (
    StudentID INT,
    Name NVARCHAR(100),
    Department NVARCHAR(50)
)

INSERT INTO @StudentTemp
SELECT StudentID, Name, Department
FROM Students
WHERE Department = 'Computer Science'
```

3. **Global Variables**
```sql
-- System global variables
SELECT @@SERVERNAME AS ServerName
SELECT @@VERSION AS SQLVersion
```

### Dynamic SQL Query
```sql
-- Dynamic query example
DECLARE @DynamicSQL NVARCHAR(MAX)
DECLARE @Department NVARCHAR(50) = 'Computer Science'

SET @DynamicSQL = 
    N'SELECT * FROM Students 
     WHERE Department = @Dept'

EXEC sp_executesql 
    @DynamicSQL, 
    N'@Dept NVARCHAR(50)', 
    @Dept = @Department
```

## 27. Control of Flow Statements

### Conditional Statements
```sql
-- IF-ELSE example
IF EXISTS (SELECT 1 FROM Students WHERE GPA > 3.5)
BEGIN
    PRINT 'High-performing students exist'
    
    -- Nested conditions
    IF (SELECT COUNT(*) FROM Students) > 1000
    BEGIN
        PRINT 'Large student population'
    END
    ELSE
    BEGIN
        PRINT 'Small student population'
    END
END
ELSE
BEGIN
    PRINT 'No high-performing students found'
END
```

### WHILE Loop
```sql
DECLARE @Counter INT = 1

WHILE @Counter <= 10
BEGIN
    INSERT INTO StudentLog (LogEntry)
    VALUES ('Processing batch ' + CAST(@Counter AS NVARCHAR(10)))
    
    SET @Counter = @Counter + 1
    
    -- Break condition
    IF @Counter > 5
        BREAK
END
```

## 28. Database Integrity

Database integrity ensures the accuracy, consistency, and reliability of data through:

1. **Entity Integrity**
- Ensures each row is uniquely identifiable
- Implemented through PRIMARY KEY constraints

2. **Referential Integrity**
- Maintains relationships between tables
- Implemented through FOREIGN KEY constraints

```sql
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName NVARCHAR(100)
)

CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name NVARCHAR(100),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
)
```

## 29. Cascade Options

### ON UPDATE CASCADE
```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) 
    REFERENCES Departments(DepartmentID)
    ON UPDATE CASCADE
)
```

### ON DELETE SET NULL
```sql
CREATE TABLE Enrollments (
    EnrollmentID INT PRIMARY KEY,
    StudentID INT,
    CourseID INT,
    FOREIGN KEY (StudentID) 
    REFERENCES Students(StudentID)
    ON DELETE SET NULL
)
```

## 30. Key Constraints Comparison

### Primary Key (PK)
- Unique identifier for a table
- Cannot contain NULL values
- Only one per table

### Foreign Key (FK)
- References primary key in another table
- Establishes relationships between tables
- Can contain multiple foreign keys

### Unique Constraint
- Ensures unique values
- Can contain NULL values
- Multiple unique constraints allowed

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,  -- Primary Key
    Email NVARCHAR(100) UNIQUE, -- Unique Constraint
    DepartmentID INT,           -- Foreign Key
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
)
```

# Database Administrator Interview Guide (Advanced Concepts)

## 31. Rules and Defaults

### Rules
Custom validation constraints for column values:

```sql
-- Create a rule to limit student age
CREATE RULE AgeValidationRule
AS @Age BETWEEN 16 AND 100

-- Bind the rule to a column
EXEC sp_bindrule 'AgeValidationRule', 'Students.Age'
```

### Defaults
Provide default values when no explicit value is specified:

```sql
-- Create a default value
CREATE DEFAULT StudentStatusDefault
AS 'Active'

-- Bind default to a column
EXEC sp_bindefault 'StudentStatusDefault', 'Students.Status'
```

## 32. Constraints vs. Rules

### Constraints
- Enforced at the database level
- Part of table definition
- Immediate validation
- Preferred modern approach

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Age INT CHECK (Age BETWEEN 16 AND 100),
    Email NVARCHAR(100) UNIQUE,
    RegistrationDate DATE DEFAULT GETDATE()
)
```

### Rules
- Less flexible
- Can be applied after table creation
- Limited to specific data types
- Gradually being replaced by constraints

## 33. Creating Custom Data Types

```sql
-- Create a custom data type
CREATE TYPE EmailAddress 
FROM NVARCHAR(100) NOT NULL

-- Use in table creation
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    Email EmailAddress
)
```

## 34. Database Schema

A schema is a logical container for database objects:

```sql
-- Create a new schema
CREATE SCHEMA Academic

-- Create table within schema
CREATE TABLE Academic.Students (
    StudentID INT PRIMARY KEY,
    Name NVARCHAR(100)
)

-- Create user with schema permissions
CREATE USER DatabaseAdministrator 
WITH PASSWORD = 'SecurePassword123!'

-- Grant schema permissions
GRANT SELECT, INSERT, UPDATE ON SCHEMA::Academic TO DatabaseAdministrator
```

## 35. User Creation and Permissions

```sql
-- Create a new database user
CREATE LOGIN DatabaseManager 
WITH PASSWORD = 'StrongPassword456!'

-- Create user in specific database
USE UniversityDB
CREATE USER DatabaseManager 
FROM LOGIN DatabaseManager

-- Grant specific database permissions
GRANT SELECT, INSERT ON Students TO DatabaseManager
GRANT EXECUTE ON SCHEMA::dbo TO DatabaseManager
```

## 36. Data Manipulation Comparison

### DELETE
- Removes specific rows
- Can be rolled back
- Triggers are fired
- Logs are generated

```sql
DELETE FROM Students WHERE StudentID = 100
```

### TRUNCATE
- Removes all rows
- Cannot be rolled back
- No triggers
- Faster than DELETE
- Resets identity column

```sql
TRUNCATE TABLE Students
```

### DROP
- Removes entire table structure
- Deletes all data
- Removes indexes, constraints
- Cannot be easily recovered

```sql
DROP TABLE Students
```

## 37. Ranking Functions

```sql
-- Demonstrate ranking functions
SELECT 
    StudentName,
    Department,
    GPA,
    RANK() OVER (PARTITION BY Department ORDER BY GPA DESC) AS DepartmentRank,
    DENSE_RANK() OVER (ORDER BY GPA DESC) AS OverallDenseRank,
    ROW_NUMBER() OVER (ORDER BY GPA DESC) AS RowNumber,
    NTILE(4) OVER (ORDER BY GPA DESC) AS GPAQuartile
FROM Students
```

## 38. Date Types and Conversions

### Date Types
- DATE: Store date only
- TIME: Store time only
- DATETIME: Date and time
- DATETIME2: More precise date and time
- DATETIMEOFFSET: Includes time zone

```sql
-- Date conversion examples
SELECT 
    CAST('2023-12-31' AS DATE) AS CastedDate,
    CONVERT(VARCHAR, GETDATE(), 101) AS FormattedDate,
    FORMAT(GETDATE(), 'yyyy-MM-dd') AS FormattedDateCustom
```

### Conversion Functions
- CAST: Standard SQL conversion
- CONVERT: SQL Server-specific with more formatting options
- FORMAT: Most flexible but slower

# Database Administrator Interview Guide (Advanced Theoretical Concepts)

## 39. Authentication vs. Authorization

### Authentication
Authentication is the process of verifying the identity of a user attempting to access a database system. It answers the fundamental question: "Who are you?"

Authentication Methods:
1. **Windows Authentication (Integrated Security)**
```sql
-- Connection string using Windows Authentication
Server=myServerAddress;Database=myDataBase;Trusted_Connection=True;
```

2. **SQL Server Authentication**
```sql
-- Connection using SQL Server login
Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;
```

### Authorization
Authorization determines what actions an authenticated user is allowed to perform. It answers the question: "What are you allowed to do?"

Authorization Levels:
```sql
-- Grant SELECT permission
GRANT SELECT ON Students TO DatabaseUser

-- Deny DELETE permission
DENY DELETE ON Students TO DatabaseUser

-- Revoke UPDATE permission
REVOKE UPDATE ON Students FROM DatabaseUser
```

## 40. GUID (Globally Unique Identifier)

A GUID is a 128-bit unique identifier used to ensure global uniqueness across systems and databases.

```sql
-- Create a table with GUID primary key
CREATE TABLE Students (
    StudentID UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Name NVARCHAR(100),
    Email NVARCHAR(100)
)

-- Generate a new GUID
DECLARE @NewGUID UNIQUEIDENTIFIER = NEWID()
```

## 41. Random Data Selection

```sql
-- Select random student
SELECT TOP 1 *
FROM Students
ORDER BY NEWID()

-- Alternative approach using TABLESAMPLE
SELECT * 
FROM Students TABLESAMPLE (10 ROWS)
```

## 42. SQL Execution Order

The logical order of SQL query execution:
1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY
7. LIMIT/TOP

Example demonstrating execution order:
```sql
SELECT 
    Department, 
    AVG(Salary) AS AverageSalary
FROM Employees
WHERE Employment = 'Full-Time'
GROUP BY Department
HAVING AVG(Salary) > 50000
ORDER BY AverageSalary DESC
```

## 43. Set Operations

### UNION vs. UNION ALL vs. INTERSECT vs. EXCEPT

```sql
-- UNION (removes duplicates)
SELECT StudentName FROM UndergraduateStudents
UNION
SELECT StudentName FROM GraduateStudents

-- UNION ALL (keeps all rows)
SELECT StudentName FROM UndergraduateStudents
UNION ALL
SELECT StudentName FROM GraduateStudents

-- INTERSECT (common rows)
SELECT StudentName FROM UndergraduateStudents
INTERSECT
SELECT StudentName FROM HonorRollStudents

-- EXCEPT (rows in first query not in second)
SELECT StudentName FROM AllStudents
EXCEPT
SELECT StudentName FROM GraduateStudents
```

## 44. Create Empty Table Copy

```sql
-- Create an empty copy of a table in HR schema
SELECT *
INTO HR.StudentsCopy
FROM Students
WHERE 1 = 0  -- False condition ensures no data is copied
```

## 45. Enhanced Entity-Relationship Diagram (EERD)

An advanced version of ERD that includes:
- Inheritance relationships
- Specialization
- Generalization
- More detailed attribute representation

Conceptual Example:
```
Person (Abstract Entity)
|
+-- Student (Specialized Entity)
|   - StudentID
|   - EnrollmentDate
|
+-- Faculty (Specialized Entity)
    - EmployeeID
    - Department
```

## 46. Specialization and Inheritance

### Types of Inheritance
1. **Total Specialization**
   - All entities in the parent class must be in a subclass
   
2. **Partial Specialization**
   - Not all parent entities must be in a subclass

### Disjoint vs. Overlapping
- **Disjoint**: An entity can belong to only one subclass
- **Overlapping**: An entity can belong to multiple subclasses

```sql
-- Modeling inheritance in database
CREATE TABLE Person (
    PersonID INT PRIMARY KEY,
    Name NVARCHAR(100),
    Email NVARCHAR(100)
)

CREATE TABLE Student (
    StudentID INT PRIMARY KEY,
    PersonID INT,
    Major NVARCHAR(50),
    FOREIGN KEY (PersonID) REFERENCES Person(PersonID)
)

CREATE TABLE Faculty (
    FacultyID INT PRIMARY KEY,
    PersonID INT,
    Department NVARCHAR(50),
    FOREIGN KEY (PersonID) REFERENCES Person(PersonID)
)
```
# Database Administrator Interview Guide (Advanced Theoretical Concepts)

## 47. Aggregate Functions and Subqueries

### Aggregate Functions
Aggregate functions perform calculations across a set of rows, providing summarized information about data.

```sql
-- Comprehensive aggregation example
SELECT 
    Department,
    COUNT(*) AS TotalStudents,
    AVG(GPA) AS AverageGPA,
    MAX(Age) AS OldestStudent,
    MIN(Age) AS YoungestStudent,
    SUM(Scholarship) AS TotalScholarshipAmount
FROM Students
GROUP BY Department
```

### Subqueries
Subqueries are nested queries that provide data to the main query, allowing complex data retrieval and filtering.

Types of Subqueries:
1. **Scalar Subquery** (Returns single value)
```sql
-- Find students with GPA higher than department average
SELECT StudentName, GPA
FROM Students s1
WHERE GPA > (
    SELECT AVG(GPA) 
    FROM Students s2 
    WHERE s2.Department = s1.Department
)
```

2. **Correlated Subquery** (Depends on outer query)
```sql
-- Find top-performing students in each department
SELECT StudentName, Department, GPA
FROM Students s1
WHERE GPA = (
    SELECT MAX(GPA) 
    FROM Students s2 
    WHERE s2.Department = s1.Department
)
```

## 48. Normalization

Normalization is a systematic approach to decomposing tables to eliminate data redundancy and improve data integrity.

### Normalization Levels (Normal Forms):

1. **First Normal Form (1NF)**
- Eliminate repeating groups
- Identify unique column(s)
- Each column contains atomic values

```sql
-- Before 1NF
StudentID | Courses
1        | Math,Physics,Chemistry

-- After 1NF
StudentID | Course
1        | Math
1        | Physics
1        | Chemistry
```

2. **Second Normal Form (2NF)**
- Must be in 1NF
- Remove partial dependencies
- All non-key columns fully depend on primary key

3. **Third Normal Form (3NF)**
- Must be in 2NF
- Remove transitive dependencies
- No non-key column depends on another non-key column

## 49. Functional Dependency

Functional dependency describes the relationship between attributes in a relation.

Example:
- StudentID → (Name, Email, Department)
- This means StudentID uniquely determines Name, Email, and Department

### Types of Functional Dependencies:
- **Full Functional Dependency**
- **Partial Functional Dependency**
- **Transitive Functional Dependency**

## 50. DML Anomalies

Anomalies occur when data is inserted, updated, or deleted, causing data inconsistencies.

Types of Anomalies:
1. **Insertion Anomaly**
2. **Update Anomaly**
3. **Deletion Anomaly**

```sql
-- Example showing potential anomaly
CREATE TABLE CourseEnrollment (
    InstructorName VARCHAR(100),
    InstructorDepartment VARCHAR(50),
    CourseCode VARCHAR(10),
    CourseName VARCHAR(100)
)

-- Insertion Anomaly: Cannot add a new course without an instructor
-- Update Anomaly: Updating instructor name requires multiple row updates
-- Deletion Anomaly: Deleting last enrollment loses course information
```

## 51. Types of Joins

```sql
-- Inner Join (Returns matching records)
SELECT s.StudentName, c.CourseName
FROM Students s
INNER JOIN Enrollments e ON s.StudentID = e.StudentID
INNER JOIN Courses c ON e.CourseID = c.CourseID

-- Left Join (All records from left table)
SELECT s.StudentName, c.CourseName
FROM Students s
LEFT JOIN Enrollments e ON s.StudentID = e.StudentID

-- Right Join (All records from right table)
SELECT s.StudentName, c.CourseName
FROM Students s
RIGHT JOIN Enrollments e ON s.StudentID = e.StudentID

-- Full Outer Join (All records from both tables)
SELECT s.StudentName, c.CourseName
FROM Students s
FULL OUTER JOIN Enrollments e ON s.StudentID = e.StudentID

-- Cross Join (Cartesian Product)
SELECT s.StudentName, c.CourseName
FROM Students s
CROSS JOIN Courses c
```

## 52. Database Mapping

Database mapping is the process of defining relationships between different database objects and schemas.

Mapping Steps:
1. Identify Entities
2. Define Attributes
3. Establish Relationships
4. Normalize Structure
5. Create Physical Schema

## 53. SQL Command Categories

1. **DML (Data Manipulation Language)**
   - SELECT
   - INSERT
   - UPDATE
   - DELETE

2. **DDL (Data Definition Language)**
   - CREATE
   - ALTER
   - DROP
   - TRUNCATE

3. **DCL (Data Control Language)**
   - GRANT
   - REVOKE

4. **TCL (Transaction Control Language)**
   - COMMIT
   - ROLLBACK
   - SAVEPOINT

Would you like me to continue with the final topics in the database administrator interview guide?
