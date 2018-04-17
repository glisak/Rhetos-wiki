## Authentication

Rhetos supports **Windows Authentication** by default.

If the web application must be accessed from outside of Windows Domain (for example, a public application), use the **Forms Authentication** instead, by including the [AspNetFormsAuth](https://github.com/Rhetos/AspNetFormsAuth) plugin package.

## Authorization

There are two subsystems available for the authorization of user access in the Rhetos applications.

1. [Basic permissions (Claims)](https://github.com/Rhetos/Rhetos/wiki/Basic-permissions)
    * The access rights are typically defined by the roles that a user has (entered by the administrator).
    * The basic element of access is an operation on an entity (read/insert/update/delete) or an action.
2. [Row permissions](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)
    * Programmable permissions.
    * The access rights are implemented on an entity to allow some users access **a subset** of the entity's records.
