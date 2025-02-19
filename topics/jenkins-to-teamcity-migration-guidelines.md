[//]: # (title: Jenkins to TeamCity Migration Guidelines)
[//]: # (auxiliary-id: Jenkins to TeamCity Migration Guidelines)

## Introduction
{instance="tc"}

This document provides the basics you need to know when migrating from Jenkins to a TeamCity CI server. TeamCity is quite different from Jenkins in terms of concepts related to managing CI and CD. To learn more about Continuous Delivery and get tips on building a production pipeline, follow our [CI/CD Guide](https://www.jetbrains.com/teamcity/ci-cd-guide/). If you want to jump in and get started with TeamCity right away, see the [Getting Started](getting-started-with-teamcity.md) instructions.

## Introduction
{instance="tcc"}

This document provides the basics you need to know when migrating from Jenkins to a TeamCity CI server. TeamCity is quite different from Jenkins in terms of concepts related to managing CI and CD. If you just want to jump in and get started, try to follow [this guide](getting-started-with-teamcity-cloud.md).


## Concepts

Jenkins and TeamCity mostly feature the same set of concepts, however, with slightly different naming. The following table provides a mapping for some of the Jenkins concepts to the TeamCity counterparts.

<snippet id="jenkins-mapping-to-teamcity">

<table><tr>

<td>

<b>Jenkins</b>

</td>

<td>

<b>TeamCity</b>

</td></tr><tr>

<td>

Jenkins Master/Node

</td>

<td>

TeamCity server

</td></tr><tr>

<td>

Dumb Slave / Permanent Agent

</td>

<td>

[Agent Pool](configuring-agent-pools.md)

</td></tr><tr>

<td>

Executor

</td>

<td>

[TeamCity Agent](build-agent.md)

</td></tr><tr>

<td>

View or [Folder](https://plugins.jenkins.io/cloudbees-folder)

</td>

<td>

[Project](project.md)

</td></tr><tr>

<td>

Job/Item/Project

</td>

<td>

[Build configuration](managing-builds.md)

</td></tr><tr>

<td>

Build

</td>

<td>

[Build Steps](configuring-build-steps.md)

</td></tr><tr>

<td>

Pre Step

</td>

<td>

[Build Steps](configuring-build-steps.md) (partially)

</td></tr><tr>

<td>

Post\-build Action

</td>

<td>

[Build Steps](configuring-build-steps.md)   (partially)

</td></tr><tr>

<td>

Build Triggers

</td>

<td>

[Build Triggers](configuring-build-triggers.md)

</td></tr><tr>

<td>

Source Control Management (SCM)

</td>

<td>

[Version Control System (VCS)](vcs-root.md)

</td></tr><tr>

<td>

Workspace

</td>

<td>

[Build Checkout Directory](build-checkout-directory.md)

</td></tr><tr>

<td>

Pipeline

</td>

<td>

[Build Chain](build-chain.md) (via snapshot dependencies)

</td></tr><tr>

<td>

Label

</td>

<td>

[Agent Requirements](configuring-agent-requirements.md)

</td></tr></table>

</snippet>

## Migrating Freestyle project

A Freestyle project is the most common project type in Jenkins so we describe the TeamCity counterparts for a Freestyle project to guide the migration.

### Project and Build Configuration

There are some conceptual differences in how the build jobs are configured in Jenkins and TeamCity.

[Build Configuration](managing-builds.md) is the TeamCitys counterpart of Jenkins Job/Item/Project. However, a Build Configuration requires a [Project](project.md) instance to be created first. In fact, the notion of Project in TeamCity is the first big difference that a user encounters when migrating from Jenkins. The Project contains a good portion of settings required for the build configurations. All settings that are assigned to a Project in TeamCity are listed [here](project.md).

### VCS Roots

Source Code Management (SCM) of a Jenkins build job corresponds to a [VCS root](vcs-root.md) in Project Settings in TeamCity. A Project may include an arbitrary number of VCS Roots. Any Build Configuration may use any number of VCS Roots of the Project.

When only a part of the VCS Root is required, it is possible to use [VCS Checkout Rules](vcs-checkout-rules.md). This allows mapping the directories from the configured VCS Root to subdirectories in the [build checkout directory](build-checkout-directory.md) on a Build Agent.

### Build Environment

The Build Environment section of the Freestyle project in Jenkins specifies additional features for the project. In TeamCity, the corresponding features may be configured in various sections, depending on the goal: general project settings, build configuration settings, VCS root settings, and so on.

Examples:

* To clean the workspace before the build, enable the [Clean build](configuring-vcs-settings.md#Checkout+Settings) checkbox in Version Control Settings of a Build Configuration.
* To cancel the build if the process is stuck, configure [failure conditions](build-failure-conditions.md) of a Build Configuration. What is more, TeamCity provides [hanging build detection](configuring-general-settings.md#Build+Options) out of the box.
* Environment variables can be accessed by specifying a special `%env.<environment variable name>%` pattern (example: `%\env.JAVA_HOME%`) in various places of the configuration.

### Triggers

A Freestyle project allows configuring optional triggers to control when Jenkins will start the builds. In most cases, a trigger in a Jenkins project will be a counterpart in TeamCity.

A [number of options](configuring-build-triggers.md) for triggering builds in TeamCity are available. It is possible to watch for changes in a source control repository, monitor an external resource, or even a Maven dependency. It is possible to schedule builds periodically, besides, the [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html) is available to trigger builds externally.

### Build

This is where the real work happens. This part is mostly identical in Jenkins and TeamCity. TeamCity provides a large number of [build runners](build-runner.md) out of the box. You can [configure several build steps](configuring-build-steps.md) to run the required tasks for the given Build Configuration.

### Building Maven project

A Jenkins Maven project doesnt have a direct counterpart in TeamCity. Instead, an appropriate build step is selected to perform the work. In fact, [TeamCity automates](maven.md) a lot of the setup for Maven projects and provides an additional build triggering [option based on dependencies](configuring-maven-triggers.md#Maven+Artifact+Dependency+Trigger).

### Post-build Actions

Post\-build Actions of the Freestyle project can be mapped either to the specific attributes of a Project/Build Configuration in TeamCity or by using an additional build step. You may also find a relevant solution to the task by adding [Build features](adding-build-features.md) to the Build Configuration in TeamCity.

An individual build step may be [configured to execute](configuring-build-steps.md) depending on the success or failure of the previous build step(s) of the same Build Configuration.

### Working with branches

TeamCity provides support for the [Feature Branches](working-with-feature-branches.md) in distributed version control systems (DVCS).

The support for Feature Branches integrates with the various TeamCity functions. For instance, the [Branch Remote Run Trigger](branch-remote-run-trigger.md) automatically starts a new personal build each time TeamCity detects changes in particular branches of the VCS roots of the build configuration.

Heres a list of some of the highlights related to Feature Branches in TeamCity:
* The build branch name can be specified in an artifact dependency
* The VCS, Schedule, and Finish build triggers support the branch filter setting
* VCS labeling also supports the branch filter
* Mercurial bookmarks can be used in the branch specification, so, if you prefer to use bookmarks instead of standard Mercurial branches, you can fully use the TeamCity feature branches with them
* Git tags can be used in the branch specification. Some teams use Git tags the same way as branches, i.e., once a new tag is set, a build should be started in TeamCity in a branch designated by the tag name.
* [Automatic Merge](automatic-merge.md) functionality to merge a branch into another after a successful build. The functionality is available as a [Build feature](adding-build-features.md) option in Build Configuration settings.

### Plugins

TeamCity comes with a lot of features built\-in by default, and can be further extended by installing [additional plugins](https://plugins.jetbrains.com/teamcity).

  

### Build pipelines

A build pipeline in TeamCity is called a [Build Chain](build-chain.md). It is a set of Build Configurations connected via [snapshot dependencies](snapshot-dependencies.md). The [TeamCity Take on Build Pipelines](https://blog.jetbrains.com/teamcity/2016/03/teamcity-take-on-build-pipelines/) article describes in detail how TeamCity handles build chains and what the implications are.

### Distributed builds

In Jenkins, to offload the master node, a permanent agent is configured to run builds.

TeamCity server doesnt run any builds itself. Instead, it always delegates the job to a [build agent](build-agent.md), which means in TeamCity builds are distributed by design.

The active agents count is visible at the top of the TeamCity server UI:

<img src="agentCount.png" width="300"/>

You can break the agents into separate groups called [agent pools](configuring-agent-pools.md) and assign those to the specific projects. In Build Configuration settings, it is possible to specify a number of [Agent Requirements](configuring-agent-requirements.md) needed for the build.

