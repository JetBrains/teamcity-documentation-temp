[//]: # (title: Build Configuration Home Page)
[//]: # (auxiliary-id: Build Configuration Home Page;Maven-related Data)

This article gives an overview of the __Build Configuration Home__ page of the new TeamCity UI. Most of its features are also available in the classic UI mode.

The build configuration details are represented on multiple tabs whose set may vary depending on your server or project configuration.

## Overview

This tab provides information on:
* Pending changes, also listed as a [separate tab](#Pending+Changes).
* The current status of the build configuration, and, if applicable:
    * Number of [queued builds](working-with-build-queue.md).
    * For a running build, the progress details with the __Stop__ option to terminate the build.
    * For a [failed build](build-state.md), the number, [agent](build-agent.md), and so on.
* [Build history](build-results-page.md#Build+History+in+Classic+UI).

## History

_Available only in the classic UI._

_Build history_ is a record of the past builds produced by TeamCity. The __History__ tab allows filtering builds by build agents, [tagging builds](build-actions.md#Add+Tags+to+Build), and filtering them by tags (if available).

## Change Log

By default, lists changes from builds finished during the last 14 active days. Use the _Show all_ link to view the complete changelog.

This tab shows the changelog with its graph of commits to the [monitored branches](build-results-page.md#Changes+Tab) of all VCS repositories used by the current build configurations and the repositories used by the [dependencies and dependent configurations](dependent-build.md) of the current configuration.

## Issue Log

Lists related issues, if [integration with an issue tracker](integrating-teamcity-with-issue-tracker.md) is configured.

## Statistics

Displays the collected statistical data as [visual charts](statistic-charts.md#Build+Configuration+Statistics).

## Compatible Agents

Lists all authorized agents. Agents meeting [agent requirements](configuring-agent-requirements.md) are listed as compatible. For each incompatible agent, the reason is provided.  
The agents belonging to the [pool(s)](configuring-agent-pools.md) associated with the current project are listed first.

## Pending Changes

Lists [changes](change-state.md) waiting to be included in the next build.

## Settings

Lists the current [build configuration settings](creating-and-editing-build-configurations.md#Configuring+Settings).

## View Investigation History

The _Investigation History_ action of the __[Actions](build-actions.md#Build+Configuration+Actions+Menu)__ menu gives access to the log of all [investigations](investigating-and-muting-build-failures.md) of the current build configuration. It is most helpful for big teams and projects when it is not as easy to determine who and when changed or resolved the investigation.

To see all the actions applied to an investigation, open its context menu and click __Investigation History__.

<img src="wn-investigation-history.png" alt="Investigation history"/>

## Maven Tab

You can find information about settings specified in your Maven project's `pom.xml` file on the dedicated __Maven__ tab. It also shows _Provided parameters_: for example, `maven.project.name`, `maven.project.groupId`, `maven.project.version`, `maven.project.artifactId`. You can use these parameters within your build and reference them within the build number pattern using the `%`-notation. For example, `%\maven.project.version%.{0}`.

