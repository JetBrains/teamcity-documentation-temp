[//]: # (title: What's New in TeamCity 2020.1)
[//]: # (auxiliary-id: What's New in TeamCity 2020.1)

## Conditional build steps

Now, you get granular control over build steps with the new __[execution conditions](build-step-execution-conditions.md)__. When running a build, TeamCity will execute each step only if __all__ its preconditions, configured by you, are satisfied in the current run.

In the advanced build step settings, click __Add condition__ and specify a logical condition for any build parameter provided by the TeamCity server or agent. You can quickly select among the example conditions, such as:
* _Run the step only in the default branch_
* _Run the step only in the release branch_
* _Skip the step in personal builds_

Or, create a custom condition, and TeamCity will automatically suggest supported parameters and values. A wide set of logical conditions and parameters is available.

<img src="wn-exec-conditions.png" alt="Adding a parameter-based execution condition"/>

## Kubernetes support out of the box

TeamCity 2020.1 comes bundled with our widely used [Kubernetes Support](https://plugins.jetbrains.com/plugin/9818-kubernetes-support) plugin.

TeamCity integration with Kubernetes allows running cloud agents inside your K8S cluster. You can choose how you want to launch a build agent: from a specific image, from a Kubernetes deployment, or based on a custom pod template.

<img src="wn-kuber-image.png" alt="Adding a Kubernetes image"/>

Read how to integrate TeamCity with Kubernetes in [our documentation](setting-up-teamcity-for-kubernetes.md).

The bundled integration will replace the installed external Kubernetes Support plugin. If you were using the Helm runner, provided by this plugin in the previous versions of TeamCity, please refer to our [upgrade notes](upgrade-notes.md#Bundled+Kubernetes+Support+plugin+does+not+contain+Helm+runner) before upgrading to 2020.1.

## Investigation History

As practice shows, our [investigation tools](investigating-and-muting-build-failures.md) are widely accepted among the TeamCity users. They help to inspect build problems and failed tests and allow assigning their investigation to the responsible members of the team. To address your feedback, we've made the investigation history accessible in the web UI. This is most helpful for big teams and projects when it is not as easy to determine who and when changed or resolved the investigation. 
 
To see all the actions applied to an investigation, open its context menu and click __Investigation History__.

<img src="wn-investigation-history.png" alt="Investigation history"/>

## Pull requests support for Azure DevOps

Now, your TeamCity builds can detect pull requests in on-premises and cloud Azure DevOps (2018 or later).

To configure the [respective build feature](pull-requests.md):


1. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Build Features"/></include>
2. Click __Add build feature__ and choose _Pull Requests_.

Note that in case with Azure DevOps TeamCity detects requests on a merge branch — not on the pull request itself as with other VCSs. Each build will be launched on a virtual branch showing an actual result of the build after merging the PR. Thus, the build will contain both the commit with changes and the virtual merge commit.

## Displaying TeamCity build information in Jira Cloud

TeamCity integration with Jira Cloud has been extended. Now, TeamCity not only gives quick access to Jira tasks, mentioned in the builds' commits, but also reports build information to Jira Cloud in real time. This way, you can preview build statuses directly in Jira, with no need to check the TeamCity server itself.

<img src="jira-cloud-integration.png" alt="TeamCity build status in Jira Cloud" width="800"/>

Refer to [our documentation](jira-cloud-integration.md) for details on configuring the integration with Jira Cloud.

## Notifications on build configuration level

TeamCity can use external channels (such as email) to notify the registered users about various build events. Previously, notifications were configured only for each TeamCity user or user group. Now, we are introducing the [Notifications](notifications.md) build feature which allows setting up notifications per build configuration and provides the same set of rules as the user-specific notifications.

This approach does not require referencing a specific TeamCity user and works better for group notifications. Currently, the feature supports notifications via email and [Slack](#Built-in+integration+with+Slack).

With this feature, it is now possible to describe the notifications in DSL and set up similar notifications for many build configurations by creating a [build configuration template](build-configuration-template.md).

## Built-in integration with Slack

Since this version, Slack integration has been built into TeamCity. It allows receiving notifications with specific build events and details in private messages or via a Slack channel.

Two notifying options are available:
* _Individual notifications_ configured per project in a user profile, as any [usual notifications](adding-notification-rules.md) in TeamCity.
* _Notifications [on a build configuration level](#Notifications+on+build+configuration+level)_ that allow setting up notifications for a single build configuration (or an entire project in a template). [Read more](notifications.md) on how to configure the responsible build feature.

Example of a TeamCity notification in Slack:

<img src="wn-slack-notification.png" alt="TeamCity notification in Slack"/>

## New Browser Notifier

In the scope of our 2020.1 release, we have presented the new TeamCity Browser Notifier extension. It automatically detects the active TeamCity session in a browser and sends real-time notifications about build statuses and events, based on your custom rules. The extension is available for Mozilla Firefox, Opera, and Google Chrome (including all Chromium-based browsers such as Microsoft Edge).

The new Browser Notifier aims at replacing the deprecated Windows Tray Notifier and automatically uses all rules configured for it, if any.

Read more about the Notifier's handy features in [our blog](https://blog.jetbrains.com/teamcity/2020/05/new-teamcity-notifier-browser-extension/).

## Downloading full agent with plugins

You can now access a full TeamCity agent, packed with all plugins currently enabled on the server. This option is the most convenient if you use scripts for creating agent images (for example, in cloud).

A regular TeamCity agent distribution does not contain plugins: the agent downloads them on the first start. The full agent contains all enabled plugins and automatically stays relevant with the current TeamCity server state. This makes its distribution archive larger but significantly reduces the time spent on the first agent run. All its instances will be synchronized with the server from the start and can run a build straight away.

Note that after starting, the full agent behaves like a regular agent. If you modify the state of plugins on the TeamCity server, all active agents will need to restart to sync with the server.

To download the full agent, go to the __Agents__ page in TeamCity, open the _Install Build Agents_ menu, and select _Full ZIP file distribution_. You can copy the link to this archive and use it in automation scripts; no authorization is required.

<img src="wn-full-agent.png" alt="Download TeamCity full agent ZIP"/>

## Managing project secure values

This release makes working with project tokens easier.

If you generate tokens for a DSL-based project in TeamCity, these tokens are saved in the project's DSL in the VCS while their respective secure values are stored in the TeamCity system settings. When you create a new TeamCity project based on the same DSL, you have to make sure secure values, corresponding to these tokens, are specified in this new project as well.
 
To take care of this procedure, TeamCity now automatically looks for [missing secure values](storing-project-settings-in-version-control.md#Managing+Tokens) of the project's tokens in other projects.

On the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Versioned Settings | Tokens__ tab, you can enter the required secure values manually. If TeamCity finds the same tokens in other projects you are permitted to edit, you will have an option to copy the values used in these projects.

<img src="wn-tokens-tab.png" alt="TeamCity Tokens tab" width="1000"/>

You can also add a new token directly on this page by clicking __Scramble secure value__.

## Support for custom encryption keys

TeamCity stores all secure values, used in project configuration files, in a scrambled form. The initial values are stored in the TeamCity Data Directory, and their safety primarily depends on the security of your environment. As an extra security level, TeamCity now supports custom encryption keys for protecting secure values. By using the custom encryption instead of the default scrambling strategy, you can minimize the risk of potential malicious actions.

Read more about this new option in [our documentation](teamcity-configuration-and-maintenance.md#encryption-settings).

## Improvements for secondary nodes

With each release, we come closer to providing a full-value TeamCity server experience on secondary nodes. Since version 2020.1, secondary nodes support the following functionality:

* __New responsibility: processing build triggers__.   
In setups with many projects and build configurations, a significant amount of the main server’s CPU is allocated to constant processing of build triggers. By enabling this responsibility for one or more secondary nodes, you distribute the trigger processing tasks and necessary CPU load between the main node and the responsible secondary ones.
* __More user-level actions__.   
Secondary nodes now allow changing user settings, checking for pending changes, performing various agent-related actions, and more. 
* __Automatic update via web UI__.   
Use our [autoupdate](upgrading-teamcity-server-and-agents.md#Automatic+Update) on secondary nodes, similarly to the main server experience.
* __Agent | Cloud tab__.   
You can now monitor cloud agents right on a secondary node.

## New features of experimental UI

<note>

Our [experimental UI](teamcity-sakura-ui.md) is a work in progress: we introduce the changes in the early stages of development so you can already benefit from the new features. Based on your feedback, we will make sure to optimize the experimental pages and migrate all the important classic features in the future releases.   
For now, if any of the familiar features are missing, you can switch an experimental page to the classic UI with the ![mammoth.png](mammoth.png){style="block"} button, or turn off the experimental UI on the server completely in __My Settings & Tools__.

__Try out the TeamCity experimental UI and leave your feedback via our [UI survey](https://surveys.jetbrains.com/s3/feedback-form-for-teamcity?tcv=2020.1)!__

</note>

### Viewing all agents

The __All Agents__ view displays the most important information on agents. This view works faster for a large number of agents. It gives a quick preview of agents' statuses and allows managing them side by side, on a single dashboard.

<img src="wn-agents.png" alt="TeamCity experimental Agents tab"/>

### Reordering projects in sidebar

One of the most requested features, missing in the experimental UI, was the ability to customize the list of projects. With this release, we address your requests by adding this functionality to the experimental UI sidebar.

To start customizing the projects' sidebar, click ![wn-pencil.png](wn-pencil.png) in its upper-right corner. You can move a project or build configuration in the list via a keyboard or using the arrow UI button. You can also mark/clear projects as favorite directly in this configuration window.

<img src="wn-configure-sidebar.png" alt="Customizing experimental sidebar" style="block"/>

### Improvements for Project Home

The __Project Home__ page, introduced in the experimental UI in TeamCity 2019.1, receives a few handy features in 2020.1:

(1) When viewing the project details, you can see the statuses of build configurations of all the subprojects, even when their blocks are collapsed. Only builds in the default branches are considered.   
(2) You can instantly add a child subproject or build configuration.   
(3) The build configuration context menu now provides quick access to its settings' sections.

<img src="wn-exp-project-handy.png" alt="Customizing experimental sidebar" width="800"/>

## Other improvements

* Java 11 has been bundled with the TeamCity server Windows installer and server Docker images instead of Java 8.
* Automatic assignment of investigations on second failure has been optimized (read more in [our documentation](investigations-auto-assigner.md#delay-auto-assign)).
* TeamCity now uses the BCrypt algorithm to make user password storage safer.
* The __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Build Schedule__ tab now has an alternative filter option: hide triggers with the enabled "_Trigger only if there are pending changes_" option. This helps to quickly find all triggers where this option is disabled if you need to investigate the builds' behavior.
* To comply with recommended security and performance practices, the TeamCity agent Docker images now:
   * run under a non-root user;
   * use Docker volumes for directories that require writing large amounts of data.
* Optimized memory usage for servers with lots of test names in the `test_names` dictionary table.

## Fixed issues

[Full list of fixed issues](https://confluence.jetbrains.com/display/TW/TeamCity+2020.1+Release+Notes)

## Upgrade notes

[Changes from 2019.2.x to 2020.1](upgrade-notes.md#Changes+from+2019.2.x+to+2020.1)

## Previous releases

* [What's New in TeamCity 2019.2](https://www.jetbrains.com/help/teamcity/2019.2/what-s-new-in-teamcity-2019-2.html)
* [What's New in TeamCity 2019.1](https://www.jetbrains.com/help/teamcity/2019.1/what-s-new-in-teamcity-2019-1.html)
