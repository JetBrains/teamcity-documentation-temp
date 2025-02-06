[//]: # (title: Build Results Page)
[//]: # (auxiliary-id: Build Results Page;Build History)

<snippet id="build-results-intro">

In TeamCity, all information about a build, whether it is queued, running or finished, is accumulated on its __Build Results__ page. To view build results, choose any configuration to view its build history and click a required build number.

<img src="dk-build-results-page-overview.png" width="706" alt="Build results page"/>

This page includes several static tabs (Overview, Changes, Build Log, Artifacts, and others), as well as contextual tabs whose visibility depends on specific configuration features. For example, the **Dependencies** tab is shown only for builds whose parent configurations belong to a [build chain](build-chain.md).

</snippet>

> Some of the page data is accessible only after the build is finished. Other details like Changes, Parameters, and Dependencies are available while the build is waiting in the queue.
> 
{style="note"}

## Internal Build ID

Knowing a build ID can be required when working with [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html) or [downloading build artifacts](patterns-for-accessing-build-artifacts.md). You can obtain this ID from the URL or the **Parameters** tab of the __Build Results__ page.

The URL of the __Build Results__ page typically looks like the following:

* In Sakura UI: `<SERVER_URL>/buildConfiguration/<CONFIGURATION_ID>/<BUILD_ID>?...`.
* In Classic UI: `<SERVER_URL>/viewLog.html?buildId=<BUILD_ID>&buildTypeId=<CONFIGURATION_ID>&...`

The `<BUILD_ID>` portion is the internal numeric build ID uniquely identifying this build in TeamCity.

You can also find the same value under the build's **Parameters** tab in TeamCity UI, stored in the `teamcity.build.id` parameter.

## Overview Tab

The **Overview** tab displays general information about the build: build duration, user or automatic trigger that started it, a build agent that was used to run it, the summary of processed changes, and so on.

<img src="exp-build-details.png" alt="Sakura Build Results page"/>

It also shows the diagnostic messages related to the current build's status.

## Changes Tab

The **Changes** tab shows the code changes associated with the build, including commit messages and authors. This tab allows you to:

* Filter changes by their author, comment, path, and revision.
* Review all changes included in the build with their corresponding [revisions](revision.md) in version control.
* View changes from related dependencies (for builds that belong to a [build chain](build-chain.md)).
    
    If the current build configuration has an artifact dependency, and the artifact downloaded in the current build has changed compared to the one downloaded in the previous build of the current configuration, the __Artifact dependencies changes__ node appears displaying the build which was used to download artifact dependencies from, and the changes which were included in that build.

* View file changes as diffs: tick the **Show files** checkbox and click a modified file.
* Download a change as a .patch diff file: click the ellipsis button (...) and select **Download patch**.
* [Label the build sources](vcs-labeling.md).
* [Trigger a custom build](running-custom-build.md) with this change.
* Review the change in an [external change viewer](external-changes-viewer.md), if configured by the administrator.
  {instance="tc"}



If enabled, the __Show graph__ checkbox displays the changes as a graph of commits to the VCS roots related to this build. Key features of this graph include:

* Hover over to a graph node to view the VCS root revision number, and a connector line between nodes to view VCS roots that were changed in this build. Each of the roots is displayed as a separate line. You can click any bar to highlight changes pertaining to this root.
* If there have been merges between the branches of the repository, the graph displays them. 
* If your VCS root has subrepositories (marked S in the list of changes), navigate to a node in the parent to see which commits in subrepositories are referenced by this revision in the parent.



## Build Log Tab

The graphic build log timeline reflects the duration of each build stage and indicates build problems:

<img src="build-timeline.png" width="600" alt="Build timeline"/>

Click any stage to open the corresponding line of the build log.

More information on build logs in TeamCity is available [here](build-log.md).

## Artifacts Tab

Provides access to [downloadable files](build-artifact.md) produced by the build (executables, images, documentation, and so on).

## Parameters Tab

The **Parameters** tab allows you to track standard and user-defined <a href="configuring-build-parameters.md">build parameters</a> associated with this build. Highlights parameters whose values changed during the build.

<include from="levels-and-priority-of-build-parameters.md" element-id="build-results-parameters-tab"/>


## Dependencies Tab

If a finished build has artifact and/or snapshot dependencies, the __Dependencies__ tab is displayed on the __Build Results__ page. Here you can explore builds whose artifacts and/or sources were used for creating this build (downloaded artifacts) as well as the builds which used the artifacts and/or sources of the current build (delivered artifacts). Additionally, you can view indirect dependencies for the build. That is, for example, if build A depends on build B which depends on builds C and D, then these builds C and D are indirect dependencies for build A.

The __Dependencies__ tab provides three modes of displaying the build dependencies: a visual timeline, structured list, and build chain. Choose the mode that is the most helpful for your current task.

<img src="exp-dependencies-tab.png" alt="Dependencies Tab of Build Results" width="706"/>

The "Timeline" and "List" view modes allow you to search for specific builds by their names.

<img src="dk-searchDependencies.png" alt="Find Panel in dependencies" width="706"/>

## Issues Tab

If a build configuration has a [integration with an issue tracker](integrating-teamcity-with-issue-tracker.md) set up and if there is at least one issue mentioned in the comments for the included changes or in the comments for the build itself, you will see the list of issues related to the current build in the __Issues__ tab.

>If you need to view all the issues related to a build configuration and not just to particular build, you can navigate to the __Issues Log__ tab available on the __Build Configuration Home__ page, where you can see all the issues mapped to the comments or filter the list to particular range of builds.

## Code Coverage Tab

If you have code coverage configured in your build runner, the __Code Coverage__ tab with the full HTML code coverage report will appear.

By clicking the links in the __Coverage Breakdown__ section, you can drill down to display statistics for different scopes: for example, Namespace, Assembly, Method, and Source Code.

## Code Inspection Tab

If configured, the results of a [Code Inspection](configuring-test-reports-and-code-coverage.md#Code+Inspection+in+TeamCity) build step are shown on the __Code Inspection__ tab. Use the left pane to navigate through the inspection results; the filtered inspections are shown in the right pane.
* Switch from the __Total__ to __Errors__ option, if you are not interested in warnings.
* Use the scope filter to limit the view to the specific directories. This makes it easier for developers to manage specific code of interest.
* Use the inspections tree view under the scope filter to display results by specific inspection.
* Note that TeamCity displays the source code line number that contains the problem. Click it to jump the code in your IDE.

## Duplicates Tab

If your build configuration has Duplicates build runner as one of the build steps, you will see the __Duplicates__ tab in the __Build Results__.

The tab consists of:
* A list of duplicates found. The __new only__ option enables you to show only the duplicates that appeared in the latest build.
* A list of files containing these duplicates. Use the left and right arrow buttons to show selected duplicate in the respective pane in the lower part of the tab.
* Two panes with the source code of the file fragments that contain duplicates.
* Scope filter in the upper-left corner lists the specific directories that contain the duplicates. This filtering makes it easier for developers to manage the code of interest.

## Tests Tab

The __Tests__ tab allows switching between failed, ignored, and succeeded tests, and use various filters.

For each failed test, you can view its stacktrace, the diff between the expected and actual values, jump to the test history, [assign a team member to investigate its failure](investigating-and-muting-build-failures.md), [open the test in your IDE](installing-tools.md) and/or start fixing it right away.

>Watch our **video tutorial** on how to [use the test report page](https://www.youtube.com/watch?v=LKJjcBJT1k0).

To see a detailed test or investigation history, click __Show test history__ or __Show investigation history__ in its context menu.

Clicking the __Show test history__ link opens the __Test details__ page where you can find following information:
* The test details, including the test success rate and test's run duration data and graph.
* A complete test history table containing information about the test's status, its duration, and information on the builds this test was run in.

### Test Duration Graph

The Test Duration graph is useful for comparing the amount of time it takes individual tests to run on the builds of this build configuration.

Test duration results are only available for the builds which are currently in the build history. Once a build has been [cleaned up](teamcity-data-clean-up.md), this data is no longer available.

You can perform the following actions on the Test Duration Graph:
* Filter out the builds that failed the test by clearing the __Show failed__ option.
* Calculate the daily average values by selecting the __Average__ option.
* Click a dot plotted on the graph to jump to the page with the results of the corresponding build.
* View a build summary in the tooltip of a dot on the graph and navigate to the corresponding __Build Results__ page.
* Filter information by agents selecting or clearing a particular agent or by clicking __All__ or __None__ links to select or clear all agents.

## Maven Build Info Tab

For each Maven build, a build agent gathers Maven-specific build details to be displayed on the __Maven Build Info__ tab after the build finish. This tab can be useful for build engineers when adjusting build configurations.

## PerfMon Tab

Shows hardware data related to this build: CPU usage statistics, memory consumption, and others. This page is available for build configuration with the <a href="performance-monitor.md">Performance Monitor</a> feature (included by default for all new configurations created via existing VCS connections).


## Classic UI Tabs

Other classic UI tabs, not yet reproduced in the new UI, are also available: click __More__ and select the required tab in the list. To view the sections described below, yu can also temporarily switch to the classic UI via the respective toggle in the upper right corner.

### Build History in Classic UI

_Build history_ is a record of the past builds produced by TeamCity. It is displayed as the __Trends__ widget in the new UI and as the menu with links in the classic UI, which is described below.

To view the history of builds in the current configuration, click __previous__ and __next__ links in the upper right corner of __Build Results__. Click __All history__ link to open the __History__ tab.

Navigation menu:

<img src="history-buttons.png" alt="Navigate through history" width="228"/>

The __History__ tab:

<img src="build-history.png" alt="Build history" width="1312"/>

### Tests History in Classic UI

The __Tests__ tab of the classic UI differs from the new version and contains a few unique features, such as the ability to download all tests in CSV and pop-up duration graph images. You can temporarily switch to it whenever you need its capabilities.


