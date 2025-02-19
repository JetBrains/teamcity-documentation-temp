[//]: # (title: What's New in TeamCity 2023.05)
[//]: # (auxiliary-id: What's New in TeamCity 2023.05)

## Dark Theme

In this release cycle we implemented one of the most upvoted requests: TeamCity Dark Theme. In addition, you can select the "System theme" option that automatically applies either Light or Dark theme depending on your current OS settings.

<img src="dk-TCDark.png" width="706" alt="TeamCity Dark Theme"/>


## Interactive Agent Terminals

With this update you can open remote terminals to agent machines directly from the TeamCity UI. These terminals allow project administrators to maintain local and cloud agents and troubleshoot issues.

<img src="dk-openInteractiveTerminal.gif" width="706" alt="Agent Terminal Window"/>

With this new terminal in place, we removed the older **Open SSM Terminal** action link from agent pages.

Learn more: [Install and Start TeamCity Agents](install-and-start-teamcity-agents.md#Debug+Agents+Remotely).


## Multinode Setup Enhancements
{instance="tc"}

### Round-Robin
{instance="tc"}

Version 2023.05 introduces a new requests distribution logic that spreads the load more effectively between TeamCity nodes and minimizes the number of negatively affected users when a node is down due to a planned maintenance or an unexpected failover.

This new logic is based on sending new requests to a random node that has the [Handling UI actions and load balancing user requests](multinode-setup.md#Handling+UI+Actions+and+Load+Balancing+User+Requests) responsibility. After this initial draw, TeamCity memorizes which node was selected and relegates subsequent requests to the same node.

Learn more: [](multinode-setup.md#Round-Robin).

### Assign the VCS Polling Responsibility to Multiple Nodes
{instance="tc"}

Prior to version 2023.05, the "VCS repositories polling" [responsibility](multinode-setup.md) (allows nodes to poll repositories for new commits and detect changes) was available for a single node in the entire cluster. Starting with this version, you can assign this responsibility to multiple nodes. This enhancement allows you to evenly distribute the load across nodes and reduce the delay before triggering new builds.

<img src="dk-multinode-overview.png" alt="Multinode Setup" width="706"/>

To specify which node should handle your current requests (for instance, adding a new build to the build queue), use the node selector in the bottom right corner of the TeamCity UI. The currently selected node is marked with the star icon on the **Nodes Configuration** page.



Learn more: [VCS Repositories Polling](multinode-setup.md#VCS+Repositories+Polling+on+Secondary+Node).

### Disable Main Node Responsibilities
{instance="tc"}

Previously, the [main TeamCity node](multinode-setup.md) automatically re-gained the *"Processing data produced by running builds"*, *"VCS repositories polling"*, and *"Processing build triggers"* responsibilities when a TeamCity cluster had no nodes with such responsibilities. In addition, when you switched the *"Main TeamCity node"* responsibility to another node, this new node automatically inherited all other responsibilities.

Starting with version 2023.05, main nodes do not automatically accept "missing" responsibilities. This change grants you more control over main nodes and allows you to reduce their load and CPU/memory consumption. The only main node responsibilities you cannot disable in TeamCity UI are "Main TeamCity node" (you can clear this checkbox only by enabling this responsibility on another node) and "Handling UI actions and load balancing user requests".


### Launch TeamCity Backup and Clean-Up from Any Node
{instance="tc"}

You can now [create backups](creating-backup-from-teamcity-web-ui.md) on any node with the [Handling UI actions and load balancing user requests](multinode-setup.md#Handling+UI+Actions+and+Load+Balancing+User+Requests) responsibility.

Clean-ups can now also be scheduled from any node with this responsibility. However, clean-ups are always executed on main nodes. This task can be performed while secondary nodes perform their regular tasks.


## VCS Integrations Enhancements

### Connect to GitHub via GitHub Apps

Starting with this release, TeamCity can work with GitHub and GitHub Enterprise instances via connections that utilize [GitHub Apps](https://docs.github.com/en/apps/creating-github-apps/setting-up-a-github-app/about-creating-github-apps). GitHub Apps is the superior way to provide access to your personal and organization repositories. It boasts fine-grained permissions, grants you more control over which repositories the app can access, and does not require keeping a dedicated "service" user to produce OAuth access tokens.

<img src="dk-add-GHApp-connection.png" alt="Create a GitHub App Connection" width="706"/>

GitHub App connections allow you to check out GitHub.com and GitHub Enterprise repositories,<!--share access tokens with [Commit Status Publisher](commit-status-publisher.md#GitHub) and [](pull-requests.md) build features,--> set up webhooks that GitHub uses to notify TeamCity about a repository change, and enable the related [authentication module](configuring-authentication-settings.md#GitHub).

Learn more: [Configuring Connections](configuring-connections.md#GitHub).

### Ignore GitHub Draft Pull Requests
{instance="tc"}

Starting from this version, you can configure the [Pull Requests build feature](pull-requests.md)
to ignore [GitHub draft pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests#draft-pull-requests) by checking the **Ignore Drafts** box in the build feature settings.
TeamCity will ignore draft pull requests until their status changes.

By default, the Pull Requests build feature loads the GitHub draft pull request information and runs builds on draft pull requests.
The build page displays the "Draft" status and grayed-out icon next to the pull request number:

<img src="dk-pullrequests-Draft1.png" alt="Draft PR" width="706"/>

When the status of the draft pull request changes to "Ready for review" in GitHub, the build page reflects the change:

<img src="dk-pullrequests-Draft2.png" alt="Ready for review PR" width="706"/>

### Reissue Refreshable Tokens for VCS Roots

If a VCS root is configured via TeamCity [connections](configuring-connections.md) to access Git repositories hosted in Bitbucket Server, Bitbucket Cloud or GitLab, the "Authentication Settings" section of this root's settings now displays the **Acquire new** button. This button allows you to instantly replace the refreshable token used by the VCS Root with a new token issued for the current user.

<img src="dk-refreshableGitToken.png" width="706" alt="Reissue Token" />

Short-lived refreshable tokens provide more security compared to passwords or personal access tokens since the TeamCity server refreshes them automatically without sharing any related data with agents.

Learn more: [Refreshable tokens](git.md#refresh-token).

### Integration with Bitbucket Server and Data Center
{instance="tc"}

In addition to Bitbucket Cloud, TeamCity now supports Bitbucket Server and Data Center. The corresponding option is available in the connection types list, and on the **Create Project** page.

<img src="dk-whatsnew202303-bbserver.png" width="708" alt="TeamCity integration with Bitbucket Server and Data Center"/>

Learn more: [Configuring Connections](configuring-connections.md#Bitbucket+Server+and+Data+Center) | [Creating and Editing Projects](creating-and-editing-projects.md#Creating+project+pointing+to+Bitbucket)


## Podman Support

Starting with version 2023.05, you can connect to image registries, run build steps inside containers, and push/pull images (via the [](command-line.md) runner) using [Podman](https://podman.io) instead of [Docker](https://www.docker.com).

<img src="dk-podman-wn.png" width="706" alt="Podman support in TeamCity 2023.05"/>

* The [](container-wrapper.md) extension (previously known as "Docker Wrapper") now pulls images via either `docker pull` or `podman pull`, depending on which Container Manager is installed on the build agent.
* The [](docker-support.md) build feature can now use Podman to log in to container registries.
* If you use the [](command-line.md) runner to execute `podman ...` commands, utilize new `container.engine`, `podman.version`, and `podman.osType` [parameters](configuring-build-parameters.md) to specify [agent requirements](configuring-agent-requirements.md) that ensure your builds run only on build agents with installed Podman.

Learn more: [](integrating-teamcity-with-container-managers.md).


## HTTPS Access Enhancements
{instance="tc"}

### Fetch HTTPS Certificates via Let's Encrypt
{id="fetch-certificates-via-lets-encrypt"}

[Let's Encrypt](https://letsencrypt.org) is a non-profit Certificate Authority (CA) that provides TLS certificates trusted by all modern browsers. Starting with version 2020.05, TeamCity can contact this CA to automatically issue and set up a valid certificate.

<img src="dk-https-letsencrypt.png" width="706" alt="Obtain certificate via Lets Encrypt"/>

Certificates issued by Let's Encrypt are automatically renewed 30 days before expiration, and cover both your TeamCity server domain and if configured, the [artifacts isolation domain](teamcity-configuration-and-maintenance.md#artifacts-domain-isolation).

Learn more: [HTTPS Server Settings](https-server-settings.md).

### Specify the Required Encryption Protocol for HTTPS Connection

If your TeamCity server allows access via HTTPS, the server's default protocol for communicating with clients is currently TLS Version 1.2. Starting with version 2023.05, you can specify the list of available encryption protocols or force TeamCity to use one specific protocol.

<img src="dk-tls-protocols.png" width="706" alt="TeamCity using the TLS 1.3 Protocol"/>

Learn more: [](https-server-settings.md#Specify+Available+Encryption+Protocols).


## New Service Messages


### Send Slack Messages and Emails via Service Messages
{instance="tc"}

TeamCity [](service-messages.md) allow you to report various information about the build by adding special messages to your build scripts. The list of available service messages now includes the `##teamcity[notification ...]` message that sends emails, Slack direct messages, and posts updates to Slack channels.

<img src="dk-serviceMessages-whatsNew.png" width="706" alt="Custom TeamCity messages in Slack and Email boxes"/>

Built-in security features ensure messages cannot be sent to wrong recipients and cannot include links to external web resources that are not configured as trusted.

Learn more: [Slack Messages](service-messages.md#Sending+Custom+Slack+Messages) | [Emails](service-messages.md#Sending+Custom+Email+Messages).

### Send Slack Messages via Service Messages
{instance="tcc"}

TeamCity [](service-messages.md) allow you to report various information about the build by adding special messages to your build scripts. The list of available service messages now includes the `##teamcity[notification ...]` message that Slack direct messages and posts updates to Slack channels.

<img src="dk-slack-service-message.png" width="706" alt="Custom TeamCity messages in Slack"/>

Built-in security features ensure messages cannot be sent to wrong recipients and cannot include links to external web resources that are not configured as trusted.

Learn more: [Slack Messages](service-messages.md#Sending+Custom+Slack+Messages)

### Add and Remove Build Tags via Service Messages


You can now send TeamCity [service messages](service-messages.md) to add and remove [build tags](build-actions.md#Add+Tags+to+Build).

<img src="dk-servicemessage-tags.png" width="706" alt="Tagging builds with OS names"/>

To add and remove tags, send the following messages:

```
##teamcity[addBuildTag 'your-custom-tag']
##teamcity[removeBuildTag 'tag-to-remove']
```

Learn more: [Service Messages](service-messages.md#Adding+and+Removing+Build+Tags).


## AWS-Related Updates
{instance="tc"}

<!--### Share AWS Connections with Child Projects

The [](aws-credentials.md) build feature can now utilize AWS connections owned by a parent TeamCity project. Previously, you had to configure a connection and a build feature within the same project.-->

<!--### Utilize Amazon Spot Placement Scores

[Spot instances](setting-up-teamcity-for-amazon-ec2.md#Amazon+EC2+Spot+Instances+support) enable you to request unused AWS EC2 instances at steep discounts (compared to On-Demand prices). As a result, you can significantly lower costs for Amazon-hosted TeamCity agents.

Starting with version 2023.05, you can allow TeamCity to request [spot placement scores](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-placement-score.html) and automatically choose AWS Regions or Availability zones in which (according to these scores) your spot requests are most likely to succeed. As a result, you can expect more stable and frequently available spot instance agents.

Learn more: [](setting-up-teamcity-for-amazon-ec2.md#Required+IAM+permissions).-->

### EC2 Instance Metadata Service v2 Support

IMDSv2 is the new version of the Amazon EC2 Instance Metadata Service that [addresses a number of IMDSv1 vulnerabilities](https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/).

TeamCity 2023.05 supports EC2 instances and Amazon Machine Images (AMIs) with both "Optional" and "Required" IMDSv2 settings. TeamCity always attempts to use IMDSv2 first, but supports older AMIs as well.

### EC2 Launch Templates Customization

Starting from this version, TeamCity allows customizing [Amazon EC2 launch templates](setting-up-teamcity-for-amazon-ec2.md#Amazon+EC2+Launch+Templates+support). You can now use the same launch template to run various instances that differ in some parameters only.
Check the **Customize Launch Template** box and modify the launch template's values as required.

<img src="dk-whatsnew-CustomizeLaunchTemplates.png" width="706" alt="Customize Launch Templates for AWS EC2"/>

Learn more: [](setting-up-teamcity-for-amazon-ec2.md#Amazon+EC2+Launch+Templates+support).


## IMDSv2 Support for Amazon Machine Images
{instance="tcc"}

IMDSv2 is the new version of the Instance Metadata Service by Amazon that [addresses a number of IMDSv1 vulnerabilities](https://aws.amazon.com/blogs/security/defense-in-depth-open-firewalls-reverse-proxies-ssrf-vulnerabilities-ec2-instance-metadata-service/).

TeamCity 2023.05 supports EC2 instances and Amazon Machine Images (AMIs) with both "Optional" and "Required" IMDSv2 settings. TeamCity always attempts to use IMDSv2 first, but supports older AMIs as well.


## Two-Factor Authentication Enhancements
{instance="tc"}

### Additional Verification for Critical Settings
{instance="tc"}

Starting with version 2023.05, users who pass the two-factor authentication have one hour to perform security-related actions: disable 2FA, change user password and/or email, and generate access tokens. Once this period expires, users must re-confirm their identities and pass a new 2FA verification before proceeding with these actions.

This new behavior adds an extra layer of protection for your TeamCity server.

Learn more: [](managing-two-factor-authentication.md#Critical+Settings+Protection).

### Force 2FA for Specific User Groups
{instance="tc"}

If the global two-factor authentication mode is "Optional", you can now force individual [user groups](creating-and-managing-user-groups.md) to use 2FA. To do so, add the `teamcity.2fa.mandatoryUserGroupKey` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) and set its value to the required group key.

```
teamcity.2fa.mandatoryUserGroupKey=SYSTEM_ADMINISTRATORS_GROUP
```

Learn more: [](managing-two-factor-authentication.md#Force+2FA+for+Individual+User+Groups).


## REST API Updates


### Manage SSH Keys

Starting with version 2023.05, you can perform the full range of operations on projects' SSH keys via [REST API](teamcity-rest-api.md): upload and generate new keys, modify VCS authentication settings, set passphrases for encrypted keys, and browse and remove uploaded keys. To do this, send required requests to the `/app/rest/projects/<project_locator>/sshKeys` endpoint.

Learn more: [SSH Keys Management](ssh-keys-management.md#REST+API).

### Manage Versioned Settings

Our REST API now allows you to manage settings related to [storing project settings in VCS](storing-project-settings-in-version-control.md). You can use this new API to modify these settings, check for changes, and load/commit changes from/to the related VCS. Explore the `/app/rest/projects/{locator}/versionedSettings/` endpoint to view available requests.

Learn more: [Manage VCS Settings](https://www.jetbrains.com/help/teamcity/rest/2023.05/manage-vcs-settings.html).

### Manage User Roles

The new `/app/rest/roles` endpoint allows you to obtain, modify, and remove existing [roles](managing-roles-and-permissions.md#Managing+Roles), as well as create new ones.

Learn more: [Manage Roles and Permissions](https://www.jetbrains.com/help/teamcity/rest/2023.05/manage-roles-and-permissions.html).

### Manage Server Authentication Settings

You can now send `GET` and `PUT` requests to the `/app/rest/server/authSettings` endpoint to manage [server authentication settings](configuring-authentication-settings.md).

Learn more: [Manage Server Authentication Settings](https://www.jetbrains.com/help/teamcity/rest/2023.05/manage-auth-settings.html#Example+1%3A+Enable+Guest+User+Access).


## .NET 8 Support
{instance="tc"}

TeamCity 2023.05 now supports [.NET 8.0](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) framework by Microsoft. This means TeamCity agents correctly recognize the corresponding SDK installed on agent machines and the [.NET build runner](net.md) successfully builds projects that target .NET 8.0.


## Additional Verification for Critical Settings
{instance="tcc"}

Starting with version 2023.05, users who pass the two-factor authentication have one hour to perform security-related actions: disable 2FA, change user password and/or email, and generate access tokens. Once this period expires, users must re-confirm their identities and pass a new 2FA verification before proceeding with these actions.

This new behavior adds an extra layer of protection for your TeamCity server.

Learn more: [](managing-two-factor-authentication.md#Critical+Settings+Protection).


## The Sakura UI Improvements


### The "Chains" Tab for Build Configurations

Build configuration pages now display the "Chains" tab. The page allows you to browse Sankey-like diagram of builds linked into a [](build-chain.md).

<img src="dk-sakura-chains.png" width="706" alt="Build Chains in Sakura"/>

Previously, this page was available only in Classic UI.

### Reorder Builds
{instance="tc"}

You can now manually reorder builds in the build queue by dragging them to the desired position in the Sakura UI.

### Improved Changes Visibility
{instance="tc"}

- The **Change Log** tab is now available for projects and build configurations.
- The **Show graph** option has been implemented on all pages and tabs related to changes. With this option enabled, the changes are displayed as a graph of commits to the related VCS roots.

<img src="dk-sakura-changelog.png" width="706" alt="Change Log tab with changes graph"/>


### Updated Parameters Tab on the Build Results Page

We have overhauled the **Parameters** tab on the [](build-results-page.md). You can now use a search box to find required parameters, view only those parameters that were added/modified during a [custom build](running-custom-build.md), and hide parameters reported by [dependent builds](build-chain.md).

Learn more: [](build-results-page.md#Parameters+Tab).


## Run Steps Only for Failed Builds
{instance="tc"}

You can now choose the "Only if build status is failed" [execution policy](configuring-build-steps.md#Execution+Policy) for individual steps. This policy allows you to create steps that will be ignored when your build finishes successfully and executed only when it fails.

<img src="dk-whatsnew2303-onlywhenfails.png" width="706" alt="Run the build step only when the build fails"/>


## Kotlin DSL: Build Failure Conditions on Custom Metrics
{instance="tc"}

You can now use Kotlin DSL to configure a build failure condition [on a custom statistic value](build-failure-conditions.md#Adding+Custom+Build+Metric)
reported by the build.
You do not need to edit the main-config.xml file for this,
which means that configuring a build failure condition on a custom metric no longer requires system administrator privileges.

The sample Kotlin DSL code is as follows:

```Kotlin
  failureConditions {
  failOnMetricChange {
    param("metricKey", "myReportedCustomStatisticValue")
    // ...
  }
}
```

We are working on improving the web UI representation of the build failure condition on custom metrics added via DSL.


## Server Health

### Health Reports for Archived Projects

You can now generate [Server Health reports](server-health.md) for archived projects. To do this, select the *&lt;Archived Projects&gt;* scope.

<img src="dk-serverHealthArhived.png" width="706" alt="Server Health Reports for Archived Projects"/>

### New Endpoints to Check the Server Status

Added two new endpoints that you can check by sending GET requests to obtain the current server status:

* the `<server_URL>/healthCheck/healthy` endpoint returns "200" if a server is running, even if it is still initializing or in [maintenance mode](teamcity-maintenance-mode.md).
  {instance="tc"}
* the `<server_URL>/healthCheck/healthy` endpoint returns "200" if a server is running, even if it is still initializing.
  {instance="tcc"}
* the `<server_URL>/healthCheck/ready` endpoint returns "200" if a server is fully initialized and ready to accept user requests. If the server is still initializing or awaits for a data upgrade, the endpoint returns "503".


## Miscellaneous

* The [Notifications build feature](notifications.md) now allows you to enter multiple recipient addresses.
* Added `env.BUILD_URL` to the list of [predefined environment variables](predefined-build-parameters.md#Predefined+Server+Build+Parameters). This variable returns a link to the current build.
* The [SSH Keys](ssh-keys-management.md) page now displays the button that allows you to generate a new key. Generating keys on TeamCity server is faster and more secure (compared to running `ssh-keygen` locally and manually uploading the keys).

* Added new `teamcity-commit-status.log` and `teamcity-pull-requests.log` [log files](teamcity-server-logs.md) that contain information related to the [](commit-status-publisher.md) and [](pull-requests.md) build features. Each log has a corresponding preset that allows TeamCity to write DEBUG-level events.
  {instance="tc"}
* [TeamCity Enterprise](https://www.jetbrains.com/teamcity/buy/#on-premises?licence=enterprise) users can now click **Help | Support** to quickly navigate to the new request form at [teamcity-support.jetbrains.com](https://teamcity-support.jetbrains.com).

  <img src="dk-helpbtn-support.png" width="460" alt="Support link"/>




## Upgrade Notes
{instance="tc"}

Before upgrading, we highly recommend reading about important changes in version [2023.05 compared to 2022.10.x](upgrade-notes.md#Changes+from+2022.10+to+2023.05).

## Fixed Issues
{instance="tc"}

See [TeamCity 2023.05 release notes](teamcity-2023-05-release-notes.md).



## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.

## We'd Love to Hear From You


We place a high value on your feedback and encourage you to share your thoughts and suggestions. See this link for more information: [Feedback](troubleshooting.md).
