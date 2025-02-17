[//]: # (title: Changing Build Configuration Status)
[//]: # (auxiliary-id: Changing Build Configuration Status)

A build configuration is characterized by its state visible in the UI which can be _paused_ or _active_. By default, when created, all configurations are active and can be paused manually as described below or automatically if the project is [archived](archiving-projects.md).

If a build configuration is _paused_, its [automatic build triggers](configuring-build-triggers.md) are disabled until the configuration is activated. Still, you can start a build of a paused configuration manually or automatically as a part of a [build chain](build-chain.md). Besides, information on paused build configurations is not displayed on the [Changes](viewing-user-changes-in-builds.md) page.

It is possible to manually pause all or selected build configurations for a project.

## Pausing Build Configuration

To pause a single build configuration, open its __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes)__ or __Home__ page. In the __Actions__ menu, click __Pause__ and enter your comment in the _Pause_ dialog (optional). To remove the builds of the paused build configuration from the [build queue](working-with-build-queue.md), check the _Cancel already queued builds_ box. Click __Pause__ to confirm.

To activate a paused build configuration, select __Activate__ in the __Actions__ menu.

## Pausing Several Build Configurations in Project

To pause several build configurations in a project:
1. On the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page, open the __Actions__ menu and click __Pause/Activate__.
2. In the dialog that opens, select the box next to the project to pause all its build configurations or check the boxes of the configurations selectively: the checked build configurations will be paused.
3. Add an optional comment.
4. To remove all the builds of the paused configurations from the [build queue](working-with-build-queue.md), check the _Cancel already queued builds if build configuration is paused_ box.
5. Click __Apply__.

To activate several build configurations in a project:

1. On the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page, open the __Actions__ menu and click __Pause/Activate__.
2. In the dialog that opens, clear the box next to the project to activate all its build configurations or clear the boxes of the configurations selectively: the unselected build configurations will be activated.
3. Add an optional comment.
4. Click __Apply__.

## Build Configuration Status

In general, a build configuration status reflects the status of its last finished build.

Note that [personal builds](personal-build.md) do not affect the build configuration status.

You can view the status of all build configurations on the __Projects__ page (for all projects) or __Project Home__ page (for a particular project), when the details are collapsed.

Build configuration status icons:

<table><tr>

<td width="250">

Icon


</td>

<td>

Description


</td></tr><tr>

<td>

![Successful.png](Successful.png)


</td>

<td>

The last build on default branch executed successfully.


</td></tr><tr>

<td>

![Failed.png](Failed.png)


</td>

<td>

The last build on the default branch executed with errors or one of the currently running builds is failing. The build configuration status will change to "failed" when there's at least one currently running and failing build, even if the last finished build was successful.


</td></tr><tr>

<td>

![investigate.png](investigate.png)

![fix.png](fix.png)

</td>

<td>

Indicates that someone has started investigating the problem, or already fixed it. See [Investigating and Muting Build Failures](investigating-and-muting-build-failures.md).


</td></tr><tr>

<td>

_no icon_

</td>

<td>

There were no finished builds for this configuration, the status is unknown.

</td></tr><tr>

<td>

![paused.png](paused.png)

</td>

<td>

The build configuration is paused: all triggers are disabled, but the build can still be triggered manually or by [dependencies](configuring-dependencies.md).

To see who paused the configuration or to unpause it, click the link next to the status icon.

</td></tr></table>

## Status Display for Set of Build Configurations

It is possible to filter out the build configurations whose status you want to be displayed in TeamCity or externally.

To display the status of selected build configurations in __TeamCity__:
* configure visible projects on the __Projects__ page to display the status of build configurations belonging to these projects only
* implement a [custom Java plugin](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html) for TeamCity to make the page available as a part of  the TeamCity web application

To display the status for a set of build configurations __externally__ (on your company's website, wiki, Confluence, or any other web page), you can:
* use the [external status widget](configuring-general-settings.md#HTML+Status+Widget)
* use the [build status icon](https://www.jetbrains.com/help/teamcity/rest/get-build-status-icon.html)
* use any of the available [visualization plugins](https://plugins.jetbrains.com/search?products=teamcity&tags=Notification%2FVisualizers)
* implement a separate page or application which will get the build configuration status via the TeamCity [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html)

<seealso>
        <category ref="reference">
            <a href="build-state.md">Build State Reference</a>
        </category>
</seealso>