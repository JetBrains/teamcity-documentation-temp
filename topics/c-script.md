[//]: # (title: C# Script)
[//]: # (auxiliary-id: C# Script;csharp script)

The _C# Script_ runner allows executing a [C#](https://docs.microsoft.com/en-us/dotnet/csharp/) script on Windows, Linux, or macOS. It uses a custom TeamCity tool for running a C# Interactive shell across platforms.

With this runner, you can perform various service tasks, such as preparing a build environment, creating an OS user, or reporting to different messengers.

Refer to [Configuring Build Steps](configuring-build-steps.md) for a description of common build steps' settings. Refer to [](container-wrapper.md) to learn how you can run this step inside a container.

>This [blog post](https://blog.jetbrains.com/teamcity/2021/11/how-to-automate-ci-cd-tasks-with-c-scripting-in-teamcity/) describes this runner and contains two tutorials: basic "Hello World" and more advanced Telegram bot integration. It also shows an example C# Script step configuration in [Kotlin DSL](kotlin-dsl.md). 

## Prerequisites

The runner's requirements:
* .NET runtime 6.0.0 or later must be installed on a build agent, or available in a Docker container if this step is run inside Docker.
* The [TeamCity.csi](https://www.nuget.org/packages/TeamCity.csi/) package must be [installed as an agent tool](installing-agent-tools.md).
{instance="tc"}

>__Use TeamCity.csi outside TeamCity__   
>You can use our custom tool to run tasks in C# from the command line, similarly to using a regular C# Interactive tool. TeamCity.csi can be run on Windows, Linux, and macOS. See its [README](https://github.com/JetBrains/teamcity-csharp-interactive) for more details.

## C# Script Settings

<table>

<tr>

<td>

Setting

</td>

<td>

Description

</td>

</tr>

<tr>

<td>

TeamCity C# script tool

</td>

<td>

Select a version of the TeamCity C# Script tool from those installed via __Administration | Tools__, or enter a custom path to this tool, relative to the [build checkout directory](build-checkout-directory.md).

</td>

</tr>

<tr>

<td>

Script type

</td>

<td>

Choose one of the two options: enter a custom script body right inside the runner or specify a path to a C# script file (`.csx`).

>If the submitted script references another script, make sure the path to the referenced script is specified relatively to the [build checkout directory](build-checkout-directory.md).

</td>

</tr>

<tr>

<td>

C# script

</td>

<td>

_Available for the __Custom Script__ type._
{id="custom-script" help-id="c-custom-script"}

Enter a code of a C# script.

</td>

</tr>

<tr>

<td>

C# script file
{id="script-file" help-id="c-script-file"}

</td>

<td>

_Available for the __Script File__ type._

Enter a path to the script file, relative to the [build checkout directory](build-checkout-directory.md).

</td>

</tr>

<tr>

<td>

Script parameters

</td>

<td>

Enter parameters of the script. Parameters are passed as the `Args` array. View [supported arguments](#Commands+and+Arguments+Supported+in+Scripts).

[Parameter references](configuring-build-parameters.md#Pass+Values+to+Simple+Script+Runners) are supported.

>__Passing secure values__   
>For security reasons, we highly recommend that you avoid using parameter references directly inside scripts if these parameters represent secure values. You can pass such values via script parameters instead. For example, to pass a token value, [add a new build parameter](configuring-build-parameters.md) with the "Password" type, refer it in the runner’s _Script parameters_ field:
>`%\access_token%`
>and call it as an argument within the script:
>`val accessToken = args[0]`
>This way, you can reuse the token’s value anywhere in the script.
>This practice will ensure that these values are available on the agent only during the build. Otherwise, if the parameters are specified directly inside the script, their resolved values will be stored on the agent machine as long as the script itself is stored, which might compromise the security of your data.
> 
{style="note"}

</td>

</tr>

<tr>

<td>

NuGet package sources

</td>

<td>

If you [refer to a package](#Commands+and+Arguments+Supported+in+Scripts) from a script, TeamCity will automatically download it from the feed specified in this field. The runner can automatically parse commands like `#r "nuget:System.Text.Json"` and `#r "nuget:System.Text.Json,4.7.2"`.

By default, TeamCity restores NuGet packages from their sources published on [NuGet.org](https://nuget.org). In this field, you can specify paths to other NuGet repositories, and TeamCity will search for packages there, by the order of declaration. If a package source cannot be found in any of the specified repositories, TeamCity will search for it on NuGet.org.

To be able to access private repositories, configure [NuGet feed credentials](nuget-feed-credentials.md).
{instance="tc"}

You can also refer a [TeamCity internal feed](using-teamcity-as-nuget-feed.md).
{instance="tc"}

</td>

</tr>

</table>

## Commands and Arguments Supported in Scripts

Supported commands:

<table>

<tr><td>Command</td><td>Description</td><td>Example</td></tr>

<tr>
<td>

`#r`

</td>
<td>

Add a reference to a NuGet package or an assembly and its dependencies.

</td>
<td>

`#r "nuget:<package_name>, 1.2.3"` or `#r "<library_name>.dll"`

</td>
</tr>

<tr>
<td>

`#load`

</td>
<td>

Load the specified script file and execute it.

</td>
<td>

`#load "<script_name>.csx"`

</td>
</tr>

<tr>
<td>

`#help`

</td>
<td>

Display Help on available commands and key bindings.

</td>
<td></td>
</tr>

<tr>
<td>

`#l`

</td>
<td>

Set a verbosity level to `quiet`, `normal`, or `trace`.

</td>
<td></td>
</tr>

</table>

<anchor name="call-args"/>

Supported arguments:
* __arguments__ provided in the _Script parameters_ field and stored in the `Args` array (for example, `WriteLine(Args[0])` to write the value of the first script parameter);
* __system parameters__ specified in __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes) | Parameters__ and stored in the `Props` dictionary (for example, `WriteLine(Props["version"])` to write the value of the `system.version` parameter).
