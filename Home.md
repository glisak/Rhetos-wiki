Welcome to the Rhetos wiki!

Rhetos is a DSL framework that enables you to create your own *domain-specific language* to build server applications.
After an application developer describes the application in that language (DSL script), Rhetos will
use it to generate the application server, including the application's database,
business layer object model (C#) and web API (REST and SOAP).

Recommended documentation:

* [Installation instructions](https://github.com/Rhetos/Rhetos/blob/master/Readme.md)
* [Rhetos DSL examples](https://github.com/Rhetos/Rhetos/wiki/Rhetos-DSL-examples)
* [Rhetos coding standard](https://github.com/Rhetos/Rhetos/wiki/Rhetos-coding-standard)
* [Rhetos RESTful web API plugin](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md)
* [Forms Authentication plugin](https://github.com/Rhetos/Rhetos/blob/master/AspNetFormsAuth/Readme.md)
* [Active Directory integration plugin](https://github.com/Rhetos/Rhetos/blob/master/ActiveDirectorySync/Readme.md), if not using Forms Authentication
* [Data migration](https://github.com/Rhetos/Rhetos/wiki/Data-migration)

Recommended plugins:

* [CommonConcepts](https://github.com/Rhetos/Rhetos/tree/master/CommonConcepts) contains basic concepts for building applications.
* [ActiveDirectorySync](https://github.com/Rhetos/Rhetos/tree/master/ActiveDirectorySync) synchronizes the Rhetos principals and roles with Active Directory by automatically adding or removing principal-role and role-role membership relations. Note then Rhetos supports Windows Authentication out-of-the-box, event without this package.
* [AspNetFormsAuth](https://github.com/Rhetos/Rhetos/tree/master/AspNetFormsAuth), provides Forms Authentication to Rhetos server applications.
* [RestGenerator](https://github.com/Rhetos/RestGenerator) automatically generates RESTful web API for all entities and other readable or writable data structures that are defined in a Rhetos application. Additionally allows executing actions and downloading reports.
* [MvcModelGenerator](https://github.com/Rhetos/MvcModelGenerator) generates ASP.NET MVC model for all entities and other queryable data structures, for use in other web application when accessing the Rhetos server.
* [ODataGenerator](https://github.com/Rhetos/ODataGenerator) generates OData interface (Open Data Protocol) for all entities and other queryable data structures that are defined in a Rhetos application.
* [I18NFormatter](https://github.com/Rhetos/I18NFormatter), enables localization of Rhetos applications using [GetText / PO](http://en.wikipedia.org/wiki/Gettext) standard.
