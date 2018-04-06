Rhetos is a DSL platform for Enterprise Application Development.

* It enables developers to create a **Domain-Specific Programming Language** and use it to write their applications.
* There are libraries available with ready-to-use implementations of many standard business and design patterns or technology integrations.

Rhetos works as a compiler that generates the business application from the source written in the DSL scripts.

* It is focused on the back-end development: It generates the business logic layer and the database.
* Rhetos application generator can be used for application scaffolding, but its real strength is utilized when using DSL scripts as the *source* and never manually modify the generated code.

## Why was Rhetos created

Enterprise business applications can be hard to develop and maintain because they typically contain a huge number of features that must work together.

1. Rhetos allows fast development of standard business patterns in the application, by simply **declaring the features**.
2. Rhetos framework puts emphasis on principles that increase maintainability of the complex applications.

The basic principles:

* **Standardization** of business and technology patterns implementation:
  * Write high-quality implementations of standard patterns and reuse them.
* Declarative programming:
  * Try to develop most of the application's business logic by simply declaring the features.
  * Reduce the amount if the imperative code in order to decrease code coupling and increase long-term maintainability.
  * Developers are encouraged to recognize business and technology patterns, and create DSL concepts to simplify the use for those patterns in the business application development.
  * DSL concept are able to encapsulate a much larger scope of patterns then just writing a class or a function. The implemented pattern can affect any part of the system, from the database structure to the web API, extend other existing features or implement the crosscutting concerns.
* Code decoupling and Extensibility:
  * Each feature is implemented as a DSL concept with minimal number of dependencies.
  * Additional features are added as an extension from outside, not increasing complexity of the existing features' implementation.

## What does the Rhetos platform contain

1. Programming language development tools:
    * A framework for developing DSL concepts (as plugins) and code generators.
2. Ready-to-use DSL libraries:
    * Implementations of standard business and design patterns
    * Web API generators (SOAP, REST, OData)
    * Authentication plugins
    * Reporting
    * Internationalization
    * ...
3. Application development & deployment:
    * Setup the Rhetos server
    * Use existing libraries with DSL concepts and technology-specific solutions (NuGet packages)
    * Develop your application: Write DSL scripts, custom DSL concept (generate new NuGet packages)
    * Deploy all the packages to the Rhetos server to generate the application's business layer, database, web service and other features.

## Rhetos DSL

Rhetos DSL (a programming language) is a set of **concepts**.

* Each concept represent a business patterns or a software design pattern.
* Each concept has defined syntax (DSL grammar).
* Each concept has code generators that generate the application code, database structure, other concepts, web API and other stuff.
* Existing libraries include many standard concepts, but the application developers can easily add new concepts to extend the programming language.

Developers write the business application's code in the DSL scripts:

* A DSL script can be understood as a set of **statements**.
* Each statement declares a feature of the application.
* DSL scripts often include SQL and C# code snippets.
* DSL scripts can contain calls to external dlls.

Rhetos works as compiler to generate the final application from the DSL scripts.

* Rhetos generates the human-readable C# code for the server application, and compiles it to create the web application.
* The generated application uses Entity Framework to access the generated database.
* There are plugins available for generating different types of web services, helper classes for client development and similar.

Database upgrade:

* The database is not generated from scratch on each deployment, it is upgraded instead.
* Rhetos protects all the data if the database structure is changed when deploying a new version of the business application.
* Data-migration SQL scripts can be used to migrate the data when needed.
