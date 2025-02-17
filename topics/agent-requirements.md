[//]: # (title: Agent Requirements)
[//]: # (auxiliary-id: Agent Requirements)

_Agent requirements_ are special conditions that define whether a [build configuration](managing-builds.md) can run on a particular [build agent](build-agent.md). Together with grouping by [agent pools](configuring-agent-pools.md), they give you a flexible control over how builds are distributed to agents.

To create an explicit agent requirement for a given build configuration:


1. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Agent Requirements"/></include>
2. Click __Add new requirement__. Each requirement represents a conditional rule for a certain parameter. While you are entering a parameter name or value, TeamCity will show you related suggestions.

To temporarily disable or delete a requirement, use its context menu.

>You can also set requirements to agents in the scope of each build step.  
>In a step's settings, add a [parametrized execution condition](build-step-execution-conditions.md) that defines a requirement to build agents. This way, this step will be executed during a build run only if the current agent satisfies this condition.

## Agent Requirements Based on Environment Variables and Properties

When a build agent registers on the TeamCity server, it provides information about its configuration, including its environment variables, system properties, and additional settings specified in the `buildAgent.properties` file. You can use these parameters to define agent requirements.

For example, if the current build configuration must run only on a Windows agent, add the following rule:
* Parameter name: `teamcity.agent.jvm.os.name`
* Condition: `equals`
* Value: `Windows`

After this requirement is created, TeamCity will check the value of the `jvm.os.name` system property on all active agents. If it does not equal `Windows` on a particular agent, this agent will be marked as incompatible with the current build configuration.

You can also use [regular expressions](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/Pattern.html) to match parameter values.
For example, if you want to select the agent by matching various parts of the agent name, add the following rule:
* Parameter name: `teamcity.agent.name`
* Condition: `matches`
* Value: `(macos|linux|win)-(m|l|xl).*`

Both compatible and incompatible agents are listed in __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes) | Agent Requirements__.

You can add multiple agent requirements for a single parameter. The agent will be considered compatible only if it satisfies all these requirements.

>If you are not sure why a certain requirement is not met, contact a responsible project or system administrator who created this requirement.

## Implicit Requirements

Any [reference](configuring-build-parameters.md#Parameter+References) (a name enclosed in `%` characters) to an unknown parameter within a build is considered an _implicit requirement_. The build will only run on an agent if:
* The agent provides this parameter, or
* The parameter is defined on the build configuration (or project) level.

The priority of the build configuration's value is higher than of the value defined on the agent.

For example, if you define a build runner parameter as a reference to another property: `%\env.JDK_16%/lib/*.jar`, this will implicitly add an agent requirement for the referenced property: that is, `env.JDK_16` must be defined. To define such properties on the agent, you can:
* Specify them in the [`buildAgent.properties`](configure-agent-installation.md) file.
* Set the [environment variable](predefined-build-parameters.md#Java-Related+Environment+Variables) `JDK_16` on the build agent.
* Specify the value on the __Parameters__ tab of a [project or configuration settings](project-administrator-guide.md#Edit+and+View+Modes). The same value of the property will be used for all build agents.

## Agent Requirements Video Guide

<video src="https://youtu.be/5gaoW8IfNJA"
title="TeamCity tutorial — Agent Requirements"/>

 <seealso>
        <category ref="concepts">
            <a href="build-agent.md">Build Agent</a>
            <a href="managing-builds.md">Build Configuration</a>
        </category>
        <category ref="admin-guide">
            <a href="assigning-build-configurations-to-specific-build-agents.md">Assigning Build Configurations to Specific Build Agents</a>
            <a href="configuring-build-agent-startup-properties.md">Configuring Build Agent Startup Properties</a>
            <a href="configuring-build-parameters.md">Configuring Build Parameters</a>
            <a href="configuring-agent-requirements.md">Configuring Agent Requirements</a>
        </category>
        <category ref="reference">
            <a href="requirement-conditions.md">Requirement Conditions</a>
        </category>
</seealso>