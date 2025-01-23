[//]: # (title: Build Chain)
[//]: # (auxiliary-id: Build Chain)

A _build chain_ is a sequence of builds interconnected by [snapshot dependencies](dependent-build.md#Snapshot+Dependency). Sometimes the build chain is called a "pipeline". Parts of a build chain linked with snapshot dependencies with enabled revisions synchronization use the same snapshot of the sources.

>See our **video guide** on how to [compose a pipeline in TeamCity](https://www.youtube.com/watch?v=p4kCMOehrqs).

## Common Use Case

The most common use case for specifying a build chain is running the same test suite of your project on different platforms. For example, before a _release build_ you want to make sure the tests are run correctly under different platforms and environments. For this purpose, you can instruct TeamCity to run tests, then an integration build first, and a release build after that.

Let's see how the build chain mechanism works in details. On triggering a dependent build of the _Release_ build configuration, TeamCity does the following:
1. Resolves a chain of all build configurations that the _Release_ build configuration snapshot depends on.
2. Checks for changes for all dependent build configurations and synchronizes them when the first build in the build chain enters a build queue.
3. Adds all the builds that need building with specific revisions to the build queue.

## Configuring Build Chains

To specify dependencies in your build configuration:

1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Dependencies"/></include>
3. Click the __Add new snapshot dependency__ button.

See also [Build Dependencies Setup](build-dependencies-setup.md) for details and an example.

## Stopping/Removing From Queue Builds from Build Chain

If a build being stopped or removed from the build queue is a part of a build chain, there is a message below the comment field: "_This build is a part of a build chain_".

If there are other running or queued parts of the build chain (i.e. other running builds or queued builds, which are connected with the build under the action), these builds will be listed below under the label "_Stop other parts_".

If a user has access rights to stop a build in the list, there is a checkbox near it. The checkbox is selected by default if stopping the current build will definitely cause the build in the list to fail (for instance, if the listed build depends on the original build being stopped).

If user has no access right to stop a build from the list, the checkbox is not visible.

Selecting the checkbox marks the selected build for a stop/removal from the queue.

If a user has no access right to view a build which is a part of the build chain, this build is not visible to the user at all. If there is at least one such build, there is a warning displayed: "_You don't have access rights to all its parts_". The stripe is shown right under the message "_This build is a part of a build chain_".

In case when all other parts of the build chain cannot be viewed by the current user, we show a yellow stripe with warning: "_You don't have access rights to see its other parts_".

If there are no running or queued builds for the build chain (i.e. all other parts of the build chain have finished), no additional information is shown.

## Disabling Revisions Synchronization Between Chain Parts

You can [disable revisions synchronization](snapshot-dependencies.md#enforce-rev-sync) for a snapshot dependency of a build configuration when promoting a build.   
This option works if you promote a build from chain part 1 to chain part 2, and the first build configuration of part 2 has this option disabled. In this case, TeamCity can use different sources revisions for builds in part 1 and part 2. See the build setup example in [Build Dependencies Setup](build-dependencies-setup.md#Turned+off+Enforced+Revisions+Synchronization).

This is useful when you need to run a dependent build without synchronizing its code revision with its dependencies (preceding builds in a chain).   
For example, you can promote an older build to a [deployment build configuration](deployment-build-configuration.md), and this build will be run using the latest deployment scripts.  
In load/acceptance testing, when you store tests in a version control system and often change them to test your system, you do not need to rebuild your application entirely; instead, you can pick up the chain directly from the testing phase.


## Build Chains Visual Representation

Basically, each build chain is a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph), that is it cannot have cycles.

Build Chains are visible in various places in the TeamCity web UI:

* [Dependencies page of build configuration settings](#Dependencies+page+of+build+configuration+settings)
* [Build Chains tab of the Project Home and Build Configuration Home pages](#Build+Chains+tab+of+Project+Home+and+Build+Configuration+Home+pages)

### Dependencies page of build configuration settings

If a build configuration is a part of a build chain, the corresponding information is displayed in __Build Configuration Settings | Dependencies | Snapshot Dependencies__. Clicking the build chain link opens the preview of the build chain and its configuration in a separate window. The preview shows builds of the chain; the builds with automatic triggering configured are marked with ths icon: ![v.png](v.png).

<img src="snapshotDepPreview.png" alt="Preview of snapshot dependencies"/>

<note>

If build configurations in a chain use feature branches, the [logical names](working-with-feature-branches.md#Logical+Branch+Name) of the branches are also represented as build labels on the chain graph. Note that these names [may differ](working-with-feature-branches.md#Dependencies) from the VCS roots actually used for the builds: if a build uses a default branch but depends on a build that uses a non-default branch, the label of the dependency build will reflect this. For example, if build A uses the default branch "master" but depends on build B that uses branch "v1.1", both builds will be labeled with "v1.1".

</note>

### Build Chains tab of Project Home and Build Configuration Home pages

You can review build chains on both project and build configuration pages: each of those pages has a __Build Chains__ tab appearing when snapshot dependencies are configured. 

The tab displays the list of build chains that contain builds of this project or this build configuration with the ability to filter them. Note that build chains are sorted so that the build chain with the last finished build appears at the top of the list.

The pie-chart icon displays the ratio of the statuses for builds that are parts of the chains. On hovering over the pie chart, the details are displayed:

<img src="buildChainsCollapsed.png" alt="Collapsed build chains"/>

When a chain is expanded, the following information is also available:

   * all builds this build chain is comprised of
   * status of these builds: not triggered, in the queue, running or finished and its details
   * the chain displays builds in order of actual execution, that is builds that start first are on the left.

Clicking a build in a chain highlights the selected build and all its direct dependencies. This page:

   * provides a compact representation of chains: if several top builds triggered the same chain of dependent builds, TeamCity displays one build chain with several top builds.
   * has additional display options: Group by projects and Hide details.
   * transitively highlights all the downstream/upstream builds when a build is selected in a build chain.

From this page you can also:
   * Continue a chain, if there are yet "not triggered" builds. Click __Run__ and a new build will be started on the chain revisions and associated with builds from this chain.
   * Click ![custom_build(1).png](custom_build_1.png)  to open the [custom build dialog](running-custom-build.md) with build chain revisions preselected. This action can be used if you want to rerun some build in the chain.

### Dependencies tab of build results page

If dependencies are configured, you can view their details on the build results page, the __Dependencies__ tab. This tab also displays indirect dependencies, for example, if a build A depends on a build B which depends on builds C and D, then these builds C and D are indirect dependencies for build A.   
The tab also displays artifacts downloaded and delivered by the builds of the chain. It also allows grouping/ungrouping builds and highlighting the builds reused from previous chains ([suitable builds](snapshot-dependencies.md#Suitable+Builds)).


## Partial Chain Execution

When two build configurations are linked by a snapshot dependency, the dependent build always requires the finished other build before starting. However, sometimes only part of the build chain needs to run. This section explains how to skip unnecessary chain configurations and run only the required ones.

### Run Custom Build Dialog

If you rarely need to run a chain portion, you can leave your chain configuration as use the [Run Custom Build dialog](running-custom-build.md) whenever you need to run a portion of a chain. Switch to the **Dependencies** tab of a dialog and choose the "Skip" option for a build configuration you want to ignore.

<img src="dk-skip-chain-builds.png" width="706" alt="Skip builds"/>

Note that the dialog allows you to skip only those build configurations that are directly linked to the configuration you are about to run. For example, starting "Build 4" of the "Build 1 &rarr; Build 2 &rarr; Build 3 &rarr; Build 4" chain allows you to skip only "Build 3". You cannot trigger "Build 4" and tell TeamCity to ignore any configurations preceding "Build 3".

### Build Parameters

Starting with version 2024.12, TeamCity supports the `teamcity.build.chain.skipTags` and `teamcity.build.chain.onlyTags` [configuration parameters](configuring-build-parameters.md). These parameters allow you to configure conditional build dependencies.

* The `teamcity.build.chain.skipTags` parameter explicitly skips unwanted configurations. The chain will run only non-excluded builds.
* The `teamcity.build.chain.onlyTags` parameter specifies configurations you want to run. However, this does not limit the build chain to the mentioned configurations only.
    * If `onlyTags` references configurations that have their own dependencies, these dependencies will be run as well.
    * Configurations in between those mentioned in `onlyTags` parameter cannot be skipped.

#### Parameter Values

Both parameters accept comma-separated tags and configuration IDs as values.

* Tags are values of the `teamcity.configuration.tags` parameter. Use this parameter to label any configuration and set the same value in `skipTags`/`onlyTags`. Multiple tag values may be separated with spaces, commas, or semicolons.
* [Configuration IDs](identifier.md) are shown in configuration settings, below the configuration name.

    <img src="dk-get-build-conf-id.png" width="706" alt="Build configuration ID"/>

    You can also copy configuration IDs from URLs of configuration-related pages. These IDs typically have the `projectName_configurationName` format. For example, the "https://foo.com/buildConfiguration/Glacier_TestWin/2554" URL corresponds to the build results page, where "Glacier_TestWin" is a configuration ID and "2554" is the internal build number.

#### Parameter Priority

When a chain (or its portion) is triggered, TeamCity reads `skipTags` and `onlyTags` parameters from the topmost configuration only. Parameters from upstream builds are ignored.

The following build chain illustrates three sample configurations linked in a single "BuildA &rarr; BuildB &rarr; BuildC" chain.

```Kotlin
// Configuration A
object BuildA : BuildType({
    id = AbsoluteId("BuildA")
    params {
        param("teamcity.configuration.tags", "extra")
    }
})
// Configuration B
object BuildB : BuildType({
    id = AbsoluteId("BuildB")
    params {
        param("teamcity.configuration.tags", "extra")
        param("teamcity.build.chain.skipTags", "extra")
    }

    dependencies { snapshot(BuildA) {} }
})
// Configuration C
object BuildC : BuildType({
    id = AbsoluteId("BuildC")
    name = "BuildC"
    dependencies { snapshot(BuildB) {} }
})
```

If you start a new "Configuration B" build, it will run as a solo build, skipping "Configuration A". However, starting "Configuration C" will trigger the entire "BuildA &rarr; BuildB &rarr; BuildC" chain.

#### Artifact Resolution

If you skip a build that [provides an artifact](artifact-dependencies.md) to another build, this downstream build will fail because no artifact was delivered. To prevent it from happening, TeamCity dynamically replaces strict artifact dependencies with [optional ones](artifact-dependencies.md#Optional+dependency) (`?:build.jar => build/libs/` instead of `+:build.jar => build/libs/`).

#### Build Reuse for Partial Builds

If a build skipped some of its upstream configurations, TeamCity considers it to be a build with modified dependencies. For that reason, further chain runs will not [reuse](snapshot-dependencies.md#Suitable+Builds) this build and instead trigger a new one.

#### Example 1: Skip Tags

The following chain includes multiple building and testing configurations with the "Build All" [composite configuration](composite-build-configuration.md) triggering the entire chain.

<img src="dk-skip-builds-full-chain.png" width="706" alt="Full chain"/>

Running the entire chain can be time- and resource-consuming. To allow "Build All" to run only core "Build..." configurations, add the `teamcity.build.chain.skipTags` parameter using either tags or IDs as a value. Both methods are identical, choose the one that most suites your needs.

<procedure title="skipTags parameter value">

<tabs>

<tab title="Configuration tags">


Add the <code>teamcity.configuration.tags</code> parameter to each configuration you want to skip. Set these parameters to any string you want, and use the same string as the <code>teamcity.build.chain.skipTags</code> parameter.

The <a href="kotlin-dsl.md">Kotlin</a> snippet below uses the "optional" tag for unwanted configurations.

<br/>

<code-block lang="kotlin">
object TestWin : BuildType({
    id("TestWin")
    params {
        param("teamcity.configuration.tags", "optional")
    }
})
object TestAndroid : BuildType({
    id("TestAndroid")
    params {
        param("teamcity.configuration.tags", "optional")
    }
})
// ...
// Other optional build configurations
// ...
object TeamcityGatedPrDemo_BuildAll : BuildType({
    id("BuildAll")
    type = BuildTypeSettings.Type.COMPOSITE
    params {
        param("teamcity.build.chain.skipTags", "optional")
    }
    dependencies {
        snapshot(BuildLinux_iOS) {}
        snapshot(BuildPlugins) {}
        snapshot(BuildPortable) {}
        snapshot(BuildWin) {}
    }
})
</code-block>

</tab>

<tab title="Configuration IDs">

Instead of marking each unwanted configuration with a tag, you can use this configuration ID as the <code>teamcity.build.chain.skipTags</code> parameter value.

<br/>

<code-block lang="kotlin">
object TestWin : BuildType({
    id("TestWin")
})
object TestAndroid : BuildType({
    id("TestAndroid")
})
// ...
// Other build configurations
// ...
object TeamcityGatedPrDemo_BuildAll : BuildType({
    id("BuildAll")
    type = BuildTypeSettings.Type.COMPOSITE
    params {
        param("teamcity.build.chain.skipTags", "TestAndroid,TestIOS,TestLinux,TestPS,TestWin,BuildPlugins")
    }
    dependencies {
        snapshot(BuildLinux_iOS) {}
        snapshot(BuildPlugins) {}
        snapshot(BuildPortable) {}
        snapshot(BuildWin) {}
    }
})
</code-block>

<br/>

</tab>

</tabs>

</procedure>


To run the entire chain, remove the `teamcity.build.chain.skipTags` parameter or set it to any value that does not match configuration tags or IDs. For example, add a [schedule trigger](configuring-schedule-triggers.md) that will run full nightly builds with configurations omitted during shortened daily builds. TeamCity trigger settings include the **Build Customization** section that allows you to add or modify build parameters.

```Kotlin
object TeamcityGatedPrDemo_BuildAll : BuildType({
    id("BuildAll")
    type = BuildTypeSettings.Type.COMPOSITE
    params {
        param("teamcity.build.chain.skipTags", "optional")
    }
    triggers {
        schedule {
            // Full "Build All" chain running daily at 3 AM
            schedulingPolicy = daily { hour = 3 }
            triggerBuild = always()
            withPendingChangesOnly = false
            buildParams {
                param("teamcity.build.chain.skipTags", "nightly-build-mode")
            }
        }
    }
})
```


#### Example 2: Only Tags

This example utilizes the same chain as in Example 1:

<img src="dk-skip-builds-full-chain.png" width="706" alt="Full chain"/>

Each individual configuration is tagged using the `teamcity.configuration.tags` parameter. For your convenience, the [Kotlin](kotlin-dsl.md) configuration of this chain is categorized into tabs.

<tabs>

<tab title="Windows">

```Kotlin
object TestWin : BuildType({
    id = AbsoluteId("TestWin")
    params { param("teamcity.configuration.tags", "tests,windows") }
})

object BuildWin : BuildType({
    id = AbsoluteId("BuildWin")
    params { param("teamcity.configuration.tags", "build,windows") }
    dependencies {
        snapshot(TestWin) {}
    }
})
```

</tab>

<tab title="Linux">

```Kotlin
object TestLinux : BuildType({
    id = AbsoluteId("TestLinux")
    params { param("teamcity.configuration.tags", "tests,linux") }
})

// Shared configuration for Linux and iOS
object BuildLinux_iOS : BuildType({
    id = AbsoluteId("BuildLinux_iOS")
    params { param("teamcity.configuration.tags", "build,linux,ios") }
    dependencies {
        snapshot(TestIOS) {}
        snapshot(TestLinux) {}
    }
})
```

</tab>


<tab title="iOS">

```Kotlin
object TestIOS : BuildType({
    id = AbsoluteId("TestIOS")
    params { param("teamcity.configuration.tags", "tests,ios") }
})

// Shared configuration for Linux and iOS
object BuildLinux_iOS : BuildType({
    id = AbsoluteId("BuildLinux_iOS")
    params { param("teamcity.configuration.tags", "build,linux,ios") }
    dependencies {
        snapshot(TestIOS) {}
        snapshot(TestLinux) {}
    }
})
```

</tab>

<tab title="Android and Playstation">

```Kotlin
object TestAndroid : BuildType({
    id = AbsoluteId("TestAndroid")
    params { param("teamcity.configuration.tags", "tests,portable") }
})

object TestPS : BuildType({
    id = AbsoluteId("TestPS")
    params { param("teamcity.configuration.tags", "tests,portable" }
})

object BuildPortable : BuildType({
    id = AbsoluteId("BuildPortable")
    params { param("teamcity.configuration.tags", "build,portable") }
    dependencies {
        snapshot(TestAndroid) {}
        snapshot(TestPS) {}
    }
})
```

</tab>

<tab title="Plugins">

```Kotlin
object BuildPlugins : BuildType({
    id = AbsoluteId("BuildPlugins")
    params { param("teamcity.configuration.tags", "build,plugins") }
})
```

</tab>

<tab title="Composite 'BuildAll'">

```Kotlin
object TeamcityGatedPrDemo_BuildAll : BuildType({
    id("BuildAll")
    type = BuildTypeSettings.Type.COMPOSITE
    params {
        select("teamcity.build.chain.onlyTags","",
            label = "Choose a sub-chain to run",description = """
            * run all — runs the entire chain
            * desktop — runs "Build (Win)" and "Build (Linux/iOS)" builds with their related tests
            * portable — runs the "Build (Portable)" build with its related Android/PS tests
            * windows — runs the "Test (Win) > Build (Win)" sequence
            * linux — runs the "Test (Linux) > Build (Linux/iOS)" sequence
            * ios  — runs the "Test (iOS) > Build (Linux/iOS)" sequence
            * plugins — runs the solo "Build Plugins" configuration
        """.trimIndent(),
        display = ParameterDisplay.PROMPT,
        options = listOf(
            "run all" to "",
            "plugins",
            "desktop" to "windows,linux,ios",
            "portable" to "BuildPortable",
            "windows",
            "linux",
            "ios"))
    }

    dependencies {
        snapshot(BuildLinux_iOS) {}
        snapshot(BuildPlugins) {}
        snapshot(BuildPortable) {}
        snapshot(BuildWin) {}
    }
})
```

</tab>

</tabs>


The composite "Build All" configuration [styles](typed-parameters.md) the `teamcity.build.chain.onlyTags` parameter as a "Select" parameter with the "Prompt" behavior mode. Whenever the chain is manually triggered, the **Run Custom Build** dialog pops up, allowing users to choose which part of a chain they want to launch.

<img src="dk-subchain-selector.png" width="706" alt="Subchain selector"/>

* run all — the label mapped to an empty string. This is the initial/default value of the `onlyTags` parameter. Runs all nine configurations of this chain.
* plugins — triggers the "Build Plugins" configuration. This configuration will run alone since there are no other configurations with the "plugins" tag, and it has no dependencies on other builds.
* desktop — runs a sub-chain of five build/test configurations that have any of the "windows", "linux", and "ios" tags.
* portable — explicitly points to "BuildPortable" configuration by its ID. This configuration depends on "TestAndroid" and "TestPS", so all three configurations will run. Identical to using the "portable" tag as the parameter value.
* windows/linux/ios — parameter values mapped to identical tags. Trigger short platform-specific "Test &rarr; Build" sub-chains. Note that "linux" and "ios" sub-chains are identical, since triggering the shared "BuildLinux_iOS" configuration forces both linked "TestLinux" and "TestIOS" configurations to run.

The list of available values does **NOT** have these options:

* build — this value will trigger all "Build ..." configurations directly, which in turn will start their linked "Test ..." configurations. As a result, the entire chain will run.
* tests — this value can confuse users since you cannot trigger only leftmost "Test ..." and rightmost "Build All" configurations, leaving "Build ..." configurations intact. This happens because TeamCity cannot skip configurations in the middle of a chain, since it is then unclear how to handle dependencies traversing configurations which never run. 


### Service Message

Send the `##teamcity[skipQueuedBuilds tags='value1,value2,...' comment='Your comment']` [service message](service-messages.md) from any build step of an upstream build configuration to cancel queued builds of downstream configurations. Same as with `skipTags` / `onlyTags` [parameters](#Build+Parameters), the `tags` argument of this message accepts:

* values of `teamcity.configuration.tags` parameters added to configurations;
* configuration IDs.

Use service messages to cancel out certain "limbs" of a build chain. For example, you may want to send the `skipQueuedBuilds` from the "Build" configuration to cancel individual tests suites. With some of the tests running and some of them skipped, the integrity of the "Build → Test → Deploy" chain remains intact.


```Shell
Build ----|---- Test Suite 1 ----|
          |---- Test Suite 2 ----|
          |---- Test Suite 3 ----|
          |---- Test Suite 4 ----|
          |---- Test Suite 5 ----|---- Deploy
```

If you skip all of these tests, the entire mid-section of this chain will be gone. This will produce a potentially confusing "Build → ??? → Deploy" chain. To avoid this, consider [cloning](copy-move-delete-build-configuration.md) configurations and making a separate "Build → Deploy" chain instead.


 <seealso>
        <category ref="concepts">
            <a href="dependent-build.md">Dependent Build</a>
        </category>
        <category ref="admin-guide">
            <a href="configuring-dependencies.md">Configuring Dependencies</a>
            <a href="build-dependencies-setup.md">Build Dependencies Setup</a>
        </category>
</seealso>
