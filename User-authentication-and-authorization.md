## Authentication

### Windows Authentication

Rhetos supports Windows Authentication by default, so the users can automatically log in to the web application inside the Windows domain.

For additional administration features, such as automatic synchronization with the **Active Directory user groups**, see [Basic permissions](https://github.com/Rhetos/Rhetos/wiki/Basic-permissions).

### Forms Authentication

If the web application must be accessed from outside of Windows Domain (for example, a public application), use the **forms authentication** instead. The users will log in by typing a username and a password.

* Install forms authentication by including the [AspNetFormsAuth](https://github.com/Rhetos/AspNetFormsAuth) plugin package and follow the additional instruction in the package's readme file.
* Optional plugin [AspNetFormsAuthImpersonation](https://github.com/Rhetos/AspNetFormsAuthImpersonation) allows the administrator to log in as another user for debugging and support.
* [SimpleSPRTEmail](https://github.com/Rhetos/SimpleSPRTEmail) plugin adds an authentication service method "Send password reset token" for sending emails if a user forgot the password.

### Additional authentication options

A custom authentication plugin may be added by implementing the [IUserInfo](https://github.com/Rhetos/Rhetos/blob/master/Source/Rhetos.Utilities/IUserInfo.cs) interface and registering the new implementation as the Rhetos server plugin.

## Authorization

There are two subsystems available for the authorization of user access in the Rhetos applications.

1. [Basic permissions (Claims)](https://github.com/Rhetos/Rhetos/wiki/Basic-permissions)
    * The access rights are typically defined by the roles that a user has (entered by the administrator).
    * The basic element of access is an operation on an entity (read/insert/update/delete) or an action.
2. [Row permissions](https://github.com/Rhetos/Rhetos/wiki/RowPermissions-concept)
    * Programmable permissions.
    * The access rights are implemented on an entity to allow some users access **a subset** of the entity's records.
