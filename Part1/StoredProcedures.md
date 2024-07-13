# Stored Procedures in SQL Server

Stored procedures are precompiled SQL queries that are stored in the database. They offer several advantages:

- **Improved Performance**: Stored procedures are compiled and optimized, which can improve execution speed compared to ad-hoc SQL queries.
- **Security**: Permissions can be granted on stored procedures, allowing fine-grained control over database access.
- **Encapsulation**: They encapsulate logic and business rules in the database, promoting code reuse and maintainability.
- **Reduced Network Traffic**: Only the procedure call is transmitted over the network, not the entire query.
- **Version Control**: Changes to stored procedures can be managed centrally within the database.

## CheckDatabaseIntegrity Stored Procedure

This stored procedure checks the integrity of a specified database using the `DBCC CHECKDB` command and captures detailed output.

### Stored Procedure Script

```sql
-- =============================================
-- Author:      [Ahmad Abbas]
-- Create date: [12/7/2024]
-- Description: This stored procedure checks the integrity of a specified database
--              using the DBCC CHECKDB command and captures detailed output.
-- =============================================
CREATE PROCEDURE CheckDatabaseIntegrity
    @DatabaseName NVARCHAR(128)
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @SQL NVARCHAR(MAX);

    -- Construct the DBCC CHECKDB command
    SET @SQL = 'DBCC CHECKDB(' + QUOTENAME(@DatabaseName) + ') WITH ALL_ERRORMSGS';

    -- Execute the DBCC CHECKDB command and capture the output
    BEGIN TRY
        -- Create a temporary table to capture the output
        CREATE TABLE #DBCCOutput (
            ID INT IDENTITY(1,1),
            [Message] NVARCHAR(MAX)
        );

        -- Insert DBCC CHECKDB output into the temporary table
        INSERT INTO #DBCCOutput
        EXEC sp_executesql @SQL;

        -- Print the output messages
        DECLARE @Message NVARCHAR(MAX);
        DECLARE @PrintCursor CURSOR;
        SET @PrintCursor = CURSOR FOR
            SELECT [Message]
            FROM #DBCCOutput
            ORDER BY ID;

        OPEN @PrintCursor;
        FETCH NEXT FROM @PrintCursor INTO @Message;

        WHILE @@FETCH_STATUS = 0
        BEGIN
            PRINT @Message;
            FETCH NEXT FROM @PrintCursor INTO @Message;
        END;

        CLOSE @PrintCursor;
        DEALLOCATE @PrintCursor;

        DROP TABLE #DBCCOutput;

        PRINT 'Integrity check completed successfully for database: ' + @DatabaseName;
    END TRY
    BEGIN CATCH
        -- Handle any errors
        DECLARE @ErrorMessage NVARCHAR(4000);
        DECLARE @ErrorSeverity INT;
        DECLARE @ErrorState INT;

        -- Retrieve the error information
        SELECT 
            @ErrorMessage = ERROR_MESSAGE(),
            @ErrorSeverity = ERROR_SEVERITY(),
            @ErrorState = ERROR_STATE();

        -- Print the error message
        PRINT 'An error occurred during the integrity check for database: ' + @DatabaseName;
        PRINT 'Error: ' + @ErrorMessage;

        -- Optionally, you can rethrow the error to be handled by the calling program
        RAISERROR (@ErrorMessage, @ErrorSeverity, @ErrorState);
    END CATCH;
END;

--To execute the stored procedure
EXEC CheckDatabaseIntegrity @DatabaseName = 'your_database_name';
