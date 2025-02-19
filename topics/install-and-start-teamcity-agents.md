[//]: # (title: Install and Start TeamCity Agents)
[//]: # (auxiliary-id: Install and Start TeamCity Agents;Setting up and Running Additional Build Agents)

>This section is about [self-hosted build agents](teamcity-cloud-subscription-and-licensing.md#cloud-self-hosted-agents). [JetBrains-hosted build agents](supported-platforms-and-environments.md#JetBrains-Hosted+Agents) are maintained by the TeamCity Cloud team and require no actions from users.
>
{type="note" instance="tcc"}

A TeamCity _build agent_ is a piece of software which listens for the commands from the TeamCity server and starts the actual build processes. A production TeamCity setup requires installing additional build agents on dedicated machines. Before that, make sure to read notes on [agent-server communication](#Agent-Server+Data+Transfer), [system requirements](system-requirements.md#TeamCity+Agent+Requirements), [conflicting software](known-issues.md#Conflicting+Software), and [security](security-notes.md#Build+Agents).
{instance="tc"}

A TeamCity _build agent_ is a piece of software which listens for the commands from the TeamCity server and starts the actual build processes. A production TeamCity setup requires installing additional build agents on dedicated machines. Before that, make sure to read notes on [agent-server communication](#Agent-Server+Data+Transfer), [system requirements](system-requirements.md#TeamCity+Agent+Requirements), [security](security-notes.md#Build+Agents), [conflicting software](known-issues.md#Conflicting+Software), and the [licensing policy](teamcity-cloud-subscription-and-licensing.md) on adding new self-hosted agents.
{instance="tcc"}

If you install TeamCity bundled with a Tomcat servlet container, or use the TeamCity installer for Windows, both the server and one build agent are installed on the same machine. This is not a recommended setup for [production purposes](configure-server-installation.md#Configuring+Server+for+Production+Use) because of [security concerns](security-notes.md). Moreover, the build procedure can slow down the responsiveness of the web UI and overall TeamCity server functioning.
{instance="tc"}

<anchor name="SettingupandRunningAdditionalBuildAgents-ServerDataTransfers"/>

<anchor name="SettingupandRunningAdditionalBuildAgents-Agent-ServerDataTransfers"/>

## Agent-Server Data Transfer

[//]: # (AltHead: Server-Agent Data Transfers)

A TeamCity agent connects to the TeamCity server via the URL configured as the `serverUrl` agent property. This is called unidirectional agent-to-server connection.

Agents use unidirectional agent-to-server connection via the polling protocol: an agent establishes an HTTP(S) connection to the TeamCity Server, and polls the server periodically for server commands.

>It is recommended using __HTTPS__ for agent-to-server communications (check related [server configuration notes](how-to.md#Configure+HTTPS+for+TeamCity+Web+UI)). If the agents and the server are deployed in a secure environment, agents can be configured to use plain HTTP URL for connections to the server as this reduces transfer overhead. Note that the data travelling through the connection established from an agent to the server includes build settings, repository access credentials and keys, repository sources, build artifacts, build progress messages, and build log. In case of using the HTTP protocol that data can be compromised via the "[man in the middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)" attack.
>
{type="warning" instance="tc"}

<!--[//]: # (Internal note. Do not delete. "Setting up and Running Additional Build Agentsd283e376.txt")-->

<anchor name="SettingupandRunningAdditionalBuildAgents-InstallingAdditionalBuildAgents"/>

## Connecting Local Agents to TeamCity Server

After you [install a build agent locally](install-teamcity-agent.md), it needs to be [configured](configure-agent-installation.md) and connected to your TeamCity server or cloud instance. Watch this video for a quick guide:

<video src="https://youtu.be/dvyDCzOJJZw"
title="TeamCity tutorial — How to connect local agents to your TeamCity server"/>

>Please note that the _Use authentication token..._ option referenced in the video is currently available only for TeamCity Cloud instances.
>
{type="warning" instance="tc"}


## Cloud Agents

Hosting TeamCity agents in the cloud allows you implement a highly scalable solutions with new agents spinning up on demand and winding down when there are no builds to process. See the [](teamcity-integration-with-cloud-solutions.md) section for more information on cloud-hosted TeamCity agents.


### Generating Authentication Token
{instance="tcc"}

The recommended approach to connecting a self-hosted agent to a TeamCity Cloud instance is to generate a unique authentication token for this agent. To do this, go to __Agents__, open the __Install Build Agents__ menu in the upper right corner of the screen, and click _Use authentication token_. There are two options:

* _Generate plain-text token_: you need to copy the generated token and enter it in the [build agent configuration](configure-agent-installation.md) file. On Windows, you will be prompted to enter it right in the _Configure Build Agent Properties_ installation dialog.
* _Download config_: enter an agent name (`name` attribute in the [build agent config](configure-agent-installation.md)) and download the entire config file. Place it as the `buildAgent.properties` file in the build agent directory.

Please generate own token or configuration file per each self-hosted agent.



## Debug Agents Remotely

<snippet id="agents-terminal">

After an agent was installed and connected, you can invoke a terminal for this agent's machine directly from the TeamCity UI. This functionality lets you remotely view agent logs, check installed software, and debug specific agent issues.

To invoke a terminal, click **Agents** in the TeamCity header, choose the required agent, and click **Open terminal**.

<img src="dk-agentTerminal-2023-11.png" width="706" alt="Agent Terminal Window"/>

You can also open this terminal from the [](build-results-page.md). In this case, the terminal opens in the [checkout directory](build-checkout-directory.md) instead of the `$HOME` folder.

<img src="dk-terminal-in-checkout-folder.png" width="706" alt="Agent Terminal Window"/>

When a terminal opens, you can click the **Open in a separate tab** link to get a bigger client area.



The **Open terminal** button is available for all types of agent machines (Linux, Windows, and macOS) and invokes terminals under the same user identity who starts TeamCity agents.

To ensure your build agent is idle while you perform maintenance, disable it but do not stop it since the terminal session requires a [running](start-teamcity-agent.md) build agent. [Stopping a build agent](start-teamcity-agent.md#Stop+Build+Agent) freezes a previously open terminal tab, preventing users from typing new commands.

For cloud agents that are automatically terminated after idling for a certain period of time, you may want to click the ["Disable for maintenance..."](build-agents-configuration-and-maintenance.md#Enabling%2FDisabling+Agents+via+UI) button to keep the agent's machine running and prevent it from shutting down while you are still investigating a build problem. 

<!-- Executing terminal commands is a valid activity that prevents automatic agent shutdown. However, if the terminal is not used, the agent will shut down as scheduled. Shortly before this, TeamCity will display a pop-up notification.

<img src="dk-agent-terminal-warning.png" width="706" alt="Terminal shutdown warning"/>

-->

> The "Open interactive terminal" link opens in the `<SERVER-URL>/plugins/teamcity-agent-terminal/agentTerminal.html?agentId:<ID>` URL in a separate panel or a browser tab. If your server is [behind a proxy](multinode-setup.md#Proxy+Configuration), ensure your proxy configuration allows websocket connections to this page.
> 
{type="note" instance="tc"}

The **Open terminal** link is visible only to users whose [role permissions](managing-roles-and-permissions.md) include the *"Invoke interactive agent terminals"* permission. This permission should be granted for all projects associated with the agent pool of the corresponding agent. Users with the "Project Administrator" and "System Administrator" roles have such a permission by default. As an additional precaution, each request to open a terminal is written as a new "Agent actions | Connect to agent" activity in the [audit log](tracking-user-actions.md).

</snippet>