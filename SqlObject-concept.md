SqlObject concept
=================

Allows creating any SQL object as a part of generated application.
**SqlObject** is intended to be used as a workaround when existing DSL concepts (**Unique**, **SqlIndex**, **SqlView**, **SqlTrigger**, etc) are not sufficient.

Implemented in Rhetos package: *CommonConcepts*.

Features
--------

* Set custom SQL script to create the object in the database.
* Set custom SQL script to remove the object from the database.
* Define creation order of dependent SQL object.
* Create SQL objects without transaction (full-text search index, e.g.). 

Dependencies and creation order of SQL objects
----------------------------------------------

Use any of the **SqlDependsOn...** concepts in an **SqlObject** to define other objects (entites, views, etc) that need to be created before the **SqlObject**.

If another object (**SqlQueryable**, for example) depends on the **SqlObject**, use **SqlDependsOnSqlObject** to make sure that the **SqlObject** is created in database before the view from the **SqlQueryable**.

    Module Demo
    {
        Entity SomeEntity { Integer I; }

        SqlObject SomeView
            'CREATE VIEW Demo.V1 AS SELECT ID, I+1 AS I1 FROM Demo.SomeEntity'
            'DROP VIEW Demo.V1'
        {
            SqlDependsOn Demo.SomeEntity;
        }

        SqlQueryable SomeEntityAdditionalInfo
            'SELECT ID, I1+1 AS I2 FROM Demo.V1'
        {
            Extends Demo.SomeEntity;
            Integer I2;
            SqlDependsOnSqlObject Demo.SomeView;
        }
    }

Creating SQL objects without transaction
----------------------------------------

Some SQL object cannot be created in a trasaction (**full-text search index** on SQL Server, e.g.).

By default, Rhetos will execute all SQL scripts in a transaction, unless the script starts with comment `/*DatabaseGenerator:NoTransaction*/`.

This example (from Rhetos unit tests) creates an SQL view, adding the transaction level (`@@TRANCOUNT`) as the suffix to the view name, to prove that the create script is executed without a transaction. 

    Module TestSqlWorkarounds
    {
        SqlObject WithoutTransaction
        
            "/*DatabaseGenerator:NoTransaction*/
                DECLARE @createView nvarchar(max);
                SET @createView = 'CREATE VIEW TestSqlWorkarounds.WithoutTransaction_' + CONVERT(NVARCHAR(max), @@TRANCOUNT) + ' AS SELECT a=1';
                EXEC (@createView);"
                
            "/*DatabaseGenerator:NoTransaction*/
                DECLARE @dropView nvarchar(max);
                SELECT @dropView = name FROM sys.objects o WHERE type = 'V' AND SCHEMA_NAME(schema_id) = 'TestSqlWorkarounds' AND name LIKE 'WithoutTransaction[_]%';
                SET @dropView = 'DROP VIEW TestSqlWorkarounds.' + @dropView;
                EXEC (@dropView);";
	}
