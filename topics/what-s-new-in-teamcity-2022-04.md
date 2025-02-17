[//]: # (title: What's New in TeamCity 2022.04)
[//]: # (auxiliary-id: What's New in TeamCity 2022.04)

## Parallel tests on multiple agents

TeamCity can now split tests of a build in batches and run each batch on a separate build agent.
This way, tests will run in parallel and the build will finish faster.
The speed boost ratio depends on the number of test classes in the build and the number of agents used.

Refer to [this article](parallel-tests.md) for details.

## Advanced code quality inspections with Qodana

The [Qodana plugin](https://www.jetbrains.com/help/qodana/teamcity.html) has been bundled with TeamCity.
Now you can enable the _Qodana_ build runner and add static analysis to your build chain,
run advanced code inspections, find code duplicates, track code quality progress of your code. For details about the build runner, refer to [Qodana](qodana.md).

If you previously installed the non-bundled Qodana plugin and used DSL, please check our [upgrade notes](upgrade-notes.md#bundled-tools-updates-2022-04)
{instance="tc"}

## Enhanced integration with Amazon Web Services
{instance="tc"}

TeamCity 2022.04 adds new features to its cloud integrations.

### Migration of build artifacts from the local storage to Amazon S3
{instance="tc"}

Version 2022.04 allows you to not only store new build artifacts in Amazon S3,
but also [move existing artifacts from TeamCity’s local storage to Amazon S3](artifacts-migration-tool.md).
This is particularly useful for teams who are just starting their migration from a self-hosted setup to a cloud platform.

### Transferring build artifacts via Amazon CloudFront
{instance="tc"}

The speed of transferring build artifacts stored in Amazon S3 depends on the geographical distance between you and the region where the S3 bucket is located.
To help you increase the artifacts' upload/download speed and reduce costs, TeamCity 2022.04 adds native support for [Amazon CloudFront](https://aws.amazon.com/cloudfront/), a content delivery network that offers low latency and high transfer speeds.

[Enabling its support for an S3 storage](storing-build-artifacts-in-amazon-s3.md#CloudFrontSettings) will allow TeamCity to transfer build artifacts through the nearest CloudFront server.


## Requiring Build Approvals

Some procedures in the production environment may require approval of more than one person.
TeamCity now allows requiring manual approval from a specified person or a group for a build to run.
To prevent users from triggering a build accidentally and to have more control over deployments,
resource consuming builds, or resource removing operations, configure [the Build Approval](build-approval.md) feature for your build.

## Limiting number of running builds per branch

TeamCity 2022.04 helps you improve the allocation of your build agents by [limiting the number of simultaneously running builds per branch](configuring-general-settings.md#Limit+Number+of+Simultaneously+Running+Builds).
For example, your main branch may have an unlimited number of builds that will occupy as many build agents as they need, while you limit your feature branches to running just one build at a time.

## Smarter VCS Integrations
TeamCity improves VCS integrations and provides the following new features.

### Space merge requests
{instance="tc"}

TeamCity integration with JetBrains Space now includes the [Pull Requests](pull-requests.md) build feature. If you enable this feature in a build configuration, TeamCity will automatically detect changes in merge requests submitted to your Space repository. For a build run on these changes, it will display the merge request details in the build's overview and send the build statuses back to JetBrains Space.

To enable this functionality in TeamCity:
1. In the [project settings](project-administrator-guide.md#Edit+and+View+Modes), configure a [connection to JetBrains Space](configuring-connections.md#connect-to-jetbrains-space) and create a Git [VCS root](configuring-vcs-roots.md) with your Space repository URL and an empty branch specification.
2. In the [build configuration settings](project-administrator-guide.md#Edit+and+View+Modes), add the [Pull Requests](pull-requests.md) build feature with the _JetBrains Space_ VCS hosting type:  
   <img src="pull-requests-feature.png" alt="Pull Requests build feature" width="706"/>  
   You can specify the [branch filter](branch-filter.md) to monitor merge requests only on target branches that match the specified criteria. For example, if you set the `+:refs/heads/feature-*` filter, TeamCity will monitor merge requests sent only to branches whose names start with `feature-`.
3. To allow publishing build statuses to the commit details in JetBrains Space, you also need to add the [Commit Status Publisher](commit-status-publisher.md) feature to this build configuration.

After the integration is configured, TeamCity will detect merge requests submitted to the JetBrains Space target branches. For builds run on the requested changes, it will display the merge request details in the **Overview** tab:

<img src="pull-request-details.png" alt="Build Overview - Pull Request Details" width="706"/>

TeamCity will also send statuses per build change to timelines of merge requests in JetBrains Space (and to details of commits, if the [Commit Status Publisher](commit-status-publisher.md) feature is enabled in the build configuration):

<img src="space-timeline.png" alt="JetBrains Space - Merge Request Timeline" width="706"/>

To protect a target branch and ensure that only verified requests are merged into it, you can set up [Quality Gates](https://www.jetbrains.com/help/space/branch-and-merge-restrictions.html#quality-gates-for-merge-requests) for your Space repository and use a TeamCity build as an external check. If a build on a merge request fails, Space will notify you that the changes have not passed the required quality check.

<img src="space-quality-gates.png" alt="JetBrains Space - Quality Gates" width="706"/>

### Integration with GitLab issues

Starting from this version, TeamCity supports [GitLab issues](gitlab.md) out of the box.

### Queued builds reporting

Now the Commit Status Publisher updates the commit status in the version control system immediately
after adding the corresponding build to the queue, providing you with the most up-to-date information.
GitHub, GitLab, Space, Bitbucket, and Azure DevOps are all supported.

### Running a custom build with a specific revision

When running a custom build, you can now specify an exact revision that may not necessarily belong to the list of changes known by the build configuration.
This gives you a lot more flexibility when you want to reproduce historical builds,
deploy older versions, debug new build configurations, and so on.


## Security

### Log4J and Log4Shell
Although TeamCity has not been affected by the Log4Shell vulnerability (CVE-2021-44228), some security scanners wrongly reported it as vulnerable. To avoid false-positive scanner reports, we have upgraded Log4J to the latest version.

### Spring and Spring4Shell
Similarly to Log4Shell, the Spring4Shell vulnerability (CVE-2022-22965) does not affect TeamCity. However, to avoid false-positive reports from security scanners, we have upgraded the Spring Framework used in TeamCity to the latest version.


## Native Git for VCS-related operations on the server
{id="Native+Git" instance="tc"}

TeamCity can now use native Git as the default option for Git operations on the server.
Switching to native Git improves the performance of the checking for changes operations on the server
in comparison with the previously used JGit implementation. It also fixes a number of issues related to large Git repositories.

Before switching, make sure a [native Git client](https://git-scm.com/downloads) version 2.29 or later is installed on your server machine,
and the path to its executable is specified in the `PATH` environment variable.
Alternatively, you can set the full path to the executable
via the `teamcity.server.git.executable.path` [internal property](server-startup-properties.md#TeamCity+Internal+Properties)
(no server restart is required). On Windows, remember to use double backslashes in the path.

To switch your TeamCity server to native Git, go to __Administration | Diagnostics__ and open the __Git__ tab.
Here you can test the connection via native Git in any VCS root on your server.
If you choose to test all VCS roots, TeamCity will check whether they successfully connect via JGit and then test their connection via native Git.
This measure helps ensure that none of your pipelines will break after switching to native Git.
If the connection test is successful, you can enable the native Git support on your server(s).

Our internal TeamCity server statistics show that using native git significantly improves the performance of the VCS-related operations on the server:
new changes and branches appear several times faster in comparison with JGit.


## Native Git for VCS-related operations on the server
{id="Native+Git" instance="tcc"}

TeamCity now uses native Git for VCS-related operations on the server.
Using native Git improves the performance of the checking for changes operations on the server
in comparison with the previously used JGit implementation. It also fixes a number of issues related to large Git repositories.


## New UI

### Editing agent pools
{instance="tc"}

>We keep reproducing the classic TeamCity features in its experimental UI, and a majority of them have already received a new implementation. If you haven't tried the new UI for a while or at all, we encourage you to give it a try this time. Our goal is to make the experimental UI not only as functional as the classic one but a lot more responsive and enhanced with easier navigation and widgets.

It is now possible to edit [agent pools](configuring-agent-pools.md) and quickly assign agents and projects to them right in the new Sakura UI — without switching to the classic UI mode.
To edit an agent pool, click **Assign agents** in its settings. In this dialog, you can choose what agents you want to assign to the pool:

<img src="edit-agent-pool.png" alt="Edit agent pool in new TeamCity UI" width="460"/>

### Changes page

The new [Changes page](viewing-user-changes-in-builds.md) comes with filters providing flexible search options allowing you to sort changes by comment (commit message), by path to the changed file, and by revision number.

<img src="new-changes-page.png" alt="Changes in new TeamCity UI" width="460" />

## Storing Docker images produced by build in a public ECR registry
{instance="tc"}

TeamCity can now store Docker images produced by a build to both private and — since this update — public ECR registries.

To be able to use this functionality, you need to add an [Amazon ECR connection](configuring-connections.md#Amazon+ECR) in [project settings](project-administrator-guide.md#Edit+and+View+Modes) and choose the _ECR Public_ registry type:

<img src="amazon-ecr-public.png" alt="Connecting to public ECR registry" width="750"/>

Remember to also enable [](docker-support.md) in your builds.

## Automatic import of user avatars from external systems
{instance="tc"}

When a user signs in to TeamCity [via a third-party account](configuring-authentication-settings.md), like GitHub or Bitbucket, for the first time, TeamCity will automatically fetch their avatar from the external system and attach it to their TeamCity user profile. Note that TeamCity will only be able to access avatars of users with verified emails (if you are using GitLab, check that a _public email_ is set in your account).

It is possible to upload a different avatar in the TeamCity user profile settings afterwards.


## Applying actions to multiple builds
{instance="tc"}

It is now possible to select multiple builds and apply actions to all of them at once:
* Pin/unpin
* Tag
* Compare with another build
* Add a comment
* Add to favorites
* Remove

On the **Overview** tab of **Build Configuration Home**, you can select the required builds with checkboxes that appear when hovering over builds. To apply an action to them, use the respective command of the pop-up context menu. If you need to select a range of builds, press **Shift** and click build checkboxes at the edges of the range to be selected.

<img src="select-multiple-builds.png" alt="Selecting multiple builds" width="750"/>

## Perforce integration: Automatic creation of Helix Swarm test runs and status synchronization with builds
{instance="tc"}

[Commit Status Publisher](commit-status-publisher.md) has a new option for sending build status reports to Perforce Helix Swarm — **Create Swarm Test**. If you enable it, TeamCity will [create a test run](https://www.perforce.com/manuals/swarm/Content/Swarm/swarm-apidoc_endpoint_integration_tests.html#Create_a__testrun_for_a_review_version) on the Helix Swarm server and update its status according to the build status in TeamCity.

## Kotlin DSL API without version in package names
{instance="tc"}

Kotlin DSL code generated by previous TeamCity versions had imports with `v2019_2` in the package names, for instance:
```Kotlin
 import jetbrains.buildServer.configs.kotlin.v2019_2.projectFeatures.*
 ```

Since TeamCity 2019.2, there were no new DSL API packages as there were no major incompatible changes in DSL API.
But the presence of several versions of packages causes constant troubles with code completion in IntelliJ IDEA,
when it suggests several similar classes from packages with different versions.

To fix the issue with code completion, a new version of DSL API Maven artifacts is introduced in TeamCity 2022.04.
These DSL API artifacts do not have a version in their package names. The newly generated Kotlin DSL code is switched to these _unversioned_ artifacts automatically. The existing Kotlin DSL projects can be switched [manually](upgrading-dsl.md#packages-without-dsl-api-version).

## Getting the public part of project's SSH keys via the UI
{instance="tc"}

You can now copy the public part of an uploaded non-encrypted SSH key from the project settings. To do this, go to **Project Settings | SSH keys** and click **Copy the public key** under the key name.

<img src="copy-public-ssh-keys.png" alt="Copying public SSH keys" width="750"/>

This way, project admins no longer need to ask the system administrator for a public SSH key whenever they need it (for example, to integrate their TeamCity projects with a VCS hosting service) — they can just get it via the TeamCity UI.

## Other updates
{instance="tc"}

* **Queued builds optimization improvements**
  Before TeamCity 2022.04 a build in the queue could be optimized to an already started or finished build only.
  For instance if there are two build chains in the queue: `A -> B` and `A -> C` then the queued build `A` in both build chains could be replaced by the build queue optimizer only if there was another running or finished build `A` which has the same settings and revisions.

  Starting with TeamCity 2022.04 this algorithm was improved to allow reusing builds which are still in the queue.
  So for the example above both build chains will be merged together while they are still in the queue, so both of the build chains will use the same queued build `A`.

* **Build failure conditions: Creating a build problem per each matching error**  
  When configuring a _Fail build on specific text in build log_ [failure condition](build-failure-conditions.md), you can now specify whether to create a build problem only for the first text occurrence found in a build log (default) or for each error that matches the specified pattern.

* **The Eclipse plugin** has been unbundled from TeamCity. [Contact our support](https://teamcity-support.jetbrains.com/hc/en-us) if you need the plugin.


## Fixed issues

See [TeamCity 2022.04 release notes](teamcity-2022-04-release-notes.md)

## Upgrade notes
{instance="tc"}

Before upgrading, we highly recommend reading about [important changes in version 2022.04 compared to 2021.2.x](upgrade-notes.md#Changes+from+2021.2+to+2022.04).


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