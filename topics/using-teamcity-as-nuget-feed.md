[//]: # (title: Using TeamCity as NuGet Feed)
[//]: # (auxiliary-id: Using TeamCity as NuGet Feed)

If you want to publish your NuGet packages to a limited audience, for example, to use them internally, you can use TeamCity [as a NuGet feed](https://docs.microsoft.com/en-us/nuget/hosting-packages/overview). You can configure multiple NuGet feeds for a TeamCity project.

The built-in TeamCity NuGet feed supports API v1/v2/v3.

<note>

TeamCity running on any of the supported operating systems (Windows, Linux, macOS) can be used as a NuGet feed.
</note>

## Enabling NuGet Feed

To start using TeamCity as a NuGet Server, you need to add a NuGet feed at the project level — in __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | NuGet Feed__. Multiple NuGet feeds can be configured for a project.

Click __Add new NuGet Feed__ to create a feed. Optionally, enable _Automatic packages indexing_ for the current project and its subprojects.

The following feed endpoints are available:
* `http://<teamcityUrl>/<authSchema>/app/nuget/feed/<projectName>/<feedName>/v1`
* `http://<teamcityUrl>/<authSchema>/app/nuget/feed/<projectName>/<feedName>/v2`
* `http://<teamcityUrl>/<authSchema>/app/nuget/feed/<projectName>/<feedName>/v3/index.json`
   
If you have enabled a [guest user](guest-user.md), you will see the toggle between Basic HTTP authentication and Guest authentication for NuGet feed endpoints.
* __Basic HTTP authentication__ (with httpAuth prefix): to access packages, a user must have the "_View project_" permission.
* __Guest authentication__ (with guestAuth prefix): if a guest user login is enabled, packages will be visible to all users that have access to the TeamCity server.

To reference the TeamCity project feeds, use parameters as follows:
 
```Shell  
`teamcity.nuget.feed.<authSchema>.<projectName>.<feedName>.<apiVersion>
```   
where:
* `authSchema` could be `guestAuth`/`httpAuth`
* `apiVersion` is `v1`/`v2`/`v3`, for instance: `teamcity.nuget.feed.httpAuth._Root.default.v2`

On upgrading TeamCity to version 2018.2 or later, the deprecated parameters referencing the global NuGet feed will be automatically converted to the references to the default NuGet feed:

<table>
<tr>
<td>

Deprecated reference

</td>

<td>

Current reference

</td></tr><tr>

<td>

`teamcity.nuget.feed.server`

</td>

<td>

`teamcity.nuget.feed.guestAuth._Root.default.v2`

</td></tr><tr>

<td>

`teamcity.nuget.feed.auth.server`

</td>

<td>

`teamcity.nuget.feed.httpAuth._Root.default.v2`

</td></tr><tr>

<td>

`system.teamcity.nuget.feed.auth.serverRootUrlBased.server`
 
</td>
 
<td>
 
`teamcity.nuget.feed.httpAuth._Root.default.v2`

</td></tr></table>

## Indexing NuGet Packages

### Indexing packages published as artifacts

By default, TeamCity will not add `.nupkg` artifacts published by builds into the project NuGet feed. You can select one of the following options:
* To index packages published by the selected build configurations only, add the [NuGet packages indexer](nuget-packages-indexer.md) build feature to these build configurations.
* To index all `.nupkg` files published as build artifacts in the project, enable _Automatic Packages Indexing_ in the __NuGet Feed__ section of the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__.
* Use the [NuGet Pack](nuget-pack.md) build step with the _Publish created packages to build artifacts_ checkbox.  
An agent indexes the `.nupkg` files while publishing build artifacts.

<note>

Deleting a NuGet feed with all its contents from a project will remove all [NuGet Packages Indexer](nuget-packages-indexer.md) features pointing to this feed.
</note>

### Using NuGet push command

To publish the `.nupkg` file into the TeamCity NuGet feed during the build, you can specify the NuGet feed URL as a package source and the `%\teamcity.nuget.feed.api.key%` value as a feed key in the following build steps:
*  [NuGet Publish](nuget-publish.md)
* .[NET CLI](net.md) with the `nuget push` command

Note that pushing a package is only possible from within a build. Any attempts to push external packages into the TeamCity feed will fail.

### Symbol Packages

[//]: # (AltHead:symbols)

Publishing of NuGet [symbol packages](http://docs.nuget.org/ndocs/create-packages/symbol-packages) to an internal TeamCity feed may cause issues when using an external source server. See the [corresponding issue](https://youtrack.jetbrains.com/issue/TW-25512) in our public tracker.

## Using TeamCity NuGet Feed

You can add TeamCity NuGet feeds as package sources on your developer machine. For example, to use packages during development, use the [`nuget sources`](https://docs.microsoft.com/en-us/nuget/tools/cli-ref-sources) command or NuGet package management in your IDE.

You can use TeamCity NuGet feeds to restore packages in your builds via the [NuGet Installer](nuget-installer.md) and [NuGet Publish](nuget-publish.md) build runners, or the [.NET](net.md) runner with the MSBuild `restore` target. Obsolete [MSBuild](msbuild.md) and [Visual Studio (sln)](visual-studio-sln.md) runners are also supported.   
TeamCity uses own [credential provider](https://docs.microsoft.com/en-us/nuget/reference/extensibility/nuget-exe-credential-providers) to automatically authenticate requests to the private TeamCity NuGet feeds.

>The packages available in the feed are bound to the builds' artifacts: they are removed from the feed when the artifacts of the build which produced them are [cleaned up](teamcity-data-clean-up.md). To avoid clean-up of artifacts in a specific build, [pin this build](build-actions.md#Pin+Build).

Internet Explorer settings may need to be set to trust the TeamCity server when working in a mixed authentication environment.

<!--[//]: # (Internal note. Do not delete. "Using TeamCity as NuGet Feedd342e197.txt")-->
