[//]: # (title: What's New in TeamCity 2023.11)
[//]: # (auxiliary-id: What's New in TeamCity 2023.11)

<!-- #REGION TC -->



## Matrix Build
{instance="tc"}

In TeamCity, you can leverage [build parameters](configuring-build-parameters.md) to replace raw values across build scripts, configuration/project settings, command-line arguments, and so on. Normally, each a parameter stores only one value. Starting with version 2023.11, you can add the **Matrix Build** feature to your configurations to specify a range of possible parameter values. When running such a configuration, TeamCity will spawn multiple builds to automatically cycle through these values.

<img src="dk-matrixbuilds-wn.png" width="706" alt="Matrix Build setup"/>

Adding multiple parameters, each with its own set of values, forms a matrix. TeamCity will run a build for each cell of this matrix, and report the results to the **Overview** page that allows you to identify at a glance which parameter/value combinations are failing.

<img src="matrix-build-summary.png" width="706" alt="Matrix Build Summary screen"/>

The Matrix Build feature offers multiple pre-configured options that allow you to quickly set up your build configuration so that it runs in environments that differ by the installed operating systems, default Java versions, and architectures.

Learn more: [](matrix-build.md).

## Amazon Web Services Integrations
{instance="tc"}

### EC2 Plugin Update
{instance="tc"}

We have overhauled the Amazon EC2 integration plugin. Apart from a refreshed look, the updated plugin features the following enhancements:

* You can now create images that utilize Mac AMIs. Mac VMs can be run only on dedicated Mac Mini hosts that should be booked for at least one day. Using the updated TeamCity EC2 plugin UI, you can now specify tags to locate a suitable host.
* You can now specify multiple instance types for a cloud image. This enhancement makes your cloud agent setups more versatile and reliable, and increases your chances to book a spot instance.
* The **Subnets** field now accepts multiple values, which allows you to specify different sets of incoming and outgoing traffic rules.
* The new **Image priority** setting allows you to range cloud images. When TeamCity needs to spin up a new cloud agent, it will prioritize an image with the highest priority number (given that this image has not yet reached its active agents limit).
* TeamCity can now automatically choose Regions or Availability Zones in which your spot requests are most likely to succeed based on their [spot placement scores](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-placement-score.html#sps-example-configs). To allow TeamCity request and utilize these scores, add the `ec2:GetSpotPlacementScores` [IAM permission](setting-up-teamcity-for-amazon-ec2.md#Required+IAM+permissions).

Learn more: [](setting-up-teamcity-for-amazon-ec2.md).


### S3 Plugin Update
{instance="tc"}

Version 2023.11 ships an updated S3 plugin that features the following enhancements:

<img src="dk-s3-storage-overview.png" width="706" alt="Updated S3 Plugin"/>

* Intuitive and streamlined UI designed with both Amazon S3 buckets and S3-compatible storages (such as [MinIO](https://min.io/product/s3-compatibility), [Backblaze B2](https://www.backblaze.com/cloud-storage), and others) in mind.
* Support for buckets with enabled [Transfer Acceleration](https://aws.amazon.com/s3/transfer-acceleration/).
* Hassle-free setup with the reduced number of settings. All connection-related properties are now retrieved from a selected [AWS Connection](configuring-connections.md#AmazonWebServices). The AWS region is automatically obtained from the selected bucket.
* The ability to disable integrity verification that TeamCity carries out by default for all custom S3 storages.


Learn more: [](storing-build-artifacts-in-amazon-s3.md) | [Upgrade Notes](upgrade-notes.md#2023-11-s3-update)


### AWS Connection Improvements
{instance="tc"}

New **Available for sub-projects** and **Available for build steps** settings in AWS connections allow you to ensure these connections are not used by unwanted TeamCity projects and features (for example, [](aws-credentials.md) or [custom S3 storages](storing-build-artifacts-in-amazon-s3.md)).

<img src="dk-shareAwsConnections.png" width="706" alt="Share AWS connections"/>

In addition, you can now refer to the new section of our "Configuring Connections" documentation article to learn how to configure secure AWS connections that follow Amazon guidelines and do not require permanent user credentials: [](configuring-connections.md#Recommended+Setup).
{instance="tc"}

Learn more: [](configuring-connections.md#AmazonWebServices).



## VCS Integrations
{instance="tc"}

### GitHub
{instance="tc"}

#### Seamless GitHub App Registration
{instance="tc"}

In version 2023.05, we introduced the [new type](configuring-connections.md#GitHub) of connections to GitHub and GitHub Enterprise. These connections utilize [GitHub Apps](https://docs.github.com/en/apps/using-github-apps/about-using-github-apps), instead of the traditional OAuth-based access to repositories.

Starting with version 2023.11, you will be able to create these connections much faster, without manually setting up and registering new apps in GitHub. Choose the **Automatic** creation mode in the **Add connection** dialog and TeamCity will do the rest.

<img src="dk-GhAppManifestButton.png" width="706" alt="GitHub Manifest App Button"/>

Learn more: [Configuring Connections](configuring-connections.md#GitHub).

#### Refreshable Tokens
{id="refreshable-tokens-github"}

You can now issue refreshable access tokens for [GitHub App connections](configuring-connections.md#GitHub).


### JetBrains Space
{instance="tc"}

#### Automated Connections
{instance="tc"}

With this release we introduce the updated hassle-free way to set up integrations between TeamCity and JetBrains Space projects. Instead of manually creating, setting up, and installing Space applications that grant TeamCity all required permissions, you can now delegate this routine to TeamCity. All you need to do is to point TeamCity to the right Space organization, and it will do the rest for you.

<img src="dk-space-newProjectConnection.png" alt="New Space Project Connection" width="706"/>

The updated integration utilizes two types of connections:

* Organization connection — creates a basic Space application that allows TeamCity to retrieve the list of projects in your organization and create new Space applications.
* Project connections — use the parent organization connection to create applications that allow TeamCity to access individual Space projects.

Learn more: [](configuring-connections.md#jetbrains-space-connection).


#### Automatic Status Publishing
{instance="tc"}

Starting with this version, TeamCity build configurations that target [JetBrains Space](https://www.jetbrains.com/space/) repositories do not require a configured **Commit Status Publisher** build feature to post build-related updates. Set up a TeamCity project via a predefined [Space connection](configuring-connections.md#jetbrains-space-connection) and build statuses will be posted automatically.

<img src="dk-csp-space.png" width="706" alt="Publish Space build statuses"/>

The option to manually set up the Commit Status Publisher feature is still available for those who wish full control over the process, as well as for custom setups where TeamCity is unable to publish build statuses automatically.

Learn more: [](commit-status-publisher.md#JetBrains+Space).

#### Refreshable Tokens
{id="refreshable-tokens-space"}

You can now issue refreshable access tokens for [JetBrains Space connections](configuring-connections.md#jetbrains-space-connection).


### Perforce
{instance="tc"}

#### Reusing Sources on Cloud Agents
{instance="tc"}

You can now reuse sources that are present on (or copied from) persistent storages mounted to your cloud agents. In previous versions this behavior was not possible for Perforce builds running on new agent machines.

Learn more: [](perforce-workspace-handling-in-teamcity.md#Reuse+Checked+Out+Sources+on+Cloud+Agents).

#### Perforce Helix Swarm Enhancements
{instance="tc"}

In version 2023.11, we have overhauled the "Perforce Helix Swarm" publisher of the [](commit-status-publisher.md) build feature. TeamCity can now utilize workflows and tests that already exist in your Swarm setup (instead of creating its own tests). In addition, the Publisher no longer requires credentials of a user with administrator access.

<img src="dk-swarm-personalbuild.png" width="706" alt="Personal build in TeamCity"/>

Learn more: [](integrating-with-helix-swarm.md).


### GitLab
{prdocut="tc"}

[Commit Status Publishers](commit-status-publisher.md#GitLab) and [Pull Requests](pull-requests.md#GitLab+Merge+Requests) features that target GitLab repositories can now utilize refreshable application tokens to pass the authentication.

<img src="dk-csp-GitLabToken.png" width="708" alt="Acquire access token for GitLab"/>


### Bitbucket Cloud
{instance="tc"}

[Pull Requests](pull-requests.md#Bitbucket+Cloud+Pull+Requests) features that track Bitbucket Cloud repositories have two new Authentication Type options:

* Refreshable access tokens issued via corresponding [OAuth connections](configuring-connections.md#Bitbucket+Cloud).

* Permanent access tokens issued for a specific repository, project, or workspace.

Learn more: [](pull-requests.md#Bitbucket+Cloud+Pull+Requests).


### Bitbucket Server and Data Center
{instance="tc"}

The [Pull Requests](pull-requests.md#Bitbucket+Server+Pull+Requests) feature can now utilize refreshable OAuth tokens to access repositories on Bitbucket Server / Data Center.

Learn more: [](pull-requests.md#Bitbucket+Server+Pull+Requests).


### Azure DevOps
{instance="tc"}

The [Commit Status Publisher](commit-status-publisher.md#Azure+DevOps) and [Pull Requests](pull-requests.md#Azure+DevOps+Pull+Requests) build features can now pass authentication using refreshable tokens obtained from [configured TeamCity connections](configuring-connections.md#azure-devops-connection).

<img src="dk-azureOauth-token.png" width="706" alt="Azure OAuth in CSP"/>

Learn more: [Commit Status Publisher](commit-status-publisher.md#Azure+DevOps) | [Pull Requests](pull-requests.md#Azure+DevOps+Pull+Requests).



## .NET
{instance="tc"}


* Build agents now report the `DotNetWorkloads_<version>` parameter that returns all [.NET workloads](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-workload-install) installed on the agent machine. Learn more: [](net.md#Parameters+Reported+by+Agent).

* The [](net.md) runner now provides the **Excluded test assemblies** setting for the `vstest` command. This field allows you to specify paths to files the command should ignore.

    <img src="dk-dotnet-vstestExclide.png" width="706" alt="Excluded assemblies for vstest"/>

* If your builds run a large amount of [parallel tests](parallel-tests.md) in each batch, TeamCity can automatically switch to an alternative test filtering mode that reduces potential performance issues. See this article for more information: [](parallel-tests.md#Alternative+Test+Filtering+for+.NET).



## Schedule Custom Builds
{instance="tc"}

You can now set a specific date and time when a build should run. To do this, invoke the [Run Custom Build](running-custom-build.md) dialog and use settings from the new **Date** section.

<img src="dk-customRun-general.png" width="706" alt="Run custom build dialog, General Settings tab"/>

Learn more: [Run Custom Build](running-custom-build.md#Date+%26+Time).


## Build Cache
{instance="tc"}

The new **Build Cache** feature allows configurations to cache files required by builds (for instance, downloaded npm packages) and reuse them in consecutive builds. This technique assists build agents in offloading excessive operations and can significantly speed up your building routines.

<img src="dk-buildCaches-singleConfDescription.png" width="706" alt="Build Cache feature"/>

In addition to sharing caches with its own future builds, a configuration that caches files can pass them to other build configurations within the same project.

Learn more: [](build-cache.md).


## Agents with Bundled JDKs
{instance="tc"}

Starting with version 2023.11, you can build distributions of TeamCity agents bundled with custom JDKs. These distributions allow you to install both an agent and a JDK it requires to operate in one go.

<img src="dk-fullAgentDistributions.png" width="706" alt="Full agent distributions page"/>

To create a custom agent distribution, navigate to **Administration | Agent JDKs** and add a new JDK option (you will need to specify the platform, the architecture, and a link for TeamCity to download this specific JDK).

<img src="dk-addAgentJDK.png" alt="Add Agent JDK" width="706"/>

When a new option is added, TeamCity will start building your custom agent distribution. You can download custom agent+JDK bundles by clicking **Install agent | Full distributions** on the **Agents | Overview** page.

Learn more: [](install-teamcity-agent.md).



## Versioned Settings: Load Additional Settings From a VCS
{instance="tc"}

Starting from this version, TeamCity can load custom snapshot dependencies, VCS roots and checkout rules from settings stored in a version control system. As a result, you now have even more flexibility to edit versioned settings and create custom branches with settings that significantly differ from those in default/stable branches.

When detecting these previously ignored settings, TeamCity dynamically creates required hidden entities (such as virtual build configurations) that are in effect only for the current build and remain hidden for other revisions/branches that use different settings.

<img src="dk-vcsSettings-5step.png" width="706" alt="5-Step Setup"/>

To enable the updated behavior, tick the **Apply changes in snapshot dependencies and version control settings** option on your project's **Versioned Settings** page.

Learn more: [](storing-project-settings-in-version-control.md#Apply+Changes+in+Snapshot+Dependencies+and+Version+Control+Settings).


<!--
## Remote Parameters
{instance="tc"}

You can now select the **Remote** parameter type when setting up a [parameter specification](typed-parameters.md#Adding+Parameter+Specification). Unlike regular parameters whose values are stored in TeamCity, remote parameters retrieve their values from an external source.

Currently, remote parameters support only [HashiCorp Vault](https://www.vaultproject.io) as an external value storage. Using this type of parameters, build agents can obtain secrets managed by KV, KV2, AWS, Google Cloud, and other Vault engines.

We are committed to expanding storage options in upcoming TeamCity releases and highly value your input. Your feedback matters; please share your suggestions on which remote sources you'd like us to prioritize for future support.

Learn more: [](hashicorp-vault.md).

> The [HashiCorp Vault](https://plugins.jetbrains.com/plugin/10011-hashicorp-vault-support) plugin is not bundled with TeamCity; you need to download and install it manually. We expect to bundle it during the next release cycle.
>
{style="note"}
-->


## Access Parallel Builds' Artifacts from a Primary Build
{instance="tc"}

When you run a build configurations that employs the [](parallel-tests.md) build feature, TeamCity splits a build into batches interconnected in an automatically generated [chain](build-chain.md). In previous version, [artifacts](build-artifact.md) produced during such builds were published in these individual batch builds, while a parent build had none.

<img src="dk-artifacts-parallelTestsMain.png" width="706" alt="Artifacts in parallel testing"/>

As a workaround, you could switch to the **Dependencies** tab when viewing completed configuration builds.

<img src="dk-artifacts-parallelTests.png" width="706" alt="Artifacts in parallel testing 2"/>

Starting with this version, artifacts produced by batch builds are aggregated in the **Artifacts** tab of a main build. You can also use the `teamcity.build.parallelTests.currentBatch` [parameter](configuring-build-parameters.md) to arrange artifacts produced by batch builds into different directories.

<img src="dk-artifacts-parallelBuildAggregate.png" width="706" alt="Aggregated artifacts"/>


Learn more: [](parallel-tests.md#Publish+Artifacts+Produced+By+Batch+Builds).



## Step Statuses and IDs
{instance="tc"}

Starting with version 2023.11, you can specify IDs for your steps (similarly to project and configuration IDs).

<img src="dk-stepID.png" width="706" alt="Step ID"/>

TeamCity uses these IDs to generate new `teamcity.build.step.status.<step_ID>` parameters that report the step exit statuses ("success", "failure", or "cancelled").

<img src="dk-parametersTab-statuses.png" width="706" alt="Step statuses"/>

You can use these values to perform custom actions depending on the statuses of previous steps. For example, you can craft custom step execution conditions.

Learn more: [](configuring-build-steps.md#Step+Status+Parameters).



## Additional ReSharper Plugins for the Inspections Runner
{instance="tc"}

The [](inspections-resharper.md) runner now features the **R# CLT Plugins** field that allows you to add your favorite ReSharper plugins (such as [StyleCop](https://plugins.jetbrains.com/plugin/11619-stylecop-by-jetbrains), [CleanCode](https://plugins.jetbrains.com/plugin/11677-cleancode), or [Unity Support](https://plugins.jetbrains.com/plugin/11629-unity-support)) downloaded from JetBrains Marketplace or installed from a local storage.

<img src="dk-inspections-plugins.png" width="706" alt="ReSharper plugins list"/>

Learn more: [](inspections-resharper.md#JetBrains+ReSharper+Command+Line+Tools+Settings).



## Service Messages
{instance="tc"}

Added the [new service message](service-messages.md#Writing+the+File+into+the+Build+Log) that allows you to track the contents of the given file and echo new lines to the build log.

<img src="dk-streamFiletoLog.png" width="706" alt="Stream file to log"/>

Learn more: [](service-messages.md#Writing+the+File+into+the+Build+Log).



## REST API
{instance="tc"}

### Move Configurations

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


### Wind Down Cloud Instances

Previously, you could send the DELETE request to a running cloud agent to terminate it.

```Shell
/app/rest/cloud/instances/<cloudInstanceLocator>
```
{prompt="DELETE"}

Starting with this version, you can stop cloud instances via POST requests to the following endpoints:

```Shell
/app/rest/cloud/instances/<cloudInstanceLocator>/actions/stop
/app/rest/cloud/instances/<cloudInstanceLocator>/actions/forceStop
```
{prompt="POST"}

Use the `...actions/stop` endpoint to issue a "soft" stop request: if the target agent is currently busy, it will stop after the build finishes.

The `...actions/forceStop` endpoint allows you to stop a cloud instance even if it is busy.

Learn more: [Start and Stop Cloud Instances](https://www.jetbrains.com/help/teamcity/rest/manage-cloud-profiles.html#Start+and+Stop+Cloud+Instances).



## Sakura UI and UX Enhancements
{instance="tc"}

* We have reworked the agent **Parameters** tab. You can navigate to this tab when viewing any TeamCity agent to instantly check this agent's configuration and environment paramters and system properties.

  <img src="dk-sakura-agentParameters.png" width="706" alt="New Agent Parameters tab"/>

* You can now [bookmark required agent pools](configuring-agent-pools.md#Favorite+Pools) to easily access them from the top of the agents and pools list.

* The [](build-results-page.md#Dependencies+Tab) now displays a find panel that allows you to search for specific dependent builds by configuration names.

* The [Interactive Agent Terminal](install-and-start-teamcity-agents.md#Debug+Agents+Remotely) introduced in version 2023.05 now opens in a panel docked to the bottom of an agent details or build results page. You can move it to a separate browser tab by clicking **Open in a separate tab**.

  <img src="dk-agentTerminal-2023-11-new.png" width="706" alt="Agent Terminal Window"/>

* [](performance-monitor.md) now shows absolute values of the consumed/total agent memory.

* You can now switch [](build-log.md) timestamps from absolute values to relative to quickly analyze how long it took the build to reach a specific stage.

  <img src="dk-relativeBuildLogTime.png" width="706" alt="Relative timestamps"/>

* You can now view and copy connection IDs from the **Connection** pages in TeamCity UI. This minor enhancement facilitates writing [](kotlin-dsl.md) code for objects that utilize connections: [AWS Credentials features](aws-credentials.md), [](docker-support.md), [S3 artifact storages](storing-build-artifacts-in-amazon-s3.md), AWS connections that [use other connections](configuring-connections.md#Recommended+Setup), and so on.

  <img src="dk-copy-connection-id.png" alt="Copy connection ID" width="706"/>

* Cloud images' **Build History** tab now displays a search box that allows you to find all builds of a specific cloud agent, even if this instance is no longer available.

  <img src="dk-deletedAgentHistory.png" width="706" alt="History for deleted agent"/>

* The navigation bar now paints the changes indicator bold for favorite projects with changes from the current TeamCity user.

  <img src="dk_boldChangesIndicator.png" width="706" alt="Highlighted projects with current user changes"/>



## Miscellaneous
{instance="tc"}

* The [DslContext](https://www.jetbrains.com/help/teamcity/kotlin-dsl-documentation/root/dsl-context/index.html?query=DslContext) object now exposes a string `serverUrl` property that allows you to get the URL of a TeamCity server in Kotlin DSL code.
* TeamCity distributes agents more effectively and processes large build chains with failing builds faster. Starting with version 2023.11, dependent builds whose ["On failed dependency" condition](snapshot-dependencies.md) is "Make build failed to start" no longer wait for an available agent when their dependencies fail or are canceled. Instead, the dependent build's status changes to "Failed to start" as soon as possible, and TeamCity proceeds to the next build in the chain.
* The [](commit-status-publisher.md) build feature now correctly publishes build statuses for configurations that target `refs/(merge-requests/*)/head` branches of GitLab repositories (the "merge result" branches). Previously, running TeamCity builds for merge result revisions caused the Publisher to encounter HTTP 404 errors.
* If users log into TeamCity using credentials of an external 2FA-protected service, TeamCity does not send additional 2FA requests. Learn more: [](managing-two-factor-authentication.md#Reduce+Excessive+Authorization+Requests).
* You can now add the `dateFormat=<value>` parameter to URLs used by your log analysis tools to retrieve build logs. Learn more: [](build-log.md#Modify+the+DateTime+Pattern).
* In addition to `builds_queued`, `builds_started`, `builds_running` and `builds_queued` [metrics](teamcity-monitoring-and-diagnostics.md#Metrics), TeamCity now reports an experimental `builds_detached` metric that allows you to view the number of [builds detached from their agents](configuring-build-steps.md#Agentless+Build+Steps).



<!-- #ENDREGION -->








<!-- #REGION TCC -->



## Matrix Build
{instance="tcc"}

In TeamCity, you can leverage [build parameters](configuring-build-parameters.md) to replace raw values across build scripts, configuration/project settings, command-line arguments, and so on. Normally, each a parameter stores only one value. Starting with version 2023.11, you can add the **Matrix Build** feature to your configurations to specify a range of possible parameter values. When running such a configuration, TeamCity will spawn multiple builds to automatically cycle through these values.

<img src="dk-matrixbuilds-wn.png" width="706" alt="Matrix Build setup"/>

Adding multiple parameters, each with its own set of values, forms a matrix. TeamCity will run a build for each cell of this matrix, and report the results to the **Overview** page that allows you to identify at a glance which parameter/value combinations are failing.

<img src="matrix-build-summary.png" width="706" alt="Matrix Build Summary screen"/>

The Matrix Build feature offers multiple pre-configured options that allow you to quickly set up your build configuration so that it runs in environments that differ by the installed operating systems, default Java versions, and architectures.

Learn more: [](matrix-build.md).

## AWS Connection Improvements
{instance="tcc"}

New **Available for sub-projects** and **Available for build steps** settings in AWS connections allow you to ensure these connections are not used by unwanted TeamCity projects and features (for example, [](aws-credentials.md)).

<img src="dk-shareAwsConnections.png" width="706" alt="Share AWS connections"/>

Learn more: [](configuring-connections.md#AmazonWebServices).


## VCS Integrations
{instance="tcc"}

### Seamless GitHub App Registration
{instance="tcc"}


In version 2023.05, we introduced the [new type](configuring-connections.md#GitHub) of connections to GitHub and GitHub Enterprise. These connections utilize [GitHub Apps](https://docs.github.com/en/apps/using-github-apps/about-using-github-apps), instead of the traditional OAuth-based access to repositories.

Starting with version 2023.11, you will be able to create these connections much faster, without manually setting up and registering new apps in GitHub. Choose the **Automatic** creation mode in the **Add connection** dialog and TeamCity will do the rest.

<img src="dk-GhAppManifestButton.png" width="706" alt="GitHub Manifest App Button"/>

Learn more: [Configuring Connections](configuring-connections.md#GitHub).

### Reusing Perforce Sources on Cloud Agents
{instance="tcc"}

You can now reuse sources that are present on (or copied from) persistent storages mounted to your cloud agents. In previous versions this behavior was not possible for Perforce builds running on new agent machines.

Learn more: [](perforce-workspace-handling-in-teamcity.md#Reuse+Checked+Out+Sources+on+Cloud+Agents).



## Schedule Custom Builds
{instance="tcc"}

When invoking new builds from the [Run Custom Build](running-custom-build.md) dialog, you now have an option to specify the specific date &amp; time when this build should run.

<img src="dk-customRun-general.png" width="706" alt="Run custom build dialog, General Settings tab"/>

Learn more: [Run Custom Build](running-custom-build.md#Date+%26+Time).



## Agents with Bundled JDKs
{instance="tcc"}

Starting with version 2023.11, you can build distributions of TeamCity agents bundled with custom JDKs. These distributions allow you to install both an agent and a JDK it requires to operate in one go.

<img src="dk-fullAgentDistributions.png" width="706" alt="Full agent distributions page"/>

To create a custom agent distribution, navigate to **Administration | Agent JDKs** and add a new JDK option (you will need to specify the platform, the architecture, and a link for TeamCity to download this specific JDK).

<img src="dk-addAgentJDK.png" alt="Add Agent JDK" width="706"/>

When a new option is added, TeamCity will start building your custom agent distribution. You can download custom agent+JDK bundles by clicking **Install agent | Full distributions** on the **Agents | Overview** page.

Learn more: [](install-teamcity-agent.md).



## Versioned Settings: Load Additional Settings From a VCS
{instance="tcc"}

Starting from this version, TeamCity can load custom snapshot dependencies, VCS roots and checkout rules from settings stored in a version control system. As a result, you now have even more flexibility to edit versioned settings and create custom branches with settings that significantly differ from those in default/stable branches.

When detecting these previously ignored settings, TeamCity dynamically creates required hidden entities (such as virtual build configurations) that are in effect only for the current build and remain hidden for other revisions/branches that use different settings.

<img src="dk-vcsSettings-5step.png" width="706" alt="5-Step Setup"/>

To enable the updated behavior, tick the **Apply changes in snapshot dependencies and version control settings** option on your project's **Versioned Settings** page.

Learn more: [](storing-project-settings-in-version-control.md#Apply+Changes+in+Snapshot+Dependencies+and+Version+Control+Settings).



## Token-Based Authentication
{instance="tcc"}

Version 2023.11 allows your [Pull Request](pull-requests.md) features to utilize tokens to access repositories hosted on Bitbucket Cloud and Server/Data Center.

* For Bitbucket Server/Data Center repositories, the Pull Requests feature can now use refreshable OAuth tokens issued via [TeamCity connections](configuring-connections.md#Bitbucket+Server+and+Data+Center).

* For Bitbucket Cloud, you can specify permanent access tokens issued for a specific repository, project, or workspace.

* For Azure DevOps projects, the [Commit Status Publisher](commit-status-publisher.md#Azure+DevOps) and [Pull Requests](pull-requests.md#Azure+DevOps+Pull+Requests) features can now utilize refreshable tokens obtained via configured OAuth connections.




## Additional ReSharper Plugins for the Inspections Runner
{instance="tcc"}

The [](inspections-resharper.md) runner now features the **R# CLT Plugins** field that allows you to add your favorite ReSharper plugins (such as [StyleCop](https://plugins.jetbrains.com/plugin/11619-stylecop-by-jetbrains), [CleanCode](https://plugins.jetbrains.com/plugin/11677-cleancode), or [Unity Support](https://plugins.jetbrains.com/plugin/11629-unity-support)) downloaded from JetBrains Marketplace or installed from a local storage.

<img src="dk-inspections-plugins.png" width="706" alt="ReSharper plugins list"/>

Learn more: [](inspections-resharper.md#JetBrains+ReSharper+Command+Line+Tools+Settings).



## REST API
{instance="tcc"}

Previously, you could send the DELETE request to a running cloud agent to terminate it.

```Shell
/app/rest/cloud/instances/<cloudInstanceLocator>
```
{prompt="DELETE"}

Starting with this version, you can stop cloud instances via POST requests to the following endpoints:

```Shell
/app/rest/cloud/instances/<cloudInstanceLocator>/actions/stop
/app/rest/cloud/instances/<cloudInstanceLocator>/actions/forceStop
```
{prompt="POST"}

Use the `...actions/stop` endpoint to issue a "soft" stop request: if the target agent is currently busy, it will stop after the build finishes.

The `...actions/forceStop` endpoint allows you to stop a cloud instance even if it is busy.

Learn more: [Start and Stop Cloud Instances](https://www.jetbrains.com/help/teamcity/rest/manage-cloud-profiles.html#Start+and+Stop+Cloud+Instances).

## Sakura UI and UX Enhancements
{instance="tcc"}

* You can now bookmark required agent pools to easily access them from the top of the agents and pools list. Learn more: [](configuring-agent-pools.md#Favorite+Pools).

* The [Interactive Agent Terminal](install-and-start-teamcity-agents.md#Debug+Agents+Remotely) introduced in version 2023.05 now opens in a panel docked to the bottom of the agent details page. You can move it to a separate browser tab by clicking **Open in a separate tab**.

  <img src="dk-agentTerminal-2023-11-new.png" width="706" alt="Agent Terminal Window"/>

* You can now switch [](build-log.md) timestamps from absolute values to relative to quickly analyze how long it took the build to reach a specific stage.

  <img src="dk-relativeBuildLogTime.png" width="706" alt="Relative timestamps"/>

* You can now view and copy connection IDs from the **Connection** pages in TeamCity UI. This minor enhancement facilitates writing [](kotlin-dsl.md) code for objects that utilize connections (for example, [AWS Credentials features](aws-credentials.md)).

  <img src="dk-copy-connection-id.png" alt="Copy connection ID" width="706"/>

* Cloud images' **Build History** tab now displays a search box that allows you to find all builds of a specific cloud agent, even if this instance is no longer available.

  <img src="dk-deletedAgentHistory.png" width="706" alt="History for deleted agent"/>

* The navigation bar now paints the changes indicator bold for favorite projects with changes from the current TeamCity user.

  <img src="dk_boldChangesIndicator.png" width="706" alt="Highlighted projects with current user changes"/>

## Miscellaneous
{instance="tcc"}


* [EC2 Cloud Images](setting-up-teamcity-for-amazon-ec2.md) now feature the **Image priority** setting that allows you to specify which images should spin up new cloud agents first. Images with higher priority values are prioritized over images with lower priorities.
* If your build runs a large amount of [parallel tests](parallel-tests.md) in each batch, TeamCity can automatically switch to an alternative test filtering mode that reduces potential performance issues. See this article for more information: [](parallel-tests.md#Alternative+Test+Filtering+for+.NET).
* The [](build-cache.md) feature is no longer in experimental stage. The **Administration | Experimental features** page is hidden.



<!-- #ENDREGION -->




## Upgrade Notes
{instance="tc"}

Before upgrading, we highly recommend reading about important changes in version [2023.11 compared to 2023.05.4](upgrade-notes.md#Changes+from+2023.05+to+2023.11).

## Fixed Issues
{instance="tc"}

See the [TeamCity 2023.11 release notes](teamcity-2023-11-release-notes.md) article for the summary of implemented features and fixed issues.



## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.

## Your Feedback Matters


We place a high value on your feedback and encourage you to share your thoughts and suggestions. See this link for more information: [Feedback](troubleshooting.md).


