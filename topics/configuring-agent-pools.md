[//]: # (title: Configuring Agent Pools)
[//]: # (auxiliary-id: Configuring Agent Pools;Agent Pools;Agent Pool)

Instead of having one common set of [build agents](build-agent.md), you can break them into separate groups called _agent pools_. A pool is a named set of agents to which you can assign projects.
* An agent can belong to _one pool only_.
* A project can use _multiple pools_ for its builds.

The number of agents authorized by the TeamCity server is limited by the number of [agent licenses](licensing-policy.md#Number+of+Agents). By default, all newly authorized agents are included into the _Default pool_.
{instance="tc"}

The number of agents authorized by the TeamCity server is limited by the number of agent licenses. By default, all newly authorized agents are included into the _Default pool_.
{instance="tcc"}

With the help of agent pools you can bind specific agents to specific projects. Project builds can be run only on build agents from the pools assigned to the project. Besides, using agent pools makes it easier to monitor the required agents' capacity.

Using agent pools allows:
* Binding specific agents to specific projects: project builds can be run only on build agents from the pools assigned to the project.
* Filtering the build queue by pools.
* Use grouping by pool on the [Agent Matrix and Agent Statistics](viewing-agents-workload.md) pages.
* Monitoring the required agents' capacity.

> You can access agent pool functionality using the TeamCity [REST API](https://www.jetbrains.com/help/teamcity/rest/manage-agent-pools.html).
> 
{style="note"}

## Required Permissions

To be able to add/remove pools and set maximum number of agents in the pool, you need to have the "_Manage agent pools_" permission. By default, these permissions are granted to the System Administrator and Agent Manager [roles](managing-roles-and-permissions.md).

Assigning and unassigning projects and agents to/from pools is restricted by the "_Change agent pools associated with project_" permission, which by default is a part of the Project Administrator role. Users can perform the operations on the pool only if they have the "_Change agent pools associated with project_" permission for _all projects_ associated with _all pools_ affected by the operation.

See also related [agent management permissions](managing-roles-and-permissions.md#Project-level+Agent+Management+Permissions).

## Managing Agent Pools

You can manage build agents on the __Agents__ page, a link to which is located in the UI header. If you are using the classic UI mode, note that it has a different navigation system than described in this article: pools are managed on the __Agents | Pools__ tab.  

The _Agents_ sidebar allows navigating between the existing agent pools and shows the agent statuses in real time.

To create a new pool, click __+__ in the sidebar and enter its name.  
By default, a pool contains an unlimited number of agents. You can set a maximum number of agents in the pool (not applicable to the _Default_ pool). If the maximum number of agents is reached, TeamСity will not allow adding any new agents to this pool. This includes moving agents from other pools and automatic authorization of cloud agents. New cloud agents will not start if the target pool is full.

To see the details of a certain pool or its nested [agent](viewing-build-agent-details.md), click its name in the sidebar.

### Agents Tab

To populate a pool with agents, click the destination pool in the sidebar, then click __Assign agents__ in the __Agents__ tab, and select the required agents from the list. Since an agent can belong to only one pool, assigning it to a pool will remove it from its previous pool. If TeamCity detects that this action might cause compatibility problems, it will show a respective warning. Removing an agent from a custom pool will return it to the _Default_ pool.

To assign a cloud agent to a pool, you need to configure it in the cloud image details of the [agent cloud profile](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images). Note that agents from all cloud profiles of the current project are automatically combined into a _project pool_.  
Only cloud agent images configured in the `<Root>` project can be moved using __Assign agents__.
{instance="tc"}

### Projects Tab

The "Projects" tab lets you specify which agent pools your projects can utilize.


* Browse the list of projects to check which projects can run their builds on agents that belong to the currently selected pool.
* Click the **Assign projects** button to allow required projects to utilize this pool.
* Click the **Dissociate** button to prevent the target project from using this pool.

<img src="dk-AgentPool-Projects.png" width="708" alt="The Projects tab of the agent pool settings page"/> 

Note that if you unbind a project from all existing agent pools, you can no longer run the build configurations of this project (unless they consist entirely of [agentless build steps](configuring-build-steps.md#Agentless+Build+Steps)).



## Favorite Pools

You can click a star icon next to the pool's name to bookmark it.

<img src="dk-favAgentPool.png" width="706" alt="Favorite pools"/>

Favorite pools are pinned to the top of the side navigation pane. You can also view all agents and cloud images from all favorite pools in the "FAVORITES" section.

<img src="dk-favAgentPool2.png" width="706" alt="Favorite pools details"/>

Favorite pools are user-specific, meaning that each TeamCity user can have their own set of favorite agent pools.



<seealso>
        <category ref="concepts">
            <a href="build-agent.md">Build Agent</a>
            <a href="working-with-build-queue.md">Working with Build Queue</a>
            <a href="configuring-agent-requirements.md">Agent Requirements</a>
        </category>
        <category ref="admin-guide">
            <a href="viewing-agents-workload.md">Viewing Agents Workload</a>
            <a href="teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images" instance="tc">Agent Cloud Profile</a>
        </category>
</seealso>
