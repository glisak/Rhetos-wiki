## Users and roles

By using a hierarchy of roles, the administrator can create groups of users and permissions, or connect them directly.

The following diagram shows the Rhetos entities in the *Common* module that determine the user's roles and permissions. The diamond shapes represent the associative entities.

[[images/claims.png|Roles and Claims]]

### Common administration activities

All administration activities are performed by modifying the data in the entities from the diagram above. For example:

* To create a new user, insert the record in the *Common.Principal* entity.
* To configure the user's permissions, enter the data in *Common.PrincipalHasRole* or *Common.PrincipalPermission*.
* The roles and the role permissions are usually preconfigured by the development team (entering the data in *Common.Role* and *Common.RolePermission*).
* If the forms authentication is used, instead of the Windows authentication, check for the additional administration activities in the [AspNetFormsAuth documentation](https://github.com/Rhetos/AspNetFormsAuth/blob/master/Readme.md).

### Additional features

1. For **automatic synchronization** of Rhetos user roles with the user groups in **Active Directory**,
  add the [ActiveDirectorySync](https://github.com/Rhetos/ActiveDirectorySync) plugin package to the Rhetos application.
  This will allow the domain administrator to indirectly set the user permissions in the Rhetos application.
2. The permission check can be turned off in a development environment by setting the following values in the Rhetos server's *web.config* file:
    * "Security.AllClaimsForUsers" - The value should contain a comma-separated list of users with server computer name (formatted `username@servername, ...`) that automatically have full permissions. Examples:
      * Domain user on a shared server: `<add key="Security.AllClaimsForUsers" value="mydomain\myusername@myserver" />`.
      * Local windows user without Windows domain: `<add key="Security.AllClaimsForUsers" value="mypc\myusername@mypc" />`.
      * Forms Authentication user "admin": `<add key="Security.AllClaimsForUsers" value="admin@myserver" />`.
    * "BuiltinAdminOverride" - If set to "True", the user that is a local administrator will have full permissions. This option works only for Windows authentication, and if the web server is able to check the user's Windows groups (usually in development environment). Use the following option otherwise.
3. If "AuthorizationAddUnregisteredPrincipals" is set to "True" (in *web.config* file), an entry will automatically be inserted to the *Common.Principal* table for each new user on the first login. The created user will have no permissions by default, but the additional functionality can be added to automatically initialize the user's roles and permissions ([ActiveDirectorySync](https://github.com/Rhetos/ActiveDirectorySync), for example).

## Claims

Each record in a *Common.Claim* entity represents a basic element of access. is an operation on an entity (read/insert/update/delete) or an action.

The list of claims is created on deployment, and does not change during run-time.
There are claims generated for each of the following objects:

* For each entity: Read, New, Edit, Remove
* For each read-only data structure: Read
* For each action: Execute
* For each report: Download

Please note that the records in *Common.Claim* table are automatically updated on each deployment. If a new custom claim is inserted into the table manually or by a DataMigration script, it will be automatically removed (deactivated) on the next deployment.

A custom claim can be created in a DSL script by using the CustomClaim concept. Example:

    CustomClaim 'SomeModule.SomeResource' 'CustomOperationName';

## Inserting the permissions on deployment

There is often a need to insert some permissions as the new version of the application is deployed. This usually means inserting or updating the data in tables *Common.Role*, *Common.RolePermission* or *Common.PrincipalPermission*.

The recommended solution is to include [Rhetos.AfterDeploy](https://github.com/Rhetos/AfterDeploy) package, and write the AfterDeploy SQL scripts that will insert, update or delete the permission records in the database to match the expected permissions.

Note: The DataMigration scripts are often the first solution that comes to mind, but there are issues with this approach:

* DataMigration scripts are intended for incremental modifications. This means that when changing permissions, new DataMigration scripts should be added to correct the existing permissions. It is simpler to maintain an AfterDeploy script that is executed on each deployment, that contains a list of wanted permissions and updates the records in database to match the list.
* The claims in *Common.Claim* table are generated automatically at the end of each deployment (see the *DeployPackages.log*). The data-migration scripts are executed at the beginning of the deployment, so the inserted permissions will not include the new claims that will be generated at the end. Possible workaround is to insert the claims along with the permissions, but the AfterDeploy script is much better solutions.

## See also

* [User authentication and authorization](https://github.com/Rhetos/Rhetos/wiki/User-authentication-and-authorization) - Other options for user authorization.
