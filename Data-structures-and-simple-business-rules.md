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

## See also

* [Rhetos DSL syntax](https://github.com/Rhetos/Rhetos/wiki/Rhetos-DSL-syntax)
