The `Action` concept is intended for implementing server commands. The code with the action will be execution in a single transaction, so that in case of an error the whole action will be canceled (all-or-nothing).

The action can be implemented in two ways:

* Using the `Action` concept, which assumes writing the C# code directly in the .rhe script, or in a referenced .cs script file.
* Using the `ExtAction` concept, where the C# code is implemented in a plugin dll.

Both ways have their pros and cons, and the choice depends on a situation. The `Action` concept is good for simple code snippets, while `ExtAction` is usually used for more complex implementations.

Tips:

* Action is executed in a single transaction. This means that it is not possible to log an exception error in the database.
* Row permission and other security claims are not checked inside the action.
    - The user need to have permission to execute the action (has the *Execute* claim on the action),
      but not for the resources that are used inside the action. The code inside the action has access to all data.
    - The user's permission can be explicitly checked in the action by using `Common.RowPermissionsReadItems`
      and `Common.RowPermissionsWriteItems` filters, or `IAuthorizationManager` for checking security claims.
* Reference property in the action parameters will not be bound the the database by the ORM (the lazy-load is not suppered).
  Also the whole entity cannot be sent as a reference property parameter.
* Exceptions that are cached in the action must be rethrown.

Example:

```
Module Demo
{
    Action CreatePrincipal '(parameters, repository, userInfo) =>
    {
        var principal = new Common.Principal
        {
            ID = parameters.ID,
            Name = parameters.Name
        };
        
        repository.Common.Principal.Insert(principal);
    }'
    {
        Guid ID;
        ShortString Name;
    }
}
```

Example for the `ExtAction` concept:

* DSL:

    ```
    Module Demo
    {
        ExtAction CreatePrincipal 'Demo.Rhetos.Principals, Demo.Rhetos' 'CreatePrincipal'
        {
            Guid ID;
            ShortString Name;
        }
        
    }
    ```

* C#:

    ```
    // This method is implemented in the assembly Demo.Rhetos, in class Principals.
    public static void CreatePrincipal(CreatePrincipal parameters, DomRepository repository, IUserInfo userInfo)
    {
            var principal = new Common.Principal
            {
                ID = parameters.ID,
                Name = parameters.Name
            }

            repository.Common.Principal.Insert(principal);

            var notification = "...";
            Notify(notification, repository, userInfo, Notify.Admin|Notify.PowerUsers); // A helper method for notifications.
    }
    ```
