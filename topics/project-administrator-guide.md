# Project Administrator Guide

This section focuses on **Project administration**. To learn more about other aspects of TeamCity, refer to the [](server-administrator-guide.md) and [](user-guide.md) sections.


## Steps, Configurations and Projects

In TeamCity, a building routine consists of the following blocks:

<img src="dk-basic-tree-diagram3.png" alt="TeamCity elements" width="706"/>

* **Build step** — an essential building block that executes a predefined set of commands. This can be a single command (like `dotnet test` or `gradle clean build`) or a series of operations (such as a custom Python or Bash script). Build steps run fully, with no partial execution.

* **Build configuration** — a sequence of build steps executed in a specific order. With a configuration, you can:

    * arrange steps in any order you need;
    * temporarily disable individual steps.
    * set [conditions](build-step-execution-conditions.md) for when steps should be executed. If a condition is not met, the corresponding step is skipped. For instance, step #3 could be set to run only if step #2 fails, and step #4 might be configured to execute only on Windows agents.

    Configurations can also be turned into [templates](build-configuration-template.md) making it easy to clone and reuse them without manually configuring each new setup. Once cloned, each copy can be customized independently.

    You can also incorporate configurations from the same or different projects into one [unified workflow](build-chain.md).


* **Project** — a collection of independent build configurations. These configurations can be run separately but are grouped under a single project to share common resources: [connections](configuring-connections.md), [parameters](configuring-build-parameters.md), [artifact storages](configuring-artifacts-storage.md), [cloud agent profiles](agent-cloud-profile.md), and so on.

    Each project can be owned by another project for the same benefits: subprojects can access entities owned by their parent projects. The topmost project, called the **Root project**, is created automatically by TeamCity. This Root project cannot be removed and is ideal for setting up globally accessible parameters, connections, cloud profiles, and other shared resources.


## Basic CI Workflow in TeamCity

The following diagram illustrates the basic TeamCity workflow:

<img src="cicd-flow.png" width="711" alt="Basic CI flow with TeamCity"/>

