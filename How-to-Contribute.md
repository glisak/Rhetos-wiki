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

* [Polymorphic concept](https://github.com/Rhetos/Rhetos/wiki/Polymorphic-concept)
* [RowPermissions concept](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)
