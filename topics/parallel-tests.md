[//]: # (title: Parallel Tests)
[//]: # (auxiliary-id: Parallel Tests)

TeamCity is now capable of parallelizing the execution of your tests by distributing them across multiple build agents, thus minimizing the overall duration of tests. The tests of a build can be automatically split into batches, and each batch will run on a separate build agent. 
This feature addresses a popular use case when a build consequently runs many independent tests on the same agent while they could technically be running in parallel, utilizing resources of multiple agents.
Previously, to emulate such behavior, some users would configure several build configurations and join them in a chain with parallel connections. This approach works, but sometimes requires implementing a non-trivial logic of distributing tests across batches. 

In TeamCity 2022.04, the tests' distribution logic is provided by TeamCity itself. In addition, such build runners as [Maven](maven.md), [Gradle](gradle.md), [IntelliJ IDEA Project](intellij-idea-project.md), and [.NET](net.md) are capable of automatic filtering of the tests on the agent without the need to change the settings of build steps.  


## Run tests in parallel

The _Parallel tests_ build feature solves the task of parallel tests execution on different agents.

To enable this feature in an existing build configuration:


1. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Build Features"/></include>
2. Click **Add build feature** and choose the _Parallel tests_ type.
3. Set the number of test batches, which also means the number of parallel agents to use in a build.

> Note: Only tests methods from different classes (or test cases) can run in parallel on different agents.

Before TeamCity splits tests into batches to run in parallel, it needs to gather test statistics of at least one preceding build. This information helps subdivide tests in semi-equally sized batches (based on tests duration) so that the total build time is as short as possible.
If you enable this feature in a freshly added build configuration, its first build will run in the normal mode; when it finishes and produces test reports, TeamCity will be able to split the second one.

TeamCity takes into account not only the latest tests run, but also the history of your tests: the next builds with parallel tests will also contribute to this statistics and allow TeamCity to make smarter decisions. 

### Under the hood

If the test statistics is available, then the following happens when a new build is triggered:

1. TeamCity generates copies of the current build configuration according to the number of batches specified in the build feature. 
These build configurations will have the same build steps as the original one.
In the future, the changes to the build steps of the original configuration will be propagated to the generated ones automatically.
2. The triggered build will be transformed into a [composite build](composite-build-configuration.md) with dependencies on the builds from the generated build configurations.
3. As soon as the first dependency build starts, the composite build will start too.

> The settings of the original build configuration are not affected by the _Parallel tests_ build feature. 
>
>On the other hand, the number and settings of the generated build configurations are fully controlled by the _Parallel tests_ build feature. The generated build configurations are read-only by default and are not intended to be modified manually.
> The generated build configurations are also placed into a subproject that is hidden.
> If the project has [versioned settings](storing-project-settings-in-version-control.md) enabled, the generated build configurations will not be committed to the VCS repository.

A build of a generated build configuration will run the same set of build steps as defined in the original build configuration. If some of these steps are 
of the [Maven](maven.md), [Gradle](gradle.md), [IntelliJ IDEA Project](intellij-idea-project.md), 
or [.NET](net.md) type, and they were executing some tests, then these build runners will automatically run only the fraction of the tests corresponding to the current batch.

>If the original build configuration has deployment steps, these steps will be performed the same number of times as the number of batches.

If you run tests differently, you can still enable the _Parallel tests_ build feature for your configuration
and benefit from automatic test division: you will obtain the information about the tests to execute from [the build parameters](#custom-tests) that will be provided by the build feature.

### Runner-specific requirements
{id='runner-requirements'}

Automatic execution of a batch of tests instead of the whole set of tests is only supported if the following requirements are met:

* [Maven](maven.md)
  * Maven minimal supported version: 3.x.
  * Maven Surefire plugin minimal supported version: 2.13.
  * Maven Failsafe Plugin minimal supported version: 2.13.

* [Gradle](gradle.md)
  * Gradle minimal supported version: 5.0.

* [.NET](net.md)
  * Microsoft.NET.Test.Sdk minimal supported version: 16.0.

### Custom execution of parallelized tests
{id="custom-tests"}

In some cases the tests are executed in such a way that TeamCity cannot affect their execution anyhow. For instance, they can be generated on the fly, or they can be reported by a third-party build runner, or imported from a file, and so on.

In such cases, the user will need to implement custom test execution logic to execute only one batch of tests. The _Parallel tests_ build feature simplifies the task by providing a number of build parameters.
These parameters are:

| Parameter name                                   | Description                                                                                    |
|--------------------------------------------------|------------------------------------------------------------------------------------------------|
| teamcity.build.parallelTests.currentBatch        | Contains the current batch number starting with 1                                              |
| teamcity.build.parallelTests.totalBatches        | Contains the total number of configured batches                                                |
| system.teamcity.build.parallelTests.excludesFile | Contains a path on the agent to a text file with tests which should be excluded from execution |

The `excludesFile` file is generated by the TeamCity server and should not be manually modified. Instead, you should read its contents to implement your own custom test execution logic. This file has the following format:

```
#version=<value>
#algorithm=<value>
#current_batch=<value>
#total_batches=<value>
#suite=<value>
<new line separated list of test classes>
```

<deflist type="narrow">

<def title="version">An integer value that is the file format version. It is implied that the custom tests execution logic checks the version and reports an error or fails a build if the version has an unexpected value.</def>

<def title="algorithm">
The type of the algorithm responsible for breaking down tests into batches. Available values:
<ul>
<li><code>DURATION</code> — prioritizes the execution time of test classes as the primary metric to create batches with approximately equal runtime.</li>
<li><code>SPLIT_SUITE</code> — splits each test class within a test suite into N batches.</li>
<li><code>GROUP_SUITES</code> — same as <code>DURATION</code> but groups all tests from the same suite into a single batch, ignoring individual test classes.</li>
</ul>
</def>

<def title="current_batch">The number of the current batch, same as <code>teamcity.build.parallelTests.currentBatch</code> <a href="configuring-build-parameters.md">parameter</a>.</def>

<def title="total_batches">The total number of batches, same as <code>teamcity.build.parallelTests.totalBatches</code> <a href="configuring-build-parameters.md">parameter</a>.</def>

<def title="suite">The test suite name. This parameter can have an empty value.</def>
</deflist>

Note: Java and .NET test frameworks usually report tests to TeamCity in the following format:

`[<suite name>: ]<fully qualified test class name>.<test method>[<test arguments>]`

Where `<suite name>` and `<test arguments>` are optional and not always present.

For example, the following Java Test class:

```java

package org.example.tests;

class TestCase1 {

  public void testMethod1() { /* ... */ }

  public void testMethod2() { /* ... */ }
}

```

will produce the following test names in TeamCity:

```

org.example.tests.TestCase1.testMethod1
org.example.tests.TestCase1.testMethod2

```

Then the parameter `system.teamcity.build.parallelTests.excludesFile` will point to a text file with the following content:

```

#version=1
#current_batch=1
#total_batches=3
#suite=
org.example.tests.TestCase1

```

> Note: the granularity of tests filtering is per class (or test case), not per test method.

The build step with custom tests' execution logic should use this file and filter out all the tests that belong to the classes mentioned there. All the other tests should be executed.



## Alternative Test Filtering for .NET

<snippet id="alternative-dotnet-parallel-filtering">

If the [](net.md) runner handles a large amount of test classes, parallel testing may produce huge test filters that are hard to parse and consume for test engines like NUnit.

To avoid potential performance issues, TeamCity automatically employs an alternative test filtering mode optimized for these cases. This mode is enabled on a per-agent basis if the following conditions are met:

* An agent that runs a test batch reports .NET CLI (SDK or runtime) 6.0 or newer.
* A test batch has 1000 or more test classes. You can change this threshold value via the `teamcity.internal.dotnet.test.suppression.test.classes.threshold` [configuration parameter](configuring-build-parameters.md).

You can prevent TeamCity from switching to this mode and force it to always run parallel .NET tests using the regular filtering mechanism in any individual project or build configuration. To do so, add the `teamcity.internal.dotnet.test.suppression=false` [parameter](configuring-build-parameters.md) to the required configuration or project.

</snippet>



## Publish Artifacts Produced By Batch Builds

Since parallel tests run inside independent batch builds, [artifacts](build-artifact.md) are produced by these batch builds that perform actual building routines, not by a parent configuration's builds.

However, TeamCity aggregates artifacts produced by batch builds in parent configuration builds to make them easily accessible from a single place.

<img src="dk-artifacts-parallelBuildAggregateDiagram.png" width="706" alt="Aggregated artifacts diagram"/>

When batch builds produce identical artifacts, only the latest batch build's artifacts are shown in the **Artifacts** tab of a parent configuration. This behavior is suitable for configurations with artifacts produced by building steps, but not for parallel tasks generating similar file sets.

To access all parallel task artifacts (for example, when each of your tests generates the "report.log" file), use the `teamcity.build.parallelTests.currentBatch` [parameter](configuring-build-parameters.md) when defining artifact paths for a parent build configuration. Since this parameter references the batch build number, it allows builds to organize their artifacts into subdirectories with unique names. For instance:

```
bin => batch-build-%\teamcity.build.parallelTests.currentBatch%/bin
```

<img src="dk-artifacts-parallelBuildAggregate.png" width="706" alt="Aggregated artifacts"/>


## Known limitations

* The [Code coverage](code-quality-tools.md#code-coverage-tools) statistics will be inaccurate for builds with parallel tests because it will be collected for a fraction of tests executed by the current batch.
* A newly added test which is not yet known to TeamCity will run in each batch during the first run. Related YouTrack ticket: [TW-75913](https://youtrack.jetbrains.com/issue/TW-75913/Newly-added-tests-run-in-all-parallel-auto-generated-dependencies).
* When TeamCity divides tests into batches, it only takes into account the duration of the test itself. The duration of setUp/tearDown or any other preparation methods is not know to TeamCity, therefore the duration of batches may not be equal.
* An agent selected in the custom build dialog for a build with parallel tests will be ignored because the build will be transformed into a composite one after triggering. Related YouTrack ticket: [TW-74905](https://youtrack.jetbrains.com/issue/TW-74905/Select-agent-in-Custom-run-dialog-for-Parallel-Tests-build-has-no-effect).
* Parameters published by the build steps via the [setParameter](service-messages.md#set-parameter) service message, as well as runner specific parameters, such as `maven.project.version`, won't be available in a composite build with parallel tests. Related YouTrack ticket: [TW-75249](https://youtrack.jetbrains.com/issue/TW-75249/Updating-parameter-from-service-message-has-no-effect-for-parallel-tests-builds).
* When it comes to the build configurations limit in the TeamCity Professional version, the automatically generated build configurations are counted as normal build configurations.

### Known bugs
 
* The [Enforce Clean Checkout action](clean-checkout.md#Enforcing+Clean+Checkout) does not work for build configurations with parallel tests configured. Related YouTrack ticket: [TW-75337](https://youtrack.jetbrains.com/issue/TW-75337/Parallel-tests-Enforce-clean-checkout-Action-does-not-affect-auto-generated-configurations).
* A subsequent start of a build with parallel tests won't reuse already existing builds of the generated build configurations even if there were no new VCS commits.
* Batch builds spawned to run [TestNG](https://testng.org/doc/) tests from an [XML suite](https://testng.org/doc/documentation-main.html#testng-xml) run all tests instead of a specified subset of tests. Related YouTrack ticket: [TW-75849](https://youtrack.jetbrains.com/issue/TW-75849/Parallel-tests-All-batches-run-all-tests-from-TestNG-xml-suite).

