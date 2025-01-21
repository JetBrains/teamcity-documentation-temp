[//]: # (title: Configuring Agent Requirements)
[//]: # (auxiliary-id: Configuring Agent Requirements)

[Agent Requirements](agent-requirements.md) are conditions that specify which agents can run your build configuration. Any agent requirement is the `parameter operator [value]` expression, where:

* `parameter` is either a predefined or a custom (user-defined) [build parameter](configuring-build-parameters.md). For example, the `teamcity.agent.jvm.os.name` parameter that reports which operating system is installed on the agent. Note that since requirements should identify whether an agent can run builds for this specific build configuration, you should only use parameters whose values can differ depending on the agent. For example, the `teamcity.serverUrl` parameter reports the same value for any agent and is useless for defining agent requirements.

* `operator` is a keyword that defines how to treat the `value` part. You can choose between "equals", "starts with", "is not more than", and other operators. See this article for the complete list: [](requirement-conditions.md).

* `value` is compared with the actual value reported by the parameter. If it fits the specified criteria, the entire expression returns `true`, which means this agent is compatible with (can run builds of) this specific configuration. Certain operators do not require values, for example the `exists` operator checks whether the parameter value is not `null`. The `matches` and `does not match` operators allow you to use regular expressions for comparing values.

## Build Step Requirements

To ensure your build is never assigned to an agents that cannot run it, TeamCity analyzes all build configurations and lists agents that do not fit configurations' building routines as incompatible.

For example, if one of your steps [runs inside a container](container-wrapper.md), TeamCity filters out agents that have neither Docker nor Podman installed via the `container.agent exists` requirement. Or if one of your steps references a build parameter (for example, a [](command-line.md) script prints the value of `%\my.custom.param%`), TeamCity uses the same `exists` operator to identify agents that do not report this parameter.

## How to Create a Custom Explicit Requirement

You can add custom agent requirements in both TeamCity UI and in [](kotlin-dsl.md).

<table><tr><td>

<tabs>
<tab title="TeamCity UI">

1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Agent Requirements"/></include>. This page displays which agents are <a href="#Build+Step+Requirements">currently eligible</a> to run your build configuration based on which building routines it performs.
   
   <img src="dk-agentrequirements-tab.png" width="706" alt="Agent Requirements Tab"/>

3. Click the **Add new requirement** button.
4. Choose a parameter, a condition, and enter a value to create a new expression. Since TeamCity scans parameters reported by agents, it shows suggestions as you type parameter names and values.

   <img src="dk-agentrequirements-addnew.png" width="706" alt="Add new requirement"/>
5. Click **Save**. Your list of compatible and incompatible agents will be updated. If TeamCity is unable to find agents that satisfy your new condition, a corresponding warning is displayed next to the requirement. Running such build configuration results in the build waiting in the [queue](working-with-build-queue.md) with the "There are no idle compatible agents which can run this build" status.
   
   <img src="dk-invalid-agent-requirement.png" width="706" alt="No compatible agents"/>

To temporarily disable or permanently remove a requirement, click the drop-down menu button next to **Edit**. Note that [implicit requirements](#Build+Step+Requirements) cannot be removed or disabled since they are directly required by the build actions performed in this configuration.

<img src="dk-remove-or-disable-requirement.png" width="706" alt="Remove or disable requirement"/>


> You can check which values different agents return for the given parameter before crafting an agent requirement. To do so, navigate to **Agents | Parameters Report** and enter a required parameter name.
> 
> <img src="dk-check-param-values.png" width="706" alt="Check parameter values"/>
> 
{style="tip"}

</tab>

<tab title="Kotlin DSL">

Add new expressions to the `requirements` block of your build configuration.

```Kotlin
object MyBuildConfig : BuildType({
requirements {
      exists("DotNetCoreSDK5.0_Path")
      startsWith("teamcity.agent.jvm.os.name", "Windows")
   }
})
```

See also: [Requirements | Kotlin DSL Documentation](https://www.jetbrains.com/help/teamcity/kotlin-dsl-documentation/root/requirements/index.html)

</tab>

</tabs>


</td></tr></table>



## Inherited Requirements

If an agent requirement is declared in a [](build-configuration-template.md), the requirement is also in effect for all configurations based on this template. The **Agent Requirements** page for these configuration marks such requirements as "inherited".

<img src="dk-inherited-requirement.png" width="706" alt="Inherited requirement"/>

Project administrators can edit, disable, and remove inherited requirements unless they are a part of an [enforced settings template](build-configuration-template.md#Enforcing+settings+inherited+from+template).


## Combining Conditions

<include from="requirement-conditions.md" element-id="combining-conditions"/>



 <seealso>
        <category ref="concepts">
            <a href="agent-requirements.md">TeamCity Data Backup</a>
            <a href="build-agent.md">Build Agent</a>
            <a href="managing-builds.md">Build Configuration</a>
        </category>
</seealso>