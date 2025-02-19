[//]: # (title: Mapping TeamCity Concepts to Other CI Terms)
[//]: # (auxiliary-id: Mapping TeamCity Concepts to Other CI Terms)

This information can be used when migrating to TeamCity from other Continuous Integration tools.

## Jenkins Mapping

<include from="jenkins-to-teamcity-migration-guidelines.md" element-id="jenkins-mapping-to-teamcity"/>

## Bamboo Mapping

<table>

<tr>
<td width="300"><p><b>Bamboo</b></p></td>
<td><p><b>TeamCity</b></p></td>
</tr>

<tr>
<td><p>Project</p></td>
<td><p><a href="project.md">Project</a></p></td>
</tr>

<tr>
<td><p>Plan</p></td>
<td><p><a href="build-chain.md">Build Chain</a></p></td>
</tr>

<tr>
<td><p>Stage</p></td>
<td><p>Build, <a href="composite-build-configuration.md">Composite Build</a>, or <a href="matrix-build.md">Matrix Build</a></p></td>
</tr>

<tr>
<td><p>Job</p></td>
<td><p><a href="creating-and-editing-build-configurations.md">Build Configuration</a></p></td>
</tr>

<tr>
<td><p>(Plan or Job) Build</p></td>
<td><p>Build</p></td>
</tr>

<tr>
<td><p>Task</p></td>
<td><p><a href="configuring-build-steps.md">Build Step</a></p></td>
</tr>

<tr>
<td><p>Artifact</p></td>
<td><p><a href="build-artifact.md">Build Artifact</a></p></td>
</tr>

<tr>
<td><p>Trigger method</p></td>
<td><p><a href="configuring-build-triggers.md">Build Trigger</a></p></td>
</tr>

<tr>
<td><p>Agent</p></td>
<td><p><a href="build-agent.md">Build Agent</a></p></td>
</tr>

<tr>
<td><p>Requirement</p></td>
<td><p><a href="configuring-agent-requirements.md">Agent Requirement</a></p></td>
</tr>

<tr>
<td><p>Capability</p></td>
<td><p>(Agent) <a href="using-build-parameters.md">Build Parameter</a></p></td>
</tr>

</table>


## GitLab CI/CD Mapping

<table>

<tr>
<td width="300"><p><b>GitLab CI/CD</b></p></td>
<td><p><b>TeamCity</b></p></td>
</tr>

<tr>
<td><p>Project</p></td>
<td><p><a href="project.md">Project</a></p></td>
</tr>

<tr>
<td><p>Pipeline</p></td>
<td><p><a href="build-chain.md">Build Chain</a></p></td>
</tr>

<tr>
<td><p>Stage</p></td>
<td><p>Build, <a href="composite-build-configuration.md">Composite Build</a>, or <a href="matrix-build.md">Matrix Build</a></p></td>
</tr>

<tr>
<td><p>Job</p></td>
<td><p><a href="creating-and-editing-build-configurations.md">Build Configuration</a></p></td>
</tr>

<tr>
<td><p>Job</p></td>
<td><p>Build</p></td>
</tr>

<tr>
<td><p>Job artifact</p></td>
<td><p><a href="build-artifact.md">Build Artifact</a></p></td>
</tr>

<tr>
<td><p>Branch pipeline</p>
<p>Merge request pipeline</p>
</td>
<td><p><a href="configuring-build-triggers.md">Build Trigger</a></p></td>
</tr>

<tr>
<td><p>Runner</p></td>
<td><p><a href="build-agent.md">Build Agent</a></p></td>
</tr>

<tr>
<td><p>Rules</p></td>
<td><p><a href="build-step-execution-conditions.md">Build Step Execution Conditions</a></p></td>
</tr>

</table>

## Concourse Mapping

<table>

<tr>
<td width="300"><p><b>Concourse</b></p></td>
<td><p><b>TeamCity</b></p></td>
</tr>

<tr>
<td><p>Pipeline</p></td>
<td><p><a href="build-chain.md">Build Chain</a></p></td>
</tr>

<tr>
<td><p>Job</p></td>
<td><p><a href="creating-and-editing-build-configurations.md">Build Configuration</a></p></td>
</tr>

<tr>
<td><p>Build</p></td>
<td><p>Build</p></td>
</tr>

<tr>
<td><p>Step</p></td>
<td><p><a href="configuring-build-steps.md">Build Step</a></p></td>
</tr>

<tr>
<td><p>Artifact</p></td>
<td><p><a href="build-artifact.md">Build Artifact</a></p></td>
</tr>

<tr>
<td><p>Get step</p></td>
<td><p><a href="configuring-build-triggers.md">Build Trigger</a></p></td>
</tr>

<tr>
<td><p>Worker</p></td>
<td><p><a href="build-agent.md">Build Agent</a></p></td>
</tr>

<tr>
<td><p>Vars</p></td>
<td><p><a href="using-build-parameters.md">Build Parameter</a></p></td>
</tr>

</table>
