[//]: # (title: NuGet Installer)
[//]: # (auxiliary-id: NuGet Installer)

The _NuGet Installer_ build runner performs NuGet [Command-line package restore](http://docs.nuget.org/docs/reference/package-restore#Command-Line_Package_Restore). It can also (optionally) automatically update package dependencies to the most recent ones.

<include from="nuget.md" element-id="nuget-OS"/>

<note>

Make sure that sources that you check out from VCS ([VCS Settings](configuring-vcs-settings.md)) include the `packages` directory from your solution folder.
</note>

>Note that TeamCity Cloud currently doesn't support automatic delivery of tools to [build agents](build-agent.md). To be able to use this runner, you need to download and install the required version of NuGet on the agent. You can do this manually (only on self-hosted agents) or via any convenient utility step at the beginning of the build (for example, [Command Line](command-line.md)). When configuring a NuGet build step, you will need to specify the path to NuGet relatively to the [build checkout directory](build-checkout-directory.md).
> 
{type="warning" instance="tcc"}

NuGet Installer settings:

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

NuGet.exe

</td>

<td>

Select NuGet version to use from the drop-down menu (you need to have [NuGet installed](nuget.md)) or specify a custom path to `NuGet.exe`.

<note>

An appropriate version of .NET Framework installed on the agent machine is required depending on the NuGet.exe version used: NuGet 2.8.6\+ requires .NET 4.5\+, earlier NuGet versions require .NET 4.0.
</note>


</td></tr><tr>

<td>

Path to solution file

</td>

<td>

Specify the path to your solution file (.sln) where packages are to be installed.

</td></tr><tr>

<td>

Restore Mode

</td>

<td>

Select `NuGet.exe restore` (requires NuGet 2.7\+) to restore all packages for an entire solution. The `NuGet.exe install` command is used to restore packages for versions prior to NuGet 2.7, but only for a single `packages.config` file.

</td></tr><tr>

<td>

Restore Options

</td>

<td>

If needed, select:

* __Exclude version from package folder names__: Equivalent to the `-ExcludeVersion` option of the `NuGet.exe install` command. If enabled, the destination folder will contain only the package name, not the version number.
* __Disable looking up packages from local machine cache__: Equivalent to the `-NoCache` option of the `NuGet.exe `

</td></tr><tr>

<td>

Packages Sources

</td>

<td>

Specify the NuGet package sources. If left blank, [`https://nuget.org`](https://nuget.org/) is used to search for your packages.

If you are using a [TeamCity NuGet feed](using-teamcity-as-nuget-feed.md), select it using the 'magic wand' icon <img src="magic-wand.png" alt="Switch to the Sakura UI" height="20" width="20"/> or manually specify the URL from the NuGet Feed section of [project settings](project-administrator-guide.md#Edit+and+View+Modes).
{instance="tc"}

If you use packages from an authenticated feed, configure the [NuGet Feed Credentials](nuget-feed-credentials.md) build feature.   

TeamCity allows you to authenticate using private NuGet feeds. Read more in [NuGet](nuget.md#Authentication+in+private+NuGet+Feeds).


</td></tr><tr>

<td>

Update Packages

</td>

<td>

__Update packages with help of NuGet update command__: Uses the `NuGet.exe update` command to update all packages under the solution. The package versions and constraints are taken from `packages.config` files.

</td></tr><tr>

<td>

Update Mode

</td>

<td>

Select one of the following:

* __Update via solution file__ — TeamCity uses Visual Studio solution file (.sln) to create the full list of NuGet packages to install. This option updates packages for the entire solution.
* __Update via packages.config__ — Select to update packages via calls to `NuGet.exe update Packages.Config` for each `packages.config` file under the solution.

</td></tr><tr>

<td>

Update Options

</td>

<td>

* __Include pre-release packages__: Equivalent to the `-Prerelease` option of the `NuGet.exe update` command
* __Perform safe update__: Equivalent to the `-Safe` option of the `NuGet.exe update` command, that looks for updates with the highest version available within the same major and minor version as the installed package.

</td></tr></table>

>To view the NuGet Installer's settings in [Kotlin DSL](kotlin-dsl.md), click __View as code__ in the sidebar.

See the [NuGet documentation](https://docs.nuget.org/docs/reference/command-line-reference) for the complete `NuGet.exe` command line reference.

When you add the NuGet Installer runner to your build configuration, each finished build will have the __NuGet Packages__ tab listing the packages used.

 <seealso>
        <category ref="blog">
            <a href="https://blog.jetbrains.com/teamcity/2013/08/nuget-package-restore-with-teamcity/">NuGet Package Restore with TeamCity</a>
        </category>
        <category ref="admin-guide">
            <a href="nuget-pack.md">NuGet Pack</a>
            <a href="nuget-publish.md">NuGet Publish</a>
        </category>
</seealso>