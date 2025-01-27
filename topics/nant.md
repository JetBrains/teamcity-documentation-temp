[//]: # (title: NAnt)
[//]: # (auxiliary-id: NAnt)


<note>

Starting with TeamCity 2024.07, the NAnt runner is deprecated. It remains bundled for compatibility but will be removed in future releases. We recommend switching to the  [](net.md) runner for testing .NET and .NET Framework applications.

To continue using the NAnt runner after it is unbundled, you can [download a separate plugin from the JetBrains Marketplace](https://plugins.jetbrains.com/plugin/26350-deprecated-net-build-runners-support).

</note>

The _NAnt_ build runner allows using [NAnt](https://nant.sourceforge.net/) build files.

TeamCity supports NAnt starting from version 0.85.

<note>

The TeamCity NAnt runner requires .NET Framework or Mono installed on the build agent.
</note>

## MSBuild Task for NAnt

The TeamCity NAnt runner includes a task called `msbuild` that allows NAnt to start MSBuild scripts. TeamCity `msbuild` task for NAnt has the same set of attributes as the [NAntContrib package](https://nantcontrib.sourceforge.net) `msbuild` task. The MSBuild build processes started by NAnt will behave exactly as if they were launched by TeamCity MSBuild/SLN2005 build runner (that is `NUnit` and/or `NUnitTeamCity` MSBuild tasks will be added to build scripts and logs and error reports will be sent directly to the build server).

<note>

`msbuild` task for NAnt makes all build configuration system properties available inside MSBuild script. Note, all property names will have `.` replaced with `_`.   
To disable this, set `false` to `set-teamcity-properties` attribute of the task.
</note>

By default, NAnt `msbuild` task checks for current value of NAnt target-framework property to select MSBuild runtime version.  
This parameter could be overriden by setting `teamcity_dotnet_tools_version` project property with required .NET Framework version, i.e. "4.0".

```XML
...
  <!-- this property enables MSBuild 4.0 -->
  <property name="teamcity_dotnet_tools_version" value="4.0"/>
  <msbuild project="SimpleEcho.v40.proj">
     ...
  </msbuild>
 ...

```

<note>

To pass properties to MSBuild, use the `property` tag instead of the explicit properties definition in the command line.
</note>

## &lt;nunit2&gt; Task for NAnt

To test all the assemblies without halting on first failed test use:

```XML
<target name="build">
       <nunit2 verbose="true" haltonfailure="false" failonerror="true" failonfailureatend="true">
        <formatter type="Plain" />
        <test haltonfailure="false">
          <assemblies>
            <include name="dll1.dll" />
            <include name="dll2.dll" />
          </assemblies>
        </test>
       </nunit2>
</target>

```

<note>

The `failonfailureatend` attribute is not defined in the original `NUnit2` task from NAnt. Note that this attribute will be ignored if the `haltonfailure` attribute is set to `true` for either the `nunit2` element or the `test` element.
</note>

Below you can find reference information about NAnt Build Runner fields.

## General Options

<table>
<tr>

<td>

Option 

</td>

<td>

Description 

</td>
</tr>
<tr>


<td>

Path to a build file 

</td>

<td>

Specify path relative to the [Build Checkout Directory](build-checkout-directory.md).

</td>
</tr>
<tr>

<td>

Build file content

</td>


<td>

Select the option, if you want to use a different build script than the one listed in the settings of the build file. When the option is enabled, you have to type the build script in the text area.

</td>
</tr>
<tr>

<td>

Targets

</td>

<td>

Specify the names of build targets defined by the build script, separated by spaces. The available targets can be viewed in the Web UI by clicking the icon next to the field and added by checking the appropriate boxes.

</td>
</tr>
<tr>

<td>

Working directory

</td>

<td>

Specify the path to the [Build Working Directory](build-working-directory.md). By default, the build working directory is set to the same path as the [build checkout directory](build-checkout-directory.md). 

</td>
</tr>
<tr>

<td>

NAnt home

</td>

<td>

Enter a path to the `NAnt.exe`. 

<tip>

Here you can specify an absolute path to the NAnt.exe file on the agent, or a path relative to the checkout directory. Such relative path allows you to provide particular NAnt.exe file to run a build of the particular build configuration.
</tip>

</td>
</tr>
<tr>

<td>

Target framework

</td>


<td>

Sets `-targetframework:` option to `NAnt` and generates appropriate agent requirements (_mono-2.0_ target framework will require _Mono_ system property, _net-2.0_ — _DotNetFramework2.0_ property, and so on). Selecting unsupported in TeamCity framework (_sscli-1.0_, _netcf-1.0_, _netcf-2.0_) won't impose any agent requirements.

<warning>

This option has no effect on framework which used to run `NAnt.exe`. `NAnt.exe` will be launched as ordinary exe file if .NET framework was found, and through mono executable, if not.
</warning>

</td>
</tr>
<tr>

<td>

Command line parameters

</td>


<td>

Specify any additional parameters for `NAnt.exe`

<tip>

TeamCity passes automatically to NAnt all defined [Configuring Build Parameters](configuring-build-parameters.md), so you do not need to specify all of the properties here via  `-D` option.
</tip>

</td>
</tr>
<tr>

<td>

Reduce test failure feedback time

</td>

<td>

Use following option to instruct TeamCity to run some tests before others.

</td>
</tr>
<tr>

<td>

Run recently failed tests first

</td>


<td>

If checked, in the first place TeamCity will run tests failed in previous finished or running builds as well as tests having high failure rate (a so called _blinking_ tests).

</td>
</tr>
</table>

## Code Coverage

To learn about configuring code coverage options, refer to the [Configuring .NET Code Coverage](configuring-.net-code-coverage.md) page.

<seealso>
        <category ref="concepts">
            <a href="build-runner.md">Build Runner</a>
            <a href="build-checkout-directory.md">Build Checkout Directory</a>
            <a href="build-working-directory.md">Build Working Directory</a>
        </category>
        <category ref="admin-guide">
            <a href="configuring-build-parameters.md">Configuring Build Parameters</a>
        </category>
</seealso>