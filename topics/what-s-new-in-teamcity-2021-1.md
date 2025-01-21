[//]: # (title: What's New in TeamCity 2021.1)
[//]: # (auxiliary-id: What's New in TeamCity 2021.1)

## Kotlin Script build runner

[Kotlin](https://kotlinlang.org/) by JetBrains is a widely adopted and concise programming language. It works on all platforms supported by TeamCity and is perfect for scripting build tasks. To meet the high demand for such a tool in TeamCity, we've designed a new [Kotlin Script](kotlin-script.md) build runner.

You can use it to automate recurring routines, such as custom checks, sending [service messages](service-messages.md), or downloading files via HTTP. If you use [Ant](ant.md) or [Command Line](command-line.md) build steps and are eager to try Kotlin instead, the new runner gives a great opportunity to do this. You can rewrite tasks in Kotlin and just switch the existing build steps to the new runner.

To configure a Kotlin Script build step, enter a script code or provide a path to it. By default, TeamCity will run it using Kotlin 1.5.0, but you can install any other compiler version in __Administration | Tools__.

<img src="kotlin-script-step.png" width="706" alt="Kotlin Script build step"/>

[Read this article](kotlin-script.md) for more details.

## Node.js build runner

To improve your experience with building JavaScript projects in TeamCity, we introduce a dedicated build runner for [Node.js](nodejs.md). It allows running [`npm`](https://www.npmjs.com/), [`yarn`](https://yarnpkg.com/), and [`node`](https://github.com/nodejs/node) commands inside your builds and reporting detailed test results.

You can add the Node.js steps manually or let TeamCity scan your project's repository. When scanning, TeamCity will parse the `package.json` file to see what frameworks are used in the project. Based on this information, it will propose adding the respective build steps: for example, to install necessary dependencies or to run tests. You can later adjust these steps as you like.

If there are [ESlint](https://eslint.org/), [Jest](https://jestjs.io/), or [Mocha](https://mochajs.org/) dependencies in your project, TeamCity will show the respective test results right in the build overview.

To configure a Node.js build step, just enter a script with necessary Shell commands:

<img src="nodejs-step.png" width="706" alt="Node.js build step"/>

Currently, all Node.js steps are run inside a Docker/LXC container, which means [Docker](https://www.docker.com/) or [Podman](https://podman.io) needs to be installed on build agents. TeamCity uses the `node:lts` version by default; but if there is an `.nvmrc` file inside your project, it will search for the image specification there.

[Read this article](nodejs.md) for more details.

## Customize automatically triggered builds

[Build triggers](configuring-build-triggers.md) now support custom parameters. This feature has been anticipated by our users, as it makes automatically triggered builds as versatile as builds started in the [custom run](running-custom-build.md) mode.

There are limitless combinations of how you can use this instrument to autorun different builds within the same configuration. Most importantly, the same script can now deploy triggered builds to different targets, depending on preconditions like a source branch or launch time.

In a build trigger's settings, you can find the new __Build Customization__ tab with the following options:
* _General Settings_: choose to delete all files in the [checkout directory](build-checkout-directory.md) if you need to start every triggered build with clean sources.
* _Build parameters_: customize the value of any [parameter](configuring-build-parameters.md) used in the current build configuration. Or, add a new parameter, and it will be available only in builds started by this trigger. You can even use it to override a parameter of a [dependency build configuration](snapshot-dependencies.md): use the [`reverse.dep.<dependencyBuildID>.<property>`](use-parameters-in-build-chains.md#Override+Parameters+of+Preceding+Configurations) syntax for this.

This feature gets even more effective if you combine it with our [build step execution conditions](build-step-execution-conditions.md). You just need to add a parameter-based condition to a step and then configure two triggers: one will run builds with this step (when the condition is satisfied) and one — without it. A popular use case is to run an extra clean-up step when a failed build is restarted with a [retry trigger](configuring-retry-build-trigger.md): this can help solve many build problems even without involving a user.

<img src="custom-trigger-params.png" width="460" alt="Customize triggered builds"/>

## Multinode setup improvements
{instance="tc"}

One of the top priorities of TeamCity is to provide a high-availability solution for your building processes. In this release, we bring multiple updates that make it easier to configure a performant and stable cluster setup.

### Switch node from secondary to main in runtime

Now, if the main node goes down for any reason, its main role and all the respective responsibilities can be promptly assigned to another server.

By default, the new "_Main TeamCity node_" responsibility belongs to the current main server, but it gets vacant if this server becomes unavailable. You can then assign it to any secondary server in __Administration | Nodes Configuration__.

<img src="main-node-role.png" width="460" alt="Main node responsibility"/>

The assigned server becomes the main node and automatically receives all its other responsibilities (processing builds, managing agents, and so on). This new main node keeps all its running builds, and the agents reconnect to it automatically if a [proxy is configured in your setup](multinode-setup.md#Proxy+Configuration).

When the previously main server starts again, it becomes a secondary node, as the "_Main TeamCity node_" responsibility is already occupied by another server. If necessary, you can repeat the procedure above to switch roles between these servers.

### External proxy as load balancer

In the previous versions of TeamCity, build agents had to send all the requests to the main node, and the main node was redirecting these requests to a suitable secondary node. If the main were to go down, the agent would not be able to communicate with its appointed secondary node. Starting with version 2021.1, TeamCity supports a new approach — using proxy as a balancer of requests between all clients (build agents and TeamCity users) and nodes.

This approach allows routing all agents to the proxy instead of the main server. This will make the server-agent communication less dependent on the main node and get your setup closer to 100% availability. Besides, since agents won't connect to the nodes directly, you can configure and maintain HTTPS settings in one place — on the proxy level.

See more details and the example proxy configuration in [this article](multinode-setup.md).

### Manage max number of builds on secondary node

If a secondary node is assigned to processing builds, it is now possible to limit the number of parallel builds it can run. Previously, if the "_[Processing data produced by running builds](multinode-setup.md#Processing+Data+Produced+by+Builds+on+Secondary+Node)_" responsibility was enabled for a node, it would process all builds. Now, you can distribute the load between multiple nodes or even keep some builds assigned to the main node.

To configure the limits, go to __Administration | Nodes Configuration__, find the required node in the list, and click __Edit__ next to its "_Processing data produced by running builds_" responsibility. In the _Limit builds_ dialog, enter a relative limit of builds allowed to run on this node. We suggest that you set the limits depending on the nodes' hardware capabilities.

<img src="node-build-limit.png" width="460" alt="Limit builds on node"/>

If the maximum limit of allowed builds is reached on all secondary nodes, TeamCity will be starting new builds on the main node — until some secondary node finishes its build.

## New search mode based on Elastic

Alternatively to the local Lucene-based search, TeamCity now provides a search mode based on [Elasticsearch](https://www.elastic.co/elasticsearch/). Both modes allow searching builds by number, tag, and many other parameters. The main difference is in the search index location: it can be stored next to the TeamCity server or on your Elastic host.

The new mode has two advantages: (1) it saves disk space on the TeamCity server machine and (2) it is better for the TeamCity performance. It is especially effective for multinode installations, as nodes spend fewer resources on maintaining a single remote index than on multiple local indexes.  
For relatively small installations, you may continue using Lucene search, with no need to reconfigure anything.

You can select the search mode on the Root project level in __Project Settings | Builds Search__. To connect to your Elastic host or cluster, enter its URL and credentials.

After you save the new settings, TeamCity will spend some time reindexing builds. The exact duration depends on the size of your server. You can track or control the progress in the _Diagnostics_ table.

<img src="elastic-search.PNG" width="706" alt="Elastic-based search"/>

## Read-only project settings

[Versioned project settings](storing-project-settings-in-version-control.md) is one of the most popular features of TeamCity. We keep adding more controls to this functionality based on your feedback: since this version, you can make project settings read-only in the TeamCity UI.

When the synchronization of a project's settings is enabled, all changes made in the UI are committed to the settings repository, and vice versa. But in some cases, it might be convenient to prohibit editing settings via the UI. For example, if you store a project settings in the same repository with its source code, you might want to track and approve all changes entirely by the means of VCS. Or, if you import settings from a read-only branch and then change them in the UI, TeamCity will fail to commit back to the repository and, therefore, apply new changes made in the VCS.

To address these and any similar cases, we've added a new option: _Allow editing project settings via UI_. If you disable it in a project, this will make this project's settings read-only in the UI and prevent TeamCity commits to the settings' repository.

To toggle the new option, go to __Project Settings | Versioned Settings | Configuration__:

<img src="versioned-settings-sync.png" width="460" alt="Read-only project settings"/>

[Read this article](storing-project-settings-in-version-control.md#SynchronizingSettingswithVCS) for more details.

## Limit permissions of access tokens

You can now create access tokens with limited permissions and use these tokens for REST API requests. This gives you more control over how your scripts integrate with TeamCity. For example, if used in combination with the timeout setting, this allows generating short-lived tokens for specific tasks.

By default, a token's _Permissions scope_ is set to "_Same as current user_". It means that the created token will grant the same [permissions](managing-roles-and-permissions.md) as those of the current user. You can use such a token both for authentication in the UI and for REST API requests.

If you change the scope to "_Limit per project_", you will be able to limit the token's access to a certain project and even select particular permissions for it. The list of available projects and permissions depend on your user role. Tokens with a limited scope can only be used for REST API requests.

<img src="limit-access-token.png" width="460" alt="Access token with limited permissions"/>

>Make sure to thoroughly configure the token's scope as some permissions might depend on another.

## Perforce support improvements

This release brings multiple improvements for projects versioned in [Perforce](https://www.perforce.com/).

### Perforce client in Linux Docker images

Linux Docker images of TeamCity agent and server now come with a Perforce client.

### Simplified setup of post-commit hooks

Post-commit hooks allow reducing the number of polling operations and offloading the TeamCity and VCS servers. In the previous versions, it was required to maintain several hooks for Perforce. Now, you can easily configure a single post-commit hook for the entire Perforce depot. To do this, follow [this instruction](configuring-vcs-post-commit-hooks-for-teamcity.md#Perforce+Server) in our documentation.

>A generic script for other VCS types is available [here](configuring-vcs-post-commit-hooks-for-teamcity.md#Generic+Post-Commit+Hook).

### Support ChangeView specification

TeamCity now supports the [ChangeView](https://www.perforce.com/manuals/p4guide/Content/P4Guide/configuration.workspace_view.changeview.html) specification in Perforce clients and understands the `@revision` syntax in the import statements of streams' definitions.

Moreover, if a [Perforce VCS root](perforce.md) is set to the _Client mapping mode_, you can use the `ChangeView` specification to limit the root's scope to a particular revision or multiple revisions. To do this, open the settings of a Perforce VCS root, choose the Client mapping connection mode, and enter the client mapping. For example:

```
//my-depot/... //team-city-agent/...
ChangeView:
    //my-depot/dir1/…@90
    //my-depot/dir2/…@automaticLabelWithRevision
```

where `90` is the number of the exact revision of `dir1` and `automaticLabelWithRevision` is the labeled revision of `dir2`. All the other revisions of these directories will not be monitored by this VCS root.

### Clean up stream workspaces on Perforce server

TeamCity allows using [Perforce streams as feature branches](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Streams). To optimally process changes in such streams, it needs to create and maintain dedicated workspaces on the Perforce server. Over time, these workspaces might consume a significant amount of resources on the Perforce server's machine. Besides, if you want to close a task stream, you won't be able to do this if there is a workspace associated with it. Both of these problems can be solved by deleting no longer necessary workspaces. Previously, there were no means to clean them up automatically, and any manual cleaning would require involving a Perforce server administrator. With the new _Perforce Administrator Access_ connection, project administrators can clean workspaces right from the TeamCity UI.

To configure this integration:

1. <include from="common-templates.md" element-id="open-project-settings"/>
2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
3. Add a new connection with the _Perforce Administrator Access_ type. Enter the host and user credentials for accessing the Perforce server (the user must have the [admin permission](https://www.perforce.com/manuals/p4sag/Content/P4SAG/protections.set.html#protections.set.access_levels)), and TeamCity will connect to it.

<img src="p4-admin-connect.png" width="460" alt="Perforce Administrator Access connection"/>

During every [clean-up](teamcity-data-clean-up.md), TeamCity will detect and delete workspaces that have been inactive for more than 7 days. Or, you can delete them anytime by clicking Delete these workspaces in the connection settings. Note that workspaces are deleted only on the server — not on build agents — and only if they were created by TeamCity.

If the support for feature branches is enabled in a Perforce root, it is also possible to delete workspaces associated with any stream available to this root. Go to __Build Configuration Home__, open the __Actions__ menu, and click __Delete Perforce stream workspaces__. By default, this action is available to all users with the Project Developer role. In this menu, you can specify a path to a stream, and TeamCity will delete the related workspaces on the Perforce server.

<img src="delete-stream-ws.png" width="296" alt="Delete Perforce stream workspaces"/>

## Onboarding UI assistant

TeamCity has a rich user interface, and some of its numerous handy features might not be obvious to beginners. To help our new users navigate around the interface, we introduce the onboarding UI assistant.

To enable it, open the __Help__ menu in the upper right corner of the screen and click __Show Hints__. You can hide them anytime by closing the assistant menu.

To see a hint for a certain element, hover over its name in the assistant menu. Some hint names also provide a ![link-to-doc.png](link-to-doc.png) link to the related documentation.

<img src="show-hints.png" width="460" alt="Onboarding UI assistant"/>

Hints are available for [experimental](teamcity-sakura-ui.md) __Project Home__, __Build Configuration Home__, and __Build Overview__, as well as for some of the __Settings__ pages.

## Experimental UI improvements

Our experimental UI is a work in progress: we introduce its features in the early stages of development so you can already benefit from them. In each release, we polish already existing pages and reproduce all the important classic features in the new UI.

The experimental UI roadmap strongly depends on our users' feedback. In version 2021.1, we've focused on improving the Build Overview page: read below about a [tree mode for tests](#Show+tests+in+tree+mode+in+Build+Overview) and a new [code coverage visualization](#New+code+coverage+preview). We've also [adjusted the design](#Refined+Project+Home) of the __Project Home__ page based on your requests.

>Leave feedback about your experience with the experimental UI via [this form](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?ticket_form_id=360001686659){nullable="true"}.

### Show tests in tree mode in Build Overview

Test results are often the most important information you seek when opening the build details. We keep improving how tests are represented in our experimental UI: our goal is to reproduce all instruments available in the classic UI and make them even handier for you. Now, the new UI supports the test tree mode in the __Build Overview__.

In the _Tests_ block, click __Show grouped tests__, and TeamCity will represent them in a tree mode, grouped together by _project → build configuration → test suite → test package → test class_. You can switch between the tree and flat structure anytime, depending on your current tasks.

<img src="test-tree-mode.png" width="706" alt="Tree mode for tests"/>

As tests within the same group have a similar purpose, you might want to quickly select them all or temporarily collapse them to focus on other groups. The tree mode works great for that, and it is especially helpful if there are many tests in your build configuration. When a group is selected, you can apply actions like __Investigate__ or __Mute__ to all its tests at once.

If you prefer a keyboard navigation, use __Up__ and __Down__ keys to navigate the tree and __Space__ to collapse or expand the selected group or test.

### New code coverage preview

TeamCity can provide code coverage for [multiple build runners](configuring-test-reports-and-code-coverage.md#Code+Coverage+in+TeamCity). With this release, the code coverage preview in __Build Overview__ gets even more visual.

If coverage is available for a build configuration, you can quickly preview a visualized statistics of the build's tests and see how it changed compared to the previous build:

<img src="code-coverage-preview.png" width="706" alt="Code coverage preview"/>

### Refined Project Home

After gathering your feedback on the experimental __Project Home__ page, we focused on two tasks: use the space on this page more effectively and make a project tree easier to navigate. We hope that the refined page will serve you as a handy dashboard for previewing and running all builds of the same project:

<img src="project-home.png" width="706" alt="Refined Project Home"/>

## Restrict to customize builds with personal patches

TeamCity allows running a custom build with your local changes — without actually changing the common project's code stored in a VCS. To run such a build, you need to send the patch with changes to the TeamCity server: either by using a [Remote Run](remote-run.md) in IDEs or by [uploading it](personal-build.md#Direct+Patch+Upload) via UI or REST API. This patch will be delivered to the build agent machine and used only in the custom build. However, as the patch is stored on the agent file system, it would be wise to ensure that it includes only trusted changes, which cannot potentially harm the next builds running on this agent.

For this purpose, we've created a new [user permission](managing-roles-and-permissions.md) — _Change build source code with a custom patch_. On upgrading to 2021.1, it will be automatically enabled for the default Project Developer role and other roles with the _Customize build parameters_ permission. By toggling this permission, you can granularly control who can patch builds or, if necessary, fully restrict this functionality in important projects.

## Customize multipart upload of artifacts to Amazon S3
{instance="tc"}

If you store build artifacts in Amazon S3, you can now control how they are uploaded. TeamCity uses a [multipart upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html) of large files by default, but it is now possible to customize its parameters: upload threshold and upload part size. This can help use the network bandwidth more effectively and improve throughput.

<img src="multipart-upload.png" width="706" alt="Multipart upload of large artifacts"/>


## Add multiple VCS triggers per build configuration

Now, you can add more than one [VCS trigger](configuring-vcs-triggers.md) to a build configuration. This is convenient if you want to trigger builds in the same configuration but according to different trigger rules and with different branch filters. This allows you to set a [quiet period](configuring-vcs-triggers.md#Quiet+Period+Settings) for builds in all branches except one — where a build will start as soon as a new change is committed. Or, you can configure a trigger to start builds on each check-in only in one given branch.

>To achieve even greater results, you can [customize the parameters of triggered builds](#Customize+automatically+triggered+builds).

## Choose checkout policy for Git roots

A Git VCS root has a new _Checkout policy_ option. You can choose between four policies and get different benefits, depending on the lifecycle of your build agents.

The default policy is AUTO, which means the decision is always up to TeamCity. But, you can enforce a certain policy for a VCS root by choosing one of the other options:
* _Use mirrors_: best for long-lived agents that run numerous builds. A repository mirror is kept on the agent, which speeds up the successive builds.
* _Do not use mirrors_: simply fetch a repository in the build checkout directory, without creating a mirror. Less optimal in terms of disk usage and preserved for backwards compatibility with existing build configurations.
* _Shallow clone_: best for short-lived agents (for example, disposable agents in the cloud). Fetches only a single revision necessary for one build. This can make the build start significantly faster, especially if the project repository has a long history of revisions.

[Read this article](git.md#git-checkout-policy) for more details.

## Viewing thread dump of process running inside a container

TeamCity allows viewing a thread dump of processes running on a build agent machine right in the Build Results. Now, you can view it even if the process runs inside a container (for example, with our [](container-wrapper.md) or [](docker-compose.md) functionality). For Windows containers, TeamCity shows Java processes and their thread dumps. For Linux containers, it shows all running processes and thread dumps for Java processes.

While the build is running, click __View thread dump__ in its __Overview__.

<img src="thread-dump.png" width="706" alt="View thread dump"/>

TeamCity will show the structured information about the agent processes:

<img src="thread-dump-docker.png" width="706" alt="Thread dump of a Docker process"/>

## Quick access to build status widget

Our [build status widget](configuring-general-settings.md#Enable+Status+Widget) shows the status of the last build in a configuration. As the widget doesn't require authentication, you can feature the build status icon anywhere outside TeamCity: for example, on GitHub. Now, you can quickly access this widget from the __Build Overview__.

>Make sure the widget support is enabled in __Build Configuration Settings | Build Options__.
>
{style="note"}

To access the widget menu from __Build Configuration Home__, open the __Actions__ menu and click __Get build status icon__:

<img src="status-icon.png" width="296" alt="Get build status icon"/>

The _Status Image_ menu shows the icon preview and allows copying its code in one of the supported formats:

<img src="status-widget.png" width="460" alt="Build status widget"/>

You can embed this code into an external page and view the build status without opening TeamCity.

>Learn how to use a custom HTML widget instead of the static icon in [this article](configuring-general-settings.md#HTML+Status+Widget).

## Other improvements

* __Cross-platform ReSharper Inspections and Duplicates Finder__  
  ReSharper [Inspections](inspections-resharper.md) and [Duplicates Finder](duplicates-finder-resharper.md) now support the cross-platform mode and can be run inside Docker.
* __Setting .NET SDK requirements for build agents__  
  The [.NET build runner](net.md) now allows setting a requirement to SDKs installed on a build agent. You can enter the list of expected SDK versions when configuring a .NET step, and TeamCity will automatically create agent requirements: the current build will only run on agents that have all the required SDKs.
* __Python runner updates__
  * The [Python](python.md) steps can be run inside a [Venv](https://docs.python.org/3/library/venv.html) virtual environment.
  * The runner can automatically install Flake8, Pylint, or Pytest if they are missing on the build agent.
  * The runner supports the [Poetry](https://python-poetry.org) environment tool for packaging and managing dependencies.
* __Marketplace plugin verification__  
  TeamCity will check if every new plugin, installed from the [JetBrains Marketplace](https://plugins.jetbrains.com/marketplace), is signed with a valid certificate. This will ensure that the installed plugin is safe to use and its source code is intact.

## Fixed issues

See [TeamCity 2021.1 release notes](teamcity-2021-1-release-notes.md).

## Upgrade notes
{instance="tc"}

Before upgrading, we highly recommend reading about [important changes in version 2021.1 compared to 2020.2.x](upgrade-notes.md#Changes+from+2020.2.x+to+2021.1).

## Previous releases
{instance="tc"}

* [What's New in TeamCity 2020.2](https://www.jetbrains.com/help/teamcity/2020.2/what-s-new-in-teamcity.html)
* [What's New in TeamCity 2020.1](https://www.jetbrains.com/help/teamcity/2020.1/what-s-new-in-teamcity.html)
* [What's New in TeamCity 2019.2](https://www.jetbrains.com/help/teamcity/2019.2/what-s-new-in-teamcity-2019-2.html)
* [What's New in TeamCity 2019.1](https://www.jetbrains.com/help/teamcity/2019.1/what-s-new-in-teamcity-2019-1.html)

## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.
