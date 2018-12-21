## Contributing

Please read the general guidelines on contributing to our **wiki articles** at [How to Contribute](How-to-Contribute).

## Structure of the tutorial article

Each tutorial article should have the structure as presented below.
The structure of the tutorial article should be compatible with a coding workshop, in a way that each topic is a task for developer with provided solution and an explanation.

The text in *square brackets* should be replaced, and the brackets omitted.

    [Introduction]

    [Table of contents]

    ## [Topic 1]

    [1. Example of a development task or an issue that needs to be solved]

    [2. Solution for the given task, usually a DSL script]

    [3. Explanation of the solution, bringing attentions to the key elements
        in the solution and other issues that the developer needs to understand]

    ## [Topic 2]

    ...

## Examples

These tutorial articles follow the given structure, and are a good example for writing new articles:

* [Data structures and relationships](https://github.com/Rhetos/Rhetos/wiki/Data-structures-and-relationships)
* [Polymorphic concept](https://github.com/Rhetos/Rhetos/wiki/Polymorphic-concept)
* [RowPermissions concept](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)

## Planned documentation

* Tutorials
  * Rhetos DSL (CommonConcepts)
    * Create your first Rhetos application (CommonConcepts, DSL scripts, RestGenerator)
    * Data structures and relations (Bookstore example)
    * Simple business rules
    * Persisting the computed data (improve, use Bookstore example)
    * Filters and validations
    * Low-level concepts (SaveMethod, RepositoryUses)
    * Event sourcing, Business process
    * Full-text search
  * Infrastructure
    * Create new DSL concepts and code generators
    * Unit testing Rhetos applications
    * Debugging Rhetos applications
    * Deployment process
    * Internationalization
    * Using the external C# methods
    * Reporting
    * Upgrading to a new Rhetos version
* Reference manual
* A sample project with best practices.
  * [Bookstore](https://github.com/Rhetos/Bookstore) demo application is created.
    We wand to expand it with additional design patterns and components.
* An application startup project
  * Build.bat (download Rhetos server binaries, add config files, MSBuild, deploy, unit tests)
  * [ProjectName]RhetosExtension (algorithms, DSL extensions, feature implementations don't reference DOM)
  * [ProjectName]RhetosRuntime (feature implementations that reference DOM)
  * Create a Yeoman generator
