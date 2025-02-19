[//]: # (title: Host Build Agents in Cloud)
[//]: # (auxiliary-id: Host Build Agents in Cloud;TeamCity Integration with Cloud Solutions;Run Build Agents in Cloud)

TeamCity integration with cloud (IaaS) solutions allows TeamCity to provide virtual machines running TeamCity agents on-demand. This allows TeamCity to automatically scale the number of active build agents depending on the current workload.

<!--autoscale-->
<!--auto-scale-->


In TeamCity Cloud, you have a choice if you want to use [preconfigured cloud agents hosted by JetBrains](teamcity-cloud-subscription-and-licensing.md#cloud-jb-hosted-agents) or configure them yourself and host them in your Cloud infrastructure. You can also combine these approaches and use them together with [hosting agents in your local environment](teamcity-cloud-subscription-and-licensing.md#cloud-self-hosted-agents)).  
One agent instance launched in your own cloud profile takes one self-hosted agent slot. It is managed according to the same licensing rules as agents hosted locally in a customer's environment.
{instance="tcc"}


## Cloud Agents and Executors

TeamCity supports two types of integrations:

* Regular cloud agents. This integration type uses cloud hosting providers (AWS, Microsoft Azure, Google Cloud, and so on) as an environment where build agents are hosted. These agents are entirely controlled by TeamCity: build queue distribution, compatibility check-ups, start-up and wind down activity, and more.

* Agentless integrations. In this scenario, a cloud provider serves as a "contractor" that handles TeamCity builds. TeamCity server sends queued builds to the "executor" and does not interfere in how it handles this queue.


Both options are available from the **[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Cloud Profiles** page.

<img src="dk-k8s-integration-overview.png" alt="K8S integration" width="706"/>

<procedure title="Regular cloud agents" instance="tc">

Available under the **Add agents to the project** section. Currently supported integrations include:

* [Amazon EC2](setting-up-teamcity-for-amazon-ec2.md)
* [VMWare vSphere](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md)
* [Kubernetes](setting-up-teamcity-for-kubernetes.md)
* [Windows Azure](https://plugins.jetbrains.com/plugin/9260-azure-resource-manager-cloud-support) (non-bundled plugin)
* [Google Cloud](https://plugins.jetbrains.com/plugin/9704-google-cloud-agents) (non-bundled plugin)
* Other integrations powered by [custom plugins](https://plugins.jetbrains.com/docs/teamcity/implement-cloud-support.html). Check out this [JetBrains Marketplace page](https://plugins.jetbrains.com/category/102-cloud-support/teamcity) for the complete list.


</procedure>


<procedure title="Regular cloud agents" instance="tcc">

Available under the **Add agents to the project** section. Currently supported integrations include:

* [Amazon EC2](setting-up-teamcity-for-amazon-ec2.md)
* [VMWare vSphere](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md)
* [Kubernetes](setting-up-teamcity-for-kubernetes.md)

</procedure>

<procedure title="Agentless Cloud Executors">

Available under the **Offload your tasks to external agents** section. Currently, this type of cloud hosting provider integrations supports only Kubernetes clusters. We expect to introduce other integrations in future release cycles.

* [Kubernetes Executor](kubernetes-executor.md)

</procedure>

This documentation section focuses primarily on regular cloud agents. For more information about the agentless executor mode, refer to this article: [](kubernetes-executor.md).


## Common Information

For each queued build, TeamCity first tries to start it on one of the self-hosted agents. If there is none available, TeamCity finds a matching cloud image with a compatible agent and starts a new instance for this image. TeamCity ensures that the number of running cloud instances limit is not exceeded.

The regular cloud agent integration requires:

* A configured virtual machine with an installed TeamCity agent in your cloud. It should be preconfigured to start the TeamCity agent on boot.
* A configured [cloud profile](#Agent+Cloud+Profiles+and+Images) in TeamCity.

Once a cloud profile is configured in TeamCity with one or several images, TeamCity does a test start of one instance for all the newly added images to learn about the agents configured on them. When the agents are connected, TeamCity stores their parameters to be able to correctly process build configurations-to-agents compatibility. An agent connected from a cloud instance started by TeamCity is automatically authorized, provided there are available agent licenses: the number of cloud agents is limited by the total number of agent licenses you have in TeamCity. After that, the agent is processed as a regular agent.

Depending on the profile settings, when TeamCity realizes it needs more agents, it can either:
* Start an existing virtual machine and stop it (after the build is finished or an idle timeout elapses). The machines that are stopped will be deallocated so the virtual machine fee does not apply when the agent is not active. The storage cost for this type of TeamCity agent will still apply.
* Create a new virtual machine from an image. Such machines will be destroyed (after the build is finished or an idle timeout elapses). This ensures that the machines will incur no further running costs.

The disconnected agent will be removed from the authorized agents list and deleted from the system to free up TeamCity build agent licenses.
{instance="tc"}

The disconnected agent will be removed from the authorized agents list and deleted from the system. On removal, one self-hosted slot will be released.
{instance="tcc"}

## Agent Cloud Profiles and Images

A _cloud profile_ is a collection of settings for TeamCity to start virtual machines with installed TeamCity agents on-demand while distributing a build queue. A cloud profile stores such general settings as:

* Credentials required to connect to a cloud provider.
* The maximum number of simultaneously active cloud agents.
* Conditions that specify when active agents should be terminated or stopped.
* TeamCity server URL that should be passed to new cloud agents when they start.

Each profile has one or multiple **cloud images** that store the following settings:

* An ID of a cloud instance to start or instance image to use.
* The container image to pull when an instance/node starts.
* Post-launch scripts.
* An [agent pool](configuring-agent-pools.md) that should own cloud agents spawned from this image.

  > You can only select the pool that contains the current project and/or its subprojects. Pools containing projects other than the current one and its subprojects will not be available for assignment. If the selected agent pool is changed in the future so that the criteria are not met or if the agent pool is not specified (or if this field is blank), TeamCity will automatically assign the cloud agents to the _project pool_.
  >
  > TeamCity automatically composes the project pool containing agents from all cloud profiles of the current project and all its subprojects. Thus, the added image will be available to all the subprojects as well. On the __Agents | Pools__ page, this pool is marked as "_\<Project name\> project pool_". Project pools cannot be deleted or modified.
  >
  {style="note"}

### Shared Profiles

A cloud profile configured in a project is available for all subprojects as well. That is, if you configure a profile in the *&lt;Root project&gt;*, all TeamCity projects will be able to start new cloud agents.

You can prevent all or individual subprojects from using cloud profiles inherited from a parent project. To do this, go to **[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Cloud profiles** and click **Change cloud integration status**.

* For a parent project: uncheck **Enable cloud integration in subprojects**.
* For a child a subproject: uncheck **Enable cloud integration in this project**.

## TeamCity Setup for Cloud Integration

This section describes general steps required for cloud integration.

The requirements for a virtual machine/image to be used for TeamCity cloud integration:
* The TeamCity agent must be correctly [installed](install-and-start-teamcity-agents.md) and configured to start [automatically](start-teamcity-agent.md#Automatic+Start) on the machine startup.
* To skip the update attempts on each agent connection to the server, make sure that the agent is up to date: start and wait for the update to complete. The agent state changes each time a plugin or tool is installed/updated/removed on the server.
* The [`buildAgent.properties`](levels-and-priority-of-build-parameters.md) file can be left "as is". The `serverUrl`, `name`, and `authorizationToken` properties can be left empty or set to any value, they are ignored when TeamCity starts the instance.

Provided these requirements are met, the usual TeamCity agent installation and cloud-provider image bundling procedures are applicable.

If you need the [connection](install-and-start-teamcity-agents.md#Agent-Server+Data+Transfer) between the server and the agent machine to be secure, you will need to set up the agent machine to establish a secure tunnel (for example, VPN) to the server on boot so that the TeamCity agent receives data via the secure channel. Keep in mind that communication between the TeamCity agent and server requires opening ports on both the agent and the server.

### Preparing Virtual Machine

1. Create and start a virtual machine with the desired OS installed.
2. Connect and log in to the virtual machine. 
3. Configure the running instance:
   1. [Install](install-and-start-teamcity-agents.md) and configure a build agent.
      * Configure the server name and agent name in the [`buildAgent.properties`](levels-and-priority-of-build-parameters.md) file — this is optional if TeamCity will be configured to launch the image, but it is useful to test the agent is configured correctly.
      * It usually makes sense to specify `tempDir` and `workDir` in `conf/buildAgent.properties` to use a non-system drive (for example, `D` drive under Windows).
   2. Install any additional software necessary for the builds on the machine (for example, Java or .NET).
   3. Start the agent and wait until it connects to the server, ensure it is operating and compatible with all the necessary build configurations (in the TeamCity UI, go to the __Agents__ page, select the build agent and view the __Compatible Configurations__ tab).
   4. Configure the system so that the agent is [started on the machine boot](start-teamcity-agent.md#Automatic+Start) (and make sure the TeamCity server is accessible on the machine boot).
   5. Check the port on which the build agent will listen for incoming data from TeamCity and open the required firewall ports (usually `9090`).
4. Test the setup by rebooting the machine and checking that the agent connects normally to the server. Once the agent connects, it will automatically update all the plugins. Wait until the agent is connected completely so that all plugins are downloaded to the agent machine.

If you want TeamCity to start an existing virtual machine and stop it after the build is finished or an idle timeout elapses, the setup above is all you need. If you want TeamCity to create and start virtual machines from an image and terminate the machine after use, the image should be captured from the virtual machine that was created.

### Capturing Image from Virtual Machine

1. Complete the steps for [creating a virtual machine](#Preparing+Virtual+Machine).
   * Remove any temporary/history information in the system.
   * Stop the agent (under Windows, stop the service but leave it in the _Automatic_ startup type).
   * (optional) Delete the content of the `logs` and `temp` directories in the [agent home](agent-home-directory.md).
   * (optional) Clean up the `<Agent Home>/conf/` directory from platform-specific files.
   * (optional) Change the [`buildAgent.properties`](levels-and-priority-of-build-parameters.md) file to remove the `name`, `serverUrl`, and `authorizationToken` properties.
2. Make a new image from the running instance. Refer your cloud provider's documentation on how to do this.

>TeamCity Agent autoupgrades whenever the agent distribution or agent plugins change on the server. If you want to reduce the agent startup time, it might make sense capturing a new virtual machine image after each TeamCity Server upgrade.

### Configuring Cloud Profile

A cloud profile is a collection of settings for TeamCity to start virtual machines with installed TeamCity agents on-demand. Cloud profiles are configured in the __Cloud Profiles__ section of __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__. See the detailed instruction for the [Amazon EC2](setting-up-teamcity-for-amazon-ec2.md), [Kubernetes](setting-up-teamcity-for-kubernetes.md), and [vSphere](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md) profiles.

## Estimating Costs

The cloud provider pricing applies. Note that the charges can depend on the specific configuration implemented to deploy TeamCity. We advise you to check your configuration and the cloud account data regularly in order to discover and prevent unexpected charges as early as possible.

Note that traffic volumes and necessary server and agent machines characteristics greatly depend on the TeamCity setup and nature of the builds run.

### Estimating Traffic
{instance="tc"}

Here are some points to help you estimate TeamCity-related traffic:

If the TeamCity server is not located within the same region or affinity group as the agent, the traffic between the server and agent is subject to usual external traffic charges imposed by your provider. When estimating traffic, remember that there are many types of traffic related to TeamCity (see the non-complete list below).

__External connections originated by server__:
* VCS servers
* Email servers
* Maven repositories
* NuGet repositories

__Internal connections originated by server__:
* TeamCity agents (checking status, sending commands, retrieving information like thread dumps, and so on)

__External connections originated by agent__:
* VCS servers (in case of agent-side checkout)
* Maven repositories
* NuGet repositories
* Any connections performed from the build process itself

__Internal connections originated by agent__:
* TeamCity server (retrieving build sources in case of server-side checkout or personal builds, downloading artifacts, and so on)

__Usual connections used by the server__:
* Web browsers
* IDE plugins

### Running Costs

Cloud providers calculate costs based on the virtual machine uptime, so it is recommended to adjust the timeout setting according to your usual builds duration. This reduces the up-time of a virtual machine. It is also highly recommended setting an execution timeout for all your builds so that a hanging build does not cause an overtime instance run with no payload.
