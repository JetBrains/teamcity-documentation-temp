[//]: # (title: What's New in TeamCity 2019.2)
[//]: # (auxiliary-id: What's New in TeamCity 2019.2)

## New flexible clean-up rules

Automatic build clean-up has been present in TeamCity since the early versions. It allows conveniently deleting old and no longer necessary build data. While the provided customization options are quite easy to configure, they cover only the most common cases and do not allow for fine-tuning.

With this release, we are introducing more options for flexible control of the clean-up process. In addition to existing "_base rule_" for a project or build configuration, you can now create multiple "_keep rules_" to specify what builds and data to preserve during the clean-up. The keep rules are more fine-grained and can cover cases like keeping all the builds with a certain tag (for example, `release`) or in a certain branch. While using the new keep rules requires a better understanding of different kinds of builds and their data, it also offers greater flexibility. 

The __Clean-up Rules__ section of __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ allows managing base and keep rules for the current project and for its subprojects and build configurations.

<img src="clean-up-rules.png" width="1000" alt="Clean-Up Rules page"/>

When adding a keep rule, you can specify:
* __Build data to preserve__: history, artifacts, logs, statistics, or everything.
* __Builds range__: what time interval or what number of last builds will be affected by the rule.
* Optionally, __filter__ the builds by their __status__, __tags__, and __branches__. Choose if to limit the builds to __personal__ or __non-personal__ only.   
You can also choose if the rule is applied per each matching branch individually or to all the builds in the selected branches as a single set.

Example of a keep rule:

<img src="keep-rule-example.png" width="350" alt="Keep Rule example"/>

Read more in [Clean-up](teamcity-data-clean-up.md).

## Metrics reporting

TeamCity now provides [Prometheus](https://prometheus.io)-compatible server metrics displayed on the __Diagnostics | Metrics__ tab and available via the `app/metrics` API endpoint.

To access the metrics, make sure your TeamCity user account has the "_View usage statistics_" permission assigned to it.

The metrics include:
* CPU, memory, and system load
* time spent on processing a build queue
* number of active user sessions,
* and more

The full list is available via `https://<teamcity-server-host>/app/metrics`.

Some of the metrics are experimental, but you can already access them by adding the `?experimental=true` query string to the endpoint URL or enabling "_Show experimental metrics_" option on the __Metrics__ tab.

To collect the metrics, we suggest that you use a Prometheus database or a combination of [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) & [InfluxDB](https://www.influxdata.com/). For a visual representation of the metrics, you can use [Grafana](https://grafana.com/) or any other similar solution.

Example of TeamCity metrics represented in Grafana:

<img src="grafana-monitoring-example.png" width="800" alt="Example of TeamCity metrics represented in Grafana"/>

## Code highlighting in build runner scripts

We’ve added automatic code highlighting and line numbering for scripts of the following build runners: [Command Line](command-line.md), [Ant](ant.md), [PowerShell](powershell.md), [Docker](docker.md), [NAnt](nant.md), and [Rake](rake.md), as well as for the configuration of [Amazon EC2 images](setting-up-teamcity-for-amazon-ec2.md).

To improve readability, you can now apply soft wraps to the code by clicking ![code-wrap-icon.png](code-wrap-icon.png) next to the input form.

Example of Dockerfile highlighting in TeamCity:

<img src="code-hl.png" width="500" alt="Example of Dockerfile highlighting in TeamCity"/>

## Support of EC2 launch templates

TeamCity now supports [Amazon EC2 launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html) for cloud instances. Many of the EC2 users appreciate the launch templates since they allow reusing a once defined launch specification for all new instances, which eliminates the need to describe the launch settings every time new instances are requested.

If your cloud profile is connected to the Amazon server, TeamCity will automatically detect launch templates available on this server. Simply select a template as an image _Source_ and specify its version, and TeamCity will request instances based on the template parameters.

Read more in [Setting Up TeamCity for Amazon EC2](setting-up-teamcity-for-amazon-ec2.md).

## Restoring backup on TeamCity startup

If you start the TeamCity server instance for the first time and want to restore the backed up data of your previous TeamCity installation, you can now restore the backup right from the startup screen UI.

<img src="backup-restore-ui.png" width="400" alt="Automatic backup restore"/>

Read more in [Restoring TeamCity Data from Backup](restoring-teamcity-data-from-backup.md).
 
## Running personal builds on unified diffs

Now you can run a personal build containing your local changes based on a diff patch uploaded via the TeamCity UI or via REST API.

Previously, it was possible to run personal builds, based on your local changes, only by integrating your TeamCity server with some [supported IDE](supported-platforms-and-environments.md#IDE+Integration). Now you can save your patch as a `.diff` file in a unified format (for example, using IntelliJ IDEA or the `git diff` command) and upload it directly to the server.

Read more about this feature in [Personal Build](personal-build.md#Direct+Patch+Upload).

Note that this is an experimental feature. TeamCity provides a stable parsing of unidiff files generated by Git and IntelliJ IDEA. Binary changes in non-binary files are not supported.

## Extended REST API for cloud profiles

TeamCity REST API now provides the `.../app/rest/cloud/profiles`, `.../app/rest/cloud/images`, and `.../app/rest/cloud/instances` endpoints that expose the same cloud integration details as those provided in the TeamCity UI.

## Predefined build parameters for pull requests

We have added a few predefined build parameters to expose valuable information on pull requests, so it can be used in the settings of a build configuration or in build scripts:
 
```Text
teamcity.pullRequest.number //pull request number
teamcity.pullRequest.title //pull request title
teamcity.pullRequest.source.branch //VCS name of the source branch; provided only if the source repository is the same as the target one
teamcity.pullRequest.target.branch //VCS name of the target branch
```
## Support of cross-platform dotCover

TeamCity now allows collecting code coverage for .NET Core projects on Linux and macOS by supporting cross-platform JetBrains dotCover, version 2019.2.3+.

dotCover 2019.2.3 for Windows is bundled with TeamCity. If you need to collect code coverage under non-Windows platforms, add the cross-platform dotCover tool under __Administration | Tools__ and enable the dotCover coverage in the [.NET CLI](net.md) build step. If you want to use cross-platform dotCover under Windows as well, make sure the agents have .NET Framework SDK 4.6.1+ installed.

Now you can also run code coverage analysis with dotCover inside a Docker container, with the [](container-wrapper.md) extension.

## Updates for multinode setup

### User-level actions on secondary nodes

In previous versions of TeamCity, secondary nodes provided a read-only interface. It was not possible to add builds to the queue, tag/pin builds, or perform any other user-level actions. With this release, it changes. Now, if a secondary node is granted any responsibility (that is it does not act as a read-only server), it will enable build actions for users.
Currently, supported user-level actions are:
* Triggering a build, including a custom or personal one
* Stopping a build
* Pinning/tagging/commenting a builds
* Deleting a build
* Assigning investigations and muting build problems and tests
* Marking a build as successful/failed
* Merging sources and label sources actions

See the full list of supported actions in our tracker: [TW-62749](https://youtrack.jetbrains.com/issue/TW-62749).

Administrator-level actions are not yet available on secondary nodes. Use the main server if you need to change the server configuration.

### Optimized server-side patches download on agents

Since this release, agents can download server-side patches from secondary nodes — not only from the main server, as it was before.

Server-side patches are mostly used when an agent cannot find a VCS client executable (for example, Git or Perforce) on an agent machine. In this case, the agent will request the server to create a patch with VCS changes and send it to the agent. Now, if you assign the "_[VCS repositories polling](multinode-setup.md#VCS+Repositories+Polling+on+Secondary+Node)_" and "_[Processing data produced by builds](multinode-setup.md#Processing+Data+Produced+by+Builds+on+Secondary+Node)_" responsibilities to a secondary node, the agents will be able to request patches from this node as well, which significantly reduces the load on the main server.

## Updates for DSL-based projects

TeamCity Kotlin DSL receives the following updates:
* DSL offers an alternative way to configure a [build chain](build-chain.md) in a pipeline style ([read more](kotlin-dsl.md#Build+Chain+DSL+Extension)).
* You can now customize the DSL generation behavior using context parameters configured in the TeamCity UI ([read more](kotlin-dsl.md#Use+Context+Parameters+in+DSL)).
* If the "_Store secure values outside of VCS_" option is enabled for a project, TeamCity allows you to see all the project tokens and generate new ones on the __Version Settings | Tokens__ tab ([read more](storing-project-settings-in-version-control.md#Managing+Tokens)).

## New features of experimental UI

<note>

Our experimental UI is a work in progress: we introduce the changes in the early stages of development so you can already benefit from the new features. Based on your feedback, we will make sure to optimize the experimental pages and migrate all the important classic features in the future releases.   
For now, if any of the familiar features are missing, you can switch an experimental page to the classic UI with the ![mammoth.png](mammoth.png){style="block"} button, or turn off the experimental UI on the server completely in __My Settings & Tools__.

__Try out the TeamCity experimental UI and leave your feedback via our [UI survey](https://surveys.jetbrains.com/s3/feedback-form-for-teamcity?tcv=2019.2)!__

</note>
 
TeamCity 2019.2 introduces the redesigned [Build Details](#Experimental+Build+Details+Page) and [Agents](#Experimental+Agents+page) pages, as well as other [experimental features](#Other+new+features+of+experimental+UI).

### Experimental Build Details Page

In our attempt to rethink the approach to displaying build details, we have redesigned the __Build Details__ page so it provides better visualization and gives quick access to all other projects via the sidebar.

<img src="exp-build-details.png" width="1000" alt="Experimental Build Details page"/>

You can instantly preview all the previous builds and their details, without leaving the current build page:

<img src="build-trends-preview.png" width="250" alt="Build trends preview"/>

A visualized build timeline reflects the duration of each stage and indicates build problems:

<img src="build-timeline.png" width="600" alt="Build timeline"/>

Click any build stage to open the corresponding line of the build log. Note that in the new UI, even a long log can be displayed directly in the preview, with no need to download it.

Apart from the build timeline and build log, the __Overview__ tab gives quick access to build problems, tests, changes, and dependencies. The corresponding tabs have also been updated and now offer new features:

* The __Changes__ tab displays more information about changes in the build. You can separately browse user and artifact changes, and optionally display changes in build settings. Click any change to preview its details. <img src="exp-changes-tab.png" width="700" alt="Experimental Changes tab" style="block"/>

* The __Tests__ tab allows quickly switching between failed, ignored, and succeeded tests. Click a test to view its details and assign an investigation. <img src="exp-tests-tab.png" width="700" alt="Experimental Tests tab" style="block"/>

* The __Dependencies__ tab provides three alternative modes of displaying the build dependencies: visual timeline, structured list, and build chain. <img src="exp-dependencies-tab.png" width="700" alt="Experimental Dependencies tab" style="block"/>

### Experimental Agents page

The experimental __Agents__ page loads faster for a large number of agents and allows quickly switching between agent details. You can use the sidebar to browse the agent pool hierarchy and search agents and pools by name. The __Reports__ section provides statistics about running, idle, and disconnected agents in every pool.

<img src="exp-agents-page.png" width="700" alt="Experimental Agents page"/>

### Other new features of experimental UI

* __Comparison of two builds__   
With the _Compare Builds_ feature, you can select two builds from the same configuration and review information about their parameters, revisions, statistics, and tests side-by-side. This allows for easier monitoring and is especially helpful when multiple users manage and monitor builds. For example, if a build has no changes in the project code but fails for no obvious reason, you can compare this build with the last successful build and analyze their differences to find the most probable cause of the failure.   
To compare a build with another, open the __Actions__ menu of this build, click __Compare with__, and select a build for comparison.
 
* __Expanded build preview in the build list__   
TeamCity now allows you to preview the most important build results directly in the list of builds. On the __Build Configuration Home__ page, click a build line in the list to see its details.
 
* __Favorite Projects section__   
We have migrated the __Projects__ section that gives quick access to your favorite projects.
 
* __Redesigned Changes pop-up__   
Build changes are now sorted chronologically and grouped by changes in code and changes in artifact dependencies. You can also filter changes by their author and toggle the display of changes made in the build configuration settings.

* __Information on build queueing reason__   
If a build has been staying in a queue for too long, you can now see a reason for the delay by clicking the "_In queue_" label when previewing the build in __Trends__ or in the build list.

## Other improvements

* The [branch filter](branch-filter.md) has been added for artifact dependencies, retry triggers, and the Pull Requests build feature.
* Autodetection of Dockerfile changes: once the [Docker Compose](docker-compose.md) build runner creates a Docker image on an agent, it is stored on this agent to be used for the later builds. Now, TeamCity will automatically detect that the Dockerfile, used for building the image, has changed and will force the Docker Compose step to rebuild this image.
* You can now enforce the [interval of VCS changes check](teamcity-configuration-and-maintenance.md#default-vcs-check-interval), specified in __Global Settings__, as a minimum polling interval for all VCS roots on the server. This way, Project Administrators will only be able to set intervals that are larger than the default one. This helps restrict the frequency of polling requests thus offloading the server.
* Project administrators with the enabled "_Change user / group notification rules in project_" permission can edit notification rules for users and user groups assigned to their projects.
* Now you can re-run personal builds, just like regular builds: open the context menu of a finished build and click Re-run this build.
* TeamCity can automatically manage the amount of memory used by the `git fetch` process.   
If you have previously used the `teamcity.git.fetch.process.max.memory` internal property to set the memory amount available for fetching in each VCS root, you can now disable it to delegate the detection of memory consumption to the TeamCity server. You can control the limit of available memory via the `teamcity.git.fetch.process.max.memory.limit` property.
* You can configure more than one SSH key for a build. It is useful if the build needs to authenticate in several external systems or repositories. To use multiple SSH keys in a build:
  * On the project level: specify these keys in SSH Keys.
  * On the build configuration level: add several SSH Agent build features, one per each key.
* TeamCity uses [improved rules](investigating-and-muting-build-failures.md#Relation+with+Branches) for resolving assigned investigations and unmuting muted problems and tests in active branches. Now it waits until the build problem (or failed test) is fixed in all active branches before unmuting it or resolving its investigation.
* The [retry build trigger](configuring-retry-build-trigger.md) now can trigger a new build with the same revision and put it to the queue top.

## Fixed issues

[Full list of fixed issues](https://confluence.jetbrains.com/display/TW/TeamCity+2019.2+Release+Notes)

## Upgrade notes

[Changes from 2019.1.x to 2019.2](upgrade-notes.md#Changes+from+2019.1.x+to+2019.2)

## Previous releases

[What's New in TeamCity 2019.1](https://www.jetbrains.com/help/teamcity/2019.1/what-s-new-in-teamcity-2019-1.html)
