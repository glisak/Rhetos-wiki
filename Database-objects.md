These concepts are used as a workaround for features that cannot be implemented with the available high-level concepts in Rhetos.

Use these concepts to avoid creating the database objects manually, bypassing Rhetos. Any database objects created without Rhetos in a schema that is controlled by Rhetos could hinder future upgrades of the application.

The SQL script that creates a database object can be written in-line (inside the .rhe script) or written in a separate file (see SqlProcedure example below).

* **SqlProcedure** *&lt;Module&gt;.&lt;Name&gt; &lt;ProcedureArguments&gt; &lt;ProcedureSource&gt;*

    ``` sql
    SqlProcedure GenerateNextAutoCode <GenerateNextAutoCode param.sql> <GenerateNextAutoCode body.sql>;
    ```

    - File "GenerateNextAutoCode param.sql"

        ``` sql
        @TableOrView NVARCHAR(256),
        @ColumnName NVARCHAR(256),
        ...
        ```

    - File "GenerateNextAutoCode body.sql"

        ``` sql
        BEGIN
            ...
        END
        ```

* **SqlTrigger** *&lt;DataStructure&gt;.&lt;Name&gt; &lt;Events&gt; &lt;TriggerSource&gt;*

    ``` sql
    SqlTrigger MailOutbox.TEMP_ToSend "AFTER INSERT"
        "SET NOCOUNT ON;
        INSERT INTO Demo.MailOutboxToSend(ID)
        SELECT
            i.ID
        FROM
            INSERTED i
        WHERE
            i.ToAddress IS NOT NULL
            OR i.ToAddress <> *";
    ```

* **SqlFunction** *&lt;Module&gt;.&lt;Name&gt; &lt;Arguments&gt; &lt;RETURNS ... AS ... FunctionSource&gt;*

    ``` sql
    SqlFunction udf_PeriodOverlap "@A_From datetime, @A_To datetime, @B_From datetime, @B_To datetime"
        "RETURNS TABLE
        AS
        RETURN
        SELECT
        ...
        FROM
        ...";
    ```

* **SqlView** *&lt;Module&gt;.&lt;Name&gt; &lt;ViewSource&gt;*

    ``` sql
    SqlView ActiveEmployees <SQL\ActiveEmployees.sql>;
    ```

    - File "ActiveEmployees.sql" in the "SQL" subfolder

        ``` sql
        SELECT
            ...
        FROM
            ...
        ```

* **SqlIndex** (with **Clustered**)

    ```
    Entity TTPayersDocumentIdentification
    {
        Reference TTPayer{ Required; }
        Bool IsPowerOfAttorney;
        SqlIndex TTPayer { Clustered; }
    }
    
    Entity Log
    {
        ...
        Guid ItemId { SqlIndex; }
    }
    ```

* **SqlIndexMultiple** (with **Clustered**)

    ```
    Entity RightToArea
    {
        Reference Principal { Detail; }
        Reference County;
        Reference CityMunicipality;
        Reference Settlement;
        SqlIndexMultiple 'Principal Settlement CityMunicipality County' { Clustered; }
    }
    ```

* **SqlDefault** -
  Creates a default constraint on the specified column in the table. This concepts intended for internal system use.
  It cannot be used for setting the default value when saving an entity through the Rhetos server,
  because the server always sends the NULL value for property without set values.

    ``` sql
    Entity Log
    {
        DateTime Created { SqlDefault <Log.Created default.sql>; Required; }
        ...
    }
    ```

    - File "Log.Created default.sql"

        ``` sql
        getdate()
        ```

* **SqlObject** *&lt;Module&gt;.&lt;Name&gt; &lt;CreateSQL&gt; &lt;RemoveSQL&gt;* -
  Use this to create other database objects that are not supported by the other concepts, for example: full-text search index.
  See the full documentation at [SqlObject concept](https://github.com/Rhetos/Rhetos/wiki/SqlObject-concept).

    ``` sql
    Module Demo
    {
        Entity Pivot10000
        {
            Integer Num;
        }
        
        SqlObject IX_Pivot10000
            "CREATE UNIQUE NONCLUSTERED INDEX IX_Pivot10000 ON Demo.Pivot10000 (Num)"
            "DROP INDEX Demo.Pivot10000.IX_Pivot10000"
        {
            SqlDependsOn Demo.Pivot10000;
        }
    }
    ```

## Dependencies between database objects

To assure correct order when creating database objects, Rhetos must be informed on the dependencies between the objects.

Automatic dependency detection is not perfect but works well in practice:

* **AutodetectSqlDependencies** &lt;Module&gt;
    - Automatically detects and generates dependencies (SqlDependsOn) for SqlQueryable, SqlView, SqlFunction, SqlProcedure, SqlTrigger and LegacyEntity view. It may be applied to any of those objects or to a whole module.
    - Acts globally for all objects in the module.

    ```
    Module Demo
    {
        AutoDetectSqlDependencies;
        ...

    ```

* **AutodetectSqlDependencies** &lt;Sql*&gt;
    - Acts locally for an SQL object.

    ``` sql
        SqlView HelloView 'SELECT * FROM Demo.Hello'
        {
            AutoDetectSqlDependencies;
        }
    ```

* Other concepts for defining dependencies are rarely used (when **AutoDetectSqlDependencies** is not applicable because of performance fine tuning)
    - `SqlDependsOn` - Declare that the concept's database implementation depends on another module, entity or a property.
    - `SqlDependsOnView`
    - `SqlDependsOnFunction`
    - `SqlDependsOnSqlObject`
    - `SqlDependsOnIndex`
    - `SqlDependsOnID` - Dependency on ID property. Use this instead of `SqlDependsOn entity` to avoid having dependencies to all properties of the entity.
