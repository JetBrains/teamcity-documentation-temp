[//]: # (title: Visual Studio 2003)
[//]: # (auxiliary-id: Visual Studio 2003)

<note>

Since version 2020.2, the Visual Studio 2003 build runner is discontinued and disabled in TeamCity.

We recommend switching to the [.NET build runner](net.md) instead. If you were actively using the VS 2003 runner and cannot easily migrate to the .NET runner, please let us know about it via any of our [feedback channels](https://teamcity-support.jetbrains.com/hc/en-us){nullable="true"}.

</note>

The _Visual Studio 2003_ build runner supports building Microsoft Visual Studio 2003 .NET projects. To build Microsoft Visual Studio 2005-2017 projects, see [Visual Studio (sln)](visual-studio-sln.md) Build Runner.

<note>

The Visual Studio 2003 build runner uses NAnt instead of MS Visual Studio 2003 to perform the build. As a result the agent is required to have .NET Framework 1.1 installed, however under certain conditions .NET Framework SDK 1.1 might be required. This NAnt solution task may behave differently than MS Visual Studio 2003. See [NAnt documentation](https://nant.sourceforge.net/release/latest/help/tasks/solution.html) for details.   
To use this runner you need to configure the [NAnt runner](nant.md).

</note>

<table><tr>

<td>

Option


</td>

<td>

Description


</td></tr><tr>

<td>

Solution file path


</td>

<td>

A path to the solution to be built is relative to the [build checkout directory](build-checkout-directory.md). For example:


```
vs-addin\addin\addin.sln

```


</td></tr><tr>

<td>

Working directory


</td>

<td>

Specify the [Build Working Directory](build-working-directory.md).


</td></tr><tr>

<td>

Configuration


</td>

<td>

Specify the name of the solution configuration to build.


</td></tr><tr>

<td>

Projects output


</td>

<td>

This group of options enables you to use the default output defined in the solution, or specify your own output path.


</td></tr><tr>

<td>

Output directory for all projects


</td>

<td>

This option is available, if __Override project output option__ is checked. Specify the directory where the compiled targets will be placed.


</td></tr><tr>

<td>

Resolve URLs via map


</td>

<td>

Click this radio button, if you want to map the URL project path to the physical project path. If this option is selected, specify mapping in the __Type the URL's map__ field.


</td></tr><tr>

<td>

Type the URL's map


</td>

<td>

Click this link and specify the desired map in the text area. Use the following format:


```
http://localhost:8111=myProjectPath/myProject

```

where

* [`http://localhost:8111`](http://localhost:8111){nullable="true"} is the host where the project will be uploaded
* `myProjectPath/myProject` is the project root


</td></tr><tr>

<td>

Resolve URLs via WebDAV


</td>

<td>

Click this radio button, if you want the URLs to be resolved via WebDav.

<warning>

Make sure that all the necessary files are properly updated. The build agent may not update information from VCS implicitly.
</warning>


</td></tr><tr>

<td>

MS Visual Studio reference path


</td>

<td>

Check this option, if you want to automatically include reference path of MS Visual Studio to the build.


</td></tr><tr>

<td>

NAnt home


</td>

<td>

Specify path to the NAnt executable to run builds of the build configuration. The path can be absolute, relative to the build checkout directory; also you can use an environment variable.


</td></tr><tr>

<td>

Command line parameters


</td>

<td>

Specify any additional parameters for `NAnt.exe`.

TeamCity passes automatically to NAnt all defined [system properties](configuring-build-parameters.md), so you do not need to specify all of the properties here via `-D` option. You can create necessary properties at the __Build Parameters__ section of the [Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes).


</td></tr><tr>

<td>

Run NUnit tests for


</td>

<td>

Specify .NET assemblies, where the NUnit tests to be run are stored. Multiple entries are comma\-separated; usage of NAnt wildcards is enabled. In the following example, TeamCity will search for the tests assemblies in all project directories and run these tests.


```
**\*.dll

```

All these wildcards are specified relative to path that contains the solution file.


</td></tr><tr>

<td>

Do not run NUnit tests for


</td>

<td>

Specify .NET assemblies that should be excluded from the list of found assemblies to test. Multiple entries are comma\-separated; usage of NAnt wildcards is enabled. In the following example, TeamCity will omit tests specified in this directory.


```
**\obj\**\*.dll

```



All these wildcards are specified relative to path that contains the solution file.

</td></tr><tr>

<td>

Reduce test failure feedback time


</td>

<td>

Use following option to instruct TeamCity to run some tests before others.


</td></tr><tr>

<td>

Run recently failed tests first


</td>

<td>

If checked, in the first place TeamCity will run tests failed in previous finished or running builds as well as tests having high failure rate (a so called _blinking_ tests)


</td></tr></table>

<seealso>
        <category ref="admin-guide">
            <a href="nunit-support.md#Using+NUnit+for+MSBuild">NUnit for MSBuild</a>
        </category>
</seealso>