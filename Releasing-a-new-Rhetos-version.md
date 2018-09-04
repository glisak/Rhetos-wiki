1. Before you start:
    * [ ] Make sure you are working on the latest version of the master branch (git pull).
    * [ ] Check out the [github network](https://github.com/Rhetos/Rhetos/network)
       and [pull requests](https://github.com/Rhetos/Rhetos/pulls) for any forgotten branches.
    * [ ] List any other core packages that need to be released with the new framework version.
    * [ ] Check the last [release number](https://github.com/Rhetos/Rhetos/releases) and decide on the new version number.
        A new release usually just increases the minor version by 1 (the second number).
      * Note: The version number format must be compliant with [semantic versioning](https://semver.org/).
2. Build:
    * [ ] Update *ChangeLog.md* file based on the commit history since the previous release.
    * [ ] Set the release version number in *Build.bat* file (probably it is already set), and the *Prerelease* to an empty value:<br/>
        `SET Prerelease=`
    * [ ] Run full build in the command prompt:<br/>
        `Clean.bat && Build.bat && Test.bat`
    * [ ] Verity that the build is successful: the console output should end with "Test.bat SUCCESSFULLY COMPLETED.".
    * [ ] In the *Install* subfolder: zip the *Rhetos* folder to *Rhetos.&lt;NEW VERSION&gt;.zip* (for example Rhetos.1.2.0.zip).
3. Publish:
    * [ ] [Private builds only] Copy the 1 zip and 2 nupkg files from Install folder to your company's shared network storage (ask your system administrator for the location).
    * [ ] Commit your repository changes, **except Build.bat file**, with comment "Release &lt;NEW VERSION&gt;.".
        For example "Release 1.2.0.".
      * Note: If there is nothing to commit, simply do the next step on the last commit.
    * [ ] In your repository create a new tag "v&lt;NEW VERSION&gt;" at the last commit ("Release ...").
        For example "v1.2.0".
    * [ ] Push your repository to GitHub (set the option *Include Tags*).
    * [ ] [Private builds only] Publish the new NuGet package to your company's NuGet gallery (ask your system administrator for the location).
    * [ ] Publish the Rhetos NuGet package to the public [NuGet gallery](https://www.nuget.org/packages/manage/upload).
    * [ ] Add the Rhetos server zip file to GitHub release: Open [tags on GitHub](https://github.com/Rhetos/Rhetos/tags)
        => At the tag for the newly released version click "Create release"
        => **Check** that the opened form displays the tag you have just selected
        => Drag & drop the zip file to the "**Attach binaries**" box
        => Click "Publish release".
4. Prepare the code for further development:
    * [ ] In *Build.bat* increase the second version number by 1 and set the third to 0 (for example from 1.2.5 to 1.3.0). Set the `Prelease` version to `auto`, so that the source is ready for the development of the next release:<br/>
          `SET Version=...current +0.+1.0`<br/>
          `SET Prerelease=auto`
    * [ ] Run *Build.bat*.
    * [ ] Commit with comment "Development &lt;NEXT VERSION&gt;". For example "Development 1.3.0.".
    * [ ] Push the repository to GitHub (set the option *Include Tags*).
