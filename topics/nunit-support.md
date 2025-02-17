[//]: # (title: NUnit Support in TeamCity)
[//]: # (auxiliary-id: NUnit Support in TeamCity;NUnit Support;TeamCity NUnit Test Launcher;NUnit for NAnt Build Runner;NUnit for MSBuild;MSBuild Service Tasks;NUnit Addins Support;TeamCity Add-in for NUnit;TeamCity Addin for NUnit)

There are two most common methods to set up NUnit tests reporting in TeamCity:
* Use the [.NET](net.md) build runner.
* Use the [NUnit](nunit.md) build runner.

Besides, you can try alternative approaches or run tests in any other runner (like [PowerShell](powershell.md) or [Command Line](command-line.md)) with the [TeamCity VSTest Adapter](https://github.com/JetBrains/TeamCity.VSTest.TestAdapter).

This article describes the specifics of the NUnit support in TeamCity and the [alternative approaches](#Alternative+Approaches) to establishing it.

## Supported NUnit Versions

The following NUnit versions are supported: 2.2.10, 2.4.1, 2.4.6, 2.4.7, 2.4.8, 2.5.0, 2.5.2, 2.5.3, 2.5.4, 2.5.5, 2.5.6, 2.5.7, 2.5.8, 2.5.9, 2.5.10, 2.6.0, 2.6.1, 2.6.2, 2.6.3, 3.0.

It is possible to have several versions of NUnit installed on an agent machine and use any of them in a build.

<warning>

NUnit version __3.4.0__ is __not__ supported by the NUnit build runner due to a problem in [NUnit](https://github.com/nunit/docs/wiki/Release-Notes#issues-resolved-1). Only version 3.4.0 was affected, other NUnit 3.x versions work fine with TeamCity.
</warning>

## NUnit Framework Compatibility

The following table represents the compatibility of TeamCity runners with the .NET implementations:

<table width="550">

<tr>

<td width="250">

</td>

<td width="100">

.NET Framework 1, 1.1, 2, 3.5

</td>

<td width="100">

.NET Framework 4+

</td>

<td width="100">

.NET Core 1+ and .NET 5+

</td>

</tr>

<tr>

<td>

__NUnit runner__
* `nunit-console.exe`
* `nunit3-console.exe`

</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td>

<td>

![error.png](error.png)

</td>

</tr>

<tr>

<td>

__.NET runner__
* `dotnest test`
* `dotnet msbuild /t:VSTest`
* `dotnet vstest`

</td>

<td>

![error.png](error.png)

</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td>

</tr>

<tr>

<td>

__VSTest adapter in other runners__

</td>

<td>

![error.png](error.png)

</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td>

</tr>

</table>

## Alternative Approaches

If using the NUnit or .NET build runners is inapplicable, TeamCity provides the following ways to configure NUnit tests reporting in TeamCity:
* The standard [NUnit for NAnt Build Runner](#NUnit+for+NAnt+Build+Runner).
* [NUnit for MSBuild](#Using+NUnit+for+MSBuild).
* The NUnit Test Launcher that can be configured in the [MSBuild build script](#Using+NUnit+for+MSBuild) or launched from the [command line](#NUnit+Test+Launcher).
* [TeamCity Add-in for NUnit](#TeamCity+Add-in+for+NUnit) is available to turn on reporting on the NUnit level without build procedure modifications. 
* The bundled [XML Test Reporting plugin](xml-report-processing.md) allows importing any XML report to TeamCity. In this case, it is not always possible to track results on the fly.   
    You can add the __XML Report Processing__ build feature to your build configuration, or use the following service message: `##teamcity[importData type='sometype' path='<path to the xml file>']`. Learn more: [XML Report Processing](xml-report-processing.md), [Importing XML Reports](service-messages.md#Importing+XML+Reports).
* Configuring test reports manually via [service messages](build-script-interaction-with-teamcity.md).

### Feature Comparison for Alternative Approaches

<table><tr>

<td>

Approach


</td>

<td>

Real-time Reporting


</td>

<td>

Execution in CLI-based runners


</td>

<td>

Tests Reordering


</td>

<td>

Implicit TeamCity .NET Coverage


</td></tr><tr>

<td>

\<nunit2\> NAnt task


</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)/![error.png](error.png)\*


</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td></tr><tr>

<td>

\<NUnit\> MSBuild task


</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)/![error.png](error.png)\*


</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td></tr><tr>

<td>

\<NUnitTeamCity\> MSBuild task


</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)/![error.png](error.png)\*


</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td></tr><tr>

<td>

TeamCity Add-in for NUnit

</td>

<td>

![check.png](check.png)

</td>

<td>

![error.png](error.png)

</td>

<td>

![error.png](error.png)

</td>

<td>

![error.png](error.png)

</td></tr><tr>

<td>

TeamCity NUnit Test Launcher

</td>

<td>

![check.png](check.png)

</td>

<td>

![error.png](error.png)

</td>

<td>

![check.png](check.png)

</td>

<td>

![check.png](check.png)

</td></tr><tr>

<td>

XML Reporting Plugin

</td>

<td>

![error.png](error.png)

</td>

<td>

![check.png](check.png)

</td>

<td>

N/A

</td>

<td>

N/A


</td></tr></table>

__\*__ TeamCity-provided tasks may have different syntax/behavior. Some workarounds may be required to run the script without TeamCity.

In addition to the common test reporting features, TeamCity allows running NUnit tests under the x86 process on the x64 machine by introducing an explicit specification of the platform and runtime environment versions. You can define whether to use .NET Framework 1.1, 2.0 or 4.0 started under an MSIL, x64 or x86 platform.

### NUnit Test Launcher
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

TeamCity provides its own NUnit tests launcher that can be used from the command line. The tests are run according to the passed parameters and, if the process is run inside the TeamCity build agent environment, the results are reported to the TeamCity agent.

>__Tips__:
>* If you need to access the path to the TeamCity NUnit launcher from some process, you can add the `%\system.teamcity.dotnet.nunitlauncher%` [environment variable](configuring-build-parameters.md).
>* Values surrounded with `%` in custom scripts in the [Command Line runner](command-line.md) are treated as TeamCity references.
> 

You can pass the following command-line options to the TeamCity NUnit Test Launcher:

```Shell
${teamcity.dotnet.nunitlauncher} <.NET Framework> <platform> <NUnit vers.> [/category-include:<list>] [/category-exclude:<list>] [/addin:<list>] <assemblies to test>
```

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

`<.NET Framework>`

</td>

<td>

Version of .NET Framework to run tests. Acceptable values are __v1.1__, __v2.0__, __v4.0__, and __ANY__.

</td></tr><tr>

<td>

`<platform>`

</td>

<td>

Platform to run tests. Acceptable values are __x86__, __x64__, and __MSIL__.

For .NET Framework 1.1, only __MSIL__ option is available.

</td></tr><tr>

<td>

`<NUnit vers.>`


</td>

<td>

The test framework to use. The value has to be specified in the following format: `NUnit-<version>`.

</td></tr><tr>

<td>

`/category-include:<list>`

</td>

<td>

The list of categories separated by `,` (optional).

</td></tr><tr>

<td>

`/category-exclude:<list>`

</td>

<td>

The list of categories separated by `,` (optional).

</td></tr><tr>

<td>

`/addin:<list>`

</td>

<td>

List of third-party NUnit add-ins to use (optional).

</td></tr><tr>

<td>

`<assemblies to test>`

</td>

<td>

List of assembly paths separated by `;` or whitespace.

</td></tr><tr>

<td>

`/runAssemblies:processPerAssembly`

</td>

<td>

Specify to run each assembly in a new process.

</td></tr></table>

#### Category Expression

Beginning with NUnit 2.4.6 and up to but not including NUnit v3.0, a _category expression_ can be used. This table shows some examples:

<table><tr>

<td>

Expression

</td>

<td>

Action

</td></tr><tr>

<td>

A|B|C

</td>

<td>

Selects tests having any of the categories A, B or C.

</td></tr><tr>

<td>

A,B,C

</td>

<td>

Selects tests having any of the categories A, B or C.

</td></tr><tr>

<td>

A+B+C

</td>

<td>

Selects only tests having all three categories assigned.

</td></tr><tr>

<td>

A+B|C

</td>

<td>

Selects tests with both A and B OR with category C.

</td></tr><tr>

<td>

A+B\-C

</td>

<td>

Selects tests with both A and B but not C.

</td></tr><tr>

<td>

-A

</td>

<td>

Selects tests not having category A assigned.

</td></tr><tr>

<td>

A+(B|C)

</td>

<td>

Selects tests having both category A and either of B or C.

</td></tr><tr>

<td>

A+B,C

</td>

<td>

Selects tests having both category A and either of B or C.

</td></tr></table>

__Note:__ As shown by the last two examples, the comma operator (`,`) is equivalent to the pipe (`|`) but has a higher precedence. The order of evaluation is as follows:
1. Unary exclusion operator (`-`).
2. High-precedence union operator (`,`).
3. Intersection and set subtraction operators (`+` and binary `-`).
4. Low-precedence union operator (`|`).

Since the operator characters have special meaning, avoid creating a category that uses any of them in its name. For example, the category `db-tests` must not be used in the command line, as it appears to mean "run category db, except for category tests". The same limitation applies to the characters having a special meaning for the shell you are using.

The following examples assume that the `teamcity.dotnet.nunitlauncher` property is set as a system property on the __[Parameters](configuring-build-parameters.md)__ page of __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes)__.

Run tests from an assembly:

```Shell
%\teamcity.dotnet.nunitlauncher% v2.0 x64 NUnit-2.2.10 Assembly.dll

```

Run tests from an assembly with the NUnit categories filter:

```Shell
%\teamcity.dotnet.nunitlauncher% v2.0 x64 NUnit-2.2.10 /category-include:C1 /category-exclude:C2 Assembly.dll

```

Run tests from assemblies:

```Shell
%\teamcity.dotnet.nunitlauncher% v2.0 x64 NUnit-2.5.0 /addin:Addin1.dll;Addin2.dll Assembly.dll Assebly2.dll

```

<!--[//]: # (Internal note. Do not delete. "TeamCity NUnit Test Launcherd319e337.txt")-->

### NUnit for NAnt Build Runner
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

This section assumes, that you already have a NAnt build script with the configured `nunit2` task in it, and want TeamCity to track test reports without making any changes to the existing build script. Otherwise, consider adding the [NUnit build runner](nunit.md) as one of the steps for your build configuration.

To track tests defined in a NAnt build via the standard `nunt2` task, TeamCity provides a custom [task](https://nant.sourceforge.net/nightly/latest/help/tasks/nunit2.html) implementation and automatically replaces the original `<nunit2>` task with its own task. When the build is triggered, TeamCity starts TeamCity NUnit Test Launcher using own implementation of `<nunit2>`. This allows you to leave your the build script without changes and receive on-the-fly test reports in TeamCity.

If you don't want TeamCity to replace the original `nunit2` task, consider the following options:
* Use the NUnit console with TeamCity Add-in for NUnit.
* Import XML test results via the XML Test Report plugin.
* Use the command-line [NUnit Test Launcher](#NUnit+Test+Launcher).
* Configure reporting tests manually via [service messages](service-messages.md).
* To disable `nunit2` task replacement, set the `teamcity.dotnet.nant.replaceTasks` [system property](configuring-build-parameters.md) to `false`.

The `nunt2` task implementation in TeamCity supports additional options that can be specified either as NAnt `<property>` tasks in the build script, or as <tooltip term="system-property">_system properties_</tooltip> under __Build Configuration | Build Parameters__.

The following options are supported for the TeamCity `<nunit2>` task implementation:

<table><tr>

<td>

Property

</td>

<td>

Description

</td></tr><tr>

<td>

`teamcity.dotnet.nant.nunit2.failonfailureatend`

</td>

<td>

Runs __all tests__ regardless of the number of failed ones. Fails if at least one test has failed.

</td></tr><tr>

<td>

`teamcity.dotnet.nant.nunit2.platform`

</td>

<td>

Sets desired runtime execution mode for .NET 2.0 on x64 machines. Supported values are __x86__, __x64__, and __ANY__(default).

</td></tr><tr>

<td>

`teamcity.dotnet.nant.nunit2.platformVersion`


</td>

<td>

Sets the required .NET Framework version. Supported values are __v1.1__, __v2.0__, __v4.0__. The default value is equal to the NAnt target framework.

</td></tr><tr>

<td>

`teamcity.dotnet.nant.nunit2.version`

</td>

<td>

Specifies which version of the NUnit runner to use. The value has to be specified in the following format: `NUnit-<version>`.

It is possible to have several versions of NUnit installed on an agent machine and use any of them in a build.

</td></tr><tr>

<td>

`teamcity.dotnet.nant.nunit2.addins`

</td>

<td>

Specifies the list of third-party NUnit add-ins used for NAnt build runner.

</td></tr><tr>

<td>

`teamcity.dotnet.nant.nunit2.runProcessPerAssembly`

</td>

<td>

Set `true` if you want to run each assembly in a new process.

</td></tr></table>

TeamCity NUnit test launcher will run tests in the .NET Framework, which is specified by the NAnt target framework: that is, on .NET Framework 1.1, 2.0 or 4.0 runtime. TeamCity also supports test categories for the `<nunit2>` task.

Adding the listed properties to the NAnt build script makes it TeamCity-dependent. To avoid this, specify properties as system properties under __Build Configuration__, or consider adding the `<if>` task.

>If you need a TeamCity test runner to support third-party NUnit add-ins, refer to the [NUnit Add-ins Support](#NUnit+Add-ins+Support) section for the details.

__Examples__

Start tests form a single assembly files under x64 mode on .NET 2.0.

```XML
<property name="teamcity.dotnet.nant.nunit2.platform" value="x64" />
<nunit2>
    <formatter type="Plain" />
    <test assemblyname="MyProject.Tests.dll" />
</nunit2>

```

Run all tests from category C1, but not C2.

```XML
<nunit2 verbose="true" haltonfailure="false" failonerror="true">
         <formatter type="Plain" />
          <test>
            <assemblies>
              <include name="dll.dll" />
            </assemblies>
            <categories>
               <include name="C1" />
               <exclude name="C2"/>
            </categories>
          </test>
        </nunit2>

```

Explicitly specify the version on NUnit to run tests with.   
Note that in this case, the following property should be added __before__ the `nunit2` task call.

```XML
<property name="teamcity.dotnet.nant.nunit2.version" value="NUnit-2.4.10" />
<nunit2> <!--....--> </nunit2>

```

### Using NUnit for MSBuild
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

This section describes how to use NUnit from MSBuild.
* For [NUnit prior to 3.0](#Working+with+NUnit+Task+in+MSBuild+Build)
* For [NUnit 3.0 and above](#Working+with+NUnit+3.0)

#### Working with NUnit Task in MSBuild Build

This section assumes that you already have an MSBuild build script with a configured `NUnit` task in it, and want TeamCity to track test reports without making any changes to the existing build script. Otherwise, consider adding the [NUnit build runner](nunit.md) as one of the steps of your build configuration.

#### Using NUnitTeamCity task in MSBuild Build Script

TeamCity provides a custom `NUnitTeamCity` task compatible with the `NUnit` task from [MSBuild Community tasks](https://github.com/loresoft/msbuildtasks) project. If you provide the `NUnitTeamCity` task in your build script, TeamCity will launch its own test runner based on the options specified within the task. Thus, you do not need to have any NUnit runner, because TeamCity will run the tests.

To use the `NUnitTeamCity` task correctly:
* Make sure the `teamcity_dotnet_nunitlauncher` <tooltip term="system-property">_system property_</tooltip> is accessible on build agents. Build agents running Windows should automatically detect these properties as environment variables. If you need to set them manually, see defining [agent-specific](predefined-build-parameters.md#Predefined+Agent+Build+Parameters) properties for more information.
* Configure your MSBuild build script with the `NUnitTeamCity` task using the following syntax:
   ```XML
   <UsingTask TaskName="NUnitTeamCity" AssemblyFile="$(teamcity_dotnet_nunitlauncher_msbuild_task)" />

   <NUnitTeamCity Assemblies="@(assemblies_to_test)" />

   ```

The `NUnitTeamCity` task supports the following attributes:

<table><tr>

<td width="200">

Property

</td>

<td>

Description

</td></tr><tr>

<td>

`Platform`

</td>

<td>

Execution mode on a x64 machine. Supported values are: __x86__, __x64__, and __ANY__.

</td></tr><tr>

<td>

`RuntimeVersion`

</td>

<td>

.NET Framework to use: __v1.1__, __v2.0__, __v4.0__, __ANY__. By default, the MSBuild runtime is used. Default is __v2.0__ for MSBuild 2.0 and 3.5. For MSBuild 4.0 the default value is __v4.0__.

</td></tr><tr>

<td>

`IncludeCategory`

</td>

<td>

As used in the `NUnit` task from the [MSBuild Community tasks](https://github.com/loresoft/msbuildtasks) project.

</td></tr><tr>

<td>

`ExcludeCategory`

</td>

<td>

As used in the `NUnit` task from the [MSBuild Community tasks](https://github.com/loresoft/msbuildtasks) project.

</td></tr><tr>

<td>

`NUnitVersion`

</td>

<td>

Version of NUnit to be used to run the tests.

To use NUnit 3.0 and above, see the [section below](#Working+with+NUnit+3.0).

</td></tr><tr>

<td>

`Addins`

</td>

<td>

List of third-party NUnit add-ins to be used. For more information on using NUnit add-ins, refer to the [NUnit Add-ins Support](#NUnit+Add-ins+Support) section.

</td></tr><tr>

<td>

`HaltIfTestFailed`

</td>

<td>

__True__ to fail task, if any test fails.

</td></tr><tr>

<td>

`Assemblies`

</td>

<td>

List of assemblies to run tests with.

</td></tr><tr>

<td>

`RunProcessPerAssembly`

</td>

<td>

Set `true`, if you want to run each assembly in a new process.

</td></tr></table>


The custom TeamCity `NUnit` task also supports additional attributes. For the list of available attributes refer to [this section](#Using+NUnitTeamCity+task+in+MSBuild+Build+Script).

If you need the TeamCity test runner to support third-party NUnit add-ins, refer to the [NUnit Add-ins Support](#NUnit+Add-ins+Support) section for the details.

Example (part of the MSBuild build script):

```XML
<Project xmlns="https://schemas.microsoft.com/developer/msbuild/2003">
  <UsingTask TaskName="NUnitTeamCity" AssemblyFile="$(teamcity_dotnet_nunitlauncher_msbuild_task)"/>
 
  <Target Name="SayHello">
     <NUnitTeamCity Assemblies="!!!*put here item group of assemblies to run tests on*!!!"/>
  </Target>
</Project>

```

__Important notes__
* Be sure to replace `.` with `_` when [using system properties](configuring-build-parameters.md#Pass+Values+to+Builders%27+Configuration+Files) in MSBuild scripts. For example, use `teamcity_dotnet_nunitlauncher_msbuild_task` instead of `teamcity.dotnet.nunitlauncher.msbuild.task`.
* TeamCity also provides [Visual Studio Solution Runner](visual-studio-sln.md) for solution files of Microsoft Visual Studio 2005 and above. It allows you to use MSBuild-style wildcards for the assemblies to run unit tests on.

__Examples__

Run NUnit tests using specific NUnit runner version:

```XML
<Target Name="build_01">
     <!-- start tests for NUnit-2.2.10 -->
     <NUnitTeamCity Assemblies="@(TestAssembly)" NUnitVersion="NUnit-2.2.10"/>
 
     <!-- start tests for NUnit-2.4.6 -->
     <NUnitTeamCity Assemblies="@(TestAssembly)" NUnitVersion="NUnit-2.4.8"/>
  </Target>

```

Run NUnit tests with custom add-ins with NUnit 2.4.6:

```XML
<Target Name="build">
     <NUnitTeamCity Assemblies="@(TestAssembly)" Addins="NUnitExtension.RowTest.AddIn.dll" NUnitVersion="NUnit-2.4.6"/>
  </Target>

```

Run NUnit tests with custom add-ins with NUnit 2.4.6 __in per-assembly mode:__

```XML
<Target Name="build">
     <NUnitTeamCity Assemblies="@(TestAssembly)" Addins="NUnitExtension.RowTest.AddIn.dll" NUnitVersion="NUnit-2.4.6" RunProcessPerAssembly="True"/>
</Target>

```

To make a TeamCity independent build script, consider the following option:

```XML
<NUnitTeamCity ... Condition=" '$(TEAMCITY_VERSION)' != '' "/>

```

The MSBuild property `TEAMCITY_VERSION` is added to MSBuild when started from TeamCity.

#### Working with NUnit 3.0

Starting from version 3.0, NUnit supports TeamCity natively, so there is no need to use a special task for MSBuild as it was done for the [earlier NUnit versions](#Working+with+NUnit+Task+in+MSBuild+Build). The simplest way is to run the NUnit console via the standard [Exec task](https://msdn.microsoft.com/en-us/library/x8zx72cd.aspx).

The [Getting Started with NUnit](getting-started-with-nunit.md) article contains details and examples.

### MSBuild Service Tasks
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

For MSBuild, TeamCity provides the following service tasks that implement the same options as the [Build Script Interaction](build-script-interaction-with-teamcity.md):

**`TeamCitySetBuildNumber`** allows changing the build number:

```XML
<TeamCitySetBuildNumber BuildNumber="1.3_{build.number}" />

```

It is possible to use `{build.number}` as a placeholder for older build number.

**`TeamCityProgressMessage`** allows writing a progress message:

```XML
<TeamCityProgressMessage Text="Progress message text" />

```

**`TeamCityPublishArtifacts`** allows publishing all artifacts from the MSBuild item group:

```XML
<ItemGroup>
    <Files Include="*.dll" />
  </ItemGroup>
  <TeamCityPublishArtifacts SourceFiles="@(Files-> '%(FullPath)' )" Condition=" '$(TEAMCITY_VERSION)' != '' "/>

```

**`TeamCityReportStatsValue`** is a handy task for publishing statistic values:


```XML
<TeamCityReportStatsValue Key="StatsValueType" Value="42" />

```

**`TeamCityBuildProblem`** reports a build problem which actually fails the build. Build problems appear on the __Build Results__ page and also affect build status text:

```XML
<TeamCityBuildProblem description="description" identity="identity"/>

```

<!--[//]: # (Internal note. Do not delete. "MSBuild Service Tasksd214e94.txt")-->

* The mandatory `description` attribute is a human-readable text describing the build problem. By default, `description` appears in the build status text.
* `identity` is an optional attribute and characterizes a particular build problem instance. It shouldn't change throughout builds if the same problem occurs: for example, the same compilation error. It should be a valid Java ID up to 60 characters long. By default, `identity` is calculated based on `description`.

**`TeamCitySetStatus`** is a task to change current build status text.

```XML
<TeamCitySetStatus Status="<status value>" Text="{build.status.text} and some aftertext" />

```

`{build.status.text` is substituted with an older status text. The status can have the `SUCCESS` value.

### NUnit Add-ins Support
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

NUnit Add-in is an extension that plugs in to the NUnit core and changes the way it operates. Refer to the [NUnit add-ins page](https://www.nunit.org/index.php?p=nunitAddins&amp;r=2.6.3) for more information. This section covers description of the NUnit add-ins support for NAnt, MSBuild, and NUnit Console Launcher.

#### Using Add-ins in NAnt Build Runner

To support NUnit add-ins in the [NAnt](nant.md) build runner, you need to add the `teamcity.dotnet.nant.nunit2.addins` property to your build script:

```XML
<property name="teamcity.dotnet.nant.nunit2.addins" value="<list of paths>" />

```

where `<list>` is the list of paths to NUnit add-ins separated by `;`.

For example:

```XML
<property name="teamcity.dotnet.nant.nunit2.addins" value="../tools/addins/MyFirst.AddIn.dll;MySecond.AddIn.dll" />

```

#### Using Add-ins in TeamCity NUnit Console Launcher

To support NUnit add-ins for the [console launcher](#NUnit+Test+Launcher), you need to provide the `/addins:<list of addins separated with ;>` command-line option.

For example:

```XML
${teamcity.dotnet.nunitlauncher} /addin:../tools/addins/MyFirst.AddIn.dll;nunit-addins/MySecond.AddIn.dll

```

#### Using Add-ins in MSBuild

This section is __applicable to NUnit versions prior to 3.0__.

To support NUnit add-ins for the MSBuild runner, specify the `Addins` property for the `NUnitTeamCity` task:

```XML
Addins="<list>"

```

where `<list>` is the list of add-ins separated by `;` or `,`.

For example:

```XML
<Project xmlns="https://schemas.microsoft.com/developer/msbuild/2003" DefaultTargets="build">
  <ItemGroup>
    <TestAssembly Include="$(MSBuildProjectDirectory)/MyTests.dll" />
  </ItemGroup>
  <Target Name="build">
    <NUnitTeamCity Assemblies="@(TestAssembly)" Addins="../tools/addins/MyFirst.AddIn.dll;nunit-addins/MySecond.AddIn.dll" />
  </Target>
</Project>

```

### TeamCity Add-in for NUnit
{initial-collapse-state="collapsed" collapsible="true" collapsible="true"}

TeamCity NUnit Add-in supports __NUnit prior to version 3.0__. For later versions, refer to the [this section](#Using+NUnit+for+MSBuild).

If you run NUnit tests via the [NUnit console](https://www.nunit.org/index.php?p=nunit-console&amp;r=2.2.10) and want TeamCity to track the test results without having to launch the TeamCity test runner, the best solution is to use TeamCity Add-in for NUnit. You can plug this add-in into NUnit, and the tests will be automatically reported to the TeamCity server.

Alternatively, you can opt to use the [XML Report Processing](xml-report-processing.md) build feature, or manually configure test reporting by means of [service messages](service-messages.md).

To be able to review test results in TeamCity:
1. In your build, set the path to the TeamCity Add-in to the system property `teamcity.dotnet.nunitaddin` (for MSBuild, it would be `teamcity_dotnet_nunitaddin`), and add the version of NUnit at the end of this path. For example:
    * For NUnit 2.4.X, use `${teamcity.dotnet.nunitaddin}-2.4.X.dll` (for MSBuild: `$(teamcity_dotnet_nunitaddin)-2.4.X.dll`).   
      Example for NUnit 2.4.7: `NAnt: ${teamcity.dotnet.nunitaddin}-2.4.7.dll`, MSBuild: `$(teamcity_dotnet_nunitaddin)-2.4.7.dll`.
    * For NUnit 2.5.0 alpha 4, use `${teamcity.dotnet.nunitaddin}-2.5.0.dll` (for MSBuild: `$(teamcity_dotnet_nunitaddin)-2.5.0.dll`).
2. Copy the `.dll` and `.pdb` TeamCity add-in files to the NUnit add-in directory.

Although you can copy these files once, it is highly recommended configuring your builds so that the TeamCity add-in files are copied to the NUnit add-in directory for __each build__, because these files could be updated by TeamCity.

The following example shows how to use the NUnit console runner with the TeamCity Add-in for NUnit 2.4.7 (on MSBuild):

```XML
<ItemGroup>
  <NUnitAddinFiles Include="$(teamcity_dotnet_nunitaddin)-2.4.7.*" />
</ItemGroup>
 
<Target Name="RunTests">
  <MakeDir Directories="$(NUnitHome)/bin/addins" />
  <Copy SourceFiles="@(NUnitAddinFiles)" DestinationFolder="$(NUnitHome)/bin/addins" />
  <Exec Command="$(NUnitHome)/bin/NUnit-Console.exe $(NUnitFileName)" />
</Target>

```


If you need to configure <tooltip term="environment-variable">_environment variables_</tooltip> for NUnit explicitly, specify an environment variable with the value reference of `%\system.teamcity.dotnet.nunitaddin%`. See [this article](configuring-build-parameters.md) for details.

__NUnit 2.4.8 Issue__  
NUnit 2.4.8 has the following known issue: NUnit 2.4.8 runner tries to load an assembly according to the created `AssemblyName` object. However, the `addins` folder of NUnit 2.4.8 is not included in application probe paths. As a result, NUnit 2.4.8 fails to load any add-in in the console mode.   
To solve the problem, we suggest that you use any of the following workarounds:
* Copy the TeamCity add-in assembly both to the NUnit `bin` and `bin/addins` directories.
* Patch `NUnit-Console.exe.config` to include the add-ins to application probe paths. Add the following code into the `config/runtime` element:
    ```XML
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <probing privatePath="addins"/>
    </assemblyBinding>
    
    ```

<seealso>
        <category ref="admin-guide">
            <a href="nunit.md">NUnit build runner</a>
            <a href="getting-started-with-nunit.md">Getting Started with NUnit</a>
            <a href="mstest-support.md">MSTest Support</a>
            <a href="running-risk-group-tests-first.md">Running Risk Group Tests First</a>
            <a href="xml-report-processing.md">XML Report Processing</a>
        </category>
</seealso>