The **Domain Object Model** is a set of C# classes that implement the application's business logic and control the application's data.

It is automatically generated from the application's DSL model (from the *.rhe* scripts) and compiled to *ServerDom.dll*.
The DOM classes are typically used in C# code snippets when implementing business logic with concepts such as Action and ComposableFilterBy.

ServerDom.dll can be directly referenced and used from a simple Visual Studio project or a [LinqPad](https://www.linqpad.net/) script.

## How to execute the examples

The examples from this article can be executed within LinqPad script `Rhetos Server DOM.linq`, available in Rhetos server folder.

```C#
void Main()
{
    ConsoleLogger.MinLevel = EventType.Info; // Use "Trace" for more details log.
    var rhetosServerPath = Path.GetDirectoryName(Util.CurrentQueryPath);
    Directory.SetCurrentDirectory(rhetosServerPath);
    using (var container = new RhetosTestContainer(commitChanges: false)) // Use this parameter to COMMIT or ROLLBACK the data changes.
    {
        var context = container.Resolve<Common.ExecutionContext>();
        var repository = context.Repository;
        
        // ... Copy-paste the example code here ...
    }
}
```

### LinqPad notes

* Downloaded LinqPad from https://www.linqpad.net/.
* The examples use the LinqPad's method `Dump()` to print the results in a grid.
* The following code examples can be executed with Visual Studio project referencing ServerDom.dll (and the dlls it uses) if the `Dump()` method is removed.

### Rhetos notes

* Note that this scripts contains `commitChanges: false`. This means that all changes in database will be rolled back at the end of the script.
* All Rhetos commands will throw an exceptions and rollback the database transaction in case of on error.

## Examples

### Query data

```C#
// Query data from the `Common.Claim` table:

var claims = repository.Common.Claim.Query()
    .Where(c => c.ClaimResource.StartsWith("Common.") && c.ClaimRight == "New")
    .ToSimple(); // Removes ORM navigation properties from the loaded objects.
    
claims.ToString().Dump("Common.Claims query");
claims.Dump("Common.Claims items");
```

### Save data

```C#
// Add and remove a record in the `Common.Principal` table:

var testUser = new Common.Principal { Name = "Test123", ID = Guid.NewGuid() };
repository.Common.Principal.Insert(new[] { testUser });
repository.Common.Principal.Delete(new[] { testUser });

// Print logged events for the `Common.Principal`:

repository.Common.LogReader.Query()
    .Where(log => log.TableName == "Common.Principal" && log.ItemId == testUser.ID)
    .ToList()
    .Dump("Common.Principal log");
```

### Execute action

```C#
// Execute the `Common.SetLock` action:

var parameter = new Common.SetLock
{
    ResourceType = "some resource",
    ResourceID = new Guid("12341234-1234-1234-1234-123412341234")
};
repository.Common.SetLock.Execute(parameter);

// The action generated a record:

repository.Common.ExclusiveLock.Query()
    .OrderByDescending(item => item.LockStart)
    .ToSimple()
    .First()
    .Dump("Common.ExclusiveLock last item");
```

### Execute recompute (ComputedFrom)

This example requires **CommonConceptsTest** plugin to be deployed on Rhetos server. It is available in Rhetos source subfolder `CommonConcepts\CommonConceptsTest`.

```C#
// Recomputing data in the `TestComputedFrom.PersistComplex` table
// to match the data from the `TestComputedFrom.Source` data structure:

repository.TestComputedFrom.PersistComplex.RecomputeFromSource(); // The recompute method's name is "RecomputeFrom" + data source name.
```

Modify `ConsoleLogger.MinLevel = EventType.Info` to `EventType.Trace`, to see the steps of the recompute analysis: Initialize new items, Load old items, Diff, Save.

### Reading the DSL model

This example uses the methods of the `IDslModel` interface for performance-efficient search on the DSL model: `FindByType`, `FindByKey` and `FindByReference`.

```C#
var dslModel = container.Resolve<IDslModel>();

// List all entities:
var entities = dslModel.FindByType<EntityInfo>().Select(entity => entity.Module.Name + "." + entity.Name);
entities.Dump("Entities");

// Find a specific entity by concept's key:
// Entity inherits DataStructure, and the concept's key always contains the base concept's type name.
var principalEntity = (DataStructureInfo)dslModel.FindByKey("DataStructureInfo Common.Principal");
principalEntity.GetUserDescription().Dump("Common.Principal entity");

// Find all ShortString properties that match `property => property.DataStructure == principalEntity`
var properties = dslModel.FindByReference<ShortStringPropertyInfo>(property => property.DataStructure, principalEntity);
properties.Select(p => p.GetUserDescription()).Dump("Common.Principal ShortString properties");
```

## Helpers for writing code snippets

### FilterBy code snippet

```C#
// This FilterBy will filter the records by prefix from the 'Common.Claim' table.

// The FilterBy's properties: `ShortString Prefix;`
var parameter = new
{
    Prefix = "Common.Prin"
};

var query = repository.Common.Claim.Query();

Common.Claim[] filteredItems =
    // Uncomment the first line of the code snippet when copying to .rhe script.
    // The FilterBy's code snippet BEGIN
    //(repository, parameter) =>
        repository.Common.Claim.Query()
            .Where(item => item.ClaimResource.StartsWith(parameter.Prefix))
            .ToSimple().ToArray()
    // The FilterBy's code snippet END
    ;
filteredItems.Dump("FilterBy items");
```

### ComposableFilterBy code snippet

```C#
// This ComposableFilterBy will filter the records by prefix from the 'Common.Claim' table.

// The ComposableFilterBy's properties: `ShortString Prefix;`
var parameter = new
{
    Prefix = "Common.Prin"
};

var query = repository.Common.Claim.Query();

IQueryable<Common.Queryable.Common_Claim> filteredQuery =
    // Uncomment the first line of the code snippet when copying to .rhe script.
    // The ComposableFilterBy's code snippet BEGIN
    //(query, repository, parameter) =>
        query.Where(item => item.ClaimResource.StartsWith(parameter.Prefix))
    // The ComposableFilterBy's code snippet END
    ;
filteredQuery.ToSimple().Dump("ComposableFilterBy query");
```

### ItemFilter code snippet

```C#
// This ItemFilter will return the records with prefix "Common.Prin" from the 'Common.Claim' table.

repository.Common.Claim.Query(
    // The ItemFilter's code snippet BEGIN
    item => item.ClaimResource.StartsWith("Common.Prin")
    // The ItemFilter's code snippet END
).ToSimple().Dump("ItemFilter query");
```

### Action code snippet

```C#
// This Action will insert a new record to the `Common.Claim` table:

// The Action's properties: `ShortString Resource;` and `ShortString Right;`.
var parameter = new
{
    Resource = "TestResource", // ShortString property of the action.
    Right = "TestRight" // ShortString property of the action.
};

var userInfo = context.UserInfo;

// Uncomment the first line of the code snippet when copying to .rhe script.
// The Action's code snippet BEGIN
// (parameter, repository, userInfo) =>
{
    repository.Common.Claim.Insert(new Common.Claim
    {
        ClaimResource = parameter.Resource,
        ClaimRight = parameter.Right,
        Active = true
    });
}
// The Action's code snippet END
```
