[//]: # (title: What's New in TeamCity 2023.09)
[//]: # (auxiliary-id: What's New in TeamCity 2023.09;)

<!--
## Agents with Bundled JDKs

Starting with version 2023.09, you can build distributions of TeamCity agents bundled with custom JDKs. These distributions allow you to install both an agent and a JDK it requires to operate in one go.

<img src="dk-fullAgentDistributions.png" width="706" alt="Full agent distributions page"/>

To create a custom agent distribution, navigate to **Administration | Agent JDKs** and add a new JDK option (you will need to specify the platform, the architecture, and a link for TeamCity to download this specific JDK).

<img src="dk-addAgentJDK.png" alt="Add Agent JDK" width="706"/>

When a new option is added, TeamCity will start building your custom agent distribution. You can download custom agent+JDK bundles by clicking **Install agent | Full distributions** on the **Agents | Overview** page.

Learn more: [](install-teamcity-agent.md).
-->

<!--
## Seamless GitHub App Registration

In version 2023.05, we introduced the [new type](configuring-connections.md#GitHub) of connections to GitHub and GitHub Enterprise. These connections utilize [GitHub Apps](https://docs.github.com/en/apps/using-github-apps/about-using-github-apps), instead of the traditional OAuth-based access to repositories.

Starting with version 2023.09, you will be able to create these connections much faster, without manually setting up and registering new apps in GitHub. Click the corresponding button in the connection description, and TeamCity will set everything up for you.

<img src="dk-GhAppManifestButton.png" width="706" alt="GitHub Manifest App Button"/>

Learn more: [Configuring Connections](configuring-connections.md#GitHub).

-->

<!--
## AWS Connection Improvements

Starting with this version, you can enable or disable the **Available for sub-projects** and **Available for build steps** settings in AWS connection settings. These options allow you to ensure the configured connections are not used by unauthorized TeamCity projects and their [](aws-credentials.md) build features.

<img src="dk-shareAwsConnections.png" width="706" alt="Share AWS connections"/>

In addition, you can now refer to the new section of our "Configuring Connections" documentation article to learn how to configure secure AWS connections that follow Amazon guidelines and do not require permanent user credentials: [](configuring-connections.md#Recommended+Setup).
{instance="tc"}

Learn more: [](configuring-connections.md#AmazonWebServices).

-->

## Enhanced Integration with Perforce Helix Swarm

In version 2023.09, we have overhauled the "Perforce Helix Swarm" publisher of the [](commit-status-publisher.md) build feature. TeamCity can now utilize workflows and tests that already exist in your Swarm setup (instead of creating its own tests). In addition, the Publisher no longer requires credentials of a user with administrator access.

<img src="dk-swarm-personalbuild.png" width="706" alt="Personal build in TeamCity"/>

Learn more: [](integrating-with-helix-swarm.md).


## Step Statuses and IDs

Starting with version 2023.09, you can specify IDs for your steps (similarly to project and configuration IDs).

<img src="dk-stepID.png" width="706" alt="Step ID"/>

TeamCity uses these IDs to generate new `teamcity.build.step.status.<step_ID>` parameters that report the step exit statuses ("success", "failure", or "cancelled").

<img src="dk-parametersTab-statuses.png" width="706" alt="Step statuses"/>

You can use these values to perform custom actions depending on the statuses of previous steps. For example, you can craft custom step execution conditions.

Learn more: [](configuring-build-steps.md#Step+Status+Parameters).


## Additional ReSharper Plugins for the Inspections Runner

The [](inspections-resharper.md) runner now features the **R# CLT Plugins** field that allows you to enter a list of required [ReSharper plugins](https://plugins.jetbrains.com/resharper).

<img src="dk-inspections-plugins.png" width="706" alt="ReSharper plugins list"/>

Learn more: [](inspections-resharper.md#JetBrains+ReSharper+Command+Line+Tools+Settings).

## Service Messages

Added the [new service message](service-messages.md#Writing+the+File+into+the+Build+Log) that allows you to track the contents of the given file and echo new lines to the build log.

<img src="dk-streamFiletoLog.png" width="706" alt="Stream file to log"/>

Learn more: [](service-messages.md#Writing+the+File+into+the+Build+Log).

## Token-Based Authentication

We continue updating TeamCity connections, build features, and VCS Root objects to support authentication via short-lived OAuth tokens, which are more secure than traditional static PATs (personal access tokens).

In version 2023.09, authorization via OAuth tokens was added for:

* [](commit-status-publisher.md) that post statuses to GitLab.
* [](pull-requests.md) features that target GitLab and Bitbucket Cloud repositories.

<img src="dk-csp-GitLabToken.png" width="708" alt="Acquire access token for GitLab"/>



Learn more: [Commit Status Publisher](commit-status-publisher.md#GitLab) | [Pull Requests](pull-requests.md).

## JetBrains Space Integration

Starting with this version, TeamCity build configurations that target [JetBrains Space](https://www.jetbrains.com/space/) repositories do not require a configured **Commit Status Publisher** build feature to post build-related updates. Set up a TeamCity project via a predefined [Space connection](configuring-connections.md#jetbrains-space-connection) and build statuses will be posted automatically.

<img src="dk-csp-space.png" width="706" alt="Publish Space build statuses"/>

The option to manually set up the Commit Status Publisher feature is still available for those who wish full control over the process, as well as for custom setups where TeamCity is unable to publish build statuses automatically.

Learn more: [](commit-status-publisher.md#JetBrains+Space).

## Miscellaneous

* TeamCity distributes agents more effectively and processes large build chains with failing builds faster. Starting with version 2023.09, dependent builds whose ["On failed dependency" condition](snapshot-dependencies.md) is "Make build failed to start" no longer wait for an available agent when their dependencies fail or are canceled. Instead, the dependent build's status changes to "Failed to start" as soon as possible, and TeamCity proceeds to the next build in the chain.
* The [](commit-status-publisher.md) build feature now correctly publishes build statuses for configurations that target `refs/(merge-requests/*)/head` branches of GitLab repositories (the "merge result" branches). Previously, running TeamCity builds for merge result revisions caused the Publisher to encounter HTTP 404 errors.
* If users log into TeamCity using credentials of an external 2FA-protected service, TeamCity does not send additional 2FA requests. Learn more: [](managing-two-factor-authentication.md#Reduce+Excessive+Authorization+Requests).
<!--* You can now bookmark required agent pools to easily access them from the top of the agents and pools list. Learn more: [](configuring-agent-pools.md#Favorite+Pools).-->
<!--* <include src="parallel-tests.md" include-id="alternative-dotnet-parallel-filtering-tcc"/>-->
* [](performance-monitor.md) now shows absolute values of the consumed/total agent memory.
* The [](build-results-page.md#Dependencies+Tab) now displays a find panel that allows you to search for specific dependent builds by configuration names.
* You can now add the `dateFormat=<value>` parameter to URLs used by your log analysis tools to retrieve build logs. Learn more: [](build-log.md#Modify+the+DateTime+Pattern).


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
