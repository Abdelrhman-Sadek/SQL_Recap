# SQL_Recap

# Comprehensive Guide to Database Administrator Interview Questions

## Overview

This document provides detailed answers and examples to common database administrator interview questions, ensuring clarity, completeness, and practical application.

---

### 1. What is the difference among Batch, Script, and Transaction?

- **Batch:** A collection of SQL statements executed as a single unit, sent to the database together.

  - *Example:* Multiple `INSERT` statements written in a single execution context.
    ```sql
    INSERT INTO Students (Name, Age) VALUES ('Alice', 20);
    INSERT INTO Students (Name, Age) VALUES ('Bob', 22);
    ```

- **Script:** A set of SQL commands stored in a file or editor, which can include batches and logic for execution.

  - *Example:* A `.sql` file containing:
    ```sql
    -- Sample script
    BEGIN TRANSACTION;
    INSERT INTO Students (Name, Age) VALUES ('Alice', 20);
    ROLLBACK;
    ```

- **Transaction:** A sequence of operations performed as a single unit of work, ensuring **ACID** properties (Atomicity, Consistency, Isolation, Durability).

  - *Example:*
    ```sql
    BEGIN TRANSACTION;
    INSERT INTO Accounts (AccountID, Balance) VALUES (101, 500);
    UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 101;
    COMMIT;
    ```

---

### 2. What are ACID properties?

- **Atomicity:** Ensures all operations in a transaction are completed; otherwise, none are applied.
- **Consistency:** Guarantees that the database remains in a valid state before and after the transaction.
- **Isolation:** Ensures that concurrent transactions do not interfere with each other.
- **Durability:** Once a transaction is committed, its changes are permanent, even in case of system failure.

---

### 3. What is a Cursor?

- A cursor is a database object used to retrieve, manipulate, and navigate through result sets row by row.
- *Example:*
  ```sql
  DECLARE my_cursor CURSOR FOR
  SELECT Name, Age FROM Students;

  OPEN my_cursor;
  FETCH NEXT FROM my_cursor INTO @Name, @Age;

  CLOSE my_cursor;
  DEALLOCATE my_cursor;
  ```

---

### 4. Write a query to display the names of students in one cell.

- Use the `STRING_AGG` function (SQL Server) or `GROUP_CONCAT` (MySQL).
  - *SQL Server Example:*
    ```sql
    SELECT STRING_AGG(Name, ', ') AS StudentNames FROM Students;
    ```
  - *MySQL Example:*
    ```sql
    SELECT GROUP_CONCAT(Name SEPARATOR ', ') AS StudentNames FROM Students;
    ```

---

### 5. What are the types of DB Backup?

- **Full Backup:** Copies the entire database.
- **Differential Backup:** Backs up data changed since the last full backup.
- **Transaction Log Backup:** Captures changes in the transaction log since the last log backup.
- **Incremental Backup:** Copies changes since the last backup of any type.
- **Copy-Only Backup:** A standalone backup that doesn’t affect the existing backup sequence.

---

### 6. Can we write values on identity columns?

- Yes, using the `SET IDENTITY_INSERT` command.
  ```sql
  SET IDENTITY_INSERT Students ON;
  INSERT INTO Students (ID, Name, Age) VALUES (1, 'Alice', 20);
  SET IDENTITY_INSERT Students OFF;
  ```

---

### 7. How to reset identity?

- Use `DBCC CHECKIDENT` to reset the seed value.
  ```sql
  DBCC CHECKIDENT ('Students', RESEED, 0);
  ```

---

### 8. What is a DB job?

- A DB job is a scheduled task in the database (using tools like SQL Server Agent) that automates administrative tasks like backups or indexing.

---

### 9. What are the types of Insert statements?

1. Insert single row.
   ```sql
   INSERT INTO Students (Name, Age) VALUES ('Alice', 20);
   ```
2. Insert multiple rows.
   ```sql
   INSERT INTO Students (Name, Age) VALUES ('Alice', 20), ('Bob', 22);
   ```
3. Insert using `SELECT`.
   ```sql
   INSERT INTO Students (Name, Age)
   SELECT Name, Age FROM OldStudents;
   ```
4. Insert with default values.
   ```sql
   INSERT INTO Students DEFAULT VALUES;
   ```
5. Bulk insert.
   ```sql
   BULK INSERT Students FROM 'file_path.csv'
   WITH (FIELDTERMINATOR = ',', ROWTERMINATOR = '\n');
   ```

---

### 10. What is a Snapshot?

- A snapshot is a read-only, static view of a database at a specific point in time. Used in SQL Server for reporting and testing.
  ```sql
  CREATE DATABASE SnapshotDB ON
  (NAME = MainDB, FILENAME = 'C:\MainDB_Snapshot.ss')
  AS SNAPSHOT OF MainDB;
  ```

---

### 11. What is SQLCLR?

- SQLCLR (SQL Common Language Runtime) enables execution of .NET code (like C#) within SQL Server.
- *Example:*
  ```sql
  CREATE ASSEMBLY HelloWorld
  FROM 'path_to_dll';
  ```

---

### 12. What is SQL Injection?

- SQL Injection is a security vulnerability where malicious SQL is executed via user input.
  - *Example of bad code:*
    ```sql
    SELECT * FROM Users WHERE UserID = '" + input + "';
    ```
  - *Solution: Use parameterized queries.*
    ```sql
    SELECT * FROM Users WHERE UserID = @UserID;
    ```

---

### 13. What is Red-Gate?

- A suite of tools for database management, monitoring, and deployment, often used with SQL Server.

---

### 14. What is Data Quality Service (DQS)?

- DQS is a SQL Server feature for improving data quality through cleansing, matching, and profiling.

---

### 15. What do you know about XML?

- XML (eXtensible Markup Language) is used to store and transport data. SQL Server supports XML data types and methods like `nodes()` and `value()`.
  ```sql
  SELECT XMLColumn.value('(/Root/Name)[1]', 'VARCHAR(50)') AS Name
  FROM Table;
  ```

---

### 16. What is a Trigger?

- A trigger is a special nd of stored procedure that is automatically executed in response to certain events on a table or view. Triggers are often used for enforcing business rules, auditing changes, or automatically updating data in related tables.
- *Example of a trigger to log deletions:*
  ```sql
  CREATE TRIGGER trgAfterDelete
  ON Students
  AFTER DELETE
  AS
  BEGIN
    INSERT INTO AuditLog (Action, ActionDate)
    VALUES ('Delete', GETDATE());
  END;
  ```

---

### 17. Create a trigger to prevent students from inserting data on Fridays.

```sql
CREATE TRIGGER trgPreventFridayInsert
ON Students
INSTEAD OF INSERT
AS
BEGIN
  IF (DATENAME(WEEKDAY, GETDATE()) = 'Friday')
  BEGIN
    RAISERROR ('Inserts are not allowed on Fridays', 16, 1);
    ROLLBACK;
  END
  ELSE
  BEGIN
    INSERT INTO Students (Name, Age)
    SELECT Name, Age FROM inserted;
  END
END;
```

---

### 18. Create a trigger to save any updates on the student table into an audit log.- What is Stored procedure and it’s properties

```sql
CREATE TRIGGER trgAuditUpdates
ON Students
AFTER UPDATE
AS
BEGIN
  INSERT INTO AuditLog (StudentID, OldName, NewName, ActionDate)
  SELECT d.ID, d.Name AS OldName, i.Name AS NewName, GETDATE()
  FROM deleted d
  JOIN inserted i ON d.ID = i.ID;
END;
```

---

