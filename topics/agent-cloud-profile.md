[//]: # (title: Agent Cloud Profile)
[//]: # (auxiliary-id: Agent Cloud Profile)

A _cloud profile_ is a collection of settings for TeamCity to start virtual machines with installed TeamCity agents on-demand while distributing a build queue. Profiles allow TeamCity to automatically scale the number of active build agents depending on the current workload.

<!--autoscale-->
<!--auto-scale-->

Configuring a cloud provider profile is one of the steps required to [enable agent cloud integration](teamcity-integration-with-cloud-solutions.md) between TeamCity and a cloud provider. The settings of profiles slightly vary depending on the cloud type.




>If you use Kotlin DSL for your TeamCity project settings, you need to update your DSL after upgrading as described [here](upgrading-dsl.md).

## Profiles and Images

To configure a cloud profile, go to **[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Cloud Profiles**. A cloud profile stores such settings as:

* Credentials required to connect to a cloud provider.
* The maximum number of simultaneously active cloud agents.
* Conditions that specify when active agents should be terminated or stopped.
* TeamCity server URL that should be passed to new cloud agents when they start.

For each cloud profile, create one or more cloud image. Images store such settings as:

* An ID of a cloud instance to start or instance image to use.
* The container image to pull when an instance/node starts.
* Post-launch scripts.
* An [agent pool](configuring-agent-pools.md) that should own cloud agents spawned from this image.

   > You can only select the pool that contains the current project and/or its subprojects. Pools containing projects other than the current one and its subprojects will not be available for assignment. If the selected agent pool is changed in the future so that the criteria are not met or if the agent pool is not specified (or if this field is blank), TeamCity will automatically assign the cloud agents to the _project pool_.
   > 
   > TeamCity automatically composes the project pool containing agents from all cloud profiles of the current project and all its subprojects. Thus, the added image will be available to all the subprojects as well. On the __Agents | Pools__ page, this pool is marked as "_\<Project name\> project pool_". Project pools cannot be deleted or modified.
   > 
   {style="note"}

When a build is queued, TeamCity attempts to run queued builds on regular (non-cloud) agents first. If none are currently available, TeamCity finds a compatible cloud image and (if the limit of simultaneously running instances is not yet reached) starts a new cloud instance.


## Supported Integrations

TeamCity supports integration with the following cloud providers:

<dl>

<dt>Amazon EC2</dt>
<dd>

TeamCity can manage static EC2 instances, start and terminate instances from a machine image (AMI), and request Spot Fleet instances.<br/>

[](setting-up-teamcity-for-amazon-ec2.md).
</dd>


<dt>Kubernetes</dt>

<dd>
TeamCity supports all types of Kubernetes clusters: hosting service (such as GKE from Google Cloud or Amazon EKS), self-hosted cloud clusters, and bare metal servers.<br/>

[Common Information](https://www.jetbrains.com/teamcity/integrations/cloud/kubernetes/)&emsp;|&emsp;[](setting-up-teamcity-for-kubernetes.md).
</dd>

<dt>VMware vSphere and vCenter</dt>

<dd>
TeamCity can start and stop TeamCity agents installed on VMware virtual machines.<br/>

[Common Information](https://blog.jetbrains.com/teamcity/2014/12/teamcity-vmware-vsphere-plugin/)&emsp;|&emsp;[](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md)
</dd>

<dt>Microsoft Azure</dt>

<dd>

TeamCity supports both Azure Classic and Azure Resource Manager deployment models via external plugins.<br/>

[Common Information](https://blog.jetbrains.com/teamcity/2016/04/teamcity-azure-resource-manager/)&emsp;|&emsp;[Azure Cloud Plugin](https://github.com/JetBrains/teamcity-azure-agent)

</dd>

<dt>Google Cloud</dt>

<dd>
The <b>Google Cloud Agents</b> plugin allows using Google Compute Engine to start cloud instances on demand to scale the pool of cloud build agents and also supports using cost-efficient preemptible virtual machines.<br/>

[Common Information](https://blog.jetbrains.com/teamcity/2017/06/run-teamcity-ci-builds-in-google-cloud/)&emsp;|&emsp;[Google Cloud Agents plugin](https://github.com/JetBrains/teamcity-google-agent)

</dd>

</dl>


## Shared Profiles

A cloud profile configured in a project is available for all subprojects as well. That is, if you configure a profile in the *&lt;Root project&gt;*, all TeamCity projects will be able to start new cloud agents.

You can prevent all or individual subprojects from using cloud profiles inherited from a parent project. To do this, go to **[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Cloud profiles** and click **Change cloud integration status**.

* For a parent project: uncheck **Enable cloud integration in subprojects**.
* For a child a subproject: uncheck **Enable cloud integration in this project**.



## Executors

Instead of setting up and maintaining your own build agents, you can delegate this task to an external orchestrator. Currently, only Kubernetes clusters are supported. See this article for more information: [](kubernetes-executor.md).




<!--### Specifying Profile Settings

The following profile settings have to be provided:

<table><tr>

<td>

Setting

</td>

<td>

Description

</td></tr><tr>

<td>

Profile name

</td>

<td>

Provide a name for the profile.

</td></tr><tr>

<td>

Description

</td>

<td>

Provide an optional profile description.

</td></tr><tr>

<td>

Cloud type

</td>

<td>

Select the cloud provider type from the drop-down menu.

</td></tr><tr instance="tc">

<td>

Server URL

</td>

<td>

The URL that the agents cloned from the image will use to connect to the TeamCity server. This URL must be available from the build agent machine.   
If this field is left empty, agents will use the [TeamCity server URL](configuring-server-url.md) specified on the __Administration | Global Settings__ page.

</td></tr><tr>

<td>

Terminate instance idle time

</td>

<td>

Instruct TeamCity to stop a cloud agent machine using this setting and the _Additional terminate conditions_ options. Specify the period (in minutes) for TeamCity to wait before stopping an idle build agent.   
Leave empty for no timeout.

When a cloud agent is in [maintenance mode](build-agents-configuration-and-maintenance.md#disable-for-maintenance), terminate conditions will not work. When you enable the cloud agent again, the settings will be applied.

</td></tr><tr>

<td>

Additional terminate conditions
{id="agent-terminate-condition"}

</td>

<td>

* _After certain work time (minutes)_:   
Specify the working time (in minutes) for the agent after which the instance will be terminated. If the time elapses while a build is in progress, the agent will wait until the current build finishes.
* _On idle, close to an hour (minutes left to the end of hour)_:   
Specify how many minutes before the full hour an idle instance should be stopped: this allows avoiding charges for partial hours if your virtual machines are billed in whole hours, for example, when using Amazon EC2 instances.
* _After the first build_:   
Select this option if you want TeamCity to stop the virtual machine immediately after the first build finishes. TeamCity will disable the build agents, and no more builds will be run on the same machine.

</td></tr></table>

Next, you need to provide the cloud access information which will differ depending on the provider. After that, you can check the connection and add an image to be used as a source for TeamCity cloud agents.

You can limit the number of instances across all images in the cloud profile (_Maximum instances count_) and/or set the limit per image (in [image settings](#Adding+Agent+Image)).

### Adding Agent Image

Click __Add Image__ and configure the required options for the image.

>Note that only one TeamCity build agent service can be run on each cloud instance.

You can specify which [agent pool](configuring-agent-pools.md) the agents should belong to. You can only select the pool that contains the current project and/or its subprojects. Pools containing projects other than the current one and its subprojects will not be available for assignment. If the selected agent pool is changed in the future so that the criteria are not met or if the agent pool is not specified, TeamCity will automatically assign the cloud agents to the _project pool_. You can also select the _\<Project pool\>_ in the drop-down menu manually.   
TeamCity automatically composes the project pool containing agents from all cloud profiles of the current project and all its subprojects. Thus, the added image will be available to all the subprojects as well. On the __Agents | Pools__ page, this pool is marked as "_\<Project name\> project pool_". Project pools cannot be deleted or modified.

After an agent cloud profile is created with one or several sources for virtual machines, TeamCity does a test start for all the virtual machines specified in the profile to learn about the agents configured on them. Once agents are connected, TeamCity calculates their build configurations-to-agents compatibility and stores this information.

When a cloud profile is changed, TeamCity detects modifications immediately and forces shutdown of the agents started prior to these changes once the agents finish the current build.

## Viewing Cloud Agent Information

The agents' information is displayed on the __Agents | Cloud__ page under the __&lt;Profile name&gt;__ drop-down menu.

## Enabling/disabling Cloud Integration in Project

You can enable or disable integration for a project and/or its subprojects via the TeamCity web UI using the _Change cloud integration status_ option on the __Cloud Profiles__ page.-->