## Development

* Contributions are very welcome. The easiest way is to fork the repository, make changes and then make a pull request from your fork. A detailed documentation is available on GitHub: [Collaborating with issues and pull requests](https://help.github.com/categories/collaborating-with-issues-and-pull-requests/). The first time you make a pull request, you may be asked to sign a Contributor Agreement.
* Please run the unit tests before sending the pull request. Rhetos contains two types of unit tests: standard unit tests (in Rhetos.sln), and integration tests (in CommonConceptsTest.sln) that can be run only after a successful deployment.

## Writing tutorial articles

### Format

The documentation is stored on GitHub [wiki](https://github.com/Rhetos/Rhetos/wiki) pages in [markdown](https://guides.github.com/features/mastering-markdown/) format.

Use an offline text editor with spell checker and markdown lint tool to make sure there are no issues with the test formatting.

* For example, use VS Code with plugins: Spell Right, markdownlint and Markdown All in One.
* Exception: Skip the first-level heading, because the GitHub wiki adds it automatically.

### Structure of the tutorial article

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

### Examples

These tutorial articles follow the given structure, and are a good example for writing new articles:

* [Data structures and relationships](https://github.com/Rhetos/Rhetos/wiki/Data-structures-and-relationships)
* [Polymorphic concept](https://github.com/Rhetos/Rhetos/wiki/Polymorphic-concept)
* [RowPermissions concept](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)

### Planned documentation

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
* An application startup project
  * Build.bat (download Rhetos server binaries, add config files, MSBuild, deploy, unit tests)
  * [ProjectName]RhetosExtension (algorithms, DSL extensions, feature implementations don't reference DOM)
  * [ProjectName]RhetosRuntime (feature implementations that reference DOM)
  * Create a Yeoman generator
