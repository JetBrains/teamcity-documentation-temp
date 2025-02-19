[//]: # (title: Assigning Build Configurations to Specific Build Agents)
[//]: # (auxiliary-id: Assigning Build Configurations to Specific Build Agents)

It is sometimes necessary to manage the [Build Agents](build-agent.md)' workload more effectively. For example, if the time-consuming performance tests are run, the Build Agents with low hardware resources may slow down. As a result, more builds will enter the [build queue](working-with-build-queue.md), and the feedback loop can become longer than desired. To avoid such situation, you can:

1. [Establish a run configuration policy](#Agent+pools) for an agent, which defines the build configurations to run on this agent.
2. Define special [Agent Requirements](configuring-agent-requirements.md), to restrict the pool of agents, on which a build configuration can run the builds. These requirements are:			
   * [Build Agent name](#Agent+pools). If the name of a build agent is made a requirement, the build configuration will run builds on this agent only.		
   * [Build Agent property](#Agent+pools). If a certain property, for example, a capability to run builds of a certain configuration or an operating system, is made a requirement, the build configuration will run builds on the agents that meet this requirement.
   
<tip>

* You can modify these parameters when setting up the project or build configuration, or at any moment you need. The changes you make to the build configurations are applied on the fly.	
* You can specify a particular build agent to run a build on when [Triggering a Custom Build](running-custom-build.md).

</tip>

## Agent pools

You could split agents into pools. Each project could be associated to a number of pools. See [Agent Pools](configuring-agent-pools.md).

## Establishing a Run Configuration Policy

To establish a Build Agent's run configuration policy:
	
1. Click the __Agents__ and select the desired build agent.
2. Click the __Compatible Configurations__ tab.
3. Set the **Current run configuration policy** option to **Run assigned configurations only**.
4. Click the **Assign configurations** button and tick the desired build configurations names to run on the build agent.

## Making Build Agent Name and Property a Build Configuration Requirement

To make a build configuration run the builds on a build agent with the specified name and properties:
	
1. Click __Administration__ and select the desired build configuration.
2. Select the **Agent Requirements** tab in the left navigation panel (see [Configuring Agent Requirements](configuring-agent-requirements.md)).
3. Click **Add new requirement** and enter "teamcity.agent.name" in the **Parameter name** field.
4. Change the condition to "equals" and enter the full agent name in the **Value** field. 

The following [Kotlin](kotlin-dsl.md) snippet illustrates the result:

```Kotlin
object Build : BuildType({
   name = "Build"
   steps {
      // build steps
   }
   // ...
   requirements {
      equals("teamcity.agent.name", "SandboxBuilder14")
   }
})
```

Depending on your needs, you can modify this requirement as follows:

* Change the **Condition** from "equals" to a non-strict operator (for example, "contains") to include multiple similarly named agents. For instance, the `startsWith("teamcity.agent.name", "NightlyAgent")` condition allows builds to run on "NightlyAgent1", "NightlyAgent_24", "NightlyAgent.A" and other build agents whose names satisfy the requirement.
* Use the `teamcity.agent.jvm.os.name` parameter to choose agents by their operating systems rather than names. For example, `equals("teamcity.agent.jvm.os.name", "Mac OS X")`.


<!--[//]: # (Internal note. Do not delete. "Assigning Build Configurations to Specific Build Agentsd17e193.txt")--> 

 <seealso>
        <category ref="concepts">
            <a href="build-agent.md">Build Agent</a>
            <a href="configuring-agent-requirements.md">Agent Requirements</a>
        </category>
        <category ref="admin-guide">
            <a href="running-custom-build.md">Triggering a Custom Build</a>
        </category>
</seealso>