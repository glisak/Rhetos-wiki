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

## Example

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

The `History` concept works in a backward-compatible way:
The basic table for this entity (`Demo.ContractStatus`) remains unchanged, with rows representing **currently active version** for each item.
A new *DateTime* property `ActiveSince` is created, it records "since when" is the current value effective.

A new database table is created `Demo.ContractStatus_Changes`, that contains previous versions for the items.
It has all properties from `Demo.ContractStatus`, and additionally a column (property) `EntityID` that references `Demo.ContractStatus`.
`ID` in the `Demo.ContractStatus_Changes` is just an internal ID of a history record, `EntityID` contains the ID value from `Demo.ContractStatus`.

View `Demo.ContractStatus_History` is created that contains all versions of the ContractStatus items.
It is a union of `ContractStatus` and `ContractStatus_Changes`.
Additionally it contains a computed property `ActiveUntil`, that returns `ActiveSince` from the next version of the record.

An inline table-valued function `Demo.ContractStatus_AtTime` is created, with a `DATETIME` parameter, that returns versions of all records that where active at the given time.
