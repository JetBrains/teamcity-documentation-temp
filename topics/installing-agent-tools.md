[//]: # (title: Installing Agent Tools)
[//]: # (auxiliary-id: Installing Agent Tools)

In TeamCity, an _agent tool_ is a type of plugin used only for distributing files to [build agents](build-agent.md). An agent tool can be a set of files or a binary distribution. Its classes are not loaded into the runtime.

TeamCity allows you to install/remove additional tools on the server and distribute them to build agents on-demand.   
In environments with many build agents, you can centralize distribution of configuration files (for example, if you want to distribute a custom configuration file/library to all agents that require it) or remove a tool on agents at once.

The __Administration | Tools__ page provides a unified interface to set up tools to be used by appropriate plugins. You can install different versions of a tool and/or change the default one. The tools will be automatically distributed to build agents that request them and used in the related runners.

The following types of tools can be managed up via the __Administration | Tools__ page:
* __IntelliJ Inspections and Duplicates Engine__ with the IntelliJ IDEA version that TeamCity automatically downloads and installs after the server first starts. If TeamCity fails to download and install IntelliJ IDEA:
  1. [Download the required version](https://www.jetbrains.com/intellij-repository/releases/com/jetbrains/intellij/idea/ideaIU/2022.1.3/ideaIU-2022.1.3.zip).
  2. Go to **Administration | Tools**.
  3. Scroll to the **IntelliJ Inspections and Duplicates Engine** section and click **Install Version...**.
  4. Select the **Upload** mode and click **Choose File** to upload your tool.

* __JetBrains dotCover Command Line Tools__ with the bundled version set as default. Used to collect code coverage for your .NET project.
* __JetBrains ReSharper Command Line Tools__: by default the tools are bundled with TeamCity and are used by Inspections (.NET), Duplicates Finder (.NET) build runners to run code analysis.
* __Maven__: several bundled versions are displayed, with 3.6.3 set as default.
* __NuGet.exe__ used in NuGet specific build steps and NuGet Dependency Trigger. NuGet packages (`.nupkg` files) with the `tools/NuGet.exe` file inside are supported.
* __NUnit 3__: different versions can be installed and the default version set/changed.
* __Sysinternals handle.exe__ used to determine processes which hold files in the checkout directory on Windows agents.
* __Sysinternals psexec.exe__ required for installing a TeamCity agent from a Windows server to a Windows host using Agent push.
* You can also upload __your own tool as a `.zip`, `.tar`, or `.tar.gz` archive__: the structure of the tool plugin is described on the [Plugins Packaging page](https://plugins.jetbrains.com/docs/teamcity/plugins-packaging.html#Tools). TeamCity will use the name of the ZIP file as the tool name on all agents. The ZIP file will be automatically unpacked on the agents to the directory with the same name.   
When the first custom tool is installed, the __Custom Archive__ section appears on the page. In this section, you can see all the tool usages, remove the tool, or install a new one.

TeamCity places installed tools into the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/plugins/.tools` and monitors the content of this directory.

An agent downloads a tool before starting the first build which requires it. Once downloaded, the tool is stored on the agent so builds don't spend time on downloading it again.

When editing settings of a build step that directly depends on a tool, you just need to select a specific version of this tool from the drop-down menu to let TeamCity know what exact tool should be delivered to an agent.   

However, there are cases when an arbitrary build script needs a certain tool. In such case, you can reference a required tool via a `%teamcity.tool.<installed_tool_ID>%` parameter in the build step settings that support the input of parameters. You can also reference a tool in [values of build configuration parameters](configuring-build-parameters.md) or in [agent requirements](configuring-agent-requirements.md). For instance, in case with agent requirements, you can define a requirement of the type `exists` for a parameter with the name `%teamcity.tool.<installed_tool_ID>%`, and this will instruct TeamCity that the build requires the referenced tool. Before starting a build, the TeamCity server scans all build steps' settings, finds all such tool references, and informs an agent what tools are required for the build.

To check that the tool appears on the agent, look for `teamcity.tool.<installed_tool_ID>` in [configuration parameters reported by the agent](predefined-build-parameters.md#Predefined+Agent+Build+Parameters) in the TeamCity UI.

If you delete a tool in the TeamCity UI, each agent that have this tool installed will detect this, delete own copy of the tool, and restart.