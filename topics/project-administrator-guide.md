# Project Administrator Guide

This section focuses on **Project administration**. To learn more about other aspects of TeamCity, refer to the [](server-administrator-guide.md) and [](user-guide.md) sections.



## Steps, Configurations and Projects



In TeamCity, a building routine consists of the following blocks:

<img src="dk-basic-tree-diagram3.png" alt="TeamCity elements" width="706"/>

* <snippet id="build-step-overview">**Build step** — an essential building block that executes a predefined set of commands. This can be a single command (like `dotnet test` or `gradle clean build`) or a series of operations (such as a custom Python or Bash script). Build steps run fully, with no partial execution.</snippet>

* <snippet id="build-configuration-overview">**Build configuration** — a sequence of build steps executed in a specific order.</snippet> With a configuration, you can:

    * arrange steps in any order you need;
    * temporarily disable individual steps.
    * set [conditions](build-step-execution-conditions.md) for when steps should be executed. If a condition is not met, the corresponding step is skipped. For instance, step #3 could be set to run only if step #2 fails, and step #4 might be configured to execute only on Windows agents.

    Configurations can also be turned into [templates](build-configuration-template.md) making it easy to clone and reuse them without manually configuring each new setup. Once cloned, each copy can be customized independently.

    You can also incorporate configurations from the same or different projects into one [unified workflow](build-chain.md).


* <snippet id="project-overview">**Project** — a collection of independent build configurations. These configurations can be run separately but are grouped under a single project to share common resources: [connections](configuring-connections.md), [parameters](configuring-build-parameters.md), [artifact storages](configuring-artifacts-storage.md), [cloud agent profiles](agent-cloud-profile.md), and so on.</snippet>

    Each project can be owned by another project for the same benefits: subprojects can access entities owned by their parent projects. The topmost project, called the **Root project**, is created automatically by TeamCity. This Root project cannot be removed and is ideal for setting up globally accessible parameters, connections, cloud profiles, and other shared resources.




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


## Working with Branches

TeamCity allows you to set up different building rules for different branches. For example, you might want to build the "production" branch whenever a new change appears, the "development" branch nightly, and ignore the "sandbox" branch. To do this, you need to specify branch specs and filters.

### Branch Specifications

Branch specifications are VCS root settings that specify which repository branches are tracked by this project. This is the entry point for any branch-related operation, individual elements like build triggers cannot work with branches excluded from branch specs.

<snippet id="common-branch-spec-syntax">

To set up branch specifications, open general VCS root settings and scroll down to the **Branch specification** field. Each specification is a new line that starts with `+:` or `-:` to include or exclude a specific branch, followed by the fully resolved branch name. The `+:` part can be omitted.

<deflist type="wide">
<def title="+:refs/heads/development">
Tracks the "development" branch
</def>
<def title="-:refs/heads/sandbox">
Ignores the "sandbox" branch
</def>
</deflist>

</snippet>


<snippet id="branch-spec-wildcard">

The `*` wildcard allows you to reference multiple branches with similar names:

<deflist type="wide">
<def title="refs/heads/*">
The default rule that tracks all existing repository feature branches.
</def>
<def title="refs/heads/dev-*">
Tracks feature branches whose names start with "dev-": "dev-2024.2", "dev-2025.1", and others.
</def>
</deflist>

</snippet>

Related article: [](working-with-feature-branches.md).

### Branch Filters

Branch filters are available for many TeamCity entities: triggers, build features, clean-up rules, and so on. These filters specify which of the branches specified in branch specification rules are available to this entity. For example, using branch filters of a [VCS Trigger](configuring-vcs-triggers.md) you can choose which branches should automatically start builds on new changes.

Branch filters use the same `+|-:BRANCH_NAME` syntax as branch specification rules, with two notable exceptions:

* certain entities accept only fully resolved branch names (`refs/heads/main`) whereas other support logical names as well (`main`);
* the additional `+|-pr:` syntax allows you to [filter Git pull request branches](branch-filter.md#Pull+Request+Branch+Filters).

Related article: [](branch-filter.md)


## Collecting Changes

Once your project is set up, TeamCity can receive information about all new changes committed to repository branches included in [branch specifications](#Branch+Specifications). New changes notifications are received in one of the following ways:



<deflist>
<def title="Periodic repository polling">

By default, every 60 seconds TeamCity automatically polls all VCS repositories targeted by its projects. The downsides of this approach are:

<ul>
<li>Inefficiency — TeamCity keeps constantly polling all repositories, even those that rarely have new changes.</li>
<li>Performance — If your TeamCity server has a large amount of projects, periodic polling can produce a significant load.</li>
<li>Delays — After a user committed their change, they can wait up to a minute for this change to show up on TeamCity configuration's <b>Pending changes</b> tab. Users can also use the <b>Actions</b> configuration menu to manually trigger the process:
<img src="dk-check-for-pending.png" width="706" alt="Check for new changes"/>
</li>
</ul>

To change the polling interval, navigate to <a href="teamcity-configuration-and-maintenance.md#Version+Control+Settings">general TeamCity server settings</a>. To override this value for individual configurations, edit the VCS root <a href="configuring-vcs-roots.md">minimum polling interval</a> setting.
</def>

<def title="Webhooks">

Webhooks allow VCS hosting providers to notify TeamCity about new changes. Compared to the default polling mechanism, this behavior has the following advantages:

* Efficiency — TeamCity server receives change notifications when these changes appear.
* Speed — Change notifications arrive as soon as they are committed.

On the downside, webhooks require manual setup for each repository/project.

<include from="configuring-vcs-post-commit-hooks-for-teamcity.md" element-id="polling-with-hooks"/>

Related article: <a href="configuring-vcs-post-commit-hooks-for-teamcity.md">Configuring VCS Post-Commit Hooks</a>

</def>
</deflist>


## Start Builds Automatically

TeamCity users can trigger new builds at any moment via the **Run** button in the configuration's top right corner. To start new builds automatically, you need to configure [triggers](configuring-build-triggers.md).

<snippet id="triggers-pa-guide">TeamCity offers various triggers to start new builds based on different events, such as time-based triggers for scheduled builds, change-based triggers for new commits, triggers that launch builds upon the completion of other configurations, and so on.</snippet>

Related article: [](configuring-build-triggers.md)



## Artifacts

Artifacts are files produced during a build. These files are available to:

* TeamCity users, who can download them from the [](build-results-page.md).
* Other TeamCity builds, who can import these files using [](artifact-dependencies.md).

To choose which files should be available as build artifacts:

1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="General Settings"/></include>
3. Set up **Artifact paths** property. You can first run a build that produces required files. Then, you will be able to click the "Select files from the latest build" button and choose files from the drop-down menu instead of manually entering their paths.

Related article: [](build-artifact.md)


## Set Up Cross-Configuration Dependencies

Real-life CI/CD pipelines often combine multiple standalone configurations. For example, "Build", "Test", and "Deploy to Staging" configurations (or Jobs) can run independently or in sequence.

TeamCity offers multiple options to create relations between standalone configurations.

<deflist>
<def title="Build Chain">

A build chain is a collection of classic TeamCity configurations interconnected using <a href="snapshot-dependencies.md">snapshot dependencies</a>.

Snapshot dependencies are right-to-left relations. For example, in the "A -> B" chain where configuration "B" has a dependency on configuration "A", "B" cannot run until "A" produces a suitable build first. The criteria for "suitable" builds depends on your setup, see the <a href="snapshot-dependencies.md#Suitable+Builds">Suitable Builds</a> section for more information. At the same time, "A" can run independently without triggering new "B" builds.

For mission-critical scenarios, you can set up dependent configurations to always force fresh upstream configuration builds, even if there were no recent changes to the project.
</def>

<def title="Finish Build Triggers">

<a href="configuring-finish-build-trigger.md">Finish build triggers</a> establish left-to-right relations. For example, you can create a similar "A -> B" sequence similar to a build chain, but with one key difference: "B" can run independently, while each new "A" build automatically triggers a new "B" build.

Finish build triggers offer a simple but inflexible way to trigger downstream builds and can often be replaced or complemented by snapshot dependencies.
</def>


<def title="Artifact Dependencies">

<a href="artifact-dependencies.md">Artifact dependencies</a> allow configurations to import files produced during other configurations' builds. For example, a "Delivery" configuration can deploy files (Docker images, NuGet packages, HTML documentation pages, and so on) produced by a "Build" configuration to a designated resource.

Artifact dependencies don’t create explicit links between configurations: both can run independently without triggering each other’s builds. If you use artifact dependencies without corresponding snapshot dependencies, a dependent build has no ability to ensure a suitable source of artifacts (an upstream configuration build) exists. For that reason, you may want to set up artifact dependencies to target pinned/tagged builds. This setup can exhibit more control on your building routine.
</def>

<def title="Pipelines">

In the lightweight TeamCity Pipelines mode, you create Pipelines instead of classic TeamCity projects, and Jobs instead of build configurations. By default, each Job of a pipeline is an separate entity that runs in parallel to other Jobs. To unite these Jobs in a sequence, use the Job settings panel or drag-and-drop Jobs in the visual editor. For each cross-Job dependency, you can additionally specify whether an upstream Job should share its artifacts to the downstream one.

Pipeline dependencies substitute classic TeamCity artifact and snapshot dependencies, but have fewer options (for example, you cannot choose to reuse only pinned/tagged builds of an upstream Job).
</def>

</deflist>

## Summary: Basic TeamCity Workflow

The following diagram illustrates the basic TeamCity workflow:

<img src="cicd-flow.png" width="711" alt="Basic CI flow with TeamCity"/>

1. The TeamCity server [detects a change in your repository](#Collecting+Changes).
2. The server writes this change to the database.
3. A [trigger](configuring-build-triggers.md) attached to the build configuration detects the relevant change in the database and initiates a build.
4. The triggered build appears in the [build queue](working-with-build-queue.md).
5. The build is assigned to a free compatible build agent.
6. The agent executes the build steps, described in the build configuration. While executing the steps, the agent reports the build progress to the TeamCity server. It sends all the log messages, test reports, code coverage results on the fly, so you can monitor the build process in real time.
7. After finishing the build, the agent sends [build artifacts](build-artifact.md) to the server.

## Tutorial: Create Your First Project in TeamCity

The [](configure-and-run-your-first-build.md) tutorial walks you through the main stages of setting up a TeamCity project: establishing a VCS connection, setting up build actions, configuring additional features, assigning users to this project, and more.