[//]: # (title: Scopes, Priority, and Lifecycle of Build Parameters)
[//]: # (auxiliary-id: Scopes, Priority, and Lifecycle of Build Parameters;Levels and Priority of Build Parameters;Project and Agent Level Build Parameters)


## Initial Parameter Values

TeamCity parameters can obtain their values from one or multiple sources listed below.

* Values from a template selected as the [enforced settings template](build-configuration-template.md#Enforcing+settings+inherited+from+template). These values cannot be disabled or overridden by users.

* The **Parameters** tab of the [Run Custom Build](running-custom-build.md) dialog.

    <img src="dk-params-runcustombuild.png" width="706" alt="Run Custom Build Dialog"/>

* Custom values entered on the **Build Configuration Settings | Parameters** page.

* Custom values entered on the **Project Settings | Parameters** page. Parameters defined within a project are inherited by all its subprojects and build configurations. If required, you can override them in individual build configurations.

* Values specified in a regular [build configuration template](build-configuration-template.md).

* Values specified in a build agent's [configuration file](configure-agent-installation.md) (the `<AGENT_HOME>/conf/buildAgent.properties` file). For example, the following sample demonstrates how to implement a custom build agents' ranking system that you can use in [agent requirements](configuring-agent-requirements.md):

  <img src="dk-params-agentTiers.png" width="706" alt="Custom Agent Ranking system"/>

  <br/>

  ```Kotlin
  // _Root project config
  object Project : Project({
      description = "Contains all other projects"
      params {
        param("agent.tier", "")
      }
  })
  ```

  ```
  # An agent's "buildAgent.properties" files
  
  ######################################
  #   Default Build Properties         #
  ######################################
  # ...
  agent.tier=Platinum
  # ...
  ```


* Values reported by an agent when it connects to the TeamCity server. These values are passed to parameters that describe the agent environment. For example, the `DotNetCoreSDK7.0_Path` parameter that stores the path to .NET 7 SDK on this specific agent.

* Values of predefined build parameters. These parameters can collect their values on a server side in the scope of a specific build (for example, the `build.number` parameter), or on the agent side right before a build starts (for example, the `teamcity.agent.work.dir.freeSpaceMb` parameter).



## Parameters' Priority


The list above also ranges parameter value sources by priority, from highest to lowest. That is, if the same parameter retrieves different values from different sources, a value from the topmost source in this list is applied. For example, if the `my.parameter` is defined inside an agent configuration file and on a build configuration settings page in TeamCity UI, the value from the configuration settings page wins.


<!--
## Initial Values Files

When a build process starts, TeamCity saves all parameters with their values to files that you can access via corresponding parameters. These files use the [Java Properties File format](https://docs.oracle.com/cd/E23095_01/Platform.93/ATGProgGuide/html/s0204propertiesfileformat01.html) (for example, special characters are backslash-escaped).


* System parameters are stored to a file whose path is stored in the `system.teamcity.build.properties.file` parameter. Parameters in this file are stored without their "system." prefix.

* Configuration parameters are written to a file that can be accessed via the `system.teamcity.configuration.properties.file` parameter.

The following sample script for the [](python.md) runner prints the contents of both files to the build log:

```Python
print("System properties:\r\r")
with open('%\system.teamcity.build.properties.file%', 'r') as sp:
    print(sp.read())

print("Configuration properties:\r\r")
with open('%\system.teamcity.configuration.properties.file%', 'r') as cp:
    print(cp.read())
```
-->




## Changing Parameter Values During a Build

TeamCity parameters can change their values as a build progresses through its stages. This can happen automatically, due to the nature of the value reported by a parameter, or in response to operations performed during a build.

For example, the `teamcity.agent.work.dir.freeSpaceMb` parameter that reports the available agent disk space changes its value as builds checkout new source files and generate new artifacts and logs.

If you need to manually change a TeamCity parameter from inside a build step, send the following [service message](service-messages.md):

```Shell
echo "##teamcity[setParameter name='myParam1' value='TeamCity Agent %\teamcity.agent.name%']"
```

<include from="configuring-build-parameters.md" element-id="change-parameter-from-build"/>


> Note that using the `setParameter` service message overrides the parameter value only in the scope of the current build or build chain. To **permanently** override a parameter value, send the REST API request from your build step like shown below:
> 
> ```Kotlin
> import jetbrains.buildServer.configs.kotlin.*
> import jetbrains.buildServer.configs.kotlin.buildSteps.script
>
> object UpdateBuildVersion : BuildType({
> name = "Update Build Version"
> 
>     steps {
>         script {
>             id = "simpleRunner"
>             scriptContent = """
>                 version=%\build.version%
>                 ((version=version+1))
>                 
>                 curl --location --request PUT 'http://<server_URL>/app/rest/projects/<project_name>/parameters/build.version' \
>                 --header 'Accept: */*' \
>                 --header 'Content-Type: text/plain' \
>                 --header 'Authorization: Bearer your_token' \
>                 --data ${'$'}version
>             """.trimIndent()
>         }
>     }
> })
> ```
>
{style="warning"}


## Checking Parameter Values


### In TeamCity UI

To check current values of agent parameters, navigate to **Agents | Parameters report** and enter the property whose value you need to check.

<img src="dk-params-checkparamsonagents.png" width="706" alt="Check the specific on agents"/>

You can also click any build agent to open the agent details page, and switch to the **Parameters** tab to view all parameters reported by this specific agent.

<img src="dk-params-allParamsOnAgent.png" width="706" alt="Check all parameters on an agent"/>


To view which values parameters had during a specific build, open this build's [results page](build-results-page.md) and switch to the **Parameters** tab.

<snippet id="build-results-parameters-tab">

<img src="dk-params-newAndUpdated.png" width="706" alt="Build parameters report"/>

This page has two tabs:

* **Parameters** — lists values for all configuration parameters, system properties, and environment variables. You can tick a related checkbox to view only those parameters that changed their values during this build.

* **Statistic values** — lists all [statistics values](custom-chart.md#Default+Statistics+Values+Provided+by+TeamCity) reported for the build (for example, build success rate or time required to check out a remote repository). The *View Chart* button (<img src="dk-viewChart.png" width="12" alt="View Chart"/>) allows you to check how these values trend throughout build runs.

</snippet>


### Using REST API

To check initial and actual parameter values of the specific build via [REST API](teamcity-rest-api.md), send GET requests to the `/app/rest/builds/[{buildLocator}](https://www.jetbrains.com/help/teamcity/rest/buildlocator.html)` endpoint and specify required payload fields according to the [Build schema](https://www.jetbrains.com/help/teamcity/rest/build.html).


* `/app/rest/builds/{buildLocator}?fields=originalProperties(*)` — returns user-defined parameters from the build configuration and their default values.
* `/app/rest/builds/{buildLocator}?fields=startProperties(*)` — returns all parameters reported by an agent and their values at the time the build started.
* `/app/rest/builds/{buildLocator}?fields=resultingProperties(*)` — returns all parameters reported by an agent and their values by the time the build finished.

You can also check initial and final values of the specific parameter. To do this, specify the name of the target parameter.

For example, if you run a custom build for the sample project illustrated in the [](#Changing+Parameter+Values+During+a+Build) section, send the following query to check the `day.of.week` parameter values.

```Shell
curl -L \
  https:<SERVER_URL>/app/rest/builds/<BUILD_LOCATOR>?fields=\
    originalProperties($locator(name:(value:(day.of.week),matchType:matches)),property),\
    startProperties($locator(name:(value:(day.of.week),matchType:matches)),property),\
    resultingProperties($locator(name:(value:(day.of.week),matchType:matches)),property)
```

If this build runs on Wednesday and you pass Sunday as the `day.of.week` parameter value via the **Run custom build** dialog, the response payload will contain the following values:

* `originalProperties` returns Monday (the default value stored in the build configuration).
* `startProperties` returns Sunday (the value from the **Run custom build** dialog, has a priority over the default value from the build configuration).
* `resultingProperties` returns Wednesday (the value calculated during the build and written via the service message).









<seealso>
        <category ref="admin-guide">
            <a href="configuring-build-parameters.md">Configuring Build Parameters</a>
            <a href="using-build-parameters.md">Using Build Parameters</a>
            <a href="predefined-build-parameters.md">List of Predefined Build Parameters</a>
        </category>
</seealso>
