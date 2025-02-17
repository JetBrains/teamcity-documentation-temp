[//]: # (title: Project Home Page)
[//]: # (auxiliary-id: Project Home Page)

This article gives an overview of the __Project Home__ page of the new TeamCity UI. Most of its features are also available in the classic UI mode.

The __Project Home__ page allows viewing the main information about the builds of the current project and its subprojects. It also gives quick access to most popular actions for managing builds.

This page has the following tabs:
* __Overview__: gives a preview of the project's nested subprojects and build configurations. It has two main view modes: __Builds__ (a hierarchy of build lists) and __Trends__ (interactive graphs to preview the builds' trends and apply quick actions). When viewing trends, you can hover over any chart bar to instantly see more information about the respective build: its duration, queue statistics, test results, used agent, and more.  
  This tab also allows you to quickly create nested projects down the hierarchy and configure favorite projects.
* __Investigations__: shows active [problem investigations](investigating-and-muting-build-failures.md) in the current project.
* __Change Log__: shows a detailed visual change log of the current project.
* __Statistics__: shows [custom statistics charts](custom-chart.md) of the current project.
* __Problems__: shows the actual [build failures and failed tests](investigating-and-muting-build-failures.md) of the current project.
* __Build Chains__: if the current project has [build chains](build-chain.md), shows their details and graphs (currently, this page uses the classic UI approach for visualization).
* __Flaky Tests__: shows [flaky tests](investigating-and-muting-build-failures.md#Flaky+Tests) of the current project.

From __Project Home__, you can switch to other view modes:
* To switch to __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__, use the __Edit project__ link in the upper right corner. To access a particular section of the settings, open the context menu to the right of this link.
* To open __Build Configuration Home__ of a particular nested build configuration, click its name in the list or its trends card.
* To open __Build Results__ of a particular build, expand its build configuration in a list and click the build's number. Or, when in the __Trends__ mode, click the respective bar on a chart. Note that only a limited number of most recent builds are displayed in the __Project Home__ mode. To see more builds, open the __Build Configuration Home__ view.

