[//]: # (title: Service Messages)
[//]: # (auxiliary-id: Service Messages)

_Service messages_ are specially constructed pieces of text that pass commands/information about the build from the build script to the TeamCity server.

To be processed by TeamCity, __they need to be written to the standard output stream of the build, that is printed or echoed from a build step__.

>It is recommended to output one message per line (by dividing them with `\n` symbols) and flush the stdout buffer after each service message.

Examples:

<tabs>

<tab title="Windows">

```Shell
echo ##teamcity[<messageName> 'value']

``` 
</tab>

<tab title="Linux">

```Shell
echo "##teamcity[<messageName> 'value']"

``` 
</tab>

<tab title="PowerShell script">

```Shell
Write-Host "##teamcity[<messageName> 'value']"

``` 
</tab>

</tabs>

A single service message cannot contain a newline character inside it, it cannot span across multiple lines.

## Service Messages Formats

Service messages support two formats:

* Single-attribute message:
    ```Shell
    ##teamcity[<messageName> 'value']
    
    ```

* Multiple-attribute message:
    ```Shell
    ##teamcity[<messageName> name1='value1' name2='value2']
    
    ```

  Multiple-attribute message can more formally be described as:   
  ```Shell
    ##teamcity[messageNameWSPpropertyNameOWSP=OWSP'value'WSPpropertyName_IDOWSP=OWSP'value'...OWSP]
    
    ```

where:
 * `messageName` is the name of the message, such as `flowStarted` or `setParameter`.
 * `propertyName` is a name of the message attribute. Must be a valid Java ID.
 * `value` is a value of the attribute. Must be an [escaped value](#Escaped+Values).
 * `WSP` is a required whitespace(s): space or tab character (`\t`).
 * `OWSP` is an optional whitespace(s).
 * `...` is any number of `WSPpropertyNameOWSP=OWSP'value'` blocks.
 
## Escaped Values

For escaped values, TeamCity uses a vertical bar `|` as an escape character. To have certain special characters properly interpreted by the TeamCity server, they must be preceded by a vertical bar. For example, the following message:

```Shell
##teamcity[testStarted name='foo|'s test']

```

will be displayed in TeamCity as `foo's test`. Refer to the table of the escaped values below.

<table>

<tr>

<td>

Character

</td>

<td>

Escape as

</td>

</tr>

<tr>

<td>

' (apostrophe)

</td>

<td>

|'

</td></tr>

<tr>

<td>

\n (line feed)

</td>

<td>

|n

</td></tr>

<tr>

<td>

\r (carriage return)

</td>

<td>

|r

</td></tr>

<tr>

<td>

\uNNNN (unicode symbol with code 0xNNNN)

</td>

<td>

|0xNNNN

</td></tr>

<tr>

<td>

| (vertical bar)

</td>

<td>

||

</td></tr>

<tr>

<td>

[ (opening bracket)

</td>

<td>

|[

<!--[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e192.txt")-->    

</td></tr>

<tr>

<td>

] (closing bracket)

</td>

<td>

|]

</td></tr></table>

## Reporting Messages to Build Log

You can report messages to a build log as follows:

```Shell
##teamcity[message text='<message text>' errorDetails='<error details>' status='<status value>']

```

where:
* `status` can take the following values: `NORMAL` (default), `WARNING`, `FAILURE`, `ERROR`.
* `errorDetails` is used only if `status` is `ERROR`, in other cases it is ignored.   
This message fails the build in case its status is `ERROR` and the "_Fail build if an error message is logged by build runner_" box is checked on the __[Build Failure Conditions](build-failure-conditions.md)__ page of the build configuration. For example:
    ```Shell
    ##teamcity[message text='Exception text' errorDetails='stack trace' status='ERROR']
    
    ```
  
>If you want to control when a build fails, you can either configure a [failure condition](build-failure-conditions.md) or send a special [message with a build problem](#Reporting+Build+Problems).

### Blocks of Service Messages

Blocks are used to group several messages in the build log.

Block opening:

The `blockOpened` system message has the `name` attribute, and you can also add its description: 

```Shell
 ##teamcity[blockOpened name='<blockName>' description='<this is the description of blockName>']

```

Block closing:

```Shell
 ##teamcity[blockClosed name='<blockName>']

```

>Note that when you close a block, all its inner blocks are closed automatically.
>
{style="note"}

### Reporting Compilation Messages

```Shell
##teamcity[compilationStarted compiler='<compiler_name>']
...
##teamcity[message text='compiler output']
##teamcity[message text='compiler output']
##teamcity[message text='compiler error' status='ERROR']
...
##teamcity[compilationFinished compiler='<compiler name>']

```

where:
* `compiler_name` is an arbitrary name of the compiler performing compilation, for example, `javac` or `groovyc`. Currently, it is used as a block name in the build log.
* Any message with status `ERROR` reported between `compilationStarted` and `compilationFinished` will be treated as a compilation error.

### Message FlowId

Any message supports the optional attribute `flowId` that allows you to group messages into separate categories (flows). For example, since every message is processed in the order it is sent, the following sample produces a hierarchy of blocks where both output messages are owned by the latest block:

```
##teamcity[blockOpened name='block 1']
##teamcity[blockOpened name='block 2']
##teamcity[message text='Message 1']
##teamcity[message text='Message 2']
```

<img src="dk-messages-noFlowId.png" alt="Block without flowID" width="706"/>

Adding the `flowId` attribute allows you to split the messages into two parallel flows.

```
##teamcity[blockOpened name='block 1' flowId='1']
##teamcity[blockOpened name='block 2' flowId='2']
##teamcity[message text='Message 1' flowId='1']
##teamcity[message text='Message 2' flowId='2']
```

<img src="dk-messages-FlowId.png" width="706" alt="Block with flowID"/>


In most cases, `flowId` is the only attribute required to start a message flow. In [](#Nested+Test+Reporting), you may want to start a flow not inside the root flow but as a subflow inside an existing flow. To do this, add the `flowStarted` parameter and specify the parent flow ID as the `parent` parameter. Flows without the specified parent start inside the root flow of the current step.

To end a subflow, use the `flowFinished` parameter. Ending a parent flow automatically closes all its subflows, but we recommend declaring the flow order explicitly:

```Shell
##teamcity[flowStarted flowId='MainFlow' ...]
##teamcity[flowStarted flowId='SubFlow1' parent='MainFlow' ...]
##teamcity[flowFinished flowId='SubFlow1' ...]
##teamcity[flowStarted flowId='SubFlow2' parent='MainFlow' ...]
##teamcity[flowFinished flowId='SubFlow2' ...]
##teamcity[flowFinished flowId='MainFlow' ...]

```

Note that the `flowStarted` and `flowFinished` messages are in effect only when emitted between the `testStarted` and `testFinished` messages.

### Reporting Tests

To use the TeamCity on-the-fly test reporting, a testing framework needs dedicated support for this feature to work (alternatively, [XML Report Processing](xml-report-processing.md) can be used). If TeamCity doesn't support your testing framework natively, it is possible to modify your build script to report test runs to the TeamCity server using service messages. This makes it possible to display test results in real-time, make test information available on the __[Tests](build-results-page.md#Tests+Tab)__ tab of the __Build Results__ page.

#### Message Creation Timestamp

Test report messages support the optional attribute `timestamp`. In the following examples, `<messageName>` is the name of the specific service message.

```Shell
##teamcity[<messageName> timestamp='timestamp' ...]

```

The timestamp format is `yyyy-MM-dd'T'HH:mm:ss.SSSZ` or `yyyy-MM-dd'T'HH:mm:ss.SSS` according to [Java SimpleDateFormat syntax](http://docs.oracle.com/javase/1.5.0/docs/api/java/text/SimpleDateFormat.html).

Example:

```Shell
##teamcity[<messageName> timestamp='2008-09-03T14:02:34.487+0400' ...]
##teamcity[<messageName> timestamp='2008-09-03T14:02:34.487' ...]

```

For .NET [DateTimeOffset](https://msdn.microsoft.com/en-us/library/az4se3k1(v=vs.110).aspx), a code like this
```JavaScript
var date = DateTimeOffset.Now;
var timestamp = $"{date:yyyy-MM-dd'T'HH:mm:ss.fff}{date.Offset.Ticks:+;-;}{date.Offset:hhmm}";

```

will result in
```Shell
##teamcity[<messageName> timestamp='2008-09-03T14:02:34.487+0400' ...]

```

#### Supported Test ServiceMessages

__Test suite messages:__ Test suites are used to group tests. TeamCity displays tests grouped by suites on the __[Tests](build-results-page.md#Tests+Tab)__ tab of the __Build Results__ page and in other places.

```Shell
##teamcity[testSuiteStarted name='suiteName']
<individual test messages go here>
##teamcity[testSuiteFinished name='suiteName']

```

All the individual test messages are to appear between `testSuiteStarted` and `testSuiteFinished` (in that order) with the same `name` attributes.

#### Nested Test Reporting

Starting another test finishes the currently started test in the same _flow_. To still report tests from within other tests, you will need to specify another [`flowId`](#Message+FlowId) in the nested test service messages.

__Test start/stop messages:__

```Shell
##teamcity[testStarted name='testName' captureStandardOutput='<true/false>']
<here go all the test service messages with the same name>
##teamcity[testFinished name='testName' duration='<test_duration_in_milliseconds>']

```

Indicates that `testName` was run. If the `testFailed` message is not present, the test is regarded successful, where
* `duration` (optional numeric integer attribute) sets the test duration in milliseconds to be reported in TeamCity UI. If omitted, the test duration will be calculated from the messages timestamps. If the timestamps are missing — from the actual time the messages were received on the server.
* `captureStandardOutput` (optional boolean attribute) — if `true`, all the standard output and standard error messages received between `testStarted` and `testFinished` messages will be considered test output. The default value is `false`: assumes usage of `testStdOut` and `testStdErr` service messages to report the test output.



> All the other test messages (except for `testIgnored`) with the same `name` attribute must appear between the `testStarted` and `testFinished` messages (in that order).   
> 
> If using Ant's `echo` task to output the messages, make sure to include the `flowId` attribute with the same value in all the messages related to the same test / test suite as otherwise they [will not be processed correctly](https://youtrack.jetbrains.com/issue/TW-5059).
> 
{style="note"}

It is highly recommended that you ensure that the pair of `test suite` + `test name` is unique within the build. For advanced TeamCity test-related features to work, test names must not deviate from one build to another (a single test must be reported under the same name in every build). Include absolute paths in the reported test names is __strongly discouraged__.

__Ignored tests:__

```Shell
##teamcity[testIgnored name='testName' message='ignore comment']

```

Indicates that `testName` is present but was not run (was ignored) by the testing framework. As an exception, the `testIgnored` message can be reported without the matching `testStarted` and `testFinished` messages.

__Test output:__

```Shell
##teamcity[testStarted name='className.testName']
##teamcity[testStdOut name='className.testName' out='text']
##teamcity[testStdErr name='className.testName' out='error text']
##teamcity[testFinished name='className.testName' duration='50']

```

The `testStdOut` and `testStdErr` service messages report the test's standard and error output to be displayed in the TeamCity UI. There must be only one `testStdOut` and one `testStdErr` message per test. An alternative but a less reliable approach is to use the `captureStandardOutput` attribute of the `testStarted` message.

__Test result:__


```Shell
##teamcity[testStarted name='MyTest.test1']
##teamcity[testFailed name='MyTest.test1' message='failure message' details='message and stack trace']
##teamcity[testFinished name='MyTest.test1']

##teamcity[testStarted name='MyTest.test2']
##teamcity[testFailed type='comparisonFailure' name='MyTest.test2' message='failure message' details='message and stack trace' expected='expected value' actual='actual value']
##teamcity[testFinished name='MyTest.test2']

```

Indicates that the `testname` test failed. Only one `testFailed` message can appear for a given test name.
* `message` contains the textual representation of the error.
* `details` contains detailed information on the test failure, typically a message and an exception stacktrace.
* `actual` and `expected` attributes can only be used together with `type='comparisonFailure'` to report comparison failure. The values will be used when opening the test in the IDE.

Here is a longer example of test reporting with service messages:

```Shell
##teamcity[testSuiteStarted name='suiteName']
##teamcity[testSuiteStarted name='nestedSuiteName']
##teamcity[testStarted name='package_or_namespace.ClassName.TestName']
##teamcity[testFailed name='package_or_namespace.ClassName.TestName' message='The number must be 20000' details='junit.framework.AssertionFailedError: expected:<20000> but was:<10000>|n|r    at junit.framework.Assert.fail(Assert.java:47)|n|r    at junit.framework.Assert.failNotEquals(Assert.java:280)|n|r...']
##teamcity[testFinished name='package_or_namespace.ClassName.TestName']
##teamcity[testSuiteFinished name='nestedSuiteName']
##teamcity[testSuiteFinished name='suiteName']

```

#### Enabling Test Retry

You can enable the support of _test retry_ for a build configuration. With this option enabled, the successful run of a test will mute its previous failure, which means that TeamCity will mute a test if it fails and then succeeds within the same build.   
Such tests will not affect the build status.

```Shell
##teamcity[testRetrySupport enabled=’true’]
```

#### Interpreting Test Names

A full test name can have a form of: `<suite name>: <package/namespace name>.<class name>.<test method>(<test parameters>)`, where `<class name>` and `<test method>` cannot have dots in the names. Only `<test method>` is a mandatory part of a full test name.

The full test name is used to compare tests between consequent builds or match tests across different build configurations.

Example of a full test name:

```JavaScript
Integration Tests: Backend: org.jetbrains.teamcity.LoginPageController.testBadPassword("incorrect password", false)
 
// in the example above, 
// suite name = "Integration Tests: Backend"
// package = org.jetbrains.teamcity
// class name = LoginPageController
// test method = testBadPassword
// test parameters = ("incorrect password", false)
```

The __[Tests](build-results-page.md#Tests+Tab)__ tab of the __Build Results__ page allows grouping by suites, packages/namespaces, classes, and tests. Usually the attribute values are provides as they are reported by your test framework and TeamCity is able to interpret test names correctly.

If a test cannot be parsed in the form above, TeamCity still tries to extract `<suite name>` from the full test name for the filtering on the Tests tab, and treats everything after the suite a non-parsable test name.

#### Reporting Additional Test Data

It is possible to attach extra information to the tests using the `testMetadata` service message. More details are available on a [separate page](reporting-test-metadata.md).

### Reporting .NET Code Coverage Results

You can configure .NET coverage processing by means of service messages. To learn more, refer to [Manually Configuring Reporting Coverage](manually-configuring-reporting-coverage.md) page.

### Reporting Inspections

You can report inspections from a custom tool to TeamCity using the service messages described below.

Among other uses, the number of inspections can be used as a build metric to [fail a build on](build-failure-conditions.md#Fail+Build+on+Metric+Change).

#### Inspection Type

Each specific warning or an error in code (inspection instance) has an inspection type — the unique description of the conducted inspection, which can be reported via


```Shell
##teamcity[inspectionType id='<id>' name='<name>' description='<description>' category='<category>']

```

where all the attributes are required and can have either numeric or textual values:

* `id` — (mandatory) limited by 255 characters
* `name` — (mandatory) limited by 255 characters
* `category` — (mandatory) limited by 255 characters. The `category` attribute examples are "Style violations" and "Calling contracts".
* `description` — (mandatory) limited by 4000 characters. The description can also be in HTML. For example,

```HTML
<html>
    <body>
    Reports unnecessary local variables, which add nothing to the comprehensibility of a method. Variables caught include local variables which are immediately returned, local variables that are immediately assigned to another variable and then not used, and local variables which always have the same value as another
    local variable or parameter.
    <!-- tooltip end -->
        <p>
        Use the first checkbox below to have this inspection ignore variables which are immediately returned or thrown. Some coding styles suggest using such variables for clarity and ease of debugging.
        </p>
        <p>
        Use the second checkbox below to have this inspection ignore variables which are annotated.
        </p>
    </body>
</html>

```

### Inspection Instance

Reports a specific defect, warning, error message. Includes location, description, and various optional and custom attributes.

```Shell
##teamcity[inspection typeId='<inspection type identity>' message='<instance description>' file='<file path>' line='<line>' additional attribute='<additional attribute>']

```

where all the attributes can have either numeric or textual values:
* `typeId` — (mandatory), reference to the `inspectionType.id` described [above](#Inspection+Type) limited by 255 characters.
* `message` — (optional) current instance description limited by 4000 characters.
* `file` — (mandatory) file path limited by 4000 characters. The path can be absolute or relative to the [checkout directory](build-checkout-directory.md).
* `line` — (optional) line of the file, integer.
* `additional attribute`– can be any attribute, `SEVERITY` is often used here, with one of the following values (mind the upper case): `INFO`, `ERROR`, `WARNING`, `WEAK WARNING`.

Example:

```Shell
##teamcity[inspectionType id='UnnecessaryLocalVariable' name='Redundant local variable' description='<html><body>Reports unnecessary local variables...</body>
</html>' category='Data flow issues']
##teamcity[inspection typeId='UnnecessaryLocalVariable' message='Local variable <code>i</code> is redundant' file='src/Test.java' line='19' SEVERITY='WARNING']

```

### Publishing Artifacts While Build is in Progress

You can publish the build artifacts while the build is still running, immediately after the artifacts are built.

To do this, you need to output the following line:

```Shell
##teamcity[publishArtifacts '<path>']

```

The `<path>` has to adhere to the same rules as the [Build Artifact specification](configuring-general-settings.md#Artifact+Paths) of the __Build Configuration Settings__. The files matching the `<path>` will be uploaded and visible as the artifacts of the running build.

The message should be printed after all the files are ready and no file is locked for reading.

>To publish multiple artifact files in one archive, you need to configure the _[Artifact paths](configuring-general-settings.md#Artifact+Paths)_ in __General Settings__ of a build configuration. If you use service messages, only artifacts for the last rule will be published to the archive.
>
{style="tip"}

Artifacts are uploaded in the background, which can take time. Make sure the matching files are not deleted till the end of the build step (for example, you can put them in a directory that is cleaned on the next build start, in a [temp directory](how-to.md#Make+Temporary+Build+Files+Erased+between+the+Builds), or use [Swabra](build-files-cleaner-swabra.md) to clean them after the build).



> The process of publishing artifacts can affect the build, because it consumes network traffic, and some disk/CPU resources (should be pretty negligible for not large files/directories).
>
{style="note"}

Artifacts that are specified in the build configuration setting will be published as usual, after the finish of the last build step.

### Passing NuGet Packages Between Steps

If you need to publish NuGet packages and then use their contents within one build, you want to guarantee they are published and indexed on time — and not at the build finish.   
For this, you can use a [NuGet Publish](nuget-publish.md) runner or send the `##teamcity[publishNuGetPackage]` service message in any step instead. This ensures the NuGet packages are published in all configured NuGet feeds right at the end of the current step and are available in the following build steps.

### Reporting Build Progress

You can use special progress messages to mark long-running parts in a build script. These messages will be shown on the projects' dashboard for the corresponding build and on the __[Build Results](working-with-build-results.md)__ page.

To log a single progress message, use:

```Shell
##teamcity[progressMessage '<message>']

```

This progress message will be shown until another progress message occurs or until the next target starts (in case of Ant builds).

If you wish to show a progress message for a part of a build only, use:

```Shell
##teamcity[progressStart '<message>']
...some build activity...
##teamcity[progressFinish '<message>']

```

> The same message should be used for both `progressStart` and `progressFinish`. This allows nesting progress blocks. Note that in case of Ant builds, progress messages will be replaced if an Ant target starts.
>
{style="note"}

### Reporting Build Problems

To fail a build directly from the build script, a build problem must be reported. Build problems affect the build status text. They appear on the __[Build Results](working-with-build-results.md)__ page. To add a build problem to a build, use:


```Shell
##teamcity[buildProblem description='<description>' identity='<identity>']

```

where:
 *  `description` (mandatory): a human-readable plain text describing the build problem. By default, the `description` appears in the build status text and in the list of build's problems. The text is limited to 4000 symbols, and will be truncated if the limit is exceeded.
 * `identity` (optional): a unique problem ID. Different problems must have different identity, same problems — same identity, which should not change throughout builds if the same problem, for example, the same compilation error occurs. It must be a valid Java ID up to 60 characters. If omitted, the `identity` is calculated based on the `description` text.

<!--[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e948.txt")-->    

### Reporting Build Status

Use the `buildStatus` to set the **build status text**.

<img src="dk-serviceMessage-customBuildStatus.png" width="706" alt="Custom build status"/>

```Shell
##teamcity[buildStatus text='NuGet packages were successfully published.']
```

Unlike [progress messages](#Reporting+Build+Progress) designed to reflect currently ongoing operations in the build status, a `buildStatus` message sets the final build status that persists after a build finishes.

You can add separate steps with different [step execution conditions](build-step-execution-conditions.md) to differentiate "green" builds from failing ones, and set different custom statuses for each of them.

```Kotlin
steps {
    // ...
    script {
        id = "set-custom-green-text"
        scriptContent = """echo "##teamcity[buildStatus text='NuGet packages were successfully published.']""""
    }
    
    script {
        id = "set-custom-red-text"
        executionMode = BuildStep.ExecutionMode.RUN_ONLY_ON_FAILURE
        scriptContent = """echo "##teamcity[buildStatus text='Build failed! NuGet packages were not updated.']""""
    }
}
```

The default status text can be referenced via the `{build.status.text}` placeholder.

```Shell
##teamcity[buildStatus text='The default status of this build is: {build.status.text}']
```

#### The Status Parameter

The optional `status` parameter of the `buildStatus` service message allows you to override the appearance of the build: paint failed builds as successful and (not recommended) successful builds as failed.

* To change the build's status from failed to successful, add the `status='SUCCESS'` parameter.
    
    ```Shell
    ##teamcity[buildStatus status='SUCCESS' text='{build.status.text}, the build is marked as successful']
    ```
    
    <img src="dk-servicemessage-buildstatus-sc.png" width="706" alt="Successful status assigned manually"/>
    
* If you want a "green" build to appear as failed, you can send the following message:
    
    ```Shell
    ##teamcity[buildStatus status='FAILURE' text='The build status was manually switched to {build.status.text}']
    ```
  
    However, since that only changes the build status and does not provide any actionable insights to anyone investigating this "failed" build, we recommend sending the [`buildProblem`](#Reporting+Build+Problems) message instead. This message allows you to genuinely fail the build (and provide specific details related to this issue) rather than merely changing the final status. As a result, project maintainers will be able to use the variety of tools TeamCity offers for resolving issues: investigations, mutes, and so on.

    ```Shell
    ##teamcity[buildProblem description='The artifact size has decreased dramatically' identity='2281488']
    ```


### Reporting Build Number

To set a custom build number directly, specify a `buildNumber` message using the following format:

```Shell
##teamcity[buildNumber '<new build number>']

```

In the &lt;new build number&gt; value, you can use the `{build.number}` substitution to use the current build number automatically generated by TeamCity. For example:

```Shell
##teamcity[buildNumber '1.2.3_{build.number}-ent']

```

### Adding or Changing Build Parameter
{id='set-parameter'}

By using a dedicated service message in your build script, you can dynamically update build parameters of the build right from a build step (the parameters need to be defined in the __[Parameters](configuring-build-parameters.md)__ section of the build configuration). The changed build parameters will be available in the build steps following the modifying one. They will also be available as build parameters and can be used in the dependent builds via [` %\dep.*% parameter references`](predefined-build-parameters.md#Dependency+Parameters), for example:

```Shell
##teamcity[setParameter name='ddd' value='fff']
```

When specifying a build parameter's name, mind the prefix:
* `system` for system properties.
* `env` for environment variables.
* No prefix for configuration parameters.

[Read more](configuring-build-parameters.md) about build parameters and their prefixes.

>Since the `setParameter` mechanism does not publish anything to the server until the build is finished, it is not possible to get updated parameters during the build via the REST API.
>
{style="note"}

### Reporting Build Statistics

In TeamCity, it is possible to configure a build script to report statistical data and then display the charts based on the data. Refer to the [Customizing Statistics Charts](customizing-statistics-charts.md#Modifying+Predefined+Project-level+Charts) page for a guide to displaying the charts on the web UI. This section describes how to report the statistical data from the build script via service messages. You can publish the build statics values in two ways:
* Using a service message in a build script directly
* [Providing data using the teamcity-info.xml file](teamcity-info-xml.md#Storing+Data+in+teamcity-info.xml)   
To report build statistics using service messages: Specify a `buildStatisticValue` service message with the following format for each statistics value you want to report:

```Shell
##teamcity[buildStatisticValue key='<valueTypeKey>' value='<value>']

```

where
* `key` must not be equal to any of [predefined keys](custom-chart.md#Default+Statistics+Values+Provided+by+TeamCity).
* `value` must be a positive/negative integer of up to 13 digits; float values with up to 6 decimal places are also supported.

### Disabling Service Messages Processing

If you need for some reason to disable searching for service messages in the output, you can disable the service messages search with the messages:

```Shell
##teamcity[enableServiceMessages]
##teamcity[disableServiceMessages]

```

Any messages that appear between these two are not parsed as service messages and are effectively ignored. For server-side processing of service messages, enable/disable service messages also supports the `flowId` attribute and will ignore only the messages with the same `flowId`.

<!--[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e1141.txt")-->    

### Importing XML Reports

In addition to the [UI build feature](xml-report-processing.md), XML reporting can be configured from within the build script with the help of the `importData` service message. Also, the message supports importing of the previously collected code coverage and code inspection/duplicates reports.

The service message format is:

```Shell
##teamcity[importData type='typeID' path='<path to the xml file>']

```

> To be processed, report XML files (or a directory) must be located in the [checkout directory](build-checkout-directory.md), and the path must be relative to this directory.
>
{style="note"}

where `typeID` can be one of the following (see also [XML Report Processing](xml-report-processing.md)):

<table>

<tr>

<td>

typeID

</td>

<td>

Description

</td>

</tr><tr>

<td colspan="2">

__Testing frameworks__

</td>

</tr><tr>

<td>

`junit`

</td>

<td>

JUnit Ant task XML reports

</td></tr><tr>

<td>

`surefire`

</td>

<td>

Maven Surefire XML reports

</td></tr>

<tr>

<td>

`nunit`

</td>

<td>

NUnit-Console XML reports

</td></tr>

<tr>

<td>

`mstest`

</td>

<td>

MSTest XML reports

</td></tr>

<tr>

<td>

`vstest`

</td>

<td>

VSTest XML reports

</td></tr>

<tr>

<td>

`gtest`

</td>

<td>

Google Test XML reports

</td></tr>

<tr>

<td colspan="2">

__Code inspection__

</td></tr>

<tr>

<td>

`intellij-inspections`

</td>

<td>

IntelliJ IDEA inspection results

</td></tr>

<tr>

<td>

`checkstyle`

</td>

<td>

Checkstyle inspections XML reports

</td></tr>

<tr>

<td>

`findBugs` <sup>2)</sup>

</td>

<td>

FindBugs inspections XML reports

</td></tr>

<tr>

<td>

`jslint`

</td>

<td>

JSLint XML reports

</td></tr>

<tr>

<td>

`ReSharperInspectCode` <sup>1)</sup>

</td>

<td>

ReSharper `inspectCode.exe` XML reports

</td></tr>

<tr>

<td>

`FxCop`<sup>1</sup>

</td>

<td>

FxCop inspection XML reports

</td></tr>

<tr>

<td>

`pmd`


</td>

<td>

PMD inspections XML reports

</td></tr>

<tr>

<td colspan="2">

__Code duplication__

</td></tr>

<tr>

<td>

`pmdCpd`

</td>

<td>

PMD Copy/Paste Detector (CPD) XML reports

</td></tr>

<tr>

<td>

`DotNetDupFinder` <sup>1)</sup>

</td>

<td>

ReSharper `dupfinder.exe` XML reports

</td></tr>

<tr>

<td colspan="2">

__Code coverage__

</td></tr>

<tr>

<td>

`dotNetCoverage` <sup>1) 3)</sup>

</td>

<td>

XML reports generated by dotcover, partcover, ncover or ncover3

</td>
</tr>
</table>

Notes:
1) Only supports a specific file in the `path` attribute.
2) Also requires the `findBugsHome` attribute specified pointing to the home directory of the installed FindBugs tool.
3) Also requires the `tool='<tool name>'` service message attribute, where the `<tool name>` is either `dotcover`, `partcover`, `ncover` or `ncover3`.

If not specially noted, the report types support Ant-like wildcards in the `path` attribute.

* `verbose='true'` attribute will enable detailed logging into the build log.
* `parseOutOfDate='true'` attribute will process all the files matching the path. By default, only those updated during the build (determined by last modification timestamp) are processed. If any not matching reports are found, the "report skipped as out-of-date" message appears in the build log.
* `whenNoDataPublished=<action>` (where `<action>` is one of the following: `info` (default), `nothing`, `warning`, `error`) will change output level if no reports matching the path specified were found.
* (__deprecated, use [Build Failure Conditions](build-failure-conditions.md) instead__)   
  `findBugs`, `pmd` or `checkstyle` importData messages also take optional `errorLimit` and `warningLimit` attributes specifying errors and warnings limits, exceeding which will cause the build failure.



> After the `importData` message is received, TeamCity agent starts to monitor the specified paths on the disk and imports matching report files in the background as soon as the files appear on disk.   
The parsing only occurs within the build step in which the messages were received. On the step finish, the agent ensures all the present reports are processed before beginning the next step. This behavior is different from that of [XML Report Processing](xml-report-processing.md) build feature, which completes files parsing only at the end of the build.   
Ensure the report files are available after the generation process ends (the files are not deleted, nor overwritten by the build script)
>
{style="note"}

To initiate monitoring of several directories or parse several types of the report, send the corresponding service messages one after another.



> Only several reports of different types can be included in a build. Processing reports of several inspections or duplicates tools in a single build is not supported. See the [related feature request](https://youtrack.jetbrains.com/issue/TW-14260).
>
{style="note"}

<!--[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e1503.txt")-->


## Writing the File into the Build Log

Send the following service message to start tracking the contents of a file and write its new lines to the build log.

```Shell
##teamcity[importData type='streamToBuildLog' filePath='path-to-file' wrapFileContentInBlock='false' charset='UTF-8']
```

<img src="dk-streamFiletoLog.png" width="706" alt="Stream file to log"/>



* `type` — always equals 'streamToBuildLog'.
* `filePath` — the path to the specific file. This path can be absolute (`filePath='%\teamcity.build.checkoutDir%/temp.txt'`) or relative (`filePath='./myFolder/temp.txt'`). Relative paths are resolved under the current working directory of the agent that runs the build (the directory returned by the `teamcity.agent.work.dir` [parameter](configuring-build-parameters.md)). If the specified file does not exist or cannot be opened, TeamCity will periodically retry to access this file (for as long as the runner that sent this service message is still running).

<!--* Use `filePattern` to specify the Ant-style file pattern and monitor every file that matches it. If the pattern includes a path, it is resolved under the current working directory. While the runner that sent this service message is still running, TeamCity will periodically look for new files that match this pattern.-->

* `wrapFileContentInBlock` (optional) — specifies whether the output of this message should be placed in a collapsible "Streaming file..." block. The default value is "true".

    <img src="dk-wrapInBlock.png" width="706" alt="Wrap in block"/>

* `charset` (optional) — a canonical name or an alias of an encoding supported by Java. If the argument is absent or the encoding cannot be resolved, UTF-8 is used.

TeamCity monitors the given file for as long as the parent runner is active. When the runner stops, the file is streamed to its end and closed. 

## Sending Custom Slack Messages


You can use TeamCity service messages to send Slack direct messages and post updates in Slack channels.

<img src="dk-slack-service-message.png" width="706" alt="Custom TeamCity messages in Slack"/>

TeamCity utilizes [Slack connections](configuring-connections.md#Slack) to send Slack messages. If you do not already have a Slack connection, go to **Administration | Project Settings | Connections** and create one.

1. Open the settings of a Slack connection and set the **Notifications limit** to any positive number. This value specifies the maximum number of messages that build configurations can send per run.

2. For security reasons, only links to the same TeamCity server are allowed in messages. Messages with URLs to external web resources are automatically blocked with a corresponding message written to the "teamcity-notifications.log" file (**Administration | Diagnostics | Server Logs**).
    
    <img src="dk-servicemessage-externalURL.png" width="706" alt="Blocked service message"/>
    
    If you need to include links to external resources, enter their hostnames in the **Allowed hostnames** field. Use comma (,) as a separator to enter multiple values and asterisk (\*) as a wildcard for any string (for example, \*.test.co.uk).

3. Once a Slack connection is configured, you can add service messages to a build script in the following format:

    ```Shell
    ##teamcity[notification notifier='slack' message='Line 1 |rLine2 |rLine3' sendTo='build_farm_alerts' connectionId='PROJECT_EXT_2']
    ```

   * `notifier` — always equals "slack".
   * `message` — the message to show. Supports [Markdown](https://api.slack.com/reference/surfaces/formatting) syntax (apart from "\n" for line breaks, use "|n" or "|r" [instead](#Escaped+Values)).
     <img src="dk-slackMessages-markdown.png" width="706" alt="Markdown-formatted service messages" style="block"/>
   * `sendTo` — specifies who should receive the message. Accepts a single Slack channel name, channel ID (starts with "C", for instance, "C052UHDRZU7"), or user ID (starts with "U", for instance, "U02K2UVKJP7") as value. If you need to send the same message to multiple recipients, create multiple service messages with different `sendTo` values.
   * `connectionID` — the optional parameter that allows you to choose a specific Slack connection that TeamCity should use to send this message. Accepts connection IDs as values. If this parameter is not specified, TeamCity will retrieve all Slack connections available for the current project and choose the one whose **Notifications limit** is not zero.
   
      > To quickly get an ID of a target [Slack connection](configuring-connections.md#Slack):
      > 1. <include from="common-templates.md" element-id="open-project-settings"/>
      > 2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
      > 3. Copy a connection ID from an overview table.
      >
      >     <img src="dk-copy-connection-id.png" alt="Copy connection ID" width="706"/>
      >
      {style="tip"}

<!--
      > Currently, you cannot retrieve Slack connection IDs from the TeamCity 
        UI.
      > 
      > As a workaround, use the [REST API](teamcity-rest-api.md) to send the 
        GET request to the `/app/rest/projects/<locator>/projectFeatures` 
          endpoint and find a &lt;projectFeature&gt; element that fits your 
            connection. For instance:
      >
      > 
      > ```XML
      > <projectFeature id="PROJECT_EXT_4" type="OAuthProvider">
      >     <properties count="7" href="/app/rest/projects/id:_Root/projectFeatures/id:PROJECT_EXT_4/properties">
      >         <property name="adHocAllowedDomainNames" value="stackoverflow.com"/>
      >         <property name="adHocMaxNotificationsPerBuild" value="20"/>
      >         <property name="clientId" value="1234.56789"/>
      >         <property name="displayName" value="Slack"/>
      >         <property name="providerType" value="slackConnection"/>
      >         <property name="secure:clientSecret"/>
      >         <property name="secure:token"/>
      >     </properties>
      > </projectFeature>
      > ```
      > The connection ID is the value of the "id" field, typically in the "PROJECT_EXT_INT" format.
      > 
      {style="tip"}
-->

4. Run the build to ensure all Slack messages are delivered.


## Sending Custom Email Messages
{instance="tc"}

You can use TeamCity service messages to send emails from inside build scripts.

<img src="dk-serviceMessagesEmail-delivered.png" width="706" alt="Email delivered by a service message"/>

1. Navigate to **Administration | Email Notifier** and set up common [Notifier](set-up-notifications.md#Email+Notifier) settings: the sender's email, SMTP login and password, connection security protocol, and others. See this link for an example: [](setting-up-google-mail-as-notification-server.md).

2. Set the **Notifications limit** setting to any positive number. This value specifies the maximum number of messages that build configurations can send per run.

3. Enter the list of recipients to the **Allowed addresses**. Messages sent to recipients outside this list are automatically blocked. Use comma (,) as a separator to enter multiple addresses and asterisk (\*) as a wildcard for any string (for example, \*@gmail.com).

4. For security reasons, only links to the same TeamCity server are allowed in messages. Messages with URLs to external web resources are automatically blocked with a corresponding message written to the "teamcity-notifications.log" file (**Administration | Diagnostics | Server Logs**).

    <img src="dk-servicemessage-externalURL.png" width="706" alt="Blocked service message"/>

   If you need to include links to external resources, enter their hostnames in the **Allowed hostnames** field. Use comma (,) as a separator to enter multiple values and asterisk (\*) as a wildcard for any string (for example, \*.test.co.uk).

5. Once an Email Notifier is set up, you can add service messages to a build script in the following format:

    ```Shell
    ##teamcity[notification notifier='email' message='Message body' subject='Email subject' address='user1@gmail.com,user2@yourcompany.com']
    ```
   
   * `notifier` — always equals "email".
   * `message` — the message body in plain text format.
   * `subject` — the email subject.
   * `address` — one or multiple addresses from the **Allowed addresses** list of the Email Notifier.

6. Run the build to ensure all emails are delivered.

## Canceling Build via Service Message

If you need to cancel a build from a script, for example, if a build cannot proceed normally due to the environment, or a build should be canceled form a subprocess, you can use the following service message:

```Shell
echo "##teamcity[buildStop comment='canceling comment' readdToQueue='true']"
```

If required, you can re-add the build to the queue after canceling it. By default, TeamCity will do 3 attempts to re-add the build into the queue.

Downstream chain builds can be cancelled by sending the `skipQueuedBuilds` message from preceding (upstream) configurations. This message accepts IDs or tags of configurations that you want to skip as the `tags` parameter.

```Shell
##teamcity[skipQueuedBuilds tags='value1,value2,...' comment='Your comment']
```

See the [](build-chain.md#Partial+Chain+Execution) article for more information about cancelling builds linked in a build chain.

## Adding and Removing Build Tags

Service messages allow you to add and remove [build tags](build-actions.md#Add+Tags+to+Build).

* `##teamcity[addBuildTag 'your-custom-tag']` — adds a new tag to the current build.
* `##teamcity[removeBuildTag 'tag-to-remove']` — removes a tag from the current build.

Both messages allow you to pass values of [configuration parameters](configuring-build-parameters.md) instead of plain strings. For example, the `##teamcity[addBuildTag '%\teamcity.agent.jvm.os.name%']` message tags builds with names of operating systems installed on agent machines.

<img src="dk-servicemessage-tags.png" width="706" alt="Tagging builds with OS names"/>

One service message can add or remove a single tag. To add or remove multiple tags, send multiple service messages.

## Libraries Reporting Results via TeamCity Service Messages

Several platform-specific libraries from JetBrains and external sources are able to report the results  via TeamCity Service messages.
* [Service messages .NET library](https://github.com/JetBrains/TeamCity.ServiceMessages) — .NET library for generating (and parsing) TeamCity service messages from .NET applications. See a [related blog post](https://blog.jetbrains.com/teamcity/2011/10/teamcity-service-messages-library-for-net/).
* [Jasmine 2.0 TeamCity reporter](https://github.com/WilliamDoman/Jasmine2.0TeamCityReporter) — support for emitting TeamCity service messages from Jasmine 2.0 reporter
* [Perl TAP Formatter](https://metacpan.org/release/THALJEF/TAP-Formatter-TeamCity-0.04) — formatter for Perl to transform TAP messages to TeamCity service messages
* [PHPUnit 5.0](https://github.com/sebastianbergmann/phpunit/blob/9e86c85be3302eb125f15037ae6f496f62750a93/ChangeLog-5.0.md#500---2015-10-02) — supports TeamCity service messages for tests. For earlier PHPUnit versions, the following external libraries can be used: [PHPUnit Listener 1](https://github.com/realweb-team/deploytools), [PHPUnit Listener 2](https://github.com/maartenba/phpunit-runner-teamcity) — listeners which can be plugged via PHPUnit's` suite.xml` to produce TeamCity service messages for tests.
* [Python Unit Test Reporting to TeamCity](https://pypi.python.org/pypi/teamcity-messages) — the package that automatically reports unit tests to the TeamCity server via service messages (when run under TeamCity and provided the testing code is adapted to use it).
* [Mocha](https://github.com/visionmedia/mocha) — on-the-fly reporting via service messages for Mocha JavaScript testing framework. See the related [post](http://richarddingwall.name/2012/06/17/running-mocha-browser-tests-in-teamcity/) with instructions.
* [Karma](https://github.com/karma-runner/karma) — support in the JavaScript testing tool to report tests progress into TeamCity using TeamCity service messages.


## Perforce Service Messages

<snippet id="p4-service-message">

[Personal builds](personal-build.md) working with [Perfoce projects](integrating-teamcity-with-perforce.md) may fail during the final "Undo personal changes" stage. This usually occurs if the code source checkout directory becomes unavailable before this stage begins (for example, when it is mounted by a [Bootstrap build step](configuring-build-steps.md#Bootstrap+Steps) and automatically unmounted after the last build step).
{instance="tc"}

[Personal builds](personal-build.md) working with [Perfoce projects](integrating-teamcity-with-perforce.md) may fail during the final "Undo personal changes" stage. This usually occurs if the code source checkout directory becomes unavailable before this stage begins.
{instance="tcc"}


In such cases, you must undo personal changes earlier than TeamCity normally does — while the build steps are still running. To do so, send the following [service message](service-messages.md) during the final build step:

```Text
##teamcity[undoPersonalPatch]
```

</snippet>
