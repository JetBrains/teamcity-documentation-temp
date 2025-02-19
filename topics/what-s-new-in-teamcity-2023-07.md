[//]: # (title: What's New in TeamCity 2023.07)
[//]: # (auxiliary-id: What's New in TeamCity 2023.07)


## EC2 Plugin Update

In version 2023.07, we have overhauled the Amazon EC2 integration plugin. Apart from a refreshed user interface, the updated plugin features the following enhancements:

* Support for Mac AMIs. Mac VMs can be run only on dedicated Mac Mini hosts that should be booked for at least one day. Using the updated TeamCity EC2 plugin UI, you can now specify tags to locate a suitable host.
* You can now specify multiple instance types for a cloud image, which increases your chances to book a spot instance.
* The **Subnets** field now accepts multiple values, which allows you to specify different sets of incoming and outgoing traffic rules.
* TeamCity can now automatically choose Regions or Availability Zones in which your spot requests are most likely to succeed based on their [spot placement scores](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-placement-score.html#sps-example-configs). To allow TeamCity request and utilize these scores, add the `ec2:GetSpotPlacementScores` [IAM permission](setting-up-teamcity-for-amazon-ec2.md#Required+IAM+permissions).

Learn more: [](setting-up-teamcity-for-amazon-ec2.md).

## Aggregate Batch Build Artifacts

When you run a build configurations that employs the [](parallel-tests.md) build feature, TeamCity splits a build into batches interconnected in an automatically generated [chain](build-chain.md). In previous version, [artifacts](build-artifact.md) produced during such builds were published in these individual batch builds, while a parent build had none.

<img src="dk-artifacts-parallelTestsMain.png" width="706" alt="Artifacts in parallel testing"/>

When viewing completed configuration builds, you could switch to the **Dependencies** tab to access these artifacts.

<img src="dk-artifacts-parallelTests.png" width="706" alt="Artifacts in parallel testing 2"/>

Starting with this version, artifacts produced by batch builds are aggregated in the **Artifacts** tab of a main build. You can also use the `teamcity.build.parallelTests.currentBatch` [parameter](configuring-build-parameters.md) to arrange artifacts produced by batch builds into different directories.

<img src="dk-artifacts-parallelBuildAggregate.png" width="706" alt="Aggregated artifacts"/>


Learn more: [](parallel-tests.md#Publish+Artifacts+Produced+By+Batch+Builds).


## VCS Integration Enhancements

### Automated Space Connections

With this release we introduce the updated hassle-free way to set up integrations between TeamCity and JetBrains Space projects. Instead of manually creating, setting up, and installing Space applications that grant TeamCity all required permissions, you can now delegate this routine to TeamCity. All you need to do is to point TeamCity to the right Space organization, and it will do the rest for you.

<img src="dk-space-newProjectConnection.png" alt="New Space Project Connection" width="706"/>

The updated integration utilizes two types of connections:

* Organization connection — creates a basic Space application that allows TeamCity to retrieve the list of projects in your organization and create new Space applications.
* Project connections — use the parent organization connection to create applications that allow TeamCity to access individual Space projects.

Learn more: [](configuring-connections.md#jetbrains-space-connection).

### Refreshable Tokens

We keep expanding the number of VCS provider connections that support [refreshable tokens](git.md#refresh-token). Refreshable tokens allow you to ditch traditional authentication methods using username/password and static PATs (personal access tokens) in favor of short-lived tokens (non-personal or issued for the current user).

<img src="dk-2023.07-refreshTokens.png" width="706" alt="Refreshable Access Tokens"/>

In version 2023.07 refreshable tokens can be issued for [GitHub App](configuring-connections.md#GitHub) and [JetBrains Space](configuring-connections.md#jetbrains-space-connection) connections.

Learn more: [Refreshable tokens](git.md#refresh-token).

## .NET

Build agents now report the `DotNetWorkloads_<version>` parameter that returns all [.NET workloads](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-workload-install) installed on the agent machine.

Learn more: [](net.md#Parameters+Reported+by+Agent).

## REST API

You can now send a `POST` request to the following endpoint to move a build configuration to another project:

```Shell
/app/rest/buildTypes/<BuildTypeLocator>/copy?<Target_ProjectLocator>
```
{prompt="POST"}

For example, the following request finds a build configuration with the "SourceProject_MyBuildConfig" ID and moves it to "MyProject2":

```Shell
http://localhost:8111/app/rest/buildTypes/id:SourceProject_MyBuildConfig/move?targetProjectId=MyProject2
```
{prompt="POST"}


## Sakura UI

We have reworked the agent **Parameters** tab. You can navigate to this tab when viewing any TeamCity agent to instantly check this agent's configuration and environment paramters and system properties. 

<img src="dk-sakura-agentParameters.png" width="706" alt="New Agent Parameters tab"/>

Learn more: [](configuring-build-parameters.md).

## Miscellaneous

* The [DslContext](https://www.jetbrains.com/help/teamcity/kotlin-dsl-documentation/root/dsl-context/index.html?query=DslContext) object now exposes a string `serverUrl` property that allows you to get the URL of a TeamCity server in Kotlin DSL code.
* The [](net.md) runner now provides the **Excluded test assemblies** setting for the `vstest` command. This field allows you to specify paths to files the command should ignore.
    
    <img src="dk-dotnet-vstestExclide.png" width="706" alt="Excluded assemblies for vstest"/>

## Upgrade Notes
{instance="tc"}

Upgrade notes are not published for TCC.

## Fixed Issues
{instance="tc"}

Release notes are not published for TCC.



## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.

## We'd Love to Hear From You


We place a high value on your feedback and encourage you to share your thoughts and suggestions. See this link for more information: [Feedback](troubleshooting.md).
