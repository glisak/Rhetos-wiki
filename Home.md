Welcome to the Rhetos wiki!

Rhetos is a DSL framework that enables you to create your own *domain-specific language* to build server applications.
After an application developer describes the application in that language (DSL script), Rhetos will
use it to generate the application server, including the application's database,
business layer object model (C#) and web API (REST and SOAP).

Recommended documentation:

* Setup the application development environment
  * [Prerequisites](https://github.com/Rhetos/Rhetos/wiki/Prerequisites)
  * [Development Environment Setup](https://github.com/Rhetos/Rhetos/wiki/Development-Environment-Setup)
* Application development
  * [Rhetos DSL examples](https://github.com/Rhetos/Rhetos/wiki/Rhetos-DSL-examples)
  * [Rhetos coding standard](https://github.com/Rhetos/Rhetos/wiki/Rhetos-coding-standard)
  * [Rhetos RESTful web API plugin](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md)
  * [Using the Domain Object Model](https://github.com/Rhetos/Rhetos/wiki/Using-the-Domain-Object-Model)
  * [Data migration](https://github.com/Rhetos/Rhetos/wiki/Data-migration)

Recommended plugins:

* [CommonConcepts](https://github.com/Rhetos/Rhetos/tree/master/CommonConcepts) contains basic concepts for building applications, such as entities, validation and computations.
* [ActiveDirectorySync](https://github.com/Rhetos/ActiveDirectorySync). Rhetos supports Windows Authentication by default. This package synchronizes Rhetos user roles with Active Directory, allowing user permissions to be defined by AD user groups.
* [AspNetFormsAuth](https://github.com/Rhetos/AspNetFormsAuth), provides Forms Authentication to the Rhetos applications.
* [RestGenerator](https://github.com/Rhetos/RestGenerator) automatically generates RESTful JSON web API for all entities and other readable or writable data structures that are defined in a Rhetos application. Additionally allows executing actions and downloading reports.
* [MvcModelGenerator](https://github.com/Rhetos/MvcModelGenerator) generates ASP.NET MVC model for all entities and other queryable data structures, for use in other web application when accessing the Rhetos server.
* [ODataGenerator](https://github.com/Rhetos/ODataGenerator) generates OData interface (Open Data Protocol) for all entities and other queryable data structures that are defined in a Rhetos application.
* [I18NFormatter](https://github.com/Rhetos/I18NFormatter), enables localization of Rhetos applications using [GetText / PO](http://en.wikipedia.org/wiki/Gettext) standard.
* [LegacyRestGenerator](https://github.com/Rhetos/LegacyRestGenerator), an old version of REST API, available for backward compatibility.
