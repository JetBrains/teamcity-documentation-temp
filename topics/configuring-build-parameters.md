[//]: # (title: Configuring Build Parameters)
[//]: # (auxiliary-id: Configuring Build Parameters)
<!--[//]: # (Internal note. Do not delete. "Configuring Build Parametersd72e3.txt")-->

Parameters are `name=value` pairs that can be referenced throughout TeamCity. There are three major parameter types in TeamCity:

<anchor name="Configuration+Parameters"/>

* **Configuration Parameters** — parameters whose primary objective is to share settings within a build configuration. You can also use these parameters to customize a configuration that was created from a [template](build-configuration-template.md) or uses a [meta-runner](working-with-meta-runner.md). TeamCity does not pass parameters of this type to a build process (that is, these parameters are not accessible by a build script engine).

<anchor name="Environment+Variables"/>


* **Environment Variables** — parameters that start with the `env.` prefix. These parameters are passed to the process of a build runner similarly to the default env variables of a system.

<anchor name="System+Properties"/>

* **System Properties** — parameters that start with the `system.` prefix. TeamCity can pass parameters of this type to configuration files of [certain runners](#Pass+Values+to+Builders%27+Configuration+Files) as variables specific to a build tool.

<anchor name="Parameter+References"/>

## Common Information

Parameters allow you to avoid using plain values in TeamCity UI and build scripts. Instead, you can reference a parameter by its name using the `%\parameter_name%` syntax. In addition, parameters can reference other parameters in their values (for example, `system.tomcat.libs=%\env.CATALINA_HOME%/lib/*.jar`).

Storing values in parameters allows you to:

* Quickly reuse frequently used values;

* Create reusable [templates](build-configuration-template.md) and [meta-runners](working-with-meta-runner.md) whose parameter values are overridden in target configurations;

* Add flexibility to your build configurations: parameter values can be quickly altered in TeamCity UI, via the [service message](service-messages.md) sent during a build, or in the [Run Custom Build dialog](running-custom-build.md);

* [Hide sensitive information](typed-parameters.md) that should not be visible to regular TeamCity developers;

* Improve the readability of your configurations by using shorter parameter references instead of lengthy plain values, and so on.

See the following article for simple use cases where you can store values in parameters: [](using-build-parameters.md).

## Main Use Cases

### Customize Template-Based Configurations

**Configuration parameters** allow you to create a base build configuration with parameters, extract it to the template, and override these parameters in configurations based on this template. Using this technique you can quickly duplicate your build configurations that have slight variations depending on your current task.

For example, the following build configuration has two steps and the Boolean `skip.optional.step` parameter. Step #2 will or will not be executed depending on this parameter value.

```Kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.script

object SourceConfig : BuildType({
    name = "SourceConfig"

    params {
        param("skip.optional.step", "false")
    }
    steps {
        script {
            name = "Mandatory Step"
            scriptContent = """echo "Mandatory step #1 is running...""""
        }
        script {
            name = "Optional Step"
            scriptContent = """echo "Optional step #2 is running...""""
            conditions {
                equals("skip.optional.step", "false")
            }
        }
    }})
```

If you extract a [template](build-configuration-template.md) from this configuration, you can create multiple copies of the same configuration. In those copies that do not need to run the optional step #2, override the `skip.optional.step` parameter and set it to `true`.


```Kotlin
import jetbrains.buildServer.configs.kotlin.*

object ConfigFromTemplate : BuildType({
    templates(SourceConfigTemplate)
    name = "Build Config Based on Template"

    params {
        param("skip.optional.step", "true")
    }
})
```

See the following section to learn more about step execution conditions: [](#Specify+Step+Execution+Conditions).




### Pass Values to Meta-Runners

A [meta-runner](working-with-meta-runner.md) allows you to extract build steps, requirements, and parameters from a build configuration and create a custom build runner.

See this article for the example of an Ant-based meta-runner that utilizes the custom `system.artifact.paths` parameter to publish artifacts via a corresponding [service message](service-messages.md): [](working-with-meta-runner.md#Preparing+Build+Configuration).





### Specify Step Execution Conditions

You can define [step execution conditions](build-step-execution-conditions.md) to specify whether individual steps should run. You can craft these conditions using [custom](typed-parameters.md) and [predefined](predefined-build-parameters.md) configuration parameters and environment variables.

To set up step execution conditions in TeamCity UI, go to step settings and click **Add condition | Other condition...**

<img src="dk-params-StepExecutionCondition.png" width="706" alt="Step execution condition"/>

For example, you can run different shell scripts depending on the build agent's operating system.

```Kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.powerShell
import jetbrains.buildServer.configs.kotlin.buildSteps.script

object StepExecutionConditions : BuildType({
    params {
        param("win.destination.path", "C:/Sources")
        param("unix.destination.path", "/Users/Admin/Sources")
    }

    steps {
        // PowerShell script runs only on Windows agents
        powerShell {
            name = "Copy File (Windows)"
            conditions {
                startsWith("teamcity.agent.jvm.os.name", "Windows")
            }
            scriptMode = script {
                content = """Copy-Item "%\system.teamcity.build.workingDir%/result.xml" -Destination %\win.destination.path%"""
            }
        }
        // Command Line runner for non-Windows agents
        script {
            name = "Copy File (Unix)"
            executionMode = BuildStep.ExecutionMode.RUN_ON_FAILURE

            conditions {
                doesNotContain("teamcity.agent.jvm.os.name", "Windows")
            }
            scriptContent = """cp "%\system.teamcity.build.workingDir%/result.xml" %\unix.destination.path%"""
        }
    }
})
```



### Specify Agent Requirements

[Agent requirements](configuring-agent-requirements.md) allow you to specify `parameter-operator-value` conditions. Only those agents that meet these conditions are allowed to build this build configuration.

You can define agent requirements using only those parameters whose values agents can report before the build starts. These parameters are:

* Predefined configuration parameters available for all agents (for example, `teamcity.agent.name`).

* Environment variables reported by agents (for example, `env.DOTNET_SDK_VERSION`).

* Custom configuration parameters that are present in agents' [buildAgent.properties](configure-agent-installation.md) files (for example, create a `custom.agent.parameter` in TeamCity UI and add the `custom.agent.parameter=MyValue` line to agents' properties files).

TeamCity automatically adds agent requirements depending on the configured build steps. For example, if a build step should be executed inside a Linux container, TeamCity adds requirements that specify an agent must have [either Docker or Podman](integrating-teamcity-with-container-managers.md) running on a Linux machine.

To define custom agent requirements in TeamCity UI, navigate to the **[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes) | Agent Requirements** tab.

<img src="dk-params-AgentRequirements.png" width="706" alt="Set agent requirements in TeamCity UI"/>

In [](kotlin-dsl.md), use the `requirements` block of your build configuration to define a requirement.

```Kotlin
import jetbrains.buildServer.configs.kotlin.*

object BuildConfig : BuildType({
    name = "Build Config"
    steps { 
        // Build steps
    }

    requirements {
        // Requirements in the following format:
        // Operator("ParameterName", "ParameterValue")
    }
})
```

For example, the following condition allows only Mac agents to run builds for the parent build configuration.

```Kotlin
matches("teamcity.agent.jvm.os.family", "Mac OS")
```
{instance="tcc"}

```Kotlin
startsWith("teamcity.agent.jvm.os.name", "Mac")
```
{instance="tc"}

The sample condition below allows builds to run only on machines with "Android" workload installed for .NET 7 SDK:

```Kotlin
contains("DotNetWorkloads_7.0", "android")
```

The following condition requires that an agent running builds has either Docker or Podman:

```Kotlin
exists("container.engine")
```






### Pass Values to Simple Script Runners

You can insert references to parameters as `%\parameter_name%` when writing scripts for [](command-line.md), [](c-script.md), and [](python.md) runners.

> Note that these runners resolve parameter values only if scripts are written directly in runner settings pages in TeamCity UI. If you include parameter references in external script files, TeamCity will not replace these references with parameter values.
> 
{style="note"}


<table><tr><td>

<tabs>

<tab title="CLI Runner">

The script below prints the [checkout directory](build-checkout-directory.md) path (configuration parameter) and TeamCity server version (environment variable) to the build log.<br/><br/>

```Shell
echo "Checkout directory: %teamcity.build.checkoutDir%"
echo "Server version: '$TEAMCITY_VERSION'"
```

The script below uses a reference to the build branch to obtain a file that should be copied to the target directory.<br/><br/>

```Shell
set -e -x

FILE=%\teamcity.build.branch%/fileToCopy.xml
if test -f "$FILE"; then
    cp "$FILE" folderA/folderB
fi
```

This sample script sends the REST API request to download the "libraries.tar.gz" archive from the server (whose URL is stored as the `serverURL` config parameter), add a build number to its name, and save it to the checkout directory. For example, the name of the archive from build #54 will be "libraries_54.tar.gz".<br/><br/>

```Shell
curl -o libraries_%\build.number%.tar.gz %\serverUrlBase%libraries.tar.gz
```

</tab>


<tab title="Python Runner">


The following script prints the [checkout directory](build-checkout-directory.md) path (configuration parameter) and TeamCity server version (environment variable) to the build log.<br/><br/>

```Python
print(f'Current checkout directory is: %teamcity.build.checkoutDir%')
print(f'TeamCity version is: %\env.TEAMCITY_VERSION%')
# or
print(f"TeamCity version is: {os.environ['TEAMCITY_VERSION']}")
```

</tab>

<tab title="C# Script Runner">

The following script obtains the [checkout directory](build-checkout-directory.md) path and appends an additional path to it:<br/><br/>

```C#
string fullPath = Path.Combine("%teamcity.build.checkoutDir%", "myFolder/bin");
Console.WriteLine(fullPath);
```

To get values of environment variables, use the [Environment.GetEnvironmentVariable](https://learn.microsoft.com/en-us/dotnet/api/system.environment.getenvironmentvariable?view=net-7.0) method:<br/><br/>

```C#
Console.WriteLine("Predefined variable value = " + System.Environment.GetEnvironmentVariable("TEAMCITY_VERSION"));
Console.WriteLine("Custom variable value = " + System.Environment.GetEnvironmentVariable("My.Custom.Env.Variable"));
```

You can also add parameter references in the `%\parameter_name%` format to the **Script parameters** field of the runner. These parameters will then be available from the global `Args` array.

See this blog post for an example of using parameters in C# scripts and .NET runner: [How to automate CI/CD tasks with C# Scripting in TeamCity](https://blog.jetbrains.com/teamcity/2021/11/how-to-automate-ci-cd-tasks-with-c-scripting-in-teamcity/).

</tab>

</tabs>

</td></tr></table>

#### Share Values Between Steps

You can use parameters to pass simple data from one step/script to another. To do this, send the `setParameter` [service message](service-messages.md) from a script that calculates new parameter values.

```Shell
echo "##teamcity[setParameter name='myParam1' value='TeamCity Agent %\teamcity.agent.name%']"
```

<snippet id="change-parameter-from-build">

In the following configuration, a C# script checks the current day of the week and writes it to the `day.of.week` parameter. A subsequent Python runner then uses the updated parameter value.

```Kotlin
object MyBuildConf : BuildType({
    params {
        param("day.of.week", "Monday")
    }
    steps {
        csharpScript {
            name = "Check the current day"
            content = """
            if ("%\day.of.week%" != DateTime.Today.DayOfWeek.ToString()) {
              string today = DateTime.Today.DayOfWeek.ToString();
              string TCServiceMessage = "##teamcity[setParameter name='day.of.week' value='" + today + "']";
              Console.WriteLine(TCServiceMessage);
            }
        """.trimIndent()
        }
        python {
            name = "Welcome message"
            command = script {
                content = "print('Hello %\teamcity.build.triggeredBy.username%, today is %\day.of.week%!')"
            }
        }
    }
})
```

</snippet>


<anchor id="Using+Build+Parameters+in+Build+Scripts"/>

### Pass Values to Builders' Configuration Files

[](net.md), [Maven](maven.md), [](gradle.md), [](ant.md) and [](nant.md) runners allow you to reference TeamCity parameters in build configuration files. This technique allows you to pass the required values to build processes.

> Parameters used in this scenario should start with either `env.` or `system.` prefixes but referenced without these prefixes. For example, use `${build.number}` in Maven configuration files to reference the predefined `system.build.number` parameter.
> 
{style="warning"}

<tabs>


<tab title=".NET">

In .NET, pass parameter values using the `$(<parameter_name>)` syntax.

> * MSBuild does not support names with dots (.), so you need to replace dots with underscores ("_") when using the parameter inside a build script.
> * The `nuget push` and `nuget delete` commands do not support parameters.
>
{style="note"}

The following sample `.csproj` file defines two custom MSBuild [targets](https://learn.microsoft.com/en-us/visualstudio/msbuild/target-element-msbuild?view=vs-2022):

```XML
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <PropertyGroup>
        <OutputZipFile>project.zip</OutputZipFile>
        <OutputUnzipDir>unzipped</OutputUnzipDir>
    </PropertyGroup>

    <Target Name="Zip">
        <ItemGroup>
            <FilesToZip Include="project.proj*" />
        </ItemGroup>
        <Exec Command="dir" />
        <Microsoft.Build.Tasks.Message Text="##teamcity[progressMessage 'Archiving files to $(OutputZipFile) file...']"/>
        <Exec Command="PowerShell -command Compress-Archive @(FilesToZip, ',') $(OutputZipFile) -Force" />
    </Target>
    <Target Name="Unzip">
        <Microsoft.Build.Tasks.Message Text="##teamcity[progressMessage 'Unzipping files to $(OutputUnzipDir) folder...']"/>
        <Exec Command="PowerShell -command Expand-Archive $(OutputZipFile) -DestinationPath $(OutputUnzipDir) -Force" />
    </Target>
</Project>
```

</tab>

<tab title="Maven">

To reference a parameter value in Maven and Ant, use the `${parameterName}` syntax.

```XML
<!--pom.xml file-->
<configuration>
    <tasks>
        <property environment="env"/>
        <echo message="TEMP = ${env.TEMP}"/>
        <echo message="TMP = ${env.TMP}"/>
        <echo message="java.io.tmpdir = ${java.io.tmpdir}"/>
        <echo message="build.number = ${build.number}"/>
    </tasks>
</configuration>
```

</tab>


<tab title="Ant">

To reference a parameter value in Maven and Ant, use the `${parameterName}` syntax.

```XML
<target name="buildmain">
    <ant dir="${teamcity.build.checkoutDir}" antfile="${teamcity.build.checkoutDir}/build-test.xml" target="masterbuild_main"/>
</target>
```

</tab>


<tab title="Gradle">

For [Gradle](gradle.md) runner, TeamCity system properties can be accessed as native Gradle properties (those defined in the `gradle.properties` file). If the property name is allowed as a Groovy identifier (does not contain dots), use the following syntax:

```Shell
println "Custom user property value is ${customUserProperty}"
```

Otherwise, if the property has dots in its name (for example, `build.vcs.number.1`), use the `project.ext["build.vcs.number.1"]` syntax instead.

</tab>

</tabs>





### Share Values Between Chain Builds

TeamCity parameters allow you to exchange values between configurations of a [build chain](build-chain.md). See this documentation article for more information: [](use-parameters-in-build-chains.md).












<seealso>
        <category ref="admin-guide">
        <a href="using-build-parameters.md">Using Build Parameters</a>
            <a href="levels-and-priority-of-build-parameters.md">Project- and Agent-Level Build Parameters</a>
            <a href="predefined-build-parameters.md">Predefined Build Parameters</a>
            <a href="typed-parameters.md">Typed Parameters</a>
            <a href="configuring-agent-requirements.md">Configuring Agent Requirements</a>
        </category>
</seealso>