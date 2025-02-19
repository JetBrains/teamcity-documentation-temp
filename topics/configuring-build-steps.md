[//]: # (title: Configuring Build Steps)
[//]: # (auxiliary-id: Configuring Build Steps)

Once you have a TeamCity project with a build configuration within it, you can configure **build steps**. A build step (also referred to as a "job") is a task to be executed by a [build runner](build-runner.md). One build configuration can include multiple steps, executed consecutively.

Build steps are configured in the __Build Steps__ section of the __[Build Configuration Settings](creating-and-editing-build-configurations.md)__ page. This page allows you to:

* Add new steps manually.
* [Autodetect steps](#Autodetecting+Build+Steps) by scanning the source VCS repository.
* Copy and delete steps.
* Temporarily enable/disable steps.

## Choosing Build Runner for Your Task

Each build step is represented by a [build runner](build-runner.md) and provides integration with a specific build or test tool. For example, calling a NAnt script before compiling VS solutions. You can add as many build steps to your build configuration as needed.

This video tutorial explains how to choose a build runner based on your project's needs:

<video src="https://youtu.be/wLmLgh5OK5o"
title="TeamCity - How to use specific runners to supercharge your builds"/>

## Build Steps Execution

Build steps are invoked sequentially.

The decision whether to run the next build step may depend on the exit status of the previous build steps, the current build status, or [execution conditions](build-step-execution-conditions.md).

The build step is considered _failed_ if (1) the build process returned a non-zero exit code and (2) the "_Fail build if build process exit code is not zero_" build failure condition is enabled (see [Build Failure Conditions](build-failure-conditions.md)); otherwise, the build step is considered _successful_.

Note that the status of a build step and the build itself can be different. All build steps can be successful, but the build can fail because of another build failure condition — not based on the exit code (like failing a test). On the other hand, if a build step has failed, the build will fail too.

For the details on configuring individual build steps, refer to the respective pages inside this section.

### Execution Policy

You can specify the step execution policy via the __Execute step__ option:

* __Only if build status is successful__ — before starting the step, the build agent requests the build status from the server, and skips the step if the status is "failed". This considers the failure conditions processed by the server, like failure on test failures or on metric change. Note that this still may be not exact as some failure conditions are processed on the server asynchronously ([TW-17015](https://youtrack.jetbrains.com/issue/TW-17015)).

* **Only if build status is failed** — same as above, but the agent skips the step if the build status is "success". This condition allows you to add steps that are executed only when a build fails. For example, you can run tasks that rollback the latest changes to the remote repository that introduced the issue.

* __If all previous steps finished successfully__ — the build analyzes only the build step status on the build agent, and doesn't send a request to the server to check the build status and considers only important step failures.

* __Even if some of the previous steps failed__ — select to make TeamCity execute this step regardless of the status of previous steps and status of the build.

* __Always, even if build stop command was issued__ — select to ensure this step is always executed, even if the build was canceled by a user. For example, if you have two steps with this option configured, stopping the build during the first step execution will interrupt this step, while the second step will still run. Issuing the stop command for the second time will result in ignoring the execution policy: the build will be terminated.

<tip>

__Tips:__

* You can copy a build step from one build configuration to another from the original build configuration settings page.
* You can reorder build steps as needed. Note that if you have a build configuration inherited from a template, you cannot reorder inherited build steps. However, you can insert custom build steps (not inherited) at any place and in any order, even before or between inherited build steps. Inherited build steps can be reordered in the original template only.
* You can disable a build step temporarily or permanently, even if it is inherited from a build configuration template, using the corresponding option in the last column of the __Build Steps__ list.

</tip>

<anchor name="execution-conditions"/>

Since TeamCity 2020.1, you can also add granular [execution conditions](build-step-execution-conditions.md) for build steps.

## Bootstrap Steps
{instance="tc"}

A bootstrap step is performed right after a build is triggered, before a build agent checks out source files. This allows you to add a runner (typically the [](command-line.md) one) that performs initial setup and ensures the following steps run as expected.

To enable bootstrap steps, first add the `teamcity.internal.bootstrap.steps.enabled=true` entry either to [](server-startup-properties.md#TeamCity+Internal+Properties) or to the individual project (as this project's [configuration parameter](configuring-build-parameters.md)). This setting allows you to do the following:

* Enable the **Run during bootstrap** option in step settings.
  <img src="dk_bootstrap_step.png" width="706" alt="Bootstrap step" style="block"/>

* On the configuration's **Build Steps** page, click **Reorder build steps** and drag the required step before the "Preparation stage" block.
  <img src="dk_bootstrap_step_drag.png" width="706" alt="Drag and drop bootstrap step" style="block"/>



You can create a build configuration with a bootstrap step and turn this configuration into a [template](build-configuration-template.md). Such template allows you to quickly create more configurations that perform required prerequisite actions before the actual building process starts.

[Meta-Runners](working-with-meta-runner.md) are considered as a single step and thus cannot be split into parts performed before and after a checkout.

To turn a regular step into a bootstrap one in [](kotlin-dsl.md), add a `teamcity.step.phase` parameter and set its value to "bootstrap".

```Kotlin
object MyConfig : BuildType({
  // ...
  steps {
    script {
      name = "Disk prep"
      id = "simpleRunner"
      scriptContent = "echo 'Initial setup'"
      param("teamcity.step.phase", "bootstrap") // Bootstrap step
    }
    // Regular steps
  }
  // ...
})
```


## Agentless Build Steps

_Agentless build steps_ are steps that can run without a [build agent](install-and-start-teamcity-agents.md) attached, in an external software.

Normally, a running build occupies a build agent until the very finish, even if its final steps are executed outside TeamCity. However, if a build does not need its agent for some remaining steps, it can detach from this agent. The agent becomes available and can be instantly assigned to another build.

This approach allows saving agents' work time and is optimal for configurations that use third-party tools for finalizing a build: for example, to run numerous tests or deploy a project. Such a build can finish outside TeamCity; the TeamCity server will detect its status reports directly, without using an agent as a mediator.

Let's consider an example build that consists of three steps: compilation, testing, and deployment. All of them are processed by an agent even though the deployment step is actually performed by the external software. The agent is just polling the external software and reporting build statuses to the TeamCity server.

<img src="agent-depend-build.png" alt="Regular build" width="556"/>

With the agentless approach, the agent does not need to handle the final deployment step and can run other builds from the queue. The TeamCity server will catch the following reports directly from the external tool.

<img src="agentless-build.png" alt="Build with agentless step" width="518"/>

<video src="https://youtu.be/O7LdxtiBkLY"
title="New in TeamCity 2020.2: Agentless Build Steps"/>

## Autodetecting Build Steps

TeamCity can scan the source VCS repository of a project and autodetect build steps in Node.js, Kotlin, Python, Ant, NAnt, Gradle, Maven, MSBuild, Visual Studio solution files, PowerShell, Xcode project files, Rake, and IntelliJ IDEA projects.

In __Build Steps__, click __Auto-detect build steps__, and then select the proposed steps you want to add to the current build configuration. You can change their settings afterwards.

When scanning the repository, TeamCity progressively searches for project files on the two highest levels of the source tree. When searching, it takes into consideration the extension of the files. For example:
* [PowerShell](powershell.md): `.ps1`
* [.NET](net.md): `project.json`, `.proj`, `.csproj`, `.vbproj`, `.sln`

If the detected steps have already been added to this configuration manually, TeamCity will skip those.


## Step Status Parameters


TeamCity provides `teamcity.build.step.status.<step_ID>` parameters that report the status of the step with the given ID. Step IDs are shown below their names, and are editable only when you create them.

<img src="dk-stepID.png" width="706" alt="Step ID"/>

The available values of the `teamcity.build.step.status.<step_ID>` parameters are:

* `success` — when a step finishes with no errors.
* `failure` — when a step failed. This status is reported even when all build problems were [muted](investigating-and-muting-build-failures.md).
* `cancelled` — when a build was cancelled while this step was running.

The `teamcity.build.step.status.<step_ID>` parameters appear only after their corresponding steps finish, and are not available right from the moment a build starts. This means neither steps that are still running, nor skipped steps have their `teamcity.build.step.status.<step_ID>` parameters available.

You can check all step statuses in the [](build-results-page.md#Parameters+Tab) of the build results page...

<img src="dk-parametersTab-statuses.png" width="706" alt="Step statuses"/>

...or via [TeamCity REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html).

```Shell
http://<SERVER_URL>/app/rest/builds/<BUILD_ID>/resulting-properties
```
{prompt="GET"}

You can utilize these parameters to create custom [](build-step-execution-conditions.md).


 <seealso>
        <category ref="concepts">
            <a href="build-runner.md">Build Runner</a>
        </category>
</seealso>
