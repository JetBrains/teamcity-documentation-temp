[//]: # (title: Configuring Agent Requirements)
[//]: # (auxiliary-id: Configuring Agent Requirements)

**Agent requirements** are conditions that specify which agents can run your build configuration. To view all currently existing requirements and create new ones, as well as check which of your agents can run the specific configuration, go to **[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes) | Agent Requirements**.


## Agent Requirements Video Guide

<video src="https://youtu.be/5gaoW8IfNJA"
title="TeamCity tutorial — Agent Requirements"/>



## Requirement Syntax

Any agent requirement is the `parameter operator [value]` expression, where:

* `parameter` is either a predefined or a custom (user-defined) [build parameter](configuring-build-parameters.md). For example, the `teamcity.agent.jvm.os.name` parameter that reports which operating system is installed on the agent. Note that since requirements should identify whether an agent can run builds for this specific build configuration, you should only use parameters whose values can differ depending on the agent. For example, the `teamcity.serverUrl` parameter reports the same value for any agent and is useless for defining agent requirements.

* `operator` is a keyword that defines how to treat the `value` part. You can choose between "equals", "starts with", "is not more than", and other operators. See this article for the complete list: [](requirement-conditions.md).

* `value` is compared with the actual value reported by the parameter. If it fits the specified criteria, the entire expression returns `true`, which means this agent is compatible with (can run builds of) this specific configuration. Certain operators do not require values, for example the `exists` operator checks whether the parameter value is not `null`. The `matches` and `does not match` operators allow you to use regular expressions for comparing values.

## Explicit Requirements

**Explicit requirements** are requirements manually configured in the **Agent Requirements** section of build configuration settings. To define an explicit requirement, do the following:

You can add custom agent requirements in both TeamCity UI and in [](kotlin-dsl.md).

1. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Agent Requirements"/></include> This page displays which agents are currently eligible to run your build configuration based on which building routines it performs.

   <img src="dk-agentrequirements-tab.png" width="706" alt="Agent Requirements Tab"/>

2. Click the <b>Add new requirement</b> button.

3. Choose a parameter, a condition, and enter a value to create a new expression. Since TeamCity scans parameters reported by agents, it shows suggestions as you type parameter names and values.

   <img src="dk-agentrequirements-addnew.png" width="706" alt="Add new requirement"/>

4. Click <b>Save</b>. Your list of compatible and incompatible agents will be updated. If TeamCity is unable to find agents that satisfy your new condition, a corresponding warning is displayed next to the requirement. Running such build configuration results in the build waiting in the [queue](working-with-build-queue.md) with the "There are no idle compatible agents which can run this build" status.

   <img src="dk-invalid-agent-requirement.png" width="706" alt="No compatible agents"/>


<deflist>
<def title="Kotlin DSL">

In [](kotlin-dsl.md), add a new expressions to the `requirements` block of your build configuration to create an explicit requirement.

```Kotlin
object MyBuildConfig : BuildType({
requirements {
      exists("DotNetCoreSDK5.0_Path")
      startsWith("teamcity.agent.jvm.os.name", "Windows")
   }
})
```
See also: [Requirements | Kotlin DSL Documentation](https://www.jetbrains.com/help/teamcity/kotlin-dsl-documentation/root/requirements/index.html)
</def>
</deflist>


 

To temporarily disable or permanently remove a requirement, click the drop-down menu button next to **Edit**.

<img src="dk-remove-or-disable-requirement.png" width="706" alt="Remove or disable requirement"/>


> You can check which values different agents return for the given parameter before crafting an agent requirement. To do so, navigate to **Agents | Parameters Report** and enter a required parameter name.
> 
> <img src="dk-check-param-values.png" width="706" alt="Check parameter values"/>
> 
{style="tip"}


## Inherited Requirements

If an agent requirement is declared in a [](build-configuration-template.md), the requirement is also in effect for all configurations based on this template. The **Agent Requirements** page for these configuration marks such requirements as "inherited".

<img src="dk-inherited-requirement.png" width="706" alt="Inherited requirement"/>

Project administrators can edit, disable, and remove inherited requirements unless they are a part of an [enforced settings template](build-configuration-template.md#Enforcing+settings+inherited+from+template).


## Build Step Requirements

**Build steps requirements** are requirements that were not manually created by a project administrator. They arise automatically based on what configuration steps are configured to do.

For example, if one of your steps is configured to [run inside a container](container-wrapper.md), an agent assigned to process this build must have a container engine (Docker or Podman). For that reason, TeamCity automatically adds the `docker.server.osType exists` parameter. Agents that do not report this parameter will not be assigned to new builds of this configuration.

## Implicit Requirements

Similarly to build steps requirements, **implicit requirements** are added automatically by TeamCity based on build step configuration. However, these requirements focus exclusively on [parameter references](configuring-build-parameters.md#Parameter+References).

For example, if a configuration has the `echo "%\myParam%"` [command-line build step](command-line.md), it is implied the `myParam` parameter exists and has value. Otherwise, if a parameter is missing or empty, there is nothing to print out. In this case, TeamCity shows a corresponding warning on the **Parameters** tab...

<img src="dk-implicit-requirement-parameters-tab.png" width="706" alt="Implicit requirement: Parameters tab"/>

...and displays a corresponding entry on the **Agent Requirements** tab.

<img src="dk-implicit-requirement-requirements-tab.png" width="706" alt="Implicit requirement: Requirements tab"/>

In other words, an implicit requirement can be interpreted as "a configuration uses an unknown parameter" warning. To resolve this issue, you need to either provide a parameter value on a configuration/project level, or define this parameter inside a build agent [`buildAgent.properties`](configure-agent-installation.md) file.

```Plain Text
## TeamCity build agent configuration file

######################################
#   Required Agent Properties        #
######################################

<...>

myParam="Hello world"
```

If at least one agent reports this parameter, it will be listed as compatible and the implicit requirement will be hidden from the **Agent Requirements** tab.

## Individual Step Requirements

Along with configuration-wide requirements, you can define requirements to agents for individual build steps. To do so, open step settings and add a [parametrized execution condition](build-step-execution-conditions.md).


## Combining Conditions

<include from="requirement-conditions.md" element-id="combining-conditions"/>



 <seealso>
        <category ref="concepts">
            <a href="configuring-agent-requirements.md">TeamCity Data Backup</a>
            <a href="build-agent.md">Build Agent</a>
            <a href="managing-builds.md">Build Configuration</a>
        </category>
</seealso>