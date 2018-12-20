## Introduction

Welcome to Rhetos wiki!

Rhetos is a DSL framework that enables you to create your own *domain-specific language* to build server applications.
After an application developer describes the application in that language (DSL script), Rhetos will
use it to generate the application server, including the application's database,
business layer object model (C#) and web API (REST, SOAP, OData, etc.).

* [What is Rhetos](https://github.com/Rhetos/Rhetos/wiki/What-is-Rhetos), an overview of the Rhetos architecture and principles
* [Roadmap](https://github.com/Rhetos/Rhetos/wiki/Rhetos-platform-roadmap)

Contributions are very welcome, both in learning resources (wiki articles and code samples) or framework and plugins development. See [How to Contribute](How-to-Contribute) for more information.

## Application development with Rhetos

Get started:

1. [Prerequisites](https://github.com/Rhetos/Rhetos/wiki/Prerequisites)
2. [Development environment setup](https://github.com/Rhetos/Rhetos/wiki/Development-Environment-Setup)

Tutorials:

* How to create a data model:
  * [Data structures and relationships](https://github.com/Rhetos/Rhetos/wiki/Data-structures-and-relationships)
  * [Implementing simple business rules](https://github.com/Rhetos/Rhetos/wiki/Implementing-simple-business-rules)
* [Persisting the computed data](https://github.com/Rhetos/Rhetos/wiki/Persisting-the-computed-data)
* Implementing entity inheritance and common interfaces: [Polymorphic concept](https://github.com/Rhetos/Rhetos/wiki/Polymorphic-concept)
* Low-level database development: [Database objects](https://github.com/Rhetos/Rhetos/wiki/Database-objects)
* How to implement complex server commands: [Action concept](https://github.com/Rhetos/Rhetos/wiki/Action-concept)
* [Temporal data and change history](https://github.com/Rhetos/Rhetos/wiki/Temporal-data-and-change-history)
* Row permissions business rules: [RowPermissions concept](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)
* Reports: [TemplaterReport](https://github.com/Rhetos/Rhetos/wiki/TemplaterReport)

Sample application:

* [Bookstore](https://github.com/Rhetos/Bookstore)

Fundamentals:

* [Rhetos DSL syntax](https://github.com/Rhetos/Rhetos/wiki/Rhetos-DSL-syntax)
* [Rhetos RESTful web API plugin](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md)
* [User authentication and authorization](https://github.com/Rhetos/Rhetos/wiki/User-authentication-and-authorization)
* [Using the Domain Object Model](https://github.com/Rhetos/Rhetos/wiki/Using-the-Domain-Object-Model)
* [Data migration](https://github.com/Rhetos/Rhetos/wiki/Data-migration)
* [Rhetos coding standard](https://github.com/Rhetos/Rhetos/wiki/Rhetos-coding-standard)

Support:

* Feel free to ask questions on the [Issues](https://github.com/Rhetos/Rhetos/issues) page by searching the existing ones or adding a new issue.
* [Release notes](https://github.com/Rhetos/Rhetos/blob/master/ChangeLog.md)

## Recommended plugins

* [CommonConcepts](https://github.com/Rhetos/Rhetos/tree/master/CommonConcepts) contains basic concepts for building applications, such as entities, validations and computations.
* [ActiveDirectorySync](https://github.com/Rhetos/ActiveDirectorySync). Rhetos supports Windows Authentication by default. This package synchronizes Rhetos user roles with Active Directory, allowing user permissions to be defined by AD user groups.
* [AspNetFormsAuth](https://github.com/Rhetos/AspNetFormsAuth), provides Forms Authentication to Rhetos applications.
* [RestGenerator](https://github.com/Rhetos/RestGenerator) automatically generates RESTful JSON web API for all entities and other readable or writable data structures that are defined in a Rhetos application. Additionally allows executing actions and downloading reports.
* [AfterDeploy](https://github.com/Rhetos/AfterDeploy), simplifies the handling of SQL scripts that need to be executed on each deployment.
* [LightDMS](https://github.com/Rhetos/LightDMS),  light document versioning system plugin for Rhetos. Supports database table storage, SQL Server filestream and Azure Blob Storage.
* [ODataGenerator](https://github.com/Rhetos/ODataGenerator) generates OData interface (Open Data Protocol) for all entities and other queryable data structures that are defined in a Rhetos application.
* [I18NFormatter](https://github.com/Rhetos/I18NFormatter), enables localization of Rhetos applications using [GetText / PO](http://en.wikipedia.org/wiki/Gettext) standard.
* [LegacyRestGenerator](https://github.com/Rhetos/LegacyRestGenerator), an old version of REST API, available for backward compatibility.
* [MvcModelGenerator](https://github.com/Rhetos/MvcModelGenerator) generates ASP.NET MVC model for all entities and other queryable data structures, for use in other web applications when accessing the Rhetos server.

## Framework development

* [How to Contribute](https://github.com/Rhetos/Rhetos/wiki/How-to-Contribute)
* [Releasing a new Rhetos version](Releasing-a-new-Rhetos-version)
