# User Guide

This section focuses on daily TeamCity operations: running builds, investigating problems, configuring notifications, and more.


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

The drop-down menu below the configuration name allows you to view builds of specific branch only (by default, **&lt;All branches&gt;** is selected).

> If a required branch is missing from this drop-down menu, ask your project administrator to check whether the [branch specification](project-administrator-guide.md#Branch+Specifications) of this configuration is correctly configured.
> 
{style="note"}

The **Branches** / **Builds** toggle of the main configuration page allows you to choose how to view builds: as a flat list or grouped by branches. A default branch is highlighted with a slightly darker background.

> In TeamCity, the [default branch](working-with-feature-branches.md#Default+Branch) depends on configuration settings (more specifically, on attached VCS root's settings). This is not necessarily the same default/main branch as in your VCS repository. The default branch is used whenever you click **Run** without selecting a specific branch first.
> 
> When a new release cycle begins, you can ask a project administrator to edit VCS root settings to switch the default branch.
> 
{style="note"}


## Icons and Colors

<include from="build-state.md" element-id="color-coding"/>

Related articles: [](build-state.md), [](change-state.md)

## Run a New Build

Depending on configuration triggers, new builds can start automatically. <include from="project-administrator-guide.md" element-id="triggers-pa-guide"/>

This section outlines how to trigger builds manually.

### Default Builds

To start a new build with default settings:

1. Open a required configuration.
2. (optional) Select a required branch in the drop-down menu below a configuration name.
3. Click **Run** in the top right corner of a screen. If you skipped step #2, TeamCity will run a build for the configuration's default branch.

<img src="dk-run-button.png" width="706" alt="Run button"/>

If any of the agents eligible to run this build is immediately available, the build will start right away. Otherwise, it will be [queued](working-with-build-queue.md).

### Custom Builds

The ellipsis button (...) next to **Run** invokes the [Run Custom Build](running-custom-build.md) dialog that allows you to trigger builds with custom settings: modified parameters, elevated build queue priority, specific changelist, and so on.

For configurations that use parameters with unspecified values, this dialog pops up whenever you click the **Run** button.

Related article: [](running-custom-build.md)

### Personal Builds

<include from="personal-build.md" element-id="personal-build-overview"/>

Related article: [](personal-build.md)


## View Build Results

<include from="build-results-page.md" element-id="build-results-intro"/>

Related article: [](build-results-page.md)



## Working with Build Failures

* [Build log](build-log.md) — The primary tool for diagnosing build failures. Examine the log for error messages and stack traces.

* [Tests](build-results-page.md#Tests+Tab) — Check the "Tests" tab for failing tests, which can often point to the source of the problem.

* [Changes](build-results-page.md#Changes+Tab) — Review the changes included in the build to see if any recent code modifications might be responsible for the failure.



## Investigations and Mutes

<include from="project-administrator-guide.md" element-id="investigations-and-mutes"/>

To view currently active investigations assigned to you, click the **My investigations** button on a sidebar.

<img src="dk-investigation-counter.png" width="706" alt="Investigations counter"/>


## Notifications

You can configure TeamCity to notify you about build events via email or other methods. This allows you to stay informed about the status of important builds without constantly checking the UI. Check with your TeamCity administrator about setting up notifications.

This guide covers the essential features of TeamCity for end-users. By following these steps, you can effectively monitor builds, investigate failures, and utilize build results. If you have further questions, please consult your team's TeamCity administrator or refer to the full TeamCity documentation.