# User Guide

This section focuses on **regular TeamCity usage**. To learn more about other aspects of TeamCity, refer to the [](project-administrator-guide.md) and [](project-administrator-guide.md) sections.


## Authorization

Your organization provides a URL to access TeamCity. Depending on authorization modules configured by your server administrator, you can log in via a regular username/login pair, or using 3rd-party services (GitHub, Google, Bitbucket Server, and others).


## TeamCity Overview

Major TeamCity elements are:

* <include from="project-administrator-guide.md" element-id="build-step-overview"/>
* <include from="project-administrator-guide.md" element-id="build-configuration-overview"/>
* <include from="project-administrator-guide.md" element-id="project-overview"/>

These elements are set up by [project admins](project-administrator-guide.md) and typically cannot be edited by regular developers.

TeamCity user permissions are project-based. If you were assigned as a regular developer to a specific project, you can see only this project (with its subprojects). Other projects that exist on the same server are not shown to you. The figure below illustrates the list of the projects on a TeamCity server, as seen by a server administrator (left) and project developer assigned to three projects (right).

<img src="dk-project-visibility-user.png" width="706" alt="Different project visibility for different users"/>


## Find Projects and Builds

The figure below illustrates main UI elements that allow you to locate projects and builds.

<img src="dk-main-ui-elements.png" width="706" alt="Main UI Elements"/>

* Projects Overview (1) — The main page lists all available projects. Clicking on a project reveals its build configurations (2). You can star projects and builds to add them to your favorites.

* Breadcrumbs (3) — Displays the path of the currently viewed configuration or project.

* Search bars — Allow you to filter the list of configurations and projects (4) and find specific builds (5).

## Run New Builds

Depending on configuration triggers, new builds can start automatically. <include from="project-administrator-guide.md" element-id="triggers-pa-guide"/>

This section outlines how to trigger builds manually.

### Default Builds

To start a new build with default settings, open a required configuration and click **Run** in the top right corner.

<img src="dk-run-button.png" width="706" alt="Run button"/>

### Custom Builds

The ellipsis button (...) next to **Run** invokes the [Run Custom Build](running-custom-build.md) dialog that allows you to trigger builds with custom settings: modified parameters, elevated build queue priority, specific changelist, and so on.

For configurations that use parameters with unspecified values, this dialog pops up whenever you click the **Run** button.

Related article: [](running-custom-build.md)

### Personal Builds

<include from="personal-build.md" element-id="personal-build-overview"/>

Related article: [](personal-build.md)


## Investigating Build Results

From the main configuration screen, click on any build number to get into the [](build-results-page.md) page. This page has multiple tabs with comprehensive data about the build.

<img src="dk-build-results-page-overview.png" width="706" alt="Build results page"/>

<deflist>
<def title="Overview">
Displays general information about the build: build duration, user or automatic trigger that started it, a build agent that was used to run it, the summary of processed changes, and so on.
</def>

<def title="Changes">
Lists the code changes associated with the build, including commit messages and authors. This tab allows you to navigate directly to the related VCS and inspect a corresponding change.
</def>

<def title="Tests">
If the build includes tests, the "Tests" tab displays the test results. You can view details of passed and failed tests, including error messages and stack traces.
</def>

<def title="Build Log">
Provides a detailed record of the build process, including any errors or warnings. Allows you to filter log messages by their importance. Use this tab to pinpoint the cause of build failures.
</def>

<def title="Artifacts">
Provides access to downloadable files produced by the build (executables, images, documentation, and so on).
</def>

<def title="Parameters">
Allows you to track service and user-defined <a href="configuring-build-parameters.md">build parameters</a> associated with this build. Highlights parameters whose values changed during the build.
</def>

<def title="PerfMon">
Shows hardware data related to this build: CPU usage statistics, memory consumption, and others. This page is available for build configuration with the <a href="performance-monitor.md">Performance Monitor</a> feature (included by default for all new configurations created via existing VCS connections).
</def>

<def title="Dependencies">
Available for builds that are parts of a <a href="build-chain.md">build chain</a>. Allows you to view this entire chain and navigate to associated builds from other configurations.
</def>

<def title="Maven Build Info">
Displays Maven-specific build details that can be useful for build engineers when adjusting build configurations.
</def>
</deflist>

## Statuses and Icons

Success: The build completed successfully.
Failure: The build failed due to an error. Investigate the build log and tests to identify the cause.
Running: The build is currently in progress.
Queued: The build is waiting for available resources to start.

## Working with Build Failures

* Build Log: The primary tool for diagnosing build failures. Examine the log for error messages and stack traces.

* Tests: Check the "Tests" tab for failing tests, which can often point to the source of the problem.

* Changes: Review the changes included in the build to see if any recent code modifications might be responsible for the failure.

* Reporting Issues: If you can't resolve the issue, report it to the development team, providing details from the build log, tests, and changes.

## My Builds and Investigations

This page provides a personalized overview of your builds.  You can see the status of your running and recently finished builds, as well as any investigations assigned to you.  It's a great way to keep track of your work in TeamCity.

## Notifications

You can configure TeamCity to notify you about build events via email or other methods. This allows you to stay informed about the status of important builds without constantly checking the UI. Check with your TeamCity administrator about setting up notifications.

This guide covers the essential features of TeamCity for end-users. By following these steps, you can effectively monitor builds, investigate failures, and utilize build results. If you have further questions, please consult your team's TeamCity administrator or refer to the full TeamCity documentation.