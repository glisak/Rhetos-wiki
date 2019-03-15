# Developing filters and other read methods

Most of the business rules in Rhetos applications are based on
filters and other similar read methods.

Developers write filters and queries to process the data.
The resulting datasets are then utilized to create validation rules,
compute business process state, define user permissions,
and other features.

This article explains how to write filters when developing a Rhetos application.

Prerequisites:

* [Entities and relationships](Data-model-and-relationships).
* Learn how to use the Rhetos Domain Object Model method to [Read the data](Using-the-Domain-Object-Model#reading-the-data).

Contents:

1. [Filters](#filters)
   1. [ItemFilter concept](#itemfilter-concept)
   2. [ComposableFilterBy concept](#composablefilterby-concept)
   3. [Predefined filters](#predefined-filters)
2. [Other read methods](#other-read-methods)
   1. [FilterBy concept](#filterby-concept)
   2. [Query concept](#query-concept)
3. [Combining filters and other read methods](#combining-filters-and-other-read-methods)
4. [Filter name vs. parameter type](#filter-name-vs-parameter-type)

## Filters

### ItemFilter concept

ItemFilter is the simplest concept for creating new filters.
You can use it on **Entity**, **Browse**, **SqlQueryable**, or any other queryable data structure
to filter the existing data with a simple lambda expression.

Consider the following example:

> 1. Write a filter that returns all books with the common misspelled word "curiousity".
>    Deny users the enter the incorrect book titles into the application.
> 2. Write another filter that returns books with 500 pages or longer.
>    The web application will display a list of long books.

Solution:

```C
Module Bookstore
{
    Entity Book
    {
        ShortString Code { AutoCode; }
        ShortString Title { Required; }
        Integer NumberOfPages;

        ItemFilter CommonMisspelling 'item => item.Title.Contains("curiousity")';
        InvalidData CommonMisspelling 'It is not allowed to enter misspelled word "curiousity". Please use "curiosity" instead.';

        ItemFilter LongBooks 'item => item.NumberOfPages >= 500';
    }
}
```

ItemFilter has to parameters: filter name and lambda expression.

* The **lambda expression** is a C# code snippet that will be internally
  be used in a Entity Framework LINQ query to load the data.
* For example, `ItemFilter CommonMisspelling` will create a LINQ query
  similar to this one in the generated application, with the lambda expression
  directory copied into the Where method:
    ```C#
    query.Where(item => item.Title.Contains("curiousity"));
    ```

The first filter is additionally used for a data validation
that will deny Save operation to the end user if entered an invalid book names.
**Both filters** can be used by a client application (or in the object model)
to query a subset of the books.

* Note that the validation filter can be useful for reading data,
  even if there should be no "invalid" data in the system.
  For example, there might exist some old data entered before this
  data validation was developed, or before some bug was corrected
  in the validation's filter.
* Rhetos allows developers to list all data validations in the system,
  run their filters on the existing data, and report any data that
  does not comply with the current version of the validations.

How to **test this filter**, or use it in your application:

1. From the REST web API (see [specification](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md)):
   * <http://localhost/BookstoreRhetosServer/rest/Bookstore/Book/?filters=[{%22Filter%22:%22Bookstore.LongBooks%22}]>
2. From the C# code (see [Using the Domain Object Model](Using-the-Domain-Object-Model#Filters))
    ```C#
    var query = repository.Bookstore.Book.Query(new Bookstore.LongBooks());		
    query.ToString().Dump(); // Print SQL query.
    query.ToSimple().ToList().Dump(); // Load and print the books.
    ```

ItemFilters are useful when you need to query data from the
**related entities**. For example:

> Write a filter that will return Book that are foreign
> (if record exists in the **extension** entity ForeignBook),
> and author's name starts with X (see the referenced entity Person).

Solution:

```C
Module Bookstore
{
    Entity Book
    {
        ShortString Code { AutoCode; }
        ShortString Title { Required; }
        Integer NumberOfPages;
        Reference Author Bookstore.Person;

        ItemFilter ForeignAuthorX 'item =>
            item.Author.Name.StartsWith("X")
            && item.Extension_ForeignBook.ID != null';
    }

    Entity Person
    {
        ShortString Name;
    }

    Entity ForeignBook
    {
        Extends Bookstore.Book;
        ShortString OriginalLanguage;
    }
}
```

### ComposableFilterBy concept




* InvalidData, Lock and similar concepts can use both ComposableFilterBy or ItemFilter.

### Predefined filters

Every entity by default supports the following predefined filters.
They behave in a same way as a ComposableFilterBy.

Here is a list of predefined filter. The examples of using these filters in C# code are available in the "Filters" chapter in [Using the Domain Object Model](Using-the-Domain-Object-Model#Filters) article.

* Lambda expression (a parameter for the Where() method in the LINQ query)
* `IEnumerable<Guid>`
* Generic property filter: `FilterCriteria` and `IEnumerable<FilterCriteria>`
  * Dopisati REST example, documentation
* `Rhetos.Dom.DefaultConcepts.FilterAll`
  * This is a rarely used filter that returns all the data. Even though it doesn't really do any processing, this filter is a simple helper that is sometimes used when you need to return a filter parameter, but you just want to select all data.

## Other read methods

These read methods work in a similar way as filters: they have the same parameters,
and they return the result in the same format.
The only difference it that they **don't have an input data to be processed (filtered)**,
instead they need to initialize the dataset each time.

These methods do not need to return a subset from the existing data from the database
(event though they are often implemented that way).
For example, they can generate new data or read it from an external system.

### FilterBy concept

### Query concept

## Combining filters and other read methods

As said before, the composable filters receive a LINQ query as an input and return a filtered query.
Other read methods can only generate new data from scratch.

In your web request or in the C# code, you can easily combine multiple filters
such as *ItemFilter*, *ComposableFilterBy* or a generic property filter.
On contrast, the *FilterBy* should only be used as a single filter parameter in.
The *Query* filter parameter can be only be used at the beginning of the filter list.

* To apply filters in C# code, see "Filters" and "Applying-multiple-filters" chapters
  in [Using the Domain Object Model](Using-the-Domain-Object-Model#Filters) article.
* See "Filter" in [RestGenerator specification](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md#filters)
  on how to apply filter in REST request.

The following table shows examples of reading data from the REST service with some filter parameters.
The "**Processing workflow**" describes how the data is processed internally by the Rhetos applications.
"Example URL" is using the entities from the [Bookstore](https://github.com/Rhetos/Bookstore) demo application.

<small>

| GET request filter parameters example | Data processing workflow | Example URL |
| --- | --- | -- |
| No filter parameter | Load() | <http://localhost/Rhetos/rest/Bookstore/Book/> |
| ItemFilter(param1) | Query() => Filter(param1) | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Filter":"CommonMisspelling"}]> |
| ItemFilter(param1), ItemFilter(param2) | Query() => Filter(param1) => Filter(param2) | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Filter":"CommonMisspelling"},{"Filter":"SomeOtherFilter"}]> |
| FilterBy(param1) | Load(param1) | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Filter":"ComplexSearch"}]> |
| FilterBy(param1), ItemFilter(param2) | Load(param1) => Filter(param2)<br>**ERROR**: *ItemFilter(param2) works only on a query, not on loaded array.* | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Filter":"ComplexSearch"},{"Filter":"CommonMisspelling"}]> |
| ItemFilter(param1), FilterBy(param2) | Query() => Filter(param1) => Load(param2)<br>**ERROR**: *FilterBy(param2) cannot take previous data as an input.* | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Filter":"CommonMisspelling"},{"Filter":"ComplexSearch"}]> |
| Query(param1), ItemFilter(param2) | Query(param1) => Filter(param2) |
| ItemFilter(param1), Query(param2) | Query() => Filter(param1) => Query(param2)<br>**ERROR**: *Query(param2) cannot take previous data as an input.* |
| GenericPropertyFilter(param1) | Query() => Filter(param1) | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Property":"Code","Operation":"Contains","Value":"2"}]> |
| FilterBy(param1), GenericPropertyFilter(param2) | Load(param1) => Filter(param2)<br>**WARNING**: *This is inefficient because the Load method already loaded more data from database then required.* | <http://localhost/Rhetos/rest/Bookstore/Book/?filters=[{"Filter":"ComplexSearch"},{"Property":"Code","Operation":"Contains","Value":"2"}]> |

</small>

In the examples above, the result will be the same it you replace *ItemFilter* with a *ComposableFilterBy*.

## Filter name vs. parameter type

Both filter and other read methods ...

* Ime filtera == tip, može biti `System.String[]`.

```C
ComposableFilterBy 'System.String[]'
    '(query, repository, parameter) =>
    {
        return query.Where(item => parameter.Contains(item.Oznaka));
    }';
```

<http://localhost/RhetosServer/Rest/Skladista/Primka/?filters=[{"Filter":"System.String[]","Value":["A","B"]}]>
