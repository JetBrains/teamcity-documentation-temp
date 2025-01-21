[//]: # (title: Setting Up TeamCity for Amazon EC2)
[//]: # (auxiliary-id: Setting Up TeamCity for Amazon EC2)

TeamCity Amazon EC2 integration allows TeamCity auto-scale its building resources by automatically starting and stopping cloud-hosted agents on-demand, depending on the current build queue workload.

## Common Information

You can set up various types of EC2 integrations in TeamCity. Depending on the settings and the source you use, cloud AWS-hosted agents can run on:

<img src="dk-ec2-overview.png" width="706" alt="Overview image"/>

* Multiple identical instances cloned from the same Amazon Machine Image (AMI). Can be launched as On-Demand or spot instances.
* A single permanent EC2 instance managed by TeamCity. Can be shared between multiple TeamCity servers.
* A set of spot instances requested from AWS ([Spot Fleet](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet.html)).


## Prerequisites

This section describes the steps that you must perform in your AWS account before setting up a cloud profile in TeamCity UI.

### Create and Setup an EC2 Instance

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).
2. Create the required instance. Refer to these Amazon tutorials for more information: [Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html), [Windows](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/EC2_GetStarted.html), [macOS](https://aws.amazon.com/getting-started/hands-on/launch-connect-to-amazon-ec2-mac-instance/).
   
   > The exact instance type (for example, `t2.small` or `x1e.xlarge`) is irrelevant: you can set the required type in TeamCity settings (see the [](#Add+a+Cloud+Image) section).
   > 
   {style="tip"}
   
3. Install an agent on your instance. Depending on the OS type, the required steps may vary. See these articles for more information:

   * [](install-teamcity-agent.md)
   * [](configure-agent-installation.md)

4. Install any additional software required to run build steps and the TeamCity agent itself: JDKs and JREs, SDKs, runtime frameworks, building tools such as Git and Docker, and so on.
   
   > When using EC2-hosted agents, you occasionally need to update software installed on corresponding instances and rebuild AMIs. Otherwise, outdated cloud agents that connect to your TeamCity server will instantly disconnect and spend some time upgrading their tools.
   > 
   > To minimize the number of software that needs regular updates, consider opting for container orchestration solutions instead. This approach allows TeamCity to dynamically create pods that pull the [Agent Docker images](https://hub.docker.com/r/jetbrains/teamcity-agent/tags) with the latest building tools.
   > 
   > TeamCity supports integration with the following solutions:
   > 
   > * [Kubernetes](setting-up-teamcity-for-kubernetes.md) (both self-hosted clusters and hosting services, such as Google Cloud GKE or Amazon EKS)
   > * [Amazon Container Service (ECS)](https://github.com/JetBrains/teamcity-amazon-ecs-plugin)
   > 
   {style="tip"}

5. Run the agent and ensure it successfully connects to your TeamCity server and is compatible with all required configurations.
6. Configure the system to [automatically run a TeamCity agent](start-teamcity-agent.md#Automatic+Start) when an instance starts.
7. For Windows instances, follow these steps: [Additional configuration for Windows agents](#Additional+Configuration+for+Windows+Agents).
8. Restart your instance to ensure the build agent starts automatically and can connect to the server.

### Create an AMI

Skip this section if you want the instance created in the previous step to directly connect to a TeamCity server and serve as a stand-alone agent machine. Otherwise, if you want TeamCity to automatically scale the number of active cloud agents based on the current workload, create an AMI from this instance.

1. Stop the build agent. For Windows instances, stop the agent service but leave its startup type _Automatic_.
2. Remove all temporary and excess data from your instance: installation wizards, downloaded archives, build logs, and so on.
3. Optional: Delete the `<agent_home>/logs` and `<agent_home>/temp` directories.
4. Optional: Delete the `<agent_home>/conf/amazon-*` file.
5. Remove the following properties from the `<agent_home>/conf/buildAgent.properties` file:
   * `name` — TeamCity will automatically assign unique names to your instances.
   * `serverUrl` — for the EC2 integration plugin, this property is safe to remove since you will provide a server URL for all instances when setting up a cloud profile. Other plugins may require this property to be present and set to a correct value.
   * (optional) `authorizationToken` — new cloud agents are authorized automatically.
6. Stop the instance.
7. On the overview page of your instance, click **Actions | Image and Templates | Create Image**.
   
   <img src="dk-ec2-createAmi.png" width="706" alt="Create an AMI"/>

## Setup EC2 Integration in TeamCity

After you have created a required instance or AMI, you can set up cloud profiles in TeamCity UI.

### Create a Cloud Profile

A **cloud profile** is a collection of general settings for TeamCity to start virtual machines.

1. <include from="common-templates.md" element-id="open-project-settings"/> If you want cloud agents in this profile to be available globally, choose the &lt;Root project&gt;. Profiles owned by individual projects can be used to spawn agents that can be used only in these projects.
2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Cloud Profiles"/></include>
3. Click **Create new profile**.
4. Set **Cloud type** to "Amazon EC2".
5. Enter your profile name and optional description.
6. Choose an [AWS Connection](configuring-connections.md#AmazonWebServices) that should be used to access your AWS-based images and instances. Note that AWS connections of the "IAM Role" type require underlying connections of either access keys or default credentials provider chain types to operate.

   > For EC2 profiles configured before version 2024.12, TeamCity will continue supporting legacy authorization methods using access keys and the default credentials provider chain. However, it will recommend migrating these settings to a separate or existing AWS connection.
   >
   > New EC2 profiles in version 2024.12 support only connection-based access.
   {style="note"}
    <!--
    5. Choose between authentication via access key/secret pair or credentials stored locally on the server machine. Regardless of the selected mode, a user or IAM role used by TeamCity to access AWS resources must have all permissions listed in this section: [Required IAM Permissions](#Required+IAM+permissions).
    
        <table><tr><td>
    
        <tabs>
    
        <tab title="Access Keys">
    
        This option lets you specify credentials that TeamCity will use to access your AWS resources. Using static credentials is the least secure approach, we recommend the **Use default credential provider chain** option instead.
    
            5.1.&ensp;Go to the [AWS Identity and Access Management](https://console.aws.amazon.com/iam/) (IAM) dashboard.
    
            5.2.&ensp;Switch to the **Users** tab and find a user whose credentials can be used by TeamCity to access your EC2 instances and AMIs.
    
            5.3.&ensp;Switch to the **Security Credentials** tab and scroll to the **Access keys** section. 
    
            5.4.&ensp;Create a new key. Paste its ID and secret to the related fields in TeamCity UI.
    
        </tab>
    
        <tab title="Local Credentials">
    
        The **Use default credential provider chain** option allows TeamCity to look for AWS credentials stored on the server machine. Typically, the `config` file with your AWS credentials is located at `~/.aws/config` on Linux or macOS, or at `C:\Users\USERNAME\.aws\config` on Windows. This approach is more stable and secure compared to using static access keys.
    
        See the following article for more information about locally stored AWS credentials: [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html).
    
        </tab>
    
        </tabs>
    
    </td></tr></table>
    -->
7. Choose an AWS region in which your instances are hosted.
8. Set up the agent limit. This number specifies the overall limit for agents created from all cloud images of this profile.
9. Specify the TeamCity server URL. This value will be automatically passed to agents' `buildAgent.properties` files. If not specified, agents will use the same value as on the __Administration | Global Settings__ page.
10. Specify the set of criteria for winding down active cloud agents. You can choose how long agents can remain idle and (or) how long they can perform actual building routines. The agent will be terminated if any condition is met, but only after that agent finishes the current build.
   
   <img src="dk-ec2-terminateConditions.png" width="460" alt="Agents terminate conditions"/>
11. Click **Apply changes** to save the profile and exit the profile settings page.


### Add a Cloud Image

Cloud profiles specify global settings, such as authorization credentials and instance regions. Each profile can have one or multiple **images** that store settings related to the specific type of cloud instance that should be started. You can add as many images to a profile as your needs dictate. However, note that the total number of agents started by all images cannot exceed the limit set in the profile settings (and the number of agents permitted by your license).


1. Click the **Add image** button.
2. Specify the optional image name.

   <anchor name="Amazon+EC2+Spot+Fleet+Support"/>
   
   <anchor name="Amazon+EC2+Spot+Instances+Support"/>

3. Choose the required image type.

   <table><tr><td>
   
   <tabs>
   
   <tab title="AMI">

   Choose an AMI that TeamCity will use to spawn identical instances.

   <anchor name="Amazon+EC2+Launch+Templates+support"/>

   3.1.&ensp;Check the **Use launch template** option if you want TeamCity to import and use a specific [launch template](https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-templates.html). When the default/latest version of the template updates on the server, TeamCity detects these changes and updates the running instances.

   3.2.&ensp;Choose a required AMI.
   
      * **Own AMI** — TeamCity scans a collection of AMIs available under credentials specified in the cloud profile settings. You can browse all found AMIs and choose a required image.
      * **AMI by ID** — Allows you to utilize a [shared AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sharing-amis.html).
      * **AMI by tags** — Specify a comma-separated list of [AWS tags](https://docs.aws.amazon.com/tag-editor/latest/userguide/tagging.html), for example, `Owner=Mike,Project=Glacier,Subnet=Public`. If the specified tags point to multiple AMIs, TeamCity will use the most recently created AMI. If no AMIs were found, the image name under the **Agents** section will be "Image name (no data)" instead of "Image name (ami-xxxxxxxxx)".
      
         <img src="dk-ec2-invalidTags.png" width="460" alt="Invalid AMI tags"/>

   3.3.&ensp;Specify one or multiple [instance types](https://aws.amazon.com/ec2/instance-types/).
      
      * Specify one type (for example, `t2.medium`) if you need to launch On-Demand or spot instances of this specific type only. You can add multiple images that target the same AMI with different instance type values. By doing so you can set different active instance limits for each type, and manually start instances of a specific type.
      * Set multiple types (for instance, `t2.small`, `t2.medium`, and `t2.large`) if you plan to order spot instances. This approach increases your chances of having a spot instance assigned.
      
      > Mac instances support only `mac1.metal` and `mac2.metal` types and are available only as bare metal instances on [Dedicated Hosts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html), with a minimum allocation period of 24 hours before you can release the Dedicated Host. Learn more: [Amazon EC2 Mac instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-mac-instances.html).
      > 
      {style="note"}

   3.4.&ensp;Optional: Specify additional image settings.
      * **IAM Role** — The IAM role that all launched instances will assume. This role specifies the permissions [granted to applications](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) running on your EC2 instances. The AWS account used by TeamCity must have `iam:ListInstanceProfiles` and `iam:PassRole` permissions to utilize IAM roles.
      * **Key pair** — Required if you may need to connect to your EC2 instances [using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html).
      * **User data** — Allows you to specify a script that will be run when an instance launches. Learn more: [Windows](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-windows-user-data.html), [Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html).
      * **Tags** — The list of comma-separated instance tags. For example `LaunchedBy=TeamCity,TeamCityCloudProfileName=MyProfile1`. Tagging requires the `ec2:*Tags` permission. See the following section for more information: [](#Tagging).

   3.5.&ensp;Check **Use spot instances** if you prefer cheaper [spot instances](https://aws.amazon.com/ec2/spot/) to On-Demand ones. The **Max price** field lets you to specify your maximum bid price for spot instances (in US dollars). The default On-Demand price will be used if the bid price is not specified.

      TeamCity can automatically choose Regions or Availability Zones in which your spot requests are most likely to succeed based on their [spot placement scores](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-placement-score.html#sps-example-configs). To allow TeamCity to request and utilize these scores, add the `ec2:GetSpotPlacementScores` [IAM permission](#Required+IAM+permissions).
   
      > It is not recommended to use spot instances for production-critical builds due to the possibility of an [unexpected spot instance termination by Amazon](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#using-spot-instances-managing-interruptions). If a spot instance is terminated, TeamCity will fail the build with a corresponding build problem and re-add this build to the build queue.
      >
      {style="note"}
   3.6.&ensp;Specify networking settings for your EC2 instances.
      * **VPC** — The [virtual private cloud](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-vpc.html) to which new instances will belong.
      * **Subnets** — The [IP addresses range](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) for your VPS.
      * **Security groups** — Specify [rules for incoming and outgoing connections](https://docs.aws.amazon.com/vpc/latest/userguide/security-groups.html) to (from) your EC2 instance.
   
   </tab>

   <tab title="Instance">
   
   This option allows you to add the specific instance to TeamCity. Compared to AMIs that allow TeamCity to start multiple identical instances, this type implies you have a static virtual machine that TeamCity can start and stop.

   3.1.&ensp;Specify the desired [instance type](https://aws.amazon.com/ec2/instance-types/). Since there can be only one active instance, you can select only one type.

   3.2.&ensp;Choose a **Key pair** if you may need to connect to your EC2 instance [using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html).
   
   
   </tab>

   
   <tab title="Spot Fleet Request">
   
   Amazon [Spot Fleet](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet.html) allows you to book a combination of regular (On-Demand) and [spot](https://aws.amazon.com/ec2/spot/)) instances based on the given criteria. See the following AWS documentation article for sample requests: [Spot Fleet example configurations](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet-examples.html).

   3.1.&ensp;In [AWS Management Console](https://aws.amazon.com/console/), Go to __Instances | Spot Requests | Request Spot Instances__.

   3.2.&ensp;Specify the required AMI, minimum compute unit, availability zones, and other request details.

   3.3.&ensp;Click __JSON config__ to download a JSON file with the spot fleet configuration parameters. Note that only fields from the [`SpotFleetRequestConfigData`](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_SpotFleetRequestConfigData.html) class are supported.
   
      <img src="SpotFleetConfigButton.png" width="600" alt="Spot fleet config generation button in Amazon"/>

   3.4.&ensp;Paste the generated JSON configuration file in the **Spot Fleet Request** field in TeamCity.

      To run the image, TeamCity will launch spot instances matching the allocation strategy specified in the spot fleet configuration.

      > TeamCity uses own values instead of the following parameters of the JSON config file:
      > * `TerminateInstancesWithExpiration`: `false`
      > * `ValidFrom`: the time when the image is created
      > * `ValidUntil`: the `ValidFrom` value plus 5 minutes
      > * `Type`: `request`
      >
      {style="note"}

      > Currently, [LaunchTemplateConfigs](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_LaunchTemplateConfig.html) are not supported. Use [LaunchSpecifications](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_SpotFleetLaunchSpecification.html) instead to describe required Spot instances.
      >
      {style="warning"}

   </tab>
   
   </tabs>

   </td></tr></table>
   
4. Enter a integer number ranging from `-10000` to `10000` in the **Image priority** field (the default priority is `0`). When TeamCity needs to spin up a new cloud agent, it chooses an image that has the highest priority value.
   
   TeamCity uses priority values to range images from all existing profiles. For example, newly queued builds can run on cloud agents spawned from profiles A and B. The Profile A has three images with priorities 20, 40, and 60. The Profile B has 10, 30, and 50 priority images. TeamCity will spin up new agents in the following order:
   
   * Profile A, image priority 60.
   * Profile B, image priority 50.
   * Profile A, image priority 40, and so on.
   
   An image with a lower priority is used only when an available image with a higher priority reaches its active agents limit.

   > The value of the **Image priority** field is assigned to the [teamcity.agent.priority](build-agent.md#Agent+Priority) property of each agent that originates from this image. You can manually specify this property for any other non-AWS-hosted agent to control which agents TeamCity should prioritize.
   > 
   {style="note"}

5. Set up the maximum number of active cloud agents starting from this image. Note that the total number of agents started from all images added to a profile cannot exceed the limit set on the settings page of this profile.
6. Specify the [agent pool](configuring-agent-pools.md) to which newly created instances will belong.
7. Click **Save** to exit the image settings page.

After you have configured an image, TeamCity winds up one test agent for this image to test whether it can start and connect to the server. When an agent is connected and authorized, TeamCity saves its parameters to correctly assign builds to compatible agents.
   
If your image has the **Use spot instances** setting checked and a test instance cannot be launched at the moment (for example, if there is no available capacity or your bid price is too low), TeamCity will repeat attempts to launch a spot instance once a minute, as long as there are queued builds which can run on this agent.


## Manage Cloud Agents

When a build is queued, TeamCity attempts to run queued builds on regular (non-cloud) agents first. If none are currently available, TeamCity finds a compatible cloud image and (if the limit of simultaneously running instances is not yet reached) starts a new instance.

You can manually start and stop cloud agents from the **Agents** tab. Note that the **Start** button is disabled if the number of active cloud agents reaches the limit specified in the profile or image settings.

<img src="dk-ec2-startAndStopAgents.png" width="706" alt="Start and stop cloud agents"/>

The **Running instances** block shows all agents started from this specific image.

* Instances that successfully started but have not yet connected to the TeamCity server have AWS instance IDs as names ("i-xxxxxxxxxxxx").
* Connected and authorized instances have names merged from a parent image name and an AWS instance ID ("Ubuntu-22.04-Large-i-xxxxxxxxxxxx").
* If a connected agent reports outdated build tools, it will automatically disconnect for an upgrade. Once all required software is updated, the agent will reconnect to the server. TeamCity shows a corresponding warning for outdated agent images.
   
   <img src="dk-ec2-outdatedAgent.png" width="706" alt="Outdated agent image"/>

You can open an [interactive terminal](install-and-start-teamcity-agents.md#Debug+Agents+Remotely) to any active agent instance via the **Open Terminal** button. The terminal allows you to debug and maintain your cloud agent machines.


## EC2-Specific Agent Parameters

AWS-hosted agents report several [system properties](configuring-build-parameters.md) that store EC2-specific information about these machines and allow you to identify a required agent. The majority of these parameters return values of standard AWS [image metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html).

<deflist type="wide">
<def title="system.ec2.ami-id">
If the agent machine was started from an Amazon Machine Image, this property returns the corresponding AMI ID.

Example: <code>ami-08b0a7588100450ff</code>.
</def>

<def title="system.ec2.ami-launch-index">
Indicates the <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMI-launch-index-examples.html">order in which the instances were launched</a>.

Example: <code>0</code>
</def>

<def title="system.ec2.ami-manifest-path">
The path to the AMI manifest file in Amazon S3. If you used an Amazon EBS-backed AMI to launch the instance, the returned result is unknown.

Example: <code>(unknown)</code>
</def>

<def title="system.ec2.instance-id">
The ID of this instance.

Example: <code>i-07a39ee4b882caa3e</code>
</def>

<def title="system.ec2.instance-life-cycle">
The purchasing option of this instance.

Example: <code>spot</code>
</def>

<def title="system.ec2.instance-type">
The <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html">instance type</a>.

Example: <code>t2.2xlarge</code>
</def>

<def title="system.ec2.local-hostname">
The <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-naming.html">private hostname</a> of the agent instance.

Example: <code>ip-10-128-93-39.eu-west-1.compute.internal</code>
</def>

<def title="system.ec2.local-ipv4">
The private IPv4 address of the instance. 

Example: <code>10.128.93.39</code>
</def>

<def title="proposed-agent-name">
The public agent name, typically in the <code>cloudProfileName-instanceID</code> format.

Example: <code>Win-Server-2022-xLarge-i-07a39ee4b882caa3e</code>
</def>

<def title="system.ec2.public-hostname">
The instance's public DNS (IPv4).

Example: <code>ec2-54-154-113-80.eu-west-1.compute.amazonaws.com</code>
</def>

<def title="public-ipv4">
The public IPv4 address. If an Elastic IP address is associated with the instance, the value returned is the Elastic IP address.

Example: <code>54.154.113.80</code>
</def>

<def title="system.ec2.reservation-id">
The ID of the reservation.

Example: <code>r-0461cf168b52cd309</code>
</def>
</deflist>



## DSL Configuration

The following Kotlin snippet illustrates a sample [DSL configuration](kotlin-dsl.md) for a cloud profile that uses locally stored credentials and includes three images with different settings.

```Kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.amazonEC2CloudImage
import jetbrains.buildServer.configs.kotlin.amazonEC2CloudProfile
// ...

version = "%product-version%"

project {

    vcsRoot(MyRoot)

    buildType(Build)

    features {
        // Image 1: On-Demand Instances for a Specific AMI
        amazonEC2CloudImage {
            id = "PROJECT_EXT_14"
            profileId = "amazon-4"
            name = "Ubuntu-22.04-Large(AMI)"
            vpcSubnetId = "mySubnet123"
            iamProfile = "john-doe-ec2-role"
            keyPairName = "john-doe-u2204-key"
            instanceType = "t2.large"
            securityGroups = listOf("security-group-1")
            instanceTags = mapOf(
                "LaunchedBy" to "TeamCity"
            )
            maxInstancesCount = 4
            source = Source("ami-1234567890")
        }
        // Image 2: Spot Instances for an AMI Located by Tags
        amazonEC2CloudImage {
            id = "PROJECT_EXT_20"
            profileId = "amazon-4"
            name = "Ubuntu-22.04-Medium-Spot(Tags)"
            vpcSubnetId = "mySubnet123"
            instanceType = "t3a.medium,t3a.small"
            securityGroups = listOf("security-group-1,security-group-2")
            useSpotInstances = true
            spotInstanceBidPrice = 1.0
            instanceTags = mapOf(
                "LaunchedBy" to "TeamCity"
            )
            maxInstancesCount = 4
            source = Source("tags")
            param("source-tags", "OS=Ubuntu, OSVersion=22.04")
        }
        // Image 3: A Single EC2 Instance
        amazonEC2CloudImage {
            id = "PROJECT_EXT_22"
            profileId = "amazon-4"
            agentPoolId = "-2"
            name = "Windows-Server2022(Instance)"
            vpcSubnetId = "mySubnet123"
            keyPairName = "john-doe-w2022s-keys"
            instanceType = "m2.xlarge"
            securityGroups = listOf("security-group-1")
            source = Source("i-1234567890")
        }
        // Parent Profile for All Images
        amazonEC2CloudProfile {
            id = "amazon-4"
            name = "AWS EC2 Profile"
            description = "A sample Amazon EC2 profile with AMI and Instance images"
            serverURL = "https://MyTeamCityBuildFarm.dev.gg"
            terminateIdleMinutes = 30
            region = AmazonEC2CloudProfile.Regions.EU_WEST_DUBLIN
            maxInstancesCount = 20
            authType = instanceIAMRole()
            param("terminate-after-build", "false")
            param("terminateTimeOut_checkbox", "true")
        }
    }
}

object Build : BuildType({
    name = "Build"
    allowExternalStatus = true

    vcs {
        root(MyRoot)
    }

    steps {
        // Build Steps
    }

    triggers {
        vcs {}
    }

    features {
        // Build Features
    }
})

object MyRoot : GitVcsRoot({
    name = "https://github.com/...#refs/heads/main"
    url = "https://github.com/..."
    branch = "refs/heads/main"
    branchSpec = "refs/heads/*"
    authMethod = password {
        userName = "JohnDoe"
        password = "..."
    }
    param("oauthProviderId", "PROJECT_EXT_8")
})
```

## Additional Setup

### Required IAM permissions

TeamCity requires the following permissions for Amazon EC2 Resources:

* `ec2:Describe*`
* `ec2:StartInstances`
* `ec2:StopInstances`
* `ec2:TerminateInstances`
* `ec2:RebootInstances`
* `ec2:RunInstances`
* `ec2:ModifyInstanceAttribute`
* [`ec2:*Tags`](#Tagging+Instances+Launched+by+TeamCity)

To use [spot instances](#Amazon+EC2+Spot+Instances+Support), grant the following permissions in addition to those listed above:

* `ec2:RequestSpotInstances`
* `ec2:CancelSpotInstanceRequests`
* `ec2:GetSpotPlacementScores` (optional, allows TeamCity to choose AWS Regions or Availability Zones based on their [spot placement scores](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-placement-score.html#sps-example-configs)).

To use [Spot Fleets](#Amazon+EC2+Spot+Fleet+Support), the following additional permissions are required:
* `ec2:RequestSpotFleet`
* `ec2:DescribeSpotFleetRequests`
* `ec2:CancelSpotFleetRequests`
* `iam:CreateServiceLinkedRole` (if you are getting _"The provided credentials do not have permission to create the service-linked role for EC2 Spot Fleet"_ error; you can safely revoke this permission once the service role is created)

To launch an instance under an assumed IAM Role (applicable to instances cloned from AMIs and launch templates), the following additional permissions are required:
* `iam:ListInstanceProfiles`
* `iam:PassRole`

To use [encrypted EBS volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html), the following additional permissions are required:
* `kms:CreateGrant`
* `kms:Decrypt`
* `kms:DescribeKey`
* `kms:GenerateDataKeyWithoutPlainText`
* `kms:ReEncrypt*`

<!--To [connect to an agent via SSM](#debugging-and-maintenance), the following additional permissions are required:
* `sts:GetFederationToken`
* `ssm:*Session`-->

The snippet below illustrates a custom IAM policy definition that allows all EC2 operations initiated by TeamCity from the specific IP address:


```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "1",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:RebootInstances",
        "ec2:RunInstances",
        "ec2:ModifyInstanceAttribute",
        "ec2:*Tags",
        "ec2:RequestSpotInstances",
        "ec2:CancelSpotInstanceRequests",
        "ec2:GetSpotPlacementScores",
        "ec2:RequestSpotFleet",
        "ec2:DescribeSpotFleetRequests",
        "ec2:CancelSpotFleetRequests"
      ],
      "Effect": "Allow",
      "Condition": {
      	"IpAddress": {
          "aws:SourceIp": "<TeamCity server IP address>"
      	}
      },
      "Resource": "*"
    },
    {
      "Sid": "2",
      "Action": [
        "kms:CreateGrant",
        "kms:Decrypt",
        "kms:DescribeKey",
        "kms:GenerateDataKeyWithoutPlainText",
        "kms:ReEncrypt*"
      ],
      "Effect": "Allow",
      "Condition": {
      	"IpAddress": {
          "aws:SourceIp": "<TeamCity server IP address>"
      	}
      },
      "Resource": "*"
    },
    {
      "Sid": "3",
      "Action": [
        "iam:CreateServiceLinkedRole",
        "iam:ListInstanceProfiles",
        "iam:PassRole"
      ],
      "Effect": "Allow",
      "Condition": {
      	"IpAddress": {
          "aws:SourceIp": "<TeamCity server IP address>"
      	}
      },
      "Resource": "*"
    }
  ]
}
```

See also: [Example policies (Linux)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ExamplePolicies_EC2.html), [Example policies (Windows)](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ExamplePolicies_EC2.html).


### Additional Configuration for Windows Agents

_To ensure proper TeamCity agent communication with EC2 API (including access to additional drives) on Windows_, add a dependency from the TeamCity Build Agent service on the [AmazonSSMAgent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html) or [EC2Launch/EC2Config](http://docs.amazonwebservices.com/AWSEC2/latest/WindowsGuide/UsingConfig_WinAMI.html) service (the service which ensures the machine is fully initialized in regard to AWS infrastructure use). This can be done, for example, via the [Registry](https://support.microsoft.com/en-us/windows/how-to-open-registry-editor-in-windows-10-deab38e6-91d6-e0aa-4b7c-8878d9e07b11) or using [sc config](https://technet.microsoft.com/en-us/library/cc990290(v=ws.11).aspx) (for instance, `sc config TCBuildAgent depend=EC2Config`).   
Alternatively, you can use the "Automatic (delayed start)" service starting mode.

<!--

Commented out, see TW-85775

> Due to the [bug in the network settings](https://forums.aws.amazon.com/thread.jspa?threadID=242194), instance metadata is not available by default on instances based on Windows Server 2016 image. It means the TeamCity agent service cannot retrieve its properties and cloud integration doesn't work (the agent does not connect to the server or is not automatically authorized). To fix this issue, do the following:
> 
> 1. Install the [latest EC2Config](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/UsingConfig_Install.html?shortFooter=true#ec2config-update-version).
> 2. Set the dependency of the `TCBuildAgent` service on both `EC2Config` and `AmazonSSMAgent` via the command: `sc config TCBuildAgent depend=Ec2Config/AmazonSSMAgent`.
>
{style="warning"}

-->



### Amazon EBS-Optimized Instances

The behavior of [EBS optimization](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSOptimized.html) in TeamCity is similar to that offered by EC2 console. When configuring an image of the Amazon [cloud profile](agent-cloud-profile.md), the optimization can be set using the corresponding box of the Instance Type. Note that:
* EBS optimization is turned on by default for `c4.*`, `d2.*`, and `m4.*` (non-configurable);
* EBS optimization is turned off by default for any other instance types and  can be turned on for instances that support it (such as `c3.xlarge`).


### Tagging

#### Tagging Instances Launched by TeamCity

The following requirements must be met for tagging instances launched by TeamCity:
* You have all `ec2:*Tags` permissions;
* The [maximum number of tags (50)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html#tag-restrictions) for your Amazon EC2 resource is not reached.

Without tagging permissions, TeamCity can still launch Amazon AMI and EBS images with no tags applied but is unable to launch Amazon EC2 spot instances

TeamCity enables users to get instance launch information by marking the created instances with the `teamcity:TeamcityData` tag containing `<server UUID>:-<cloud profile ID>:-<image reference>`. __This tag is necessary for TeamCity integration with EC2 and must not be deleted.__

Custom tags can be applied to EC2 cloud agent instances: when configuring Cloud profile settings, in the __Add Image/Edit Image__ dialog use the __Instance tags__: field to specify tags in the format of `<key1>=<value1>,<key2>=<value2>`. [Amazon tag restrictions](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html#tag-restrictions) need to be considered.

When using the equal(=) sign in the tag value, no escaping is needed. For instance, the string `extraParam=name=John` will be parsed into `<key=extraParam>` and value `<name=John>.`

#### Tagging Instance-Dependent Resources

When launching Amazon EC2 instances, TeamCity tags all the resources (for example, volumes and network adapters) associated with the created instances, which is important when evaluating the overall cost of an instance (taking into account the storage drive type and size, I/O operations (for standard drives), network (transfers out), and so on.

#### Sharing an EBS Instance Between Multiple TeamCity Servers

As mentioned [above](#Tagging+Instances+Launched+by+TeamCity), TeamCity tags every instance it launches with the `teamcity:TeamcityData` tag that stores information about a server, cloud profile, and source (AMI or EBS-instance). So, when several TeamCity servers try to use the same EBS instance, the second one will see the following message "Instance is used by another TeamCity server. Unable to start/stop it". If you are sure that no other TeamCity servers are working with this instance, you can delete the `teamcity:TeamcityData` tag and the instance will become available for all TeamCity servers again.

### Proxy settings
{instance="tc"}

If your TeamCity server needs to use a proxy to connect to AWS API endpoint, configure the following server [internal properties](server-startup-properties.md#TeamCity+Internal+Properties) to connect to Amazon AWS addresses.

* `teamcity.http.proxy.host.ec2` — proxy server host name
* `teamcity.http.proxy.port.ec2` — proxy server port

For proxy server authentication: 

* `teamcity.http.proxy.user.ec2` — proxy access username 
* `teamcity.http.proxy.password.ec2` — proxy access user password

For NTLM authentication: 
* `teamcity.http.proxy.domain.ec2` — proxy user domain for NTLM authentication 
* `teamcity.http.proxy.workstation.ec2` — proxy access workstation for NTLM authentication

## Estimating EC2 Costs

Standard Amazon EC2 pricing applies. Amazon charges can depend on the specific configuration implemented to deploy TeamCity. We advise you to regularly check your configuration and Amazon account data to discover and prevent unexpected expenses as soon as possible.

Note that traffic volumes and necessary server and agent machines characteristics depend a big deal on the TeamCity setup and nature of the builds run. See also [Estimate Hardware Requirements for TeamCity](system-requirements.md#Estimating+External+Database+Capacity).
{instance="tc"}


### Estimating Traffic

Here are some points to help you estimate TeamCity-related traffic:

* If TeamCity server is not located within the same EC2 region or availability zone that is configured in TeamCity EC2 settings for agents, traffic between the server and agent is subject to usual Amazon EC2 external traffic charges.
* When estimating traffic, bear in mind that there are lots types of traffic related to TeamCity (see a non-complete list below).

__External connections originated by a server__:

* VCS servers
* Email servers
* Maven repositories

__Internal connections originated by a server__:

* TeamCity agents (checking status, sending commands, retrieving information like thread dumps, and so on)

__External connections originated by a agent__:
* VCS servers (in case of agent-side checkout)
* Maven repositories
* Any connections performed from the build process itself

__Internal connections originated by an agent__:
* TeamCity server (retrieving build sources in case of server-side checkout or personal builds, downloading artifacts, and so on)

__Usual connections served by a server__:
* Web browsers
* IDE plugins

### Uptime Costs

As Amazon rounds machine uptime to the full hour for some configurations (more at [How are Amazon EC2 instance hours billed?](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-instance-hour-billing/)), adjust timeout setting on the EC2 image setting on TeamCity cloud integration settings according to your usual builds length.

It is also highly recommended to set execution timeout for all your builds so that a build hanging does not cause prolonged instance running with no payload.
