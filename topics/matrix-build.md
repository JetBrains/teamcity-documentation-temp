[//]: # (title: Matrix Build)
[//]: # (auxiliary-id: Matrix Build)

The _Matrix Build_ [build feature](adding-build-features.md) enables you to define a collection of builds by iterating over specified parameter values and generating a build for every combination.

For example, given a matrix build configured with the following parameters:

```
Browser: Chrome, Safari, Firefox
env.ShouldFail: true, false
Java: 11, 17, 21
```

When the matrix build is triggered, it runs builds for every combination of the specified values of `Browser`, `env.ShouldFail`, and `Java`, generating the following build summary (in the **Overview** tab of the matrix build):

<img src="matrix-build-table.png" alt="Summary table of matrix build" width="706" border-effect="line"/>


## Configuring a Matrix Build


### Configuring the Matrix Parameters

The **Matrix Build** dialog enables you to define the parameters of the matrix build, where each parameter definition consists of a parameter name and a list of associated values.

For example, suppose you want to configure a matrix build with the following matrix parameters:
```
Browser: Chrome, Firefox
Java: jdk-17, jdk-21
```

<tabs>
    <tab title="Edit in UI">
        <procedure title="Configure a Matrix Build in the UI"
                   style="steps"
                   hide-from-structure="true">
            <step><p>From the <b>Add Build Feature</b> dialog, choose <b>Matrix Build</b>.</p></step>
            <step><p>Configure the first parameter, as follows:</p>
                    <list type="alpha-lower">
                        <li><p>Enter the parameter name: <code>Browser</code></p>
                        <tip>A parameter name can be a new parameter, an existing parameter, or a predefined matrix parameter (see <a href="#Predefined+Matrix+Parameters"/>).</tip>
                        </li>
                        <li><p>Enter the associated parameter values: <code>Chrome</code>, <code>Firefox</code></p>
                    <tip>
                    <p>Parameter values are specified in the format, <code>[OptionalDisplayLabel=>]Value</code>.</p>
                    <p>Where <code>Value</code> consists of a single line of text and (optionally) some <a href="configuring-build-parameters.md#Parameter+References"/>. If specified, <code>OptionalDisplayLabel</code> is shown instead of the raw parameter value in the matrix build overview.</p>
                    </tip>
                    <note><p>If you need to add more than two parameter values, click <b>Add another value</b>.</p></note>
                        </li>
                    </list>
            </step>
            <step><p>To configure the second parameter, click <b>Add parameter</b>.</p>
               <list>
                  <li>Enter the parameter name: <code>Java</code></li>
                  <li>Enter the parameter values with labels: <code>JDK 17=>jdk-17</code> and <code>JDK 21=>jdk-21</code></li>
               </list>
               <tip><p>If you need to delete a parameter, click the trash icon beside the parameter name.</p></tip>
            </step>
            <step><p>Click <b>Save</b> to confirm the settings and enable the matrix build feature.</p></step>
        </procedure>
    </tab>
    <tab title="Kotlin DSL">
        <procedure title="Configure a Matrix Build in Kotlin DSL"
                   style="steps"
                   hide-from-structure="true">
            <step><p>To configure the matrix parameters, add a <code>matrix</code> block to the <code>features</code> block in the build type configuration, and create a sequence of <code>param</code> objects inside it:</p>


```Kotlin
package _Self.buildTypes

// Other imports not shown
import jetbrains.buildServer.configs.kotlin.matrix

object Build : BuildType({
    // Other code blocks not shown
    
    features {
        matrix {
            param(
                "Browser", listOf(
                    value("Chrome"),
                    value("Firefox")
                )
            )
            param(
                "Java", listOf(
                    value("jdk-17", label = "JDK 17"),
                    value("jdk-21", label = "JDK 21")
                )
            )
        }
    }
})
``` 
</step>
            <step><p>When configuring parameter names and parameter values:</p>
                    <list type="alpha-lower">
                        <li><p>A parameter name can be:</p>
                        <list>
                            <li>A new parameter name (for example, <code>Browser</code>)</li>
                            <li>An existing parameter name, already defined in the build configuration</li>
                            <li>A predefined matrix parameter (see <a href="#Predefined+Matrix+Parameters"/>)</li>
                        </list>
                        </li>
                        <li><p>A parameter value consists of a single line of text and (optionally) some <a href="configuring-build-parameters.md#Parameter+References">parameter references</a>. If a value is long or hard to read, you can specify a label, which is shown instead of the raw parameter value in the matrix build overview.</p></li>
                    </list>
            </step>
        </procedure>
    </tab>
</tabs>

### Predefined Matrix Parameters

TeamCity provides predefined matrix parameters to cover some of the common matrix build use cases:

<table>
<tr>
<th><p>Parameter</p></th>
<th><p>Description</p></th>
</tr>

<tr>
<td><p><code>arch</code></p></td>
<td><p>Enables you to run builds under different chipset architectures.</p>
<p>Applies a constraint equivalent to the agent requirement: <code>equals("teamcity.agent.jvm.os.arch", "%\arch%")</code>.</p></td>
</tr>

<tr>
<td><p><code>env.JAVA_HOME</code></p></td>
<td>
<p>Enables you to run builds with different JDK versions.</p>
<p>Each value that you select from the dropdown list (for example, <code>%\env.JDK_11_0%</code>) applies a constraint to run the build only on agents that define the referenced variable (<code>env.JDK_11_0</code>).</p>
<tip><p>The list of values you see in the dropdown is collected from the environments of all your agents.</p></tip>
</td>
</tr>

<tr>
<td><p><code>os</code></p></td>
<td><p>Enables you to run builds under different operating systems.</p>
<p>Applies a constraint equivalent to the agent requirement: <code>contains("teamcity.agent.jvm.os.name", "%\os%")</code>.</p>
</td>
</tr>

</table>

#### Kotlin DSL examples

Examples of configuring predefined parameters in Kotlin DSL:

<tabs>
<tab title="arch">

```Kotlin
object Build : BuildType({
    
    features {
        matrix {
           param("arch", listOf(
              value("x86"),
              value("ARM"),
              value("AMD64")
           ))
        }
    }
})
```
</tab>
<tab title="env.JAVA_HOME">

```Kotlin
object Build : BuildType({
    
    features {
        matrix {
           param("env.JAVA_HOME", listOf(
              value("%\env.JDK_17_0%", label = "JDK 17"),
              value("%\env.JDK_17_0_ARM64%", label = "JDK 17 ARM64")
           ))
        }
    }
})
```

</tab>
<tab title="os">


```Kotlin
object Build : BuildType({
    
    features {
        matrix {
            os = listOf(
                value("Linux"),
                value("Mac OS")
            )
       }
    }
})
```

</tab>
</tabs>


### Configuring Agent Requirements

You can reference matrix parameters in [agent requirements](configuring-agent-requirements.md) to ensure TeamCity chooses the right build agent for each value

For example, when setting up automated UI testing against different browser types, you might define the following `Browser` matrix parameter:

```
Browser: Firefox, Chrome, Edge
```

Given that the agents define environment variables to specify browser versions:

```
env.Chrome=119.0.6045.123
env.Firefox=119.0.1
```

You could define the following agent requirement to select a suitable agent based on the existence of the corresponding environment variable:

* Parameter name: `env.%\Browser%`
* Condition: `exists`


### Using Matrix Parameters in Configuration

There are many different contexts where it can be useful to reference the matrix parameters. Here are just a few examples:

* You can use [conditional build steps](build-step-execution-conditions.md) to make execution of a particular build step conditional on the value of a matrix parameter.
   > For example, if your matrix build has a deployment step that must be executed exactly once, you could define a conditional build step that executes the deployment step for just one combination of matrix parameter values.
   >
   {style="tip"}

* To reference resources needed by the build. For example, if the `Java` matrix parameter has the possible values `java-17` or `java-21`, you might reference it directly in the definition of the JDK path for your build:
    ```
    /usr/lib/jvm/%\Java%-openjdk-amd64
    ```
* If you have a build step that builds a Dockerfile for a particular Java version, you could reference the `Java` matrix parameter in the generated image file name, setting the **Image name:tag** field to `myapp:%\Java%`.
* For more complex scenarios, you could define a build step that runs a script to perform different tasks for different values of the matrix parameter.


### Publishing Artifacts

When you run a matrix build, artifacts from all of the generated builds are aggregated to the same location in the parent build. This can result in artifact files being overwritten.

To avoid overwriting artifact files, it is better to sort the generated artifacts using a directory name defined by the combination of matrix parameter values, for example:
```
%\Browser%-%\Java%
```

You can then define the artifact path as:
```
ch-simple/simple/target/*.jar => %\Browser%-%\Java%
```

The artifacts from the generated builds are then written to separate directories:
```
Chrome-JDK_17/
Chrome-JDK_21/
Firefox-JDK_17/
Firefox-JDK_21/
```


## Running a Matrix Build

You can run a matrix build in the same way as any regular build: by clicking **Run**, by configuring build triggers, or by making REST API calls.

When a matrix build starts, TeamCity runs the build, as follows:
1. The first time the matrix build runs, it generates new virtual build configurations for every combination of matrix parameter values. The matrix build effectively behaves like a parent configuration for these generated snapshot dependencies
   > Each of the generated build configurations has the same build steps as the parent configuration. Any subsequent changes to the build steps in the parent configuration will be propagated to the generated build configurations automatically.
   >
   {style="note"}

2. TeamCity runs the generated builds. Each build is added separately to the build queue and is subject to the usual rules for build priority and agent selection.
   > Normally, the generated builds can run in parallel on multiple agents. If you choose a specific agent in the [custom build options](running-custom-build.md#General+Options), however, the generated builds will run one-by-one on the specified agent.
   >
   {style="note"}

3. As soon as the first generated build starts to run, TeamCity starts the parent build (effectively, a type of [composite build](composite-build-configuration.md) with dependencies on the generated builds), which aggregates the build results from all the generated builds.
4. After the matrix build is complete, you can view the summary table on the **Overview** tab of the matrix build.

If you only need to build a part of the matrix, running a custom build is a convenient way of doing this (you can also upvote the [TW-84312](https://youtrack.jetbrains.com/issue/TW-84312/Matrix-builds-Allow-to-exclude-parameter-combinations-run-subset-of-matrix-parameters) request for a dedicated UI to run individual combinations). Switch to the **Parameters** tab of the **Run Custom Build** dialog and specify required parameter values (use comma as a separator). Parameters that you did not override in this dialog will cycle through all values specified in the Matrix Builds feature.

<img src="dk-matrix-partial.png" alt="Running a portion of the matrix" width="706"/>

In particular, when configuring a [build trigger](configuring-build-triggers.md) on the matrix build, you can configure the **Build Customization** tab in the trigger configuration dialog to customize the matrix parameter values used for the triggered builds.

> When setting matrix parameters in the **Build Customization** tab, you must enter the parameter values as a comma-separated list. For example, to iterate over two browser values, you could set the parameter name to `Browser` and the parameter value to `Chrome, Firefox`.
> 
{style="note"}


## Viewing a Matrix Build

You can view a matrix build on two different levels:

* Parent build — configures and organizes the generated builds
  * Defines the matrix parameters configuration
  * Defines configuration settings common to all builds
  * Is responsible for triggering the builds
  * Provides a tabular summary of the builds
  * Does not contain detailed results from the individual builds
* Generated build — represents the build for a single combination of parameter values
  * Inherits readonly configuration settings from the parent build
  * Has the same build steps as the parent build
  * Provides detailed results from the particular build

When you drill down to a specific build in the matrix, you see what looks like a typical build page with complete information about the build. However, the build configuration is _readonly_, because the build inherits its configuration settings from the parent build and, in particular, you cannot run this build configuration directly.

> If the project has [versioned settings](storing-project-settings-in-version-control.md) enabled, the generated build configurations are not committed to the VCS repository.
>
{style="warning"}


## Matrix Builds in a Build Chain

Matrix builds can be chained together using [snapshot dependencies](snapshot-dependencies.md) or [artifact dependencies](artifact-dependencies.md), just like a regular build.

If a regular build has a dependency on a matrix build, the dependency is linked to the parent build of the matrix. Generated builds are readonly, so you cannot link dependencies directly to them. Nevertheless, it is possible to consume artifacts from the generated builds, by ensuring that the artifacts are sorted according to the parameter combination of the respective build.

In this section, we focus on the scenario `MatrixBuild1 -> RegularBuild2`, where there is an artifact dependency defined between a matrix build and a regular build. In this case, you need to be careful how you handle the artifacts from the generated builds.


### Build Chain Example

Consider an ordinary (non-matrix) build chain with two stages:

* _Build1_ has a Maven build runner configured to generate a Java package. The **Artifact paths** field in the general settings section of the build configuration is configured to capture the generated package as an artifact:
  ```
  ch-simple/simple/target/*.jar => packages
  ```
* _Build2_ is configured with an [artifact dependency](artifact-dependencies.md) on Build1, with the following **Artifacts rules** setting:
  ```
  packages => dependencies
  ```

In order to extend the testing to multiple browsers and Java versions, Build1 needs to be refactored as a matrix build, covering the following browser and Java version combinations:

```
Browser: Chrome, Firefox
Java: JDK_17, JDK_21
```

After configuring the matrix parameters on Build1, you also need to update the artifact settings:

* In _Build1_, modify the **Artifact paths** field in the general settings section of the build configuration to sort the aggregated artifacts by parameter combination:
   ```
   ch-simple/simple/target/*.jar => packages/%\Browser%-%\Java%
   ```


## Known Limitations

* Reverse dependency parameters, `reverse.dep.*`, do not work correctly in matrix builds. Related YouTrack ticket: [TW-84730](https://youtrack.jetbrains.com/issue/TW-84730).
* When a matrix build is configured with a snapshot dependency on a preceding build, test results from the preceding build are reported in the matrix build, which is an unexpected behavior. Related YouTrack ticket: [TW-75412](https://youtrack.jetbrains.com/issue/TW-75412).
* When a matrix build is configured with a snapshot dependency on a preceding build, the reported build time of the matrix build increases, because it includes the durations of all the preceding builds in the chain. This might require you to revise timeout settings. Related YouTrack ticket: [TW-76020](https://youtrack.jetbrains.com/issue/TW-76020).
* The **Run Custom Build** dialog does not allow you to assign labels to new values.
* If a value that was specified in the feature settings has a label and its raw value includes a comma, you cannot reference it in a custom build via the label only; both label and raw value portions must be entered.
