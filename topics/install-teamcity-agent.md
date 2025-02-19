[//]: # (title: Install TeamCity Agent)
[//]: # (auxiliary-id: Install TeamCity Agent)

Before installing a TeamCity build agent, make sure to read the [system requirements](system-requirements.md#TeamCity+Agent+Requirements).

Choose a convenient installation option:
* using an [installer](#Install+from+Windows+Executable+File) on Windows
* [installing manually from a ZIP file](#Install+from+ZIP+File) on any platform
* preparing a container based on the [official TeamCity Agent image](https://hub.docker.com/r/jetbrains/teamcity-agent/)

## Install from Windows Executable File

1. Open the __Agents__ page in TeamCity.
2. Click __Install Build Agents__ and select _Windows Installer_ to download the installer.
3. On the agent machine, run `agentInstaller.exe` and follow the installation instructions.

Ensure that the user account used for running the agent service has proper [permissions](system-requirements.md#Common+Requirements).

## Install from ZIP File

This option allows you to download agents as archives that can copied to your agent machines.


### Available Agent Distributions

You can choose to download full or minimal agent distributions.

* The **minimal agent distribution** is a regular build agent with no plugins. The minimal agent downloads all required plugins upon its first startup.
* The **full agent distribution** includes relevant versions of all plugins currently enabled on the server. This makes the full distribution archive larger but significantly reduces the time spent on the first agent run.

Full agents are preferable if you use scripts for creating agent images (for example, [in cloud profiles](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images)). All cloud instances with full agents are synchronized with the server from the moment they start, and can run builds right away.
{instance="tc"}

Full agents are preferable if you use scripts for creating agent images. All cloud instances with full agents are synchronized with the server from the moment they start, and can run builds right away.
{instance="tcc"}


Full agent distributions are also available in two variations:

* Regular agent distributions without Java Development Kits. If you download and install this variation, make sure that the agent machine has the required JDK version installed (see [](supported-platforms-and-environments.md#Supported+Java+Versions+for+TeamCity+Agent)) and the `JRE_HOME` or `JAVA_HOME` environment variables point to the correct installation.
* Distributions bundled with OS-specific JDKs. These distributions allow you to install an agent and a JDK it requires in a single go. To download these distributions, add required JDK versions on the **Administration | Agent JDKs** page and wait for TeamCity to build a related agent distribution.

    <img src="dk-addAgentJDK.png" alt="Add Agent JDK" width="706"/>
    
    Visit the [Supported Platforms and Environments](supported-platforms-and-environments.md#Supported+Java+Versions+for+TeamCity+Agent) documentation article for the information on which Java versions are supported by TeamCity agents.



### How to Install Agents from ZIP Files

1. Open the __Agents__ page in TeamCity and select **Overview** in the side navigation bar.
2. Click the **Install agent** button and choose the required [option](#Available+Agent+Distributions).

    <img src="dk-agentsZipInstallations.png" width="706" alt="Agent ZIP distribution download options"/>

    If you choose the **Minimal ZIP file distribution** option, the minimal OS-independent archive will start downloading.

    If you opt for the **Full disributions** option, you will be redirected to the **Agent Distributions** page that groups available agent archives into two categories: with and without a bundled JDK.

    <img src="dk-fullAgentDistributions.png" width="706" alt="Full agent distributions page"/>

    Agents without bundled JDKs are available as regular archives and as [Docker images](https://hub.docker.com/r/jetbrains/teamcity-agent).

3. Extract the downloaded archive.

4. Open the `<installation path>\conf` directory and rename the `buildAgent.dist.properties` file to `buildAgent.properties`.

5. Edit the `buildAgent.properties` file to specify the TeamCity server URL (HTTPS is recommended, see [these notes](install-and-start-teamcity-agents.md#Agent-Server+Data+Transfer)), the name of the agent, and the [authentication token](install-and-start-teamcity-agents.md#Generating+Authentication+Token). Refer to [this article](configure-agent-installation.md) for details on the agent configuration.
   {instance="tcc"}

   > You can edit the `buildAgent.properties` file manually or by running the `configure` command with required parameters. See this section for more information: [](configure-agent-installation.md#The+Configure+Command).
   >
   {style="note"}

5. Edit the `buildAgent.properties` file to specify the TeamCity server URL (HTTPS is recommended, see [these notes](install-and-start-teamcity-agents.md#Agent-Server+Data+Transfer)) and the name of the agent. Refer to [this article](configure-agent-installation.md) for details on the agent configuration.
   {instance="tc"}

   > You can edit the `buildAgent.properties` file manually or by running the `configure` command with required parameters. See this section for more information: [](configure-agent-installation.md#The+Configure+Command).
   >
   {style="note"}




On Linux, you may need to give execution permissions to the `bin/agent.sh` shell script.

On Windows, you may want to install the [build agent Windows service](start-teamcity-agent.md#Build+Agent+as+Windows+Service) instead of using the manual agent startup.



## Install via Agent Push

TeamCity provides the Agent Push functionality that allows installing a build agent to a remote host. Supported combinations of the server host platform and targets for build agents:
* From a Unix-based TeamCity server, build agents can be installed to Unix hosts only (via SSH).
* From a Windows-based TeamCity server, build agents can be installed to Unix (via SSH) or Windows (via psexec) hosts.

<note>

__SSH note__

Make sure the "Password" or "Public key" authentication is enabled on the target host according to your preferred authentication method.
</note>

### Remote Host Requirements

There are several requirements for the remote host:

<table><tr>

<td width="150">

Platform

</td>

<td>

Prerequisites

</td></tr><tr>

<td>

Linux

</td>

<td>

* Installed JDK or JRE (see [](supported-platforms-and-environments.md#Supported+Java+Versions+for+TeamCity+Agent)). The JVM should be reachable via the `JAVA_HOME` or `JRE_HOME` global environment variable or be in the global path (instead of being specified in, for example, user's `.bashrc` file).
* The `unzip` utility.
* Either `wget` or `curl`.

</td></tr><tr>

<td>

Windows


</td>

<td>


<!--[//]: # (Internal note. Do not delete. "Setting up and Running Additional Build Agentsd283e644.txt")-->

* Installed JDK or JRE (see [](supported-platforms-and-environments.md#Supported+Java+Versions+for+TeamCity+Agent)).
* `Sysinternals psexec.exe` has to be installed on the TeamCity server and accessible in paths. You can install it in __Administration | Tools__. Note that [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) applies additional requirements to a remote Windows host. Make sure the following preconditions are satisfied:
    * [Administrative share](https://en.wikipedia.org/wiki/Administrative_share) on a remote host is enabled and accessible.
    * Remote services work (MMC snap-in can connect to the machine).
    * Remote registry works (`regedit` can connect to the machine via `services.msc`).
    * Server and workstation services are running (check via `services.msc`).
    * [Classic network authentication](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-access-sharing-and-security-model-for-local-accounts) is enabled.

You can test the connection by the following commands:
  ```Console
    net use \\target\Admin$ /user:Administrator 
    dir \\target\Admin$ 
   ```

</td></tr></table>

### Installation

Note that to access the __Agent Push__ page, a user needs to have the _Administer build agent machines_ permission.

1. In the TeamCity UI, open __Agents | Agent Push__ and click __Install Agent__.   
   If you want to use the same settings for several target hosts, you can __create a preset__ with these settings and use it each time when installing an agent to another remote host.
2. In the _Install agent_ dialog, either select a saved preset or choose "_Use custom settings_", specify the target host platform, and configure corresponding settings. Agent Push to a Linux system via SSH supports custom ports (the default is 22) specified as the _SSH port_ parameter. The port specified in a preset can be overridden in the host name (for example, `hostname.domain:2222`), during the actual agent installation.
3. You may need to download `Sysinternals psexec.exe`, in which case you will see the corresponding warning and a link to __Administration | Tools__ where you can download it.

>You can use Agent Push presets in [Agent Cloud profile](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images) settings to automatically install a build agent to a started cloud instance.
> 
{instance="tc"}
