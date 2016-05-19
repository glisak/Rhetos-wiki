Table of contents:

1. [Create a new entity](#create-a-new-entity)
2. [Extending an entity with computed data](#extending-an-entity-with-computed-data)
3. [Implementing simple business rules](#implementing-simple-business-rules)
4. [Row permissions business rules](#row-permissions-business-rules)
5. [Implementing entity inheritance and common interfaces](#implementing-entity-inheritance-and-common-interfaces)
6. [Database objects](#database-objects)
7. [Action concept](#action-concept)
8. [Reports](#reports)
9. [Entity with change history](#entity-with-change-history)
10. [Persisting the computed data](#persisting-the-computed-data)

## Create a new entity

The Rhetos platform allows simple and fast development of new entities. All you need to do is define its properties and their types.

```
Module Demo
{
    Entity School
    {
        Logging { AllProperties; }
        
        ShortString Code { AutoCode; }
        ShortString Name { Required; }
        Reference Region;
    }

    Entity Region
    {
        ShortString Name { Required; }
    }
}
```

You can mark any property to be `Required`.
A property marked with `AutoCode` will automatically increase by one for each new record.

## Extending an entity with computed data

Most commonly used way of adding computed data is `SqlQueryable` concept, using SQL query to define the computation.

Examples:

```
Entity Employee
{
    Integer IdentificationNumber;
    ShortString LastName { Required; }
    ShortString FirstName { Required; }
}

SqlQueryable EmployeeInfo1
    "SELECT
        d.ID,
        d.IdentificationNumber, -- Not used in this SqlQueryable.
        LastNameFirstName = LastName + ', ' + FirstName,
        FirstNameLastName = FirstName + ' ' + LastName,
    FROM
        Demo.Employee"
{
    Extends Demo.Employee;
            
    ShortString LastNameFirstName;
    ShortString FirstNameLastName;

    SqlDependsOn Demo.Employee;
}

SqlQueryable EmployeeInfo2
    "SELECT
        ID,
        LastNameFirstName,
        FirstNameLastName,
        IdentificationNumber,
    FROM
        Demo.EmployeeInfo1"
{
    Extends Demo.Employee;

    ShortString LastNameFirstName;
    ShortString FirstNameLastName;
    Integer IdentificationNumber;

    SqlDependsOn Demo.EmployeeInfo1;
}
```

The `SqlQueryable` concept creates a view in the database.

The `Extends` concept declares that this computed data extend the base entity.
Therefore it is important for the SQL query to return the `ID` column,
so that the computed data can be connected to the base entity.
The `Extends` concept creates an extension to the entity in the object model.
The extension can be accessed from the base entity using the property `Extension_<EntityName>`
(or `Extension_<ModuleName>_<EntityName>`, if the extension is from a different module).

For example, if we want to access the FirstNameLastName from the Employee in the C# code, we can write:
```
Employee d;
string s = d.Extension_EmployeeInfo1.FirstNameLastName;
```


The `SqlDependsOn` keyword on EmployeeInfo1 declares that the `SqlQueryable` depends on the Employee entity.
That information is needed for Rhetos to know in what order these database objects should be created:
The Employee table should be created before EmployeeInfo1 view because the view depends on the table.
Concepts `SqlDependsOnView`, `SqlDependsOnFunction` and `SqlDependsOnSqlObject` are used in a similar way.
`AutodetectSqlDependencies` can be used instead to automatically detect the dependencies by analyzing the SQL snippet.

## Implementing simple business rules

Simple business rules can be declared beside the property in the .rhe script.
The following concepts can be used:

* `MinValue` - Limit the smallest allowed value of the property.
* `MaxValue` - Limit the largest allowed value of the property.
* `MinLength` - Limit the smallest allowed length of the string.
* `MaxLength` - Limit the largest allowed length of the string.
* `RegExMatch` - Use a regular expression to validate the property value.

For more complex validations a specific filter can be developed. Such validations are built in two steps:

1. Create a filter on the entity. In the example below, the filter FinishBeforeStart returns every Employee with invalid data (WorkFinished is before WorkStarted).
2. Create a validation for that filter with the message for the used. The validation will return the error message to the user it he or she tries to enter the invalid data.

```
Entity Employee
{
    Integer IdentificationNumber;
    ShortString LastName { Required; }
    ShortString FirstName { Required; }
    ShortString Jmbg { RegExMatch "\d{13}" "Must contain 13 digits."; }
    DateTime WorkStarted { Required; }
    DateTime WorkFinished;
    Integer TestPeriod { MinValue 1; MaxValue 12; }
    ShortString Iban { Required; MinLength 21; MaxLength 21; }

    ItemFilter FinishBeforeStart 'item => item.WorkFinished!=null && item.WorkFinished.Value < item.WorkStarted.Value';
    InvalidData FinishBeforeStart 'It is not allowed to enter a WorkFinished time before the WorkStarted time.';
}
```

## Row permissions business rules

See [RowPermissions concept](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept).

## Implementing entity inheritance and common interfaces

See [Polymorphic concept](https://github.com/Rhetos/Rhetos/wiki/Polymorphic-concept).

## Database objects

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
    SqlView ActiveEmployees <SQL\1_View\ActiveEmployees.sql>;
    ```

    - File "ActiveEmployees.sql"

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

### Dependencies between database objects

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

## Action concept

The `Action` concept is intended for implementing server commands. The code with the action will be execution in a single transaction, so that in case of an error the whole action will be canceled (all-or-nothing).

The action can be implemented in two ways:

* Using the `Action` concept, which assumes writing the C# code directly in the .rhe script, or in a referenced .cs script file.
* Using the `ExtAction` concept, where the C# code is implemented in a plugin dll.

Both ways have their pros and cons, and the choice depends on a situation. The `Action` concept is good for simple code snippets, while `ExtAction` is usually used for more complex implementations.

Tips:

* Action is executed in a single transaction. This means that it is not possible to log an exception error in the database.
* Row permission and other security claims are not checked inside the action.
    - The user need to have permission to execute the action (has the *Execute* claim on the action),
      but not for the resources that are used inside the action. The code inside the action has access to all data.
    - The user's permission can be explicitly checked in the action by using `Common.RowPermissionsReadItems`
      and `Common.RowPermissionsWriteItems` filters, or `IAuthorizationManager` for checking security claims.
* Reference property in the action parameters will not be bound the the database by the ORM (the lazy-load is not suppered).
  Also the whole entity cannot be sent as a reference property parameter.
* Exceptions that are cached in the action must be rethrown.

Example:

```
Module Demo
{
    Action SaveTTPayer '(parameters, repository, userInfo) =>
    {
        var ttPayer = new Demo.TTPayer
        {
            ID = parameters.ID,
            Pin = parameters.Pin,
            Name = parameters.Name
        };
        
        repository.Demo.TTPayer.Insert(ttPayer);
        
        var principal = new Common.Principal
        {
            ID = parameters.ID,
            Name = parameters.Pin
        }
        
        repository.Common.Principal.Insert(principal);
    }'
    {
        Guid ID;
        ShortString Pin;
        ShortString Name;
    }
}
```

Example for the `ExtAction` concept:

* DSL:

    ```
    Module Demo
    {
        ExtAction SaveTTPayer 'Demo.Rhetos.TTPayer, Demo.Rhetos', 'SaveTTPayer'
        {
            Guid ID;
            ShortString Pin;
            ShortString Name;
        }
        
    }
    ```

* C#:

    ```
    // This method is implemented in the assembly Demo.Rhetos, in class TTPayer.
    public static void SaveTTPayer(SaveTTPayer parameters, DomRepository repository, IUserInfo userInfo)
    {
            var ttPayer = new Demo.TTPayer
            {
                ID = parameters.ID,
                Pin = parameters.Pin,
                Name = parameters.Name
            };
            
            repository.Demo.TTPayer.Insert(ttPayer);
            
            var principal = new Common.Principal
            {
                ID = parameters.ID,
                Name = parameters.Pin
            }

            InsertPrincipal(principal, repository, Role.TTPayer); // A helper method that creates a new used and assigns default permissions.
            var notification = "...";
            Notify(notification, repository, userInfo, Notify.Admin|Notify.PowerUsers); // A helper method for notifications.
    }
    ```

## Reports

Simple reports can be created using a commercial Rhetos plugin *TemplaterReport*, while complex reports are usually created using Reporting Services
(view and SQL procedures used for those reports are also written in the Rhetos scripts).

The following example contains a report that uses SQL query SqlInvoiceReport as a main data source, and SqlInvoiceProductReport for details.

If a report uses multiple data sources, declare them with `DataSources` concept.

This example uses an external template file "Invoice.doc".

```
Module Demo
{
    TemplaterReport PrintInvoice "Invoice.doc"
    {
        Guid InvoiceID;
                
        DataSources 'SqlInvoiceReport, SqlInvoiceProductReport';
    }
    
     FilterBy SqlInvoiceReport.'Demo.PrintInvoice' '(repository, parameter) =>
        repository.Demo.SqlInvoiceReport.Query()
            .Where(item => item.ID == parameter.InvoiceID.Value).ToArray()';

     FilterBy SqlInvoiceProductReport.'Demo.PrintInvoice' '(repository, parameter) =>
        repository.Demo.SqlInvoiceProductReport.Query()
            .Where(item => item.InvoiceID == parameter.InvoiceID.Value).ToArray()';
}
```

```
SqlQueryable SqlInvoiceReport
    "SELECT
        i.ID,
        i.InvoiceNumber,
        c.VATID,
        Contact = u.LastName + ' ' + u.FirstName,
        i.Total
    FROM
        Demo.Invoice i
        LEFT JOIN Demo.Company c ON c.ID = i.CompanyID
        LEFT JOIN Demo.User u ON u.ID = i.ContactID"
{
    Integer InvoiceNumber;
    ShortString VATID;
    ShortString Contact;
    Money Total;
}
```

```
SqlQueryable SqlInvoiceProductReport
    "SELECT
        ip.ID,
        ip.InvoiceID,
        OrdinalNumber = CAST(ROW_NUMBER() OVER (PARTITION BY ip.InvoiceID ORDER BY ip.CreatedTime) AS VARCHAR(5)) + '.',
        ProductName = p.Name,
        ip.Amount
    FROM
        Demo.InvoiceProduct ip
        LEFT JOIN Demo.Product p ON p.ID = ip.ProcuctID"
{
    Guid InvoiceID;
    ShortString OrdinalNumber;
    ShortString ProductName;
    Decimal Amount;
}
```

## Entity with change history

Use the `History` concept to manage temporal data in an entity.
The the concept is used on an entity, the last version of each record is written in the main table,
while older versions are archived in the "\_Changes" table.
The *ActiveSince* property is automatically added to the entity, each version of the record is effective from that time.

The following objects are created in the database:

* "ActiveSince" column in the entity's table.
* "\_Changes" table - A copy of the entity's table, contains EntityID reference to the ID of the main table.
* "\_History" view - Union of the current record and the archived records.
* "\_ChangesActiveUntil" view - Computed the active range for the records in "\_Changes" table
* "\_AtTime(@ContextTime DATETIME)" function - Returns the record that was active at the specified time.

**Rhetos:**

```
Module Demo
{
    Entity ContractStatus
    {
        History { AllProperties; }
        ShortString Name { Required; Unique; LookupVisible; }
        Bool Active { Required; }
    }
}
```

**SQL:**

``` sql
CREATE TABLE [Demo].[ContractStatus] (
    [ID] [uniqueidentifier] NOT NULL,
    [ActiveSince] [datetime] NULL,
    [Active] [bit] NULL,
    [Name] [nvarchar](256) NULL,
    CONSTRAINT [PK_ContractStatus] PRIMARY KEY NONCLUSTERED ([ID] ASC)
)
GO
ALTER TABLE [Demo].[ContractStatus] ADD  CONSTRAINT [DF_ContractStatus_ID]  DEFAULT (newid()) FOR [ID]
GO
```

``` sql
CREATE TABLE [Demo].[ContractStatus_Changes] (
    [ID] [uniqueidentifier] NOT NULL,
    [EntityID] [uniqueidentifier] NULL,
    [ActiveSince] [datetime] NULL,
    [Active] [bit] NULL,
    [Name] [nvarchar](256) NULL,
    CONSTRAINT [PK_ContractStatus_Changes] PRIMARY KEY NONCLUSTERED ([ID] ASC)
) ON [PRIMARY]
GO
ALTER TABLE [Demo].[ContractStatus_Changes] ADD  CONSTRAINT [DF_ContractStatus_Changes_ID]  DEFAULT (newid()) FOR [ID]
GO
ALTER TABLE [Demo].[ContractStatus_Changes]  WITH CHECK ADD  CONSTRAINT [FK_ContractStatus_Changes_ContractStatus_EntityID] FOREIGN KEY([EntityID])
REFERENCES [Demo].[ContractStatus] ([ID])
ON DELETE CASCADE
GO
ALTER TABLE [Demo].[ContractStatus_Changes] CHECK CONSTRAINT [FK_ContractStatus_Changes_ContractStatus_EntityID]
GO
```

``` sql
CREATE VIEW [Demo].[ContractStatus_History] AS
SELECT
    ID = entity.ID,
    EntityID = entity.ID,
    ActiveUntil = CAST(NULL AS DateTime),
    ActiveSince = entity.ActiveSince,
    Active = entity.Active,
    Name = entity.Name/*EntityHistoryInfo SelectEntityProperties Demo.ContractStatus*/
FROM
    Demo.ContractStatus entity

UNION ALL

SELECT
    ID = history.ID,
    EntityID = history.EntityID,
    au.ActiveUntil,
    ActiveSince = history.ActiveSince,
    Active = history.Active,
    Name = history.Name/*EntityHistoryInfo SelectHistoryProperties Demo.ContractStatus*/
FROM
    Demo.ContractStatus_Changes history
    LEFT JOIN Demo.ContractStatus_ChangesActiveUntil au ON au.ID = history.ID
GO
```

``` sql
ALTER VIEW [Demo].[ContractStatus_ChangesActiveUntil] AS
SELECT
    history.ID,
    ActiveUntil = COALESCE(MIN(newerVersion.ActiveSince), MIN(currentItem.ActiveSince))
FROM
    Demo.ContractStatus_Changes history
    LEFT JOIN Demo.ContractStatus_Changes newerVersion ON newerVersion.EntityID = history.EntityID AND newerVersion.ActiveSince > history.ActiveSince
    INNER JOIN Demo.ContractStatus currentItem ON currentItem.ID = history.EntityID
GROUP BY
    history.ID
GO
```

``` sql
CREATE FUNCTION [Demo].[ContractStatus_AtTime] (@ContextTime DATETIME)
RETURNS TABLE
AS
RETURN
SELECT
    ID = history.EntityID,
    ActiveUntil,
    EntityID = history.EntityID,
    ActiveSince = history.ActiveSince,
    Active = history.Active,
    Name = history.Name/*EntityHistoryInfo SelectHistoryProperties Demo.ContractStatus*/
FROM
    Demo.ContractStatus_History history
    INNER JOIN
    (
        SELECT EntityID, Max_ActiveSince = MAX(ActiveSince)
        FROM Demo.ContractStatus_History
        WHERE ActiveSince <= @ContextTime
        GROUP BY EntityID
    ) last ON last.EntityID = history.EntityID AND last.Max_ActiveSince = history.ActiveSince
GO
```

## Persisting the computed data

Persisting the computed data is done by developing the data source that computed the data (usually an `SqlQueryable`),
and saving the result into the database table.

Rhetos contains concepts that help automate the implementation:
`KeepSynchronized`, `ChangesOnLinkedItems`, `ChangesOnChangedItems` and `ChangesOnBaseItem`.

Example:

**Rhetos:**
```
Module Demo
{
    SqlQueryable SubjectDs
    "
        SELECT
            s.ID,
            NrlCode = CAST(s.NrlCode AS NVARCHAR(100)),
            subjectLastVersion.RnkAsc,
            subjectLastVersion.RnkDesc,
            IsFirstVersion = CAST(CASE WHEN subjectLastVersion.RnkAsc = 1 THEN 1 ELSE 0 END AS BIT),
            IsLastVersion = CAST(CASE WHEN subjectLastVersion.RnkDesc = 1 THEN 1 ELSE 0 END AS BIT),
            StatusSubjectID = ss.ID
        FROM
            Demo.Subject s
            LEFT JOIN
            (
                SELECT
                    s.ID,
                    RnkAsc = ROW_NUMBER() OVER (PARTITION BY s.NrlCode ORDER BY s.Version ASC),
                    RnkDesc = ROW_NUMBER() OVER (PARTITION BY s.NrlCode ORDER BY s.Version DESC),
                    StatusProductID =
                        (SELECT TOP 1
                            ssh.StatusSubjectID
                        FROM
                            Demo.StatusSubjectHistory ssh
                        WHERE
                            s.ID = ssh.SubjectID AND ssh.EntityType = 'Subject'
                        ORDER BY
                            ssh.SetupTime DESC)
                FROM
                    Demo.Subject s
            ) subjectLastVersion ON subjectLastVersion.ID = s.ID
            LEFT JOIN Demo.StatusProduct ss
                WITH (INDEX(IX_StatusProduct_ID_Includes))
                ON ss.ID = subjectLastVersion.StatusProductID
            LEFT JOIN Demo.NrlCountry country ON country.ID = s.NrlCountryID
    "
    {
        Extends Demo.Subject;
        
        ChangesOnLinkedItems Demo.StatusSubjectHistory.Subject;

        ChangesOnChangedItems Demo.Subject 'Guid[]' ' changedItems =>
        {
            var changedItemIds = changedItems.Select(item => item.ID).ToList();
            var nrlCodes = _domRepository.Demo.Subject.Query()
                .Where(s => changedItemIds.Contains(s.ID))
                .Select(s => s.NrlCode)
                .Distinct()
                .ToList();

            return _domRepository.Demo.Subject.Query()
                .Where(item => nrlCodes.Contains(item.NrlCode))
                .Select(item => item.ID)
                .ToArray();
        }';
        
        ShortString NrlCode;
        Integer RnkAsc;
        Integer RnkDesc;
        Bool IsFirstVersion;
        Bool IsLastVersion;
        Reference StatusSubject Demo.StatusProduct;
    }
}
```

```
Module Demo
{
    Entity SubjectPersisted
    {
        ComputedFrom Demo.SubjectDs
        {
            AllProperties;
            KeepSynchronized;
        }
    }
}
```
