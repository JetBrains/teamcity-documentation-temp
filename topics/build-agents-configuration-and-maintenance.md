[//]: # (title: Build Agents Configuration and Maintenance)
[//]: # (auxiliary-id: Build Agents Configuration and Maintenance)

## Viewing TeamCity Agents Details

The __Agents__ page of the TeamCity web UI provides the comprehensive information on the TeamCity agents. The number of tabs on the page may differ depending on your agent setup.

### Connected / Disconnected

The __Connected__ and __Disconnected__ tabs display the agents by [Agent pool](configuring-agent-pools.md) (default). To view the agents alphabetically, uncheck the __Group by agent pool__ box.   
For each pool TeamCity displays the status of its build agents. Clicking the arrow next to the pool displays the list of the pools agents with their statuses.

#### Enabling/Disabling Agents via UI

TeamCity distributes builds only among the enabled agents. Agents can be manually enabled/disabled via the web UI by clicking the status icon (1) next to the agent's name. Optionally, you can tell TeamCity to automatically disable/enable the agent after a period of time and enter your comment. TeamCity will follow the instructions and show the comment icon (2). Hovering over the icons will display the related information (3).

<img src="agent-enable-disable.png" width="483" alt="Enable/disable an agent"/>

When you click **Disable for maintenance**, a cloud agent switches to maintenance mode. In this mode, the agent will not be terminated, and you can log in to the agent, view its log, and perform other operations. Also, the agent will not be available for new builds by default.
{id="disable-for-maintenance"}

<img src="disable-for-maintenance.png" width="483" alt="Enable/disable an agent"/>

### Pools 

Refer to a [separate page](configuring-agent-pools.md) for information on configuring agent pools in TeamCity.

### Parameters Report

Filter all available agents using a specified parameter.

### Matrix and Statistics 

Refer to a separate page for information on [viewing the agents workload](viewing-agents-workload.md).

### Cloud

Lists all configured [agent cloud profiles](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images).
{instance="tc"}

Lists all configured [JetBrains-hosted agents](teamcity-cloud-subscription-and-licensing.md#cloud-jb-hosted-agents).
{instance="tcc"}

### Diff 

Compare two agents and see their differences highlighted.

## Debug Agents Remotely
{instance="tc"}

<include from="install-and-start-teamcity-agents.md" element-id="agents-terminal"/>

## Installing Software to Self-Hosted Agents
{instance="tcc"}

You can manually install any [supported software](supported-platforms-and-environments.md) to a [self-hosted build agent](teamcity-cloud-subscription-and-licensing.md#cloud-self-hosted-agents) and keep multiple versions of one tool on the same agent, if necessary.

