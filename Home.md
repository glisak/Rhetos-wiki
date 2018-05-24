## Introduction

Welcome to Rhetos wiki!

Rhetos is a DSL framework that enables you to create your own *domain-specific language* to build server applications.
After an application developer describes the application in that language (DSL script), Rhetos will
use it to generate the application server, including the application's database,
business layer object model (C#) and web API (REST, SOAP, OData, ...).

For an overview of the Rhetos architecture and principles, see:
* [What is Rhetos](https://github.com/Rhetos/Rhetos/wiki/What-is-Rhetos)

## Application development

Setup the application development environment:

1. [Prerequisites](https://github.com/Rhetos/Rhetos/wiki/Prerequisites)
2. [Development Environment Setup](https://github.com/Rhetos/Rhetos/wiki/Development-Environment-Setup)

Rhetos DSL examples:
1. How to create a data model: [Data structures and simple business rules](https://github.com/Rhetos/Rhetos/wiki/Data-structures-and-simple-business-rules)
2. [Persisting the computed data](https://github.com/Rhetos/Rhetos/wiki/Persisting-the-computed-data)
3. Implementing entity inheritance and common interfaces: [Polymorphic concept](https://github.com/Rhetos/Rhetos/wiki/Polymorphic-concept)
4. Low-level database development: [Database objects](https://github.com/Rhetos/Rhetos/wiki/Database-objects)
5. How to implement complex server commands: [Action concept](https://github.com/Rhetos/Rhetos/wiki/Action-concept)
6. [Temporal data and change history](https://github.com/Rhetos/Rhetos/wiki/Temporal-data-and-change-history)
7. Row permissions business rules: [RowPermissions concept](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)
8. Reports: [TemplaterReport](https://github.com/Rhetos/Rhetos/wiki/TemplaterReport)

Recommended documentation:

* [Rhetos DSL syntax](https://github.com/Rhetos/Rhetos/wiki/Rhetos-DSL-syntax)
* [Rhetos coding standard](https://github.com/Rhetos/Rhetos/wiki/Rhetos-coding-standard)
* [Rhetos RESTful web API plugin](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md)
* [User authentication and authorization](https://github.com/Rhetos/Rhetos/wiki/User-authentication-and-authorization)
* [Using the Domain Object Model](https://github.com/Rhetos/Rhetos/wiki/Using-the-Domain-Object-Model)
* [Data migration](https://github.com/Rhetos/Rhetos/wiki/Data-migration)

## Recommended plugins

* [CommonConcepts](https://github.com/Rhetos/Rhetos/tree/master/CommonConcepts) contains basic concepts for building applications, such as entities, validations and computations.
* [ActiveDirectorySync](https://github.com/Rhetos/ActiveDirectorySync). Rhetos supports Windows Authentication by default. This package synchronizes Rhetos user roles with Active Directory, allowing user permissions to be defined by AD user groups.
* [AspNetFormsAuth](https://github.com/Rhetos/AspNetFormsAuth), provides Forms Authentication to Rhetos applications.
* [RestGenerator](https://github.com/Rhetos/RestGenerator) automatically generates RESTful JSON web API for all entities and other readable or writable data structures that are defined in a Rhetos application. Additionally allows executing actions and downloading reports.
* [MvcModelGenerator](https://github.com/Rhetos/MvcModelGenerator) generates ASP.NET MVC model for all entities and other queryable data structures, for use in other web applications when accessing the Rhetos server.
* [ODataGenerator](https://github.com/Rhetos/ODataGenerator) generates OData interface (Open Data Protocol) for all entities and other queryable data structures that are defined in a Rhetos application.
* [I18NFormatter](https://github.com/Rhetos/I18NFormatter), enables localization of Rhetos applications using [GetText / PO](http://en.wikipedia.org/wiki/Gettext) standard.
* [LegacyRestGenerator](https://github.com/Rhetos/LegacyRestGenerator), an old version of REST API, available for backward compatibility.
