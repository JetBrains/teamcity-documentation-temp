[//]: # (title: Configure Java for Agent)
[//]: # (auxiliary-id: Configure Java for Agent)

A TeamCity build agent is a Java application (see [supported Java versions](supported-platforms-and-environments.md#TeamCity+Agent)).

A build agent runs two Java processes:
* Agent Launcher — launches the agent process.
* Agent — the main process for a build agent; runs as a child process for the agent launcher.

Using 64-bit JDK (not JRE) is recommended. JDK is required for some build runners like [IntelliJ IDEA Project](intellij-idea-project.md), Java [Inspections](inspections.md), and [Duplicates](duplicates-finder-java.md). If you do not have Java builds, you can install JRE instead of JDK.

<anchor name="java-paths"/>

## Path to Java on Agent Machine

The `.exe` TeamCity agent distribution comes bundled with 64-bit Amazon Corretto 17. For the `.zip` agent installation, you need to install the appropriate Java version. Make it available via `PATH` or in one of the following places:
* the `<Agent home>/jre` directory
* the directory pointed to by the `TEAMCITY_JRE`, `JAVA_HOME`, or `JRE_HOME` environment variables (check that you only have one of the variables defined)
* if you plan to run the agent as a Windows service, make sure to set the `wrapper.java.command` property in the `<agent home>\launcher\conf\wrapper.conf` file to a valid path to the Java executable


## Upgrading Java on Agents
{help-id="UpgradeJavaOnAgents"}

If you are trying to launch an agent, and it is not able to find the required Java version in any of the [default locations](#Path+to+Java+on+Agent+Machine), the agent will report an error on starting, the process will not launch, and the agent will be shown as disconnected in the TeamCity UI.

If a build agent uses a Java version earlier than Java 8, you will see the corresponding warning on the agent's page and a [health item](server-health.md) in the UI.

Information on Java versions currently supported by TeamCity Agent is available [here](supported-platforms-and-environments.md#Supported+Java+Versions+for+TeamCity+Agent).

To update Java on agents, do one of the following:

* If an agent detects a newer Java version of the same bitness installed, the agent details page in the TeamCity UI displays the automatic update action. Click this action to restart an agent with a newer Java employed. Agents restart after they finish with their current builds and become idle.

* (Windows) Since the build agent Windows installer comes bundled with the required Java, you can just manually reinstall the agent using the Windows installer (`.exe`) obtained from the TeamCity server __Agents__ page. See [installation instructions](install-teamcity-agent.md#Install+from+Windows+Executable+File). It is important to uninstall the previous version of the agent before installing the updated agent: invoke `Uninstall.exe` in the [Agent Home Directory](agent-home-directory.md), clear all the "_Remove_" checkboxes, and click __Uninstall__.

* Install a required Java version on the agent into one of the [standard locations](#Path+to+Java+on+Agent+Machine), and restart the agent — the agent should then detect it and provide an action to use a newer Java in the UI.

* Install a required Java on the agent and configure the agent to use it.
    
See this article for more information: [How to Install Non-Bundled Version of Java](how-to.md#Install+Non-Bundled+Version+of+Java).
{instance="tc"}


>In a rare case of updating Java for the process that launches the TeamCity agent, use one of the options for the agent Java upgrade. Another way for an agent started as a Windows service, is to stop the service, change the `wrapper.java.command` variable in `buildAgent\launcher\conf\wrapper.conf` to point to the new `java.exe` binary, and restart the service.
