[//]: # (title: What's New in TeamCity 2021.12)
[//]: # (auxiliary-id: What's New in TeamCity 2021.12)

## Running builds in your own Cloud infrastructure
{instance="tcc"}

TeamCity Cloud can run builds _in a customer's local environment_ or in a _JetBrains-managed cloud_. Usually, the first approach works best for companies who want to have full control over their build processes, while the second one suits teams whose processes rely on IaaS. Since this update, we want to introduce the third approach that stays somewhere in between: **_running builds in a customer's cloud_**.  
If you have a cloud infrastructure in Amazon EC2, Kubernetes, or VMware vSphere/vCenter, you can now integrate your TeamCity Cloud instance with it. This opens all the capabilities of cloud solutions (like custom launch policies and spot fleets) and gives you more control over the build environment than using JetBrains-hosted agents. In terms of subscription, such agents are considered [self-hosted](teamcity-cloud-subscription-and-licensing.md#cloud-self-hosted-agents), which means you purchase monthly slots for concurrent builds — there is no limit to their duration.  
All three approaches can be combined for higher flexibility and fault-tolerance of your pipelines.

Users familiar with TeamCity On-Premises might already know that TeamCity integrates with Cloud providers with the help of _cloud profiles_. And now, their functionality becomes available in TeamCity Cloud.

To integrate your TeamCity Cloud project with your cloud infrastructure:
1. On the cloud side, configure a virtual machine image with an [installed TeamCity agent](install-and-start-teamcity-agents.md).
2. In the TeamCity project's settings, create a [cloud profile](teamcity-integration-with-cloud-solutions.md#Agent+Cloud+Profiles+and+Images). If you want it to be available around the whole server, create it in the Root project's settings.
  >You can add an indefinite number of cloud profiles with different settings.
3. Configure the profile's common settings as described [here](teamcity-integration-with-cloud-solutions.md) and then follow the guidelines specific to your provider:
   * [Amazon EC2](setting-up-teamcity-for-amazon-ec2.md)
   * [Kubernetes](setting-up-teamcity-for-kubernetes.md)
   * [VMware](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md)

After the cloud profile is configured, TeamCity will be able to launch agents in your cloud and assign builds to them. If given a variety of agent environments (local and cloud), it will always try to choose the least expensive option for each build.

## Prepaid JetBrains-hosted build agents
{instance="tcc"}

Another improvement related to build agents is an alternative pricing model: now, JetBrains-hosted agents can be prepaid on a monthly basis. This option is the most convenient if you run a lot of builds on agents of certain types (6 or more hours a workday). As JetBrains-hosted agents are charged per build time, some customers could greatly benefit from paying a fixed amount of credits monthly, without caring about the builds' duration.

Note that while the builds' duration is not limited, their concurrent number is. For example, if you prepay for 2 Linux Small cloud agents, only a maximum of 2 of those types of agents can be running concurrently with the prepaid rate. Any further concurrent builds on the same type will revert to the per-minute pricing.

To prepay JetBrains-hosted agents, go to **Administration | Subscription &amp; Resources** , click **Buy agents** under the _Per-Month Agents_ section, and choose an [instance type](supported-platforms-and-environments.md#JetBrains-Hosted+Agents) you want to acquire. You can also define how many instances of this type can be run in parallel.

<img src="prepaid-agents.png" alt="JetBrains-hosted agents prepaid monthly" width="460"/>

There is no need to wait until the end of the next month: whenever you prepay agents, the amount of charged build credits will depend on the number of days left till the end of the current month.

If you plan to reduce the number of prepaid agents in the next month, you can adjust this number straight away. The setting will only be applied since the next month, and you won't have to worry about forgetting to change it at the end of the current one.

## Single sign-on authentication via SAML 2.0
{instance="tcc"}

TeamCity Cloud now supports a new authentication method: Single Sign-On (SSO) via [SAML 2.0](https://en.wikipedia.org/wiki/SAML_2.0). Enabling it in your instance will allow users to authenticate in TeamCity with their account stored in one of the major identity providers: [Okta](https://www.okta.com/), [OneLogin](https://www.onelogin.com/), [AWS SSO](https://aws.amazon.com/single-sign-on/), [AD FS](https://docs.microsoft.com/en-us/windows-server/identity/active-directory-federation-services), and so on.

With a multitude of services an average software developer uses in their daily work, having one account for authentication in all of them can significantly offload the clutter. More importantly, using SSO identity managers is a good security practice, and TeamCity wants to encourage it by providing easy integration.

>The SAML 2.0 support is implemented as a plugin written by a third-party developer and verified by the TeamCity team as safe to use. It was previously available for download and successfully used by some of our customers, but now it's available out of the box.
> 
>See the [SAML 2.0 plugin's GitHub page](https://github.com/morincer/teamcity-plugin-saml) for more details about it.
>
>We are eager to see that such quality and high-demanded tools are created by our users. If you are curious to learn how to write custom plugins for TeamCity, visit our dedicated [Help for external plugin writers](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html)!

You can enable SSO in TeamCity as any other authentication module: in **Administration | Authentication**.

<img src="enable-saml.png" alt="Enabling SAML in TeamCity" width="460"/>

After that, the new **SAML Settings** page will appear in the **Administration** section. Use it to customize the module's behavior and to establish the integration on the TeamCity side. On the provider's side, you will need to create a dedicated app for a service connection to TeamCity. Its settings may vary depending on the provider, but the key steps are the same:
1. Enter the TeamCity single sign-on URL, copied from the **SAML Settings** page, in the app's settings.
2. Copy the values of the app's SSO URL, issuer URL, and certificate.
3. Enter them in the **SAML Settings** in TeamCity.

<img src="saml-settings.png" alt="SAML settings in TeamCity" width="706"/>

See more details about this authentication module in [this article](configuring-authentication-settings.md#HTTP+SAML+2.0).

As soon as the connection between TeamCity Cloud and your SSO provider is established, users will see a new authentication button on the TeamCity login form. To sign in, they will need to click it and confirm the authentication on the provider's side.

Depending on the module settings, TeamCity can prohibit authentication to accounts whose emails don't match emails of existing TeamCity users, or it can create a new user profile for such an account.

## Running builds on JetBrains Space merge requests

TeamCity integration with JetBrains Space now includes the [Pull Requests](pull-requests.md) build feature. If you enable this feature in a build configuration, TeamCity will automatically detect changes in merge requests submitted to your Space repository. For a build run on these changes, it will display the merge request details in the build's overview and send the build statuses back to JetBrains Space.

To enable this functionality in TeamCity:
1. In the [project settings](project-administrator-guide.md#Edit+and+View+Modes), configure a [connection to JetBrains Space](configuring-connections.md#connect-to-jetbrains-space) and create a Git [VCS root](configuring-vcs-roots.md) with your Space repository URL and an empty branch specification.
2. In the build configuration settings, add the [Pull Requests](pull-requests.md) build feature with the _JetBrains Space_ VCS hosting type:  
   <img src="pull-requests-feature.png" alt="Pull Requests build feature" width="706"/>  
  You can specify the [branch filter](branch-filter.md) to monitor merge requests only on target branches that match the specified criteria. For example, if you set the `+:refs/heads/feature-*` filter, TeamCity will monitor merge requests sent only to branches whose names start with `feature-`.
3. To allow publishing build statuses in JetBrains Space, you also need to add the [Commit Status Publisher](commit-status-publisher.md) feature to this build configuration.

After the integration is configured, TeamCity will detect merge requests submitted to the JetBrains Space target branches. For builds run on the requested changes, it will display the merge request details in the **Overview** tab:

<img src="pull-request-details.png" alt="Build Overview - Pull Request Details" width="706"/>

If the [Commit Status Publisher](commit-status-publisher.md) feature is enabled in the build configuration, TeamCity will also send statuses per build change to timelines of merge requests in JetBrains Space:

<img src="space-timeline.png" alt="JetBrains Space - Merge Request Timeline" width="706"/>

To protect a target branch and ensure that only verified requests are merged into it, you can set up [Quality Gates](https://www.jetbrains.com/help/space/branch-and-merge-restrictions.html#quality-gates-for-merge-requests) for your Space repository and use a TeamCity build as an external check. If a build on a merge request fails, Space will notify you that the changes have not passed the required quality check.

<img src="space-quality-gates.png" alt="JetBrains Space - Quality Gates" width="706"/>

## New UI: Editing scope of agent pools

It is now possible to edit the scope of [agent pools](configuring-agent-pools.md) and quickly assign agents and projects to them right in the new Sakura UI — without switching to the classic UI mode.

>We keep reproducing the classic TeamCity features in its experimental UI, and a majority of them have already received a new implementation. If you haven't tried the new UI for a while or at all, we encourage you to give it a try this time. Our goal is to make the experimental UI not only as functional as the classic one but a lot more responsive and enhanced with easier navigation and widgets.
> 
{instance="tc"}

To edit an agent pool, click **Assign agents** in its settings. In this dialog, you can choose what agents you want to assign to the pool:

<img src="edit-agent-pool.png" alt="Edit agent pool in new TeamCity UI" width="460"/>

Note that whenever you select a cloud image, you actually assign all its instances to the pool. If this pool has a limit of agent slots, each cloud instance will take a single slot, just like any regular agent.

Similarly, you can also associate projects with this pool: open the **Projects** tab of the pool's settings and click **Assign projects**. This way, agents assigned to this pool will be allowed to run builds only in the selected projects.

## Other updates

**Change the project scope of an investigation or mute**  
  If a build problem occurs in multiple subprojects of the same project, it is convenient to assign its investigation (or mute it) within the whole parent project. However, users often forget to change the project scope in the _Investigation/Mute_ dialog, and TeamCity applies the action only within the current subproject. Now, project administrators can set a parent project as a preferred scope for all its subprojects. [Read how](investigating-and-muting-build-failures.md#Changing+Project+Scope+of+Investigation+or+Mute).

**Build failure conditions: Create a build problem per each matching error**  
  When configuring a _Fail build on specific text in build log_ [failure condition](build-failure-conditions.md), you can now specify whether to create a build problem only for the first text occurrence found in a build log (default) or for each error that matches the specified pattern.

## Fixed issues
{instance="tcc"}

See [TeamCity Build 107109 release notes](teamcity-release-notes-build-107109.md).

## Upgrade notes

- To comply with the common identifier format of .NET tests, TeamCity now uses a different format of names for .NET assemblies (omitting a file extension). After updating to 2021.12, this format will be applied within all the tests launched via the `test` or `vstest` command of the [.NET](net.md) runner, but the investigations and history of these tests might be reset.
- TeamCity stops supporting the [Microsoft Edge Legacy](https://techcommunity.microsoft.com/t5/microsoft-365-blog/new-microsoft-edge-to-replace-microsoft-edge-legacy-with-april-s/ba-p/2114224) web browsers.
- It is now impossible to automatically [trigger builds via REST API](https://www.jetbrains.com/help/teamcity/rest/start-and-cancel-builds.html#Advanced+Build+Run) when the [queue limit](https://www.jetbrains.com/help/teamcity/2021.12/ordering-build-queue.html#Limiting+Maximum+Size+of+Build+Queue) is reached on the server.
- Updates in TeamCity Agent Docker images:
{instance="tc"}
    - Bundled .NET Core SDK has been updated to 6.0.100.
{product="tс"}
    - Bundled two versions of .NET Core Runtime: 3.1.21 and 5.0.12.
{product="tс"}
- Bundled IntelliJ IDEA has been updated to version 2021.2.3. Note that this version requires Java 11.
- The [SBT](https://www.scala-sbt.org/) launcher, used in the [Simple Build Tool (Scala)](https://www.jetbrains.com/help/teamcity/2021.12/simple-build-tool-scala.html) plugin, has been updated to version 1.5.5.
- The [Octopus Deploy integration plugin](https://plugins.jetbrains.com/plugin/9038-octopus-deploy-integration) bundled with TeamCity Cloud has been updated to version 6.1.8.
{instance="tcc"}
- The [Unity Support plugin](https://plugins.jetbrains.com/plugin/11453-unity-support) bundled with TeamCity Cloud has been updated to version SNAPSHOT-20211116104228.
{instance="tcc"}


## Previous releases
{instance="tc"}

* [What's New in TeamCity 2021.2](https://www.jetbrains.com/help/teamcity/2021.2/what-s-new-in-teamcity.html)
* [What's New in TeamCity 2021.1](https://www.jetbrains.com/help/teamcity/2021.1/what-s-new-in-teamcity.html)
* [What's New in TeamCity 2020.2](https://www.jetbrains.com/help/teamcity/2020.2/what-s-new-in-teamcity.html)
* [What's New in TeamCity 2020.1](https://www.jetbrains.com/help/teamcity/2020.1/what-s-new-in-teamcity.html)
* [What's New in TeamCity 2019.2](https://www.jetbrains.com/help/teamcity/2019.2/what-s-new-in-teamcity-2019-2.html)
* [What's New in TeamCity 2019.1](https://www.jetbrains.com/help/teamcity/2019.1/what-s-new-in-teamcity-2019-1.html)

## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.