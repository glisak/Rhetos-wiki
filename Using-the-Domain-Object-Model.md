The **Domain Object Model** is a set of C# classes that implement the application's business logic and control the application's data.

It is automatically generated from the application's DSL model (from the *.rhe* scripts) and compiled to *ServerDom.dll*.
The DOM classes are typically used in C# code snippets when implementing business logic with concepts such as Action and ComposableFilterBy.

ServerDom.dll can be directly referenced and used from a simple Visual Studio project or a [LINQPad](https://www.linqpad.net/) script.

1. [How to execute the examples](#how-to-execute-the-examples)
   1. [LINQPad notes](#linqpad-notes)
   2. [Rhetos notes](#rhetos-notes)
2. [Reading the data](#reading-the-data)
   1. [Load data](#load-data)
   2. [Query data](#query-data)
   3. [Filters](#filters)
   4. [Overview of the read methods](#overview-of-the-read-methods)
3. [Modifying the data](#modifying-the-data)
   1. [Save data](#save-data)
   2. [Sketch a code snippet when developing a new Action](#sketch-a-code-snippet-when-developing-a-new-action)
   3. [Execute action](#execute-action)
   4. [Execute action with parameters](#execute-action-with-parameters)
   5. [Execute recompute (ComputedFrom)](#execute-recompute-computedfrom)
4. [Analyze the DSL model](#analyze-the-dsl-model)
5. [Helpers for writing code snippets](#helpers-for-writing-code-snippets)
   1. [ComposableFilterBy code snippet](#composablefilterby-code-snippet)
   2. [ItemFilter code snippet](#itemfilter-code-snippet)
   3. [FilterBy code snippet](#filterby-code-snippet)
   4. [Action code snippet](#action-code-snippet)

## How to execute the examples

The examples from this article can be executed within LINQPad script `Rhetos Server DOM.linq`, available in the Rhetos server folder,
or create a playground console app in Visual Studio with ConsoleDump NuGet plugin.

Options:

* LINQPad - Simple to use (no setup), nicer output format, but IntelliSense (autocomplete) requires buying a license.
* Playground console app in Visual Studio - 10 minutes setup, IntelliSense included.

In both cases, to run any example from this article, copy the code snippet from each example to the marked position in the Main method:

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

        // <<< Copy-paste the example code here >>>
    }
}
```

### LINQPad notes

* Downloaded LINQPad from <https://www.linqpad.net/>.
* The examples use the LINQPad's method `Dump()` to print the results in a grid.
* LINQPad keeps the Rhetos server's process active between executions. This improves performance, but you will need to stop the LINQPad process ("Cancel All Thread and Reset" menu option) before calling DeployPackages.exe, to unlock the dll files.
* Instead of using LINQPad, the following code examples can be executed with Visual Studio project referencing ServerDom.*.dll (and the dlls it uses) if the `Dump()` method is removed.

### Rhetos notes

* Note that this scripts contains `commitChanges: false`. This means that all changes in database will be rolled back at the end of the script.

## Reading the data

### Load data

**Load** method returns array of simple objects.

```C#
var allBooks = repository.Bookstore.Book.Load();
allBooks.Dump();

var someBooks = repository.Bookstore.Book.Load(b => b.Title.StartsWith("B"));
someBooks.Dump();
```

### Query data

**Query** method returns Entity Framework LINQ query.

```C#
var query = repository.Bookstore.Book.Query();

var query2 = query
    .Where(b => b.Title.StartsWith("B"))
    .Select(b => new { b.Title, b.Code });

// Entity Framework overrides ToString to return the generated SQL query.
query2.ToString().Dump("Generated SQL query");

// ToList will force Entity Framework to load the data from the database.
var items = query2.ToList();
items.Dump();
```

**ToSimple** is a Rhetos method that removes ORM navigation properties from the loaded objects.

```C#
// Query data from the `Common.Claim` table:

var query = repository.Common.Claim.Query()
    .Where(c => c.ClaimResource.StartsWith("Common.") && c.ClaimRight == "New");

query.ToString().Dump("Claims query SQL");
query.Dump("Claims query items"); // With navigation properties.

query.ToSimple().ToString().Dump("Claims ToSimple SQL"); // Same as above.
query.ToSimple().Dump("Claims ToSimple items"); // Without navigation properties.
```

### Filters

Both Load and Query method support a filter parameter.
There are different kind of filters that can be used here.

1. Using the **specific filters** implemented in the DSL script (concepts FilterBy, ComposableFilterBy ItemFilter, Query).

    ```C#
    var filterParameter = new Bookstore.CommonMisspelling();
    repository.Bookstore.Book.Load(filterParameter).Dump();

    // ToString will report the generated SQL query.
    repository.Bookstore.Book.Query(filterParameter).ToString().Dump();
    ```

2. Predefined filter - **Get by ID**.

    ```C#
    Guid id = new Guid("c9860617-7e3e-4158-9d57-bbf4dd1f986e");
    repository.Bookstore.Book.Load(new[] { id }).Single().Dump();
    ```

3. Predefined filter - **Generic property filter**
(for available operations see [filters](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md#filters) documentation from the RestGenerator)

    ```C#
    // Generic property filter:
    var filter1 = new FilterCriteria("Title", "StartsWith", "B");
    repository.Bookstore.Book.Query(filter1).Dump();

    // IEnumerable of generic filters:
    var filter2 = new FilterCriteria("Title", "Contains", "ABC");
    var filters = new[] { filter1, filter2 };
    var filtered = repository.Bookstore.Book.Query(filters);
    filtered.ToString().Dump(); // Note that the SQL query contains both filters.
    filtered.ToSimple().Dump();
    ```

Note that the examples above will work on both **Load** and **Query** methods.

### Overview of the read methods

The repository class for Entity, SqlQueryable or Browse, contains the following methods:

| | Get a list or an array of simple items<br>Returns `IEnumerable<SimpleClass>` | Get a LINQ query<br>Returns `IQueryable<QueryableClass>` |
| --- | --- | --- |
| **All items** | `var items = rep.Load()` | `var query = rep.Query()` |
| **Single filter or initialization** | `var items = rep.Load(filterParameter)` | `var query = rep.Query(filterParameter)` |
| **Apply multiple filters** | `items = rep.Filter(items, filterParameter)`<br>Note: this is inefficient | `query = rep.Filter(query, filterParameter)` |

The following table shows what repository methods are generated by which DSL concept:

| DSL Concept | Available repository methods |
| --- | --- |
**Entity**<br>(includes some predefined filters) | `rep.Load()`<br>`rep.Query()`<br>`=> ComposableFilterBy IEnumerable<Guid>`<br>`=> ComposableFilterBy lambda`<br>`=> ComposableFilterBy FilterCriteria`<br>`=> ComposableFilterBy …` |
**FilterBy** filterParameter | `rep.Load(filterParameter)` |
**ComposableFilterBy** filterParameter | `rep.Load(filterParameter)`<br>`rep.Query(filterParameter)`<br>`rep.Filter(query, filterParameter)` |
**ItemFilter** filterParameter | `=> ComposableFilterBy filterParameter` |
**Query** filterParameter | `rep.Load(filterParameter)`<br>`rep.Query(filterParameter)` |

## Modifying the data

**Note:**
If you want the check the modifies data in the database after running the following examples,
you need to modify `commitChanges: false` to `true` in the testing script of console app.
By default, if commitChanges is false, all the changes **will be rolled back** at the end of the script.

### Save data

```C#
// Insert a record in the `Common.Principal` table:
var testUser = new Common.Principal { Name = "Test123", ID = Guid.NewGuid() };
repository.Common.Principal.Insert(new[] { testUser });

// Update an existing record:
var loadedUser = repository.Common.Principal.Load(new[] { testUser.ID }).Single();
loadedUser.Name = loadedUser.Name + "xyz";
repository.Common.Principal.Update(loadedUser);

// Delete:
repository.Common.Principal.Delete(new[] { testUser });

// Print logged events for the `Common.Principal`:
repository.Common.LogReader.Query()
    .Where(log => log.TableName == "Common.Principal" && log.ItemId == testUser.ID)
    .ToList()
    .Dump("Common.Principal log");

// Output:
// TableName        | Action | Description                       | ...
// Common.Principal | Insert |                                   | ...
// Common.Principal | Update | < PREVIOUS Name = "Test123" />    | ...
// Common.Principal | Delete | < PREVIOUS Name = "Test123xyz" /> | ...
```

Note that in Entity Framework applications, saving the modified data to the database
is usually done with the Flush method, or implicitly at the end of the request.

In Rhetos, any modified data must by explicitly saved to the database with the `Update` method
(see the example above).

### Sketch a code snippet when developing a new Action

[Action](Action-concept.md) concept in Rhetos represents a custom server-side command that returns no data.

Using LINQPad, or a playground console app, write a code for the Bookstore demo application
that **inserts five books**.

```C#
for (int i = 0; i < 5; i++)
{
    var newBook = new Bookstore.Book { Code = "+++", Title = "New book" };
    repository.Bookstore.Book.Insert(newBook);
}
```

Write an [Action](Action-concept.md) concept in DSL scripts that inserts five books,
copy the code snippet above into the concept implementation.

```C
Module Bookstore
{
    Action Insert5Books
        '(parameter, repository, userInfo) =>
        {
            for (int i=0; i<5; i++)
            {
                var newBook = new Bookstore.Book { Code = "+++", Title = "New book" };
                repository.Bookstore.Book.Insert(newBook);
            }
        }';
}
```

### Execute action

This example uses the [Action](Action-concept.md) created in the previous example.
Don't forget to run `DeployPackages.exe` after implementing Action in the DSL script.

```C#
// Null argument is send because this action has no parameters.
repository.Bookstore.Insert5Books.Execute(null);
```

### Execute action with parameters

The code below uses `Common.SetLock` as an example of the [Action](Action-concept.md) concept
that is included in the CommonConcepts Rhetos package.

```C#
// Execute the `Common.SetLock` action:

var parameter = new Common.SetLock
{
    ResourceType = "some resource",
    ResourceID = new Guid("12341234-1234-1234-1234-123412341234")
};
repository.Common.SetLock.Execute(parameter);

// Test that the action generated a record:

repository.Common.ExclusiveLock.Query()
    .OrderByDescending(item => item.LockStart)
    .ToSimple()
    .First()
    .Dump("Common.ExclusiveLock last item");
```

### Execute recompute (ComputedFrom)

"Recompute" refers to the process of updating the persisted computed data,
see [Persisting the computed data](Persisting-the-computed-data.md).

This example requires **CommonConceptsTest** plugin to be deployed on Rhetos server.
It is available in Rhetos source subfolder `CommonConcepts\CommonConceptsTest`.

```C#
// Recomputing data in the `TestComputedFrom.PersistComplex` table
// to match the data from the `TestComputedFrom.Source` data structure:

// The recompute method's name is "RecomputeFrom" + data source name.
repository.TestComputedFrom.PersistComplex.RecomputeFromSource();
```

Modify `ConsoleLogger.MinLevel = EventType.Info` to `EventType.Trace`, to see the steps of the recompute analysis: Initialize new items, Load old items, Diff, Save.

## Analyze the DSL model

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

The scripts in the following chapters will help you with development and testing of code snippets for filters and actions. The code snippet can then be copied to the *.rhe* script.

### ComposableFilterBy code snippet

```C#
// This ComposableFilterBy will filter the records by prefix from the 'Common.Claim' table.

// The filter has parameter: `ShortString Prefix;`
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

After developing and testing the ComposableFilterBy's code snippet, use it to put the filter (`DemoFilter`) in the *.rhe* script:

```C
Module Common
{
    Entity Claim
    {
        ComposableFilterBy DemoFilter '(query, repository, parameter) =>
            query.Where(item => item.ClaimResource.StartsWith(parameter.Prefix))';
    }

    Parameter DemoFilter
    {
        ShortString Prefix;
    }
}
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

### FilterBy code snippet

```C#
// This FilterBy will filter the records by prefix from the 'Common.Claim' table.

// The filter has parameter: `ShortString Prefix;`
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

### Action code snippet

```C#
// This Action will insert a new record to the `Common.Claim` table:

// The action has parameters: `ShortString Resource;` and `ShortString Right;`.
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
