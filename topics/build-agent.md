[//]: # (title: Build Agent)
[//]: # (auxiliary-id: Build Agent)

A TeamCity _build agent_ is a piece of software which listens for the commands from the TeamCity server and starts the actual build processes. It is [installed and configured](install-and-start-teamcity-agents.md) separately from the TeamCity server. An agent can be installed on the same computer as the server or on a different machine (the latter is a preferred setup for server performance reasons); an agent can run the same operating system (OS) as the TeamCity server or a different OS.
{instance="tc"}

A TeamCity _build agent_ is a piece of software which listens for the commands from the TeamCity server and starts the actual build processes. There are two types of agents in TeamCity Cloud: JetBrains-hosted and self-hosted. The first ones are maintained and configured by JetBrains. They are started on-demand as soon as each new build requires to be run. The second ones are [stored and configured](install-and-start-teamcity-agents.md) by the customer. Both types of agents can be successfully combined in one TeamCity Cloud installation. Please see [Subscription and Licensing](teamcity-cloud-subscription-and-licensing.md) on details between these agents in terms of the TeamCity Cloud subscription.
{instance="tcc"}

> This article covers the basic build agent concepts. See the article links in the page footer for details of administering build agents.
> 
{style="tip"}

A TeamCity build agent contains [two processes](configuring-build-agent-startup-properties.md):   
* Agent launcher — a Java process that launches the agent process.
* Agent — the main process for a build agent; runs as a child process for the agent launcher.

An agent typically checks out the source code, downloads artifacts of other builds and runs the build process. An agent can run a single build at a time. The number of agents basically limits the number of parallel builds and environments in which your build processes are run.   
An agent can run builds of any compatible build configuration.

The TeamCity server monitors all the connected agents and assigns queued builds to the agents based on [compatibility requirements](agent-requirements.md), [agent pools](configuring-agent-pools.md), build configuration restrictions configured for an agent and the selection algorithm described [here](working-with-build-queue.md#Build+Queue+Optimization+by+TeamCity).

## Build Agent Status

In TeamCity, a build agent can have following statuses:

<table><tr>

<td width="200">

Status

</td>

<td>

Description

</td></tr><tr>

<td>

__Connected/ Disconnected__

</td>

<td>

An agent is connected if it is registered on the TeamCity server and responds to server commands, otherwise it is __disconnected__. This status is determined automatically.

>If an agent stays disconnected during 14 days, its state changes to _Unauthorized_. If you try to reconnect it to the server, you will have to authorize it again.  
>The default timeout duration (14 days) can be adjusted by changing the `teamcity.server.cleanup.agents.inactivityDays` [internal property](server-startup-properties.md#TeamCity+Internal+Properties).
>
{type="note" instance="tc"}

</td></tr><tr>

<td>

__Authorized/ Unauthorized__
{id="agent-authorization"}

</td>

<td>

Agents are manually authorized via the web UI on the __Agents__ page (except for the agents from the machines launched by the [cloud integrations](teamcity-integration-with-cloud-solutions.md)). Only authorized build agents can run builds. The number of authorized agents at any given time cannot exceed the number of [agent licenses](licensing-policy.md#Number+of+Agents) entered on the server. When an agent is unauthorized, a license is freed and a different build agent can be authorized. Purchase additional licenses to expand the number of agents that can concurrently run builds. When a new agent is registered on the server for the first time, it is __unauthorized__ by default and requires manual authorization to run the builds.
{instance="tc"}

Agents are manually authorized via the web UI on the __Agents__ page. Only authorized build agents can run builds. The number of authorized agents at any given time cannot exceed the number of agent licenses entered on the server. When an agent is unauthorized, a license is freed and a different build agent can be authorized. Purchase additional licenses to expand the number of agents that can concurrently run builds. When a new agent is registered on the server for the first time, it is __unauthorized__ by default and requires manual authorization to run the builds.
{instance="tcc"}

If a build agent is installed and running on the same computer as the TeamCity build server, it is authorized automatically.

</td></tr><tr>

<td>

__Enabled/ Disabled (Disabled for maintenance for cloud agents)__
{id="enable-agent"}

</td>

<td>

Agents are manually enabled/disabled via the [web UI](build-agents-configuration-and-maintenance.md#Enabling%2FDisabling+Agents+via+UI). The TeamCity server only distributes builds to agents that are enabled.

 Agent disabling does not affect (stop) the build which is currently running on the agent.

__Disabled__ agents can still run builds when the build is assigned to a special agent (for example, by [triggering a custom build](running-custom-build.md)). This feature is generally used to temporarily remove agents from the <tooltip term="build-grid">_build grid_</tooltip> to investigate agent-specific issues.

</td></tr></table>

All agents connected to the server must have unique agent names.

Only users with certain roles can manage agents. See [this article](managing-roles-and-permissions.md) for more information.

For a build agent configuration, refer to [this section](configure-agent-installation.md).

## Agent Upgrade
{instance="tc"}

TeamCity agents are automatically upgraded when needed. Typically, this happens when:

* the server is [upgraded](upgrading-teamcity-server-and-agents.md)
* an agent plugin is [added](installing-additional-plugins.md) or [updated](https://plugins.jetbrains.com/docs/teamcity/plugins-packaging.html#PluginsPackaging-AgentUpgradeonUpdatingPlugins) on the server
* [a new tool is installed](installing-agent-tools.md)

Note that updating agent plugins and receiving new files following the server upgrade may trigger an agent restart for the changes to take effect. If agents run under user accounts with [sufficient permissions](system-requirements.md#Common+Requirements), all restarts happen automatically and do not require your input.


## Agent Upgrade
{instance="tcc"}

Both JetBrains-hosted and self-hosted agents upgrade automatically when the server is upgraded. Note that receiving new files following the server upgrade may trigger an agent restart for the changes to take effect. If your self-hosted agents run under user accounts with [sufficient permissions](system-requirements.md#Common+Requirements), all restarts happen automatically and do not require your input.


## Agent Priority
{instance="tcc"}

If you have a mix of [JetBrains-hosted](supported-platforms-and-environments.md#JetBrains-Hosted+Agents) and [self-hosted](supported-platforms-and-environments.md#Self-Hosted+Agents) agents, TeamCity uses the following set of rules to pick an optimal agent that is the most balanced in terms of both performance and price:

* Self-hosted agents have priority over JetBrains-hosted agents
* [Per-month agents](managing-subscription-and-resources.md#Prepay+JetBrains-Hosted+Build+Agents+Monthly) have priority over per-minute agents
* If agents on any platform can run a build, TeamCity prioritizes Linux agents first, then Windows, and lastly, macOS.
* Priorities of AWS-hosted agents depend on their instance types. Smaller agents have priority over larger ones.
* Agents with the latest OS versions have priority over agents with older versions.
* Agents installed on x86_64 machines have priority over ARM agents.

You can manually lower or raise the priority of a self-hosted agent by modifying its integer `teamcity.agent.priority` property. This property accepts values in the `-10000` ~ `10000` range with the default value of `0`. For [EC2 build agents](setting-up-teamcity-for-amazon-ec2.md), you can set this property on the Cloud Image settings page:

<img src="dk-agentpriority.png" width="706" alt="Set the image priority for a EC2 Cloud Image"/>

For other agent types, add the following line to the [&lt;TeamCity_Agent_Home&gt;/conf/buildAgent.properties](configure-agent-installation.md) file:

```XML
teamcity.agent.priority=54
```

Note that TeamCity recognizes agent properties only after the agent is fully booted and connected to the server. For that reason, priorities for non-EC2 cloud agents apply only to active/running instances. Currently, only EC2 cloud images relay agent priorities before instances start.

## Agent Priority
{instance="tc"}

TeamCity employs an advanced agent selection logic, considering factors like CPU count, past building performance, agent sources (cloud or local), and more, to match your builds with the most suitable agents for the job.

You can manually lower or raise the priority of a any agent by modifying its integer `teamcity.agent.priority` property. This property accepts values in the `-10000` ~ `10000` range with the default value of `0`. For [AWS-hosted cloud agents](setting-up-teamcity-for-amazon-ec2.md), you can set this property on the Cloud Image settings page:

<img src="dk-agentpriority.png" width="706" alt="Set the image priority for a EC2 Cloud Image"/>

For other agent types, add the following line to the [&lt;TeamCity_Agent_Home&gt;/conf/buildAgent.properties](configure-agent-installation.md) file:

```XML
teamcity.agent.priority=54
```

Note that TeamCity recognizes agent properties only after the agent is fully booted and connected to the server. For that reason, priorities for non-EC2 cloud agents apply only to active/running instances. Currently, only EC2 cloud images relay agent priorities before instances start.

<seealso>
        <category ref="installation">
            <a href="install-and-start-teamcity-agents.md">Install and Start TeamCity Agents</a>
        </category>
        <category ref="concepts">
            <a href="agent-work-directory.md">Agent Work Directory</a>
            <a href="managing-roles-and-permissions.md">Roles and Permissions</a>
        </category>
        <category ref="admin-guide">
            <a href="build-agents-configuration-and-maintenance.md">Build Agents Configuration and Maintenance</a>
            <a href="configuring-agent-pools.md">Agent Pools</a>
            <a href="assigning-build-configurations-to-specific-build-agents.md">Assigning Build Configurations to Specific Build Agents</a>
            <a href="licensing-policy.md" instance="tc">Licensing Policy</a>
        </category>
</seealso>