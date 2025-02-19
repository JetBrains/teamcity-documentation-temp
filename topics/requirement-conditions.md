[//]: # (title: Requirement Conditions)
[//]: # (auxiliary-id: Requirement Conditions)

This page explains conditions used in [agent requirements](configuring-agent-requirements.md) or [build step execution requirements](build-step-execution-conditions.md).


<table>
<tr><td>Condition</td><td>Description and Example</td></tr>

<tr>
<td><p><b>exists</b></p>
</td>
<td><p>Returns <code>true</code> if the specified property exists.</p>
<p>For example:</p>
<p><code>exists("DotNetCLI_Path")</code></p>
</td>
</tr>

<tr>
<td><p><b>does not exist</b></p>
</td>
<td><p>Returns <code>true</code> if the specified property does not exist.</p>
<p>For example:</p>
<p><code>doesNotExist("env.TEAMCITY_GIT_PATH")</code></p>
</td>
</tr>

<tr>
<td><p><b>equals</b></p>
</td>
<td><p>Returns <code>true</code> if the specified property exists and equals the given value.</p>
<p>You can leave the <b>Value</b> field empty to check whether the specified property exists but is empty.</p>
<p>For example:</p>
<p><code>equals("docker.server.osType", "Linux")</code></p>
</td>
</tr>

<tr>
<td><p><b>does not equal</b></p>
</td>
<td><p>Returns <code>true</code> if a value of the specified property differs from the given value, or if this property does not exist.</p>
<p>You can leave the <b>Value</b> field empty to check whether the specified property exists and is not empty (has a value).</p>
<p>For example:</p>
<p><code>doesNotEqual("system.teamcity.buildType.id", "Tests")</code></p>
</td>
</tr>

<tr>
<td><p><b>is not more than</b></p>
</td>
<td rowspan="4"><p>Return <code>true</code> if a value of the target numeric property meets the corresponding condition.</p>
<p>For example:</p>
<p><code>moreThan("build.number", "256")</code></p>
<p><code>lessThan("teamcity.agent.hardware.cpuCount", "10")</code></p>
</td>
</tr>

<tr>
<td><p><b>is more than</b></p>
</td>
</tr>

<tr>
<td><p><b>is less than</b></p>
</td>
</tr>

<tr>
<td><p><b>is not less than</b></p>
</td>
</tr>

<tr>
<td>
<p><b>version is more than</b></p>
</td>
<td rowspan="4"><p>Compare software versions with the given value. Supports multiple value formats, including dot (<code>.</code>) delimiters, leading zeroes, common suffixes like <code>beta</code> or <code>EAP</code>. If the version number contains alphabetic characters, they are compared as well, for instance, <code>1.1e &lt; 1.1g</code>.</p>
<p>For example:</p>
<p><code>moreThanVer("maven", "3.0")</code></p>
</td>
</tr>


<tr>
<td><p><b>version is not more than</b></p>
</td>
</tr>

<tr>
<td><p><b>version is less than</b></p>
</td>
</tr>

<tr>
<td><p><b>version is not less than</b></p>
</td>
</tr>

<tr>
<td><p><b>contains</b></p>
</td>
<td><p>Returns <code>true</code> if a value of the specified property includes the given value.</p>
<p>For example:</p>
<p><code>contains("teamcity.serverUrl", "localhost")</code></p>
</td>
</tr>

<tr>
<td><p><b>does not contain</b></p>
</td>
<td><p>Returns <code>true</code> if a value of the specified property does not include the given value, or if this property does not exist.</p>
<p>For example:</p>
<p><code>doesNotContain("system.agent.name", "_local")</code></p>
</td>
</tr>

<tr>
<td><p><b>starts with</b></p>
</td>
<td rowspan="2"><p>Returns <code>true</code> if a value of the specified property starts (ends) with the given value.</p>
<p>For example:</p>
<p><code>startsWith("teamcity.agent.jvm.os.name", "Mac")</code></p>
<p><code>endsWith("teamcity.agent.work.dir", "/work")</code></p>
</td>
</tr>

<tr>
<td><p><b>ends with</b></p>
</td>
</tr>

<tr>
<td><p><b>matches</b></p>
</td>
<td rowspan="2"><p>Returns <code>true</code> if the specified property matches (does not match) the given <a href="https://java.sun.com/j2se/1.5.0/docs/api/java/util/regex/Pattern.html">regular expression</a> pattern.</p>
<p>For example, use <code>matches("CUSTOM_PARAMETER", ".*(,|^)foo(,|$).*")</code> to match all occurrences of <code>foo</code>, <code>foo,bar</code>, <code>bar,foo</code>, and <code>bar,foo,xxx</code>.
</p>
</td>
</tr>

<tr>
<td><p><b>does not match</b></p>
</td>
</tr>

</table>



## Combining Conditions

<snippet id="combining-conditions">

When multiple requirements are defined, they are implicitly joined by boolean AND. For example, the following set of conditions requires that both the `env.JDK_17_0` parameter AND the `env.JDK_21_0` parameter exist:

```Kotlin
requirements {
    exists("env.JDK_17_0")
    exists("env.JDK_21_0")
}
```

There is no mechanism available for joining requirements with boolean OR.

</snippet>