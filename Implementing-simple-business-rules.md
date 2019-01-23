1. [Property value constraints](#property-value-constraints)
2. [Deny data modifications](#deny-data-modifications)
3. [See also](#see-also)

## Property value constraints

Property value constraints are simple business rules that can be declared beside the property in the .rhe script.
The following concepts are available in CommonConcepts package:

* `MinValue` - Limit the smallest allowed value of the property.
* `MaxValue` - Limit the largest allowed value of the property.
* `MinLength` - Limit the smallest allowed length of the string.
* `MaxLength` - Limit the largest allowed length of the string.
* `RegExMatch` - Use a regular expression to validate the property value.

For more complex data validations a specific filter can be developed. Such validations are built in two steps:

1. Create a filter on the entity. In the example below, the filter FinishBeforeStart returns every Employee with invalid data (WorkFinished is before WorkStarted).
2. Create a validation for that filter with the message for the used. The validation will return the error message to the user it he or she tries to enter the invalid data.

```C
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

## Deny data modifications

The following business rules are similar to data validations, but actually act more like action permissions.
Instead of of checking if a certain data value *is valid or not*, they just block certain operations depending on the context.
Examples of the following concepts are available in a unit testing DSL script
[Validations.rhe](https://github.com/Rhetos/Rhetos/blob/master/CommonConcepts/CommonConceptsTest/DslScripts/Validations.rhe).

* `Lock` - Deny update and delete of the entity records, for records in a certain state (provided by a filter).
* `LockProperty` - Deny update of a property, for records in a certain state.
* `LockExcept` - Deny update (except for properties from the provided list) and delete of the entity records, for records in a certain state.
* `DenyUserEdit` on an entity - Client application is not allowed to directly insert, update or delete the entity records (no condition).
* `DenyUserEdit` on a property - Client application is not allowed to directly insert or update the property. On insert and update it is only allowed to send a *null* value (it will be interpreted as "no change" on update), or a previous value.

Similar features and alternatives:

1. If some data operation should be denied based on *which user* that is performing it, you should probably use [Row permissions](RowPermissions-concept) instead of the concepts listed above.
2. If you need a similar operation, but not listed above (for example to deny deleting records in a certain state), it would best to create you own concept based on the implementation of similar one. A more pragmatic option is to directly insert a C# code snippet into the entity's Save method (by using low-level concepts SaveMethod with Initialization or OnSaveValidate), and check it the Save operation should be blocked. Some code samples are available in [DataStructure.rhe](https://github.com/Rhetos/Rhetos/blob/master/CommonConcepts/CommonConceptsTest/DslScripts/DataStructure.rhe) script, with a realistic example at `OnSaveValidate DenyChangeOfLockedName`.

## See also

Examples of many DSL concepts are available in the unit testing setup script in the Rhetos framework source folder [CommonConcepts\CommonConceptsTest\DslScripts](https://github.com/Rhetos/Rhetos/tree/master/CommonConcepts/CommonConceptsTest/DslScripts):

* The script [SimpleBusinessLogic.rhe](https://github.com/Rhetos/Rhetos/blob/master/CommonConcepts/CommonConceptsTest/DslScripts/SimpleBusinessLogic.rhe) contains usage examples for some of the concepts mentioned in this article.
