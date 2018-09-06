## Simple validations

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