1. The TeamCity server detects a change in your repository. This happens in one of the following ways:
    * Default setup: every 60 seconds TeamCity automatically [polls](teamcity-configuration-and-maintenance.md#Version+Control+Settings) all VCS repositories targeted by its projects.
    * Webhooks: if [a post-commit hook](configuring-vcs-post-commit-hooks-for-teamcity.md) is configured, any commit sends this webhook to TeamCity. This approach has two major benefits: decreased server load since the server no longer periodically polls the repo, and shortened feedback loop (the server gets notified about changes immediately after a change was introduced).
        > For building GitHub-hosted projects, you can utilize [](github-checks-trigger.md) that does not require manually configuring post-commit hooks.
2. The server writes this change to the database.
3. A [trigger](configuring-build-triggers.md) attached to the build configuration detects the relevant change in the database and initiates a build.
4. The triggered build appears in the [build queue](working-with-build-queue.md).
5. The build is assigned to a free compatible build agent.
6. The agent executes the build steps, described in the build configuration. While executing the steps, the agent reports the build progress to the TeamCity server. It sends all the log messages, test reports, code coverage results on the fly, so you can monitor the build process in real time.
7. After finishing the build, the agent sends [build artifacts](build-artifact.md) to the server.


## VCS Roots

A **VCS root** is a cornerstone of the TeamCity &larr;&rarr; VCS repository communication. This integral element implements repository access required to perform a wide range of operations: repository checkout, code sources tagging, communicating build statuses back to VCS, and so on.

VCS roots store the following information:

* Fetch and push URLs that TeamCity uses to pull and push remote files.
* Branch information: the list of repository branches TeamCity should track and which branch is the default (main) one.
* Authentication settings: credentials TeamCity uses to access a repo.
* Checkout settings: specify how remote files should be stored and whether submodules should be checked out along with the main repository.
* Custom changes polling settings that allow you to override the default 60-second interval.

Sections related to VCS roots are available in both project and configuration settings.

<img src="dk-roots-in-projects-and-configs-v.png" alt="Root settings in projects and configs" width="706"/>

However, configurations never own roots. You can "attach" a VCS root to a configuration, but roots are always stored in (owned by) projects. This technique results in the following:

* A VCS root can be attached to multiple configurations, meaning that multiple build configurations can access the same repository with the same auth and checkout settings.
* A single configuration may have multiple VCS roots attached, which allows you to work with different repositories within one configuration.
* Editing VCS roots affects all configurations that use it. When modifying VCS root settings, you have an option to duplicate this root and store updated settings in this new clone, keeping the original root unchanged. This allows you to customize one build configuration but leave other configurations that share this root unaffected.

Although a VCS root is an existential part of any build configuration that works with a remote repository, in many scenarios TeamCity generates roots automatically and does not require that you create them by hand for each new build configuration. See [this tutorial](configure-and-run-your-first-build.md) for an example.


## Artifacts

Artifacts are files produced during a build. These files are available to:

* TeamCity users, who can download them from the [](build-results-page.md).
* Other TeamCity builds, who can import these files using [](artifact-dependencies.md).

To choose which files should be available as build artifacts:

1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="General Settings"/></include>
3. Set up **Artifact paths** property. You can first run a build that produces required files. Then, you will be able to click the "Select files from the latest build" button and choose files from the drop-down menu instead of manually entering their paths.

Related article: [](build-artifact.md)


## Build Chains

A build chain is a sequence of build configurations. Key features of a build chain include:

* Configurations linked in a single chain can belong to the same or different TeamCity projects.
* Linked configurations can share artifacts. In other words, artifacts produced by one configuration can be imported and used by configurations running further down the chain.
* Configurations use right-to-left dependency mechanism. In a sample "Build &rarr; Test &rarr; Deploy" build chain:
    * "Build" can be run as a solo configuration and does not trigger any parts of the chain.
    * "Test" depends on "Build" and requires "Build" to finish before it can start.
    * "Deploy" depends on "Test", which in turn depends on "Build". Triggering the deployment configuration will result in the entire chain running.
    > TeamCity also supports left-to-right dependencies where builds of one configuration trigger new builds of another configuration. See [](configuring-finish-build-trigger.md) article for more information.
* Build chains can reuse previous successful builds to avoid redundant operations. For example, if the "Build &rarr; Deploy" chain has successfully finished and no new commits were made, further "Deploy" runs will reuse that last successful "Build" results. This behavior is customizable, you can allow mission-critical dependent configurations to force fresh builds every time.
* TeamCity allows you to create snapshot and artifact dependencies.
    <deflist type="medium">
    <def title="Snapshot dependency">
    Defines a relation between two build configurations. If "Configuration B" has a snapshot dependency to "Configuration A", new "B" builds require previously finished "A" builds, and have an ability to trigger a new "A" build if no suitable one exists.
    </def>
    <def title="Artifact dependency">
    Imports artifacts produced by a target configuration build into the current build. Can be used with or without a snapshot dependency. If used solo, the dependent build has no ability to force a new build of the upstream configuration in case there is no suitable one. For that reason, you may want to set up artifact dependencies to target pinned/tagged builds. This setup can exhibit more control on your building routine: team developers inspect build "A", verify it is stable, and pin it. New builds "B" then use an artifact dependency to utilize this pinned build instead of newest (unverified) "A" builds.
    </def>
    </deflist>
    Since artifact dependencies do not establish explicit relationships between configurations, only configurations connected through snapshot dependencies are considered part of a build chain.

Related article: [](configuring-dependencies.md)

## Tutorial: Create Your First Project in TeamCity

The [](configure-and-run-your-first-build.md) tutorial walks you through the main stages of setting up a TeamCity project: establishing a VCS connection, setting up build actions, configuring additional features, assigning users to this project, and more.