[//]: # (title: NUnit)
[//]: # (auxiliary-id: NUnit)

The _NUnit_ build runner is designed to run NUnit tests on the TeamCity server. However, you can also use the [](net.md) runner to launch NUnit tests and view test reports in TeamCity. Refer to the [](nunit-support.md) article for more information.

<anchor name="NUnit3Extensions"/>

> NUnit build runner supports only [.NET Framework](https://docs.microsoft.com/en-us/dotnet/framework/get-started/overview). To run tests for [.NET](https://docs.microsoft.com/en-us/dotnet/framework/get-started/net-core-and-open-source) projects (and .NET Framework projects targeting version 4.0 or later), use the [.NET](net.md) build runner with the [`test`](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-test) command instead. See the [NUnit Support in TeamCity](nunit-support.md) article for details.
> 
{style="note"}

## Legacy and Updated NUnit Runners

<snippet id="2024-07-nunit">

Version 2024.07 introduces an updated NUnit runner that, compared to the legacy runner, does not allow you to select a .NET Runtime or .NET Framework version. If needed, use the **Additional command line parameters** field to specify these settings. In addition, the updated runner no longer supports outdated NUnit 2.x.x versions.

In version 2024.07, both updated and legacy runners are fully functional and available from the [Build Steps](configuring-build-steps.md) page. In the following release cycles we expect to unbundle the legacy runner and move it to a separate plugin. When it is done, you will need to manually [install this plugin from JetBrains Marketplace](https://plugins.jetbrains.com/plugin/26350-deprecated-net-build-runners-support) to continue using the legacy runner. As such, we recommend migrating your projects to either updated NUnit or regular [](net.md) runners.

</snippet>

## Installing NUnit

<snippet id="installing-nunit">

To use the NUnit build runner, you need to install the [NUnit NuGet package](https://www.nuget.org/packages/NUnit/) on TeamCity agents via one of the following options:
* Instruct the first build step to install NUnit from a NuGet package. 

  For example, you can add a [Command Line](command-line.md) build step which will install the `NUnit.Console` NuGet package as follows:

    ```Shell
    %\teamcity.tool.NuGet.CommandLine.DEFAULT%\tools\nuget.exe install NUnit.Console -version 3.6.0 -o packages
   
    ```
    
    The `%\teamcity.tool.NuGet.CommandLine.DEFAULT%` string is a reference to the NuGet installed under the TeamCity agent.

    You can install NuGet on agents on the __Administration | Tools__ page, where you can also mark any of the installed NuGet versions as default.
    
    After that, the `%\teamcity.tool.NuGet.CommandLine.DEFAULT%` parameter reference should properly resolve to the NuGet installation path on the agent.   
    Then nunit3-console should appear under the package directory.

* Install NUnit manually to a standard location on all the build agents and configure the path to `nunit-console.exe` in your NUnit build step.

### Installing Extensions
[//]: # (AltHead: NUnit3Extensions)

Starting from version 3.2.0, NUnit requires the `NUnit.Extension.NUnitProjectLoader` extension to be installed on the TeamCity agent. Starting from version 3.4.1, NUnit requires the `NUnit.Extension.TeamCityEventListener` extension to be installed on the TeamCity agent.  
If the extensions are not found in versions 3.2.0 and 3.2.1, the build will fail without a warning. Since version 3.4.1, a message will be displayed suggesting you install them.   

The extensions can be installed as separate packages or in bulk using the [NUnit Console Version 3](https://www.nuget.org/packages/NUnit.Console) NuGet package.

</snippet>

<anchor name="NUnit-settings"/>

## NUnit Test Settings

Legacy and updated NUnit runners exhibit a few differences.

### Updated Runner Settings

<table><tr>

<td>

Setting

</td>

<td>

Description

</td></tr><tr>

<td>

NUnit Console
{id="pathToNUnitConsoleTool" help-id="NUnit-pathToNUnitConsoleTool"}

</td>

<td>

Select a preinstalled console tool or specify a custom path to the `nunit3-console.exe` console executable, including the filename.

</td></tr><tr>

<td>

Working directory
{id="workingDirectory" help-id="NUnit-workingDirectory"}

</td>

<td>


Specify the path to the [build working directory](build-working-directory.md) if it differs from the directory of the testing assembly.

</td></tr><tr>

<td>

Path to application configuration file
{id="appConfigFile" help-id="NUnit-appConfigFile"}

</td>

<td>

Specifу the path to the application configuration file to be used when running tests. The path should be either absolute or relative to the [checkout directory](build-checkout-directory.md).

</td></tr><tr>

<td id="NUnit-cmdParameters">

Additional command line parameters

</td>

<td>

Enter [additional command line parameters](https://docs.nunit.org/articles/nunit/running-tests/Console-Command-Line.html) to pass to `nunit-console.exe`.

</td></tr><tr>

<td>

Run tests from

</td>

<td>

Specify the .NET assemblies where the NUnit tests stored. Multiple entries should be comma-separated. The MSBuild wildcards are supported. The paths to assembly files should be absolute or relative to the [checkout directory](build-checkout-directory.md).

In the example, TeamCity will search for the tests assemblies in all the project directories and run these tests:

```Shell
**\*.dll

```

Wildcards are specified relatively to the checkout directory.

</td></tr><tr>

<td>

Do not run tests from

</td>

<td>

Specify .NET assemblies that should be excluded from the list of assemblies to test. Multiple entries should be comma-separated. The MSBuild wildcards are supported. The paths to assembly files should be absolute or relative to the [checkout directory](build-checkout-directory.md).

In the example, TeamCity will omit tests specified in this directory:


```Shell
**\obj\**\*.dll

```


Wildcards are specified relatively to the checkout directory.

</td></tr><tr>

<td>

Include categories

</td>

<td>

Specify NUnit categories of tests to be run. Multiple entries should be comma-separated.    
[Category expressions](nunit-support.md#Category+Expression) are supported as well. Commas, semicolons, and new lines are treated as global __or__ operations (prior to the expression parsing).

Since NUnit 3.0, category expressions are not supported and must be set via the command line as described in the [NUNit documentation](https://github.com/nunit/docs/wiki/Test-Selection-Language).

</td></tr><tr>

<td>

Exclude categories

</td>

<td>

Specify NUnit categories to be excluded from the tests to be run. Multiple entries should be comma-separated.   
[Category expressions](nunit-support.md#Category+Expression) are supported here as well. Commas, semicolons, and new lines are treated as global __or__ operations (prior to the expression parsing).

Since NUnit 3.0, category expressions are not supported_and must be set via the command line as described in the [NUNit documentation](https://github.com/nunit/docs/wiki/Test-Selection-Language).

</td></tr><tr>

<td>

Reduce test failure feedback time

</td>

<td>

Use this option to instruct TeamCity to run some tests before others.

</td></tr></table>

### Legacy Runner Settings
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

<table><tr>

<td>

Setting

</td>

<td>

Description

</td></tr><tr>

<td id="runner">

<anchor name="NUnit-runner"/>

NUnit runner<br/><br/>


</td>

<td>

Select the NUnit version to be used to run the tests. The set of available settings varies depending on the selected version.

</td></tr><tr>

<td>

NUnit Console

</td>

<td>

Select a preinstalled console tool or specify a custom path to the `nunit3-console.exe` console executable, including the filename.

This option is available only if NUnit 3.0 is selected.

</td></tr><tr>

<td>

Working directory

</td>

<td>


Specify the path to the [build working directory](build-working-directory.md) if it differs from the directory of the testing assembly.

This option is available only if NUnit 3.0 is selected.

</td></tr><tr>

<td>

Path to application configuration file

</td>

<td>

Specifу the path to the application configuration file to be used when running tests. The path should be either absolute or relative to the [checkout directory](build-checkout-directory.md).

This option is available only if NUnit 3.0 is selected.

</td></tr><tr>

<td>

Additional command line parameters

</td>

<td>

Enter [additional command line parameters](https://docs.nunit.org/articles/nunit/running-tests/Console-Command-Line.html) to pass to `nunit-console.exe`.

This option is available only if NUnit 3.0 is selected.

</td></tr><tr>

<td>

.NET Runtime

</td>

<td>

From the __Platform__ drop-down menu, select the desired execution mode on an x64 machine. Supported values are: `Auto (MSIL)`, `x86`, and `x64`.

From the __Version__ drop-down menu, select the required .NET Framework version. Supported values are: `v2.0`, `v4.0`; and `auto`, _available if NUnit 3.0 is selected_.   
_For NUnit 3.0_, if `auto` is selected, tests will run under the framework they are compiled with.


</td></tr><tr>

<td>

Run tests from

</td>

<td>

Specify the .NET assemblies where the NUnit tests stored. Multiple entries should be comma-separated. The MSBuild wildcards are supported. The paths to assembly files should be absolute or relative to the [checkout directory](build-checkout-directory.md).

In the example, TeamCity will search for the tests assemblies in all the project directories and run these tests:

```Shell
**\*.dll

```

Wildcards are specified relatively to the checkout directory.

</td></tr><tr>

<td>

Do not run tests from

</td>

<td>

Specify .NET assemblies that should be excluded from the list of assemblies to test. Multiple entries should be comma-separated. The MSBuild wildcards are supported. The paths to assembly files should be absolute or relative to the [checkout directory](build-checkout-directory.md).

In the example, TeamCity will omit tests specified in this directory:


```Shell
**\obj\**\*.dll

```


Wildcards are specified relatively to the checkout directory.

</td></tr><tr>

<td>

Include categories

</td>

<td>

Specify NUnit categories of tests to be run. Multiple entries should be comma-separated.    
[Category expressions](nunit-support.md#Category+Expression) are supported as well. Commas, semicolons, and new lines are treated as global __or__ operations (prior to the expression parsing).

Since NUnit 3.0, category expressions are not supported and must be set via the command line as described in the [NUNit documentation](https://github.com/nunit/docs/wiki/Test-Selection-Language).

</td></tr><tr>

<td>

Exclude categories

</td>

<td>

Specify NUnit categories to be excluded from the tests to be run. Multiple entries should be comma-separated.   
[Category expressions](nunit-support.md#Category+Expression) are supported here as well. Commas, semicolons, and new lines are treated as global __or__ operations (prior to the expression parsing).

Since NUnit 3.0, category expressions are not supported_and must be set via the command line as described in the [NUNit documentation](https://github.com/nunit/docs/wiki/Test-Selection-Language).

</td></tr><tr>

<td>

Run process per assembly

</td>

<td>

Select this option if you want to run each assembly in a new process.

This option is available only for the <a href="#Legacy+and+Updated+NUnit+Runners">legacy NUnit runner</a> with NUnit versions prior to 3.0.

</td></tr><tr>

<td>

Reduce test failure feedback time

</td>

<td>

Use this option to instruct TeamCity to run some tests before others.

</td></tr></table>

## NUnit Code Coverage

To learn about configuring code coverage options, refer to the [this article](configuring-.net-code-coverage.md).

For NUnit 3.x, only [JetBrains dotCover](jetbrains-dotcover.md) is supported as a coverage tool.
 
<seealso>
        <category ref="admin-guide">
            <a href="configuring-test-reports-and-code-coverage.md">Configuring Unit Testing and Code Coverage</a>
            <a href="nunit-support.md">NUnit Support</a>
            <a href="getting-started-with-nunit.md">Getting Started with NUnit</a>
        </category>
</seealso>