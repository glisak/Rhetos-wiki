This article describes how to setup the development environment for developing applications with Rhetos framework.

Table of contents:

1. [Prerequisites](#prerequisites)
2. [Get Rhetos server binaries](#get-rhetos-server-binaries)
3. [Database setup](#database-setup)
4. [Packages configuration](#packages-configuration)
5. [IIS setup](#iis-setup)
6. [IIS Express setup](#iis-express-setup)
7. [Configure your text editor for DSL scripts (*.rhe)](#configure-your-text-editor-for-dsl-scripts-rhe)

## Prerequisites

Before following the instructions in this article, **make sure that all the [Prerequisites](https://github.com/Rhetos/Rhetos/wiki/Prerequisites) are installed**.

## Get Rhetos server binaries

Inside your application's development folder, create a subfolder named "<MyApplicationName>RhetosServer" or "RhetosServer", where the Rhetos server binaries will be placed. The following documentation will refer to this folder as "RhetosServer folder".

Option A: Download the zip file for selected release version from <https://github.com/Rhetos/Rhetos/releases>.

1. Unpack the zip to the RhetosServer folder.

Option B: Build the binaries from source:

1. Use git to clone the repository <https://github.com/Rhetos/Rhetos.git> to a new source folder on your disk:
    * In the command prompt run `git clone https://github.com/Rhetos/Rhetos.git RhetosSource`
2. Open the command prompt in the created Rhetos source folder and run `Build.bat`. Verify that the last printed line is "Build.bat SUCCESSFULLY COMPLETED".
3. The Rhetos server binaries are created in the subfolder **Install\Rhetos**. Copy the concept of **Install\Rhetos** to your application's RhetosServer folder.

## Database setup

1. Create an empty database (SQL Server 2008 or newer, Developer Edition is recommended) named "Rhetos" or "&lt;MyApplicationName&gt;Rhetos".
2. Open folder **RhetosServer\bin** and copy the template file "Template.ConnectionStrings.config" to "ConnectionStrings.config" in the same folder.
3. Edit the existing configuration line in the **ConnectionStrings.config** file to reference the created database. For example, if your database is named "Rhetos" on the local SQL Server with default instance name, simply replace "theServer" to "localhost" and "theDatabase" to "Rhetos".

## Packages configuration

Rhetos packages define the content of the Rhetos server. This section describes **the initialization of the packages configuration files**, but with no packages selected. That will result with an *empty* Rhetos server being deployed.

1. In the **RhetosServer** folder:
    * Copy "Template.RhetosPackages.config" file to "RhetosPackages.config", if the target file does not already exist.
    * Copy "Template.RhetosPackageSources.config" file to "RhetosPackageSources.config", if the target does not already exist.
2. Verify that the RhetosServer is configured correctly by opening command prompt at **RhetosServer\bin** folder and running `DeployPackages.exe`.
    * The last printed line should be "*[Trace] DeployPackages: Done.*".
    * The output may include "*[Error] DeploymentConfiguration: No packages*" and "*[Error] DeployPackages: WARNING: Empty assembly...*", because no packages are provided in the "RhetosPackages.config".
      This is ok for now.

## IIS setup

Follow the steps in this chapter if using IIS (recommended) instead of IIS Express.

1. Start "Internet Information Services (IIS) Manager".
2. Select "Application Pools" => "Add Application Pool...", enter the following data:
    * Name: "RhetosAppPool"
    * NET framework version: "v4.x"
    * Manged pipeline mode: "Integrated"
3. Open "RhetosAppPool" => "Advanced settings" => Change "Identity" => Select "Custom account" => Enter a domain user account that will be used for Rhetos server (on a development computer enter the **developer's account**). If you are **not using** a Windows domain account, see the next paragraph for setting the custom account.
    * Note: This account must have *db_owner* rights for the Rhetos database (see Database setup).
4. *Only for older versions*: If using Rhetos v1.1 or older, "RhetosAppPool" => "Advanced settings" => check "Enable 32-Bit Application".
5. Select "Sites" => Right click "Default Web Site" => "Add application...", enter the following data:
    * Enter Alias: "RhetosServer"
    * Application pool: "RhetosAppPool"
    * Physical path: select the path of the **RhetosServer** folder inside your application's development folder.
6. Select the created "RhetosServer" web application => Open "Authentication" icon:
    * Enable "Windows Authentication"
    * *Disable* all other authentication options.
7. Start your browser, and then type <http://localhost/RhetosServer/> in the address. Verify that the web site is working and the Rhetos server status is displayed on the web page.

If is is not possible to use Windows domain account, the Rhetos service can be set up to use ApplicationPoolIdentity in a development environment:

1. **Skip the following steps** if you are using a Windows domain account.
2. Modify the *RhetosAppPool* to use built-in account "ApplicationPoolIdentity", instead of the developers domain account (Advanced Settings => Identity). This is the default user for a new app pool.
3. Open SQL Server Management Studio:
    * On the SQL Server add a Security => Logins => New Login... => Enter "BUILTIN\IIS_IUSRS".
    * Open the created  login entry (BUILTIN\IIS_IUSRS) => Open "User Mapping" => Select the Rhetos database and mark the "db_owner" checkbox.
4. Allow IIS system accounts read access to the Rhetos server folder and write access to the Rhetos logs folder (the "Logs" subfolder or directly in the Rhetos server folder, depending on the settings in *web.config*), by entering these commands to the command prompt *as administrator*, in the "RhetosServer" folder:

        ICACLS . /grant "BUILTIN\IIS_IUSRS":(OI)(CI)(RX)
        ICACLS . /grant "NT AUTHORITY\IUSR":(OI)(CI)(RX)
        IF NOT EXIST Logs\ MD Logs
        ICACLS .\Logs /grant "BUILTIN\IIS_IUSRS":(OI)(CI)(M)
        ICACLS .\Logs /grant "NT AUTHORITY\IUSR":(OI)(CI)(M)

5. If the web application fails with a file access error, set the required permissions for the IIS system accounts "BUILTIN\IIS_IUSRS" and "NT AUTHORITY\IUSR":
    * Read permissions to the RhetosServer folder.
    * Write permissions to the RhetosServer logs folder ("RhetosServer\Logs", or directly "RhetosServer" folder).
6. Open "RhetosServer\Web.config" file and set the "Security.AllClaimsForUsers" parameter value to your username and computer name, formatted "username@computername". Note that your username may include the domain name prefix.

## IIS Express setup

Follow the steps in this chapter if using IIS Express instead of IIS.
IIS is recommended (see the previous chapter), but IIS Express can be useful if a restricted environment is needed on the development machine.

`SetupRhetosServer.bat` will set up web environment for IIS Express.
It requires few parameters based on which it sets up database, configures
Rhetos server and sets up IIS Express local config which is enough to start Rhetos
server and developing.

Open command prompt in "Source\Rhetos" subfolder and run:

    SetupRhetosServer.bat <WebsiteName> <Port> <SqlServer> <DatabaseName>

SetupRhetosServer.bat command-line arguments:

* *WebsiteName* - name of website in IISExpress config
  (choose any name, it is not directly related to URL of Rhetos web service)
* *Port* - port that Rhetos web service will be listening to if using IIS Express
  (1234, for example)
* *SqlServer* - Microsoft SQL Server instance on which there is/will be database for Rhetos server
* *DatabaseName* - name of database that will Rhetos use, script will create
  database if it doesn't exist.

After running this script and before running Rhetos server, you can manually
configure `IISExpress.config`. It is template config for IIS Express site
with modified `<sites>` section and added `<location>` part at the end of config.
Those two defines which port on localhost Rhetos will be listening and that
security used is based on windows authentication.

If one wants to use different database it is defined in ConnectionStrings.config
in "Source\Rhetos\bin" subfolder.

Note that besides configuring IIS and Database, `SetupRhetosServer.bat` also
deploys *CommonConcepts* and user defined packages to that database. If one chooses
manual setup it is necessary to run ApplyPackages.bat in "Source\Rhetos".

Once configured, Rhetos server can be started by running following command
in cmd while positioned in "Source\Rhetos":

    CALL "C:\Program Files (x86)\IIS Express\IISExpress.exe" /config:IISExpress.config

If using Rhetos v1.1 or older, use the 32-bit "Program Files" folder.

## Configure your text editor for DSL scripts (*.rhe)

The **syntax highlighting** plugins are available for the following text editors.

Visual Studio Code:

1. Open Visual Studio Code => Press Ctrl-Shift-P => Select "Extensions: Open Extensions Folder".
2. In the opened folder, use git to clone the <https://github.com/Rhetos/RhetosVSCode> repository to the subfolder "RhetosVSCode".
3. Restart Visual Studio Code.

Notepad++:

1. Download the [RhetosNppSyntaxHighlight.xml](https://raw.githubusercontent.com/Rhetos/RhetosNPP/master/RhetosNppSyntaxHighlight.xml) file from <https://github.com/Rhetos/RhetosNPP>.
2. Open "Notepad++" => Menu "Language" => "Define your language" => Click "Import..." => Select the downloaded XML file.

SublimeText3:

1. Install the *PackageControl* plugin by following the instructions at <https://packagecontrol.io/installation>.
2. Install the *RhetosDSL* sublime text package: Ctrl-Shift-P, select "install package", select "RhetosDSL".
    * Note: The source code is available at <https://github.com/Hugibeer/RhetosDSLSyntax>.
