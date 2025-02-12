[//]: # (title: Adding Build Features)
[//]: # (auxiliary-id: Adding Build Features)

<snippet id="intro">

A _build feature_ is a piece of functionality that can be added to any build configuration to enable additional capabilities. For example, the [](commit-status-publisher.md) build feature publishes TeamCity build results to a VCS that stores your code files, and [](investigations-auto-assigner.md) identifies users whose latest changes broke the build and automatically tasks them with resolving these issues. Configured build features can be temporarily disabled at any moment without the need to remove them.

</snippet>

## TeamCity Build Features

TeamCity ships with the following build features that can be added to your configurations:

<dl>

<dt>AssemblyInfo Patcher</dt>
<dd>Allows you to set a build number to an assembly automatically, without having to patch the AssemblyInfo.cs files manually.&emsp;<a href="assemblyinfo-patcher.md">Learn more...</a></dd>

<dt>Automatic Merge</dt>
<dd>Tracks builds in required branches and merges them into a target branch if a build satisfies the configured condition (for example, the build is successful).&emsp;<a href="automatic-merge.md">Learn more...</a></dd>

<dt>AWS Credentials</dt>
<dd>Shares an AWS connection configured in a TeamCity project to build steps.&emsp;<a href="aws-credentials.md">Learn more...</a></dd>

<dt>Build Approval</dt>
<dd>Allows you to specify TeamCity users whose approval is required to start a new build.&emsp;<a href="build-approval.md">Learn more...</a></dd>

<dt>Build Files Cleaner (Swabra)</dt>
<dd>Cleans files produced during a build.&emsp;<a href="build-files-cleaner-swabra.md">Learn more...</a></dd>

<dt>Build Cache</dt>
<dd>Allows builds to obtain files acquired by previously finished builds (for example, downloaded <code>npm</code> packages.&emsp;<a href="build-cache.md">Learn more...</a></dd>

<dt>Commit Status Publisher</dt>
<dd>Reports build stages and final results to an external system.&emsp;<a href="commit-status-publisher.md">Learn more...</a></dd>

<dt>Docker Support</dt>
<dd>Allows build configurations to automatically sign in a DockerHub or other container registry before the build start.&emsp;<a href="docker-support.md">Learn more...</a></dd>

<dt>File Content Replacer</dt>
<dd>Uses regular expressions to replace contents of text files before the build starts, and rolls back all changes after the build finishes.&emsp;<a href="file-content-replacer.md">Learn more...</a></dd>

<dt>Free Disc Space</dt>
<dd>Checks whether a build agent has enough free disc space before it runs a build, and cleans up old build data in case it does not. &emsp;<a href="free-disk-space.md">Learn more...</a></dd>

<dt>Golang</dt>
<dd>Enables the real-time reporting and history of Go test results in TeamCity.&emsp;<a href="golang.md">Learn more...</a></dd>

<dt>Investigations Auto Assigner</dt>
<dd>Analyzes build problems and test failures, and identifies users whose commits potentially led to these problems.&emsp;<a href="investigations-auto-assigner.md">Learn more...</a></dd>

<dt>Jira Cloud Integration</dt>
<dd>Allows reporting build statuses directly to Jira Cloud in real time.&emsp;<a href="jira-cloud-integration.md">Learn more...</a></dd>

<dt>Matrix Build</dt>
<dd>Allows you to set up a matrix where multiple parameters have several potential values each. Running such build configurations spawns N separate builds that test each individual parameter/value combination.&emsp;<a href="matrix-build.md">Learn more...</a></dd>

<dt>Notifications</dt>
<dd>Notifies required users about build statuses and events via e-mails or Slack messages.&emsp;<a href="notifications.md">Learn more...</a></dd>

<dt>NuGet Feed Credentials</dt>
<dd>Allows builds to interact with NuGet feeds that require authentication.&emsp;<a href="nuget-feed-credentials.md">Learn more...</a></dd>

<dt instance="tc">NuGet Packages Indexer</dt>
<dd instance="tc">Indexes NuGet packages and adds them to TeamCity remote private feeds, with no need for additional authorization.&emsp;<a href="nuget-packages-indexer.md">Learn more...</a></dd>

<dt>Performance Monitor</dt>
<dd>Allows you to get the statistics on the CPU, disk I/O, and memory usage during a build run on a build agent. &emsp;<a href="performance-monitor.md">Learn more...</a></dd>

<dt>Pull Requests</dt>
<dd>Allows you to build changes that have not yet been merged to a target repository branch.&emsp;<a href="pull-requests.md">Learn more...</a></dd>

<dt>Ruby Environment Configurator</dt>
<dd>Adds the selected Ruby interpreter and gems bin directories to the system <code>PATH</code> environment variable and configures other necessary environment variables in case of the RVM interpreter.&emsp;<a href="ruby-environment-configurator.md">Learn more...</a></dd>

<dt>Parallel Tests</dt>
<dd>Breaks down a huge number of tests into smaller batches, and employs multiple TeamCity agents to run each batch.&emsp;<a href="parallel-tests.md">Learn more...</a></dd>

<dt>Shared Resources</dt>
<dd>Allows limiting concurrently running builds using a shared external (to the CI server) resource (for example, a test database or a server with a limited number of connections).&emsp;<a href="shared-resources.md">Learn more...</a></dd>

<dt>SSH Agent</dt>
<dd>Runs an SSH agent with the selected uploaded SSH key during a build. &emsp;<a href="ssh-agent.md">Learn more...</a></dd>

<dt>VCS Labeling</dt>
<dd>Enables automatic and manual labelling (tagging) build sources in your Version Control System.&emsp;<a href="vcs-labeling.md">Learn more...</a></dd>

<dt>XML Report Processing</dt>
<dd>Parses XML reports produced by external tools and displays results on build pages.&emsp;<a href="xml-report-processing.md">Learn more...</a></dd>
</dl>


## How to Add, Remove, and Disable Build Features

### In TeamCity UI

1. Navigate to **Administration | &lt;YOUR_PROJECT&gt; | &lt;YOUR_BUILD_CONFIGURATION&gt;** and switch to the **Build Features** tab.

    <img src="dk-add-new-build-feature.png" width="706" alt="Add New Build Feature"/>

2. Click **Add build feature** and choose a required feature from the list.

3. Specify required feature settings and click **Save**.

The **Build Features** page also allows you to temporarily disable or permanently remove a configured build feature.

<img src="dk-disable-or-delete-feature.png" width="706" alt="Disable or delete a build feature"/>

### In Kotlin DSL

Add a corresponding feature to the **Approval** block of your [Kotlin DSL](kotlin-dsl.md). For example, the following sample adds the [](build-approval.md) feature that requires one user to approve new configuration builds.


```Kotlin
object MyBuildConfig : BuildType({
    // ...
   features {
       approval {
           approvalRules = "user:Valravn"
       }
   }
})
```

See also: [Build Features | Kotlin DSL Documentation](https://www.jetbrains.com/help/teamcity/kotlin-dsl-documentation/root/build-feature/index.html).

### Using REST API

Send GET, POST, and DELETE requests to the `/app/rest/buildTypes/<build_type_locator>/features` endpoint to obtain and remove existing build features and add new ones.

For example, a POST request using the following body adds the [](pull-requests.md) feature to the configuration.

<tabs>

<tab title="JSON">

```JSON
{
    "type": "pullRequests",
    "properties": {
        "property": [
            { 
                "name": "providerType",
                "value": "github"
            },
                        { 
                "name": "ignoreDrafts",
                "value": "true"
            }
        ]
    }
}
```

</tab>

<tab title="XML">

```XML
<feature type="pullRequests">
    <properties count="2">
        <property name="ignoreDrafts" value="true"/>
        <property name="providerType" value="github"/>
    </properties>
</feature>
```

</tab>

</tabs>

To disable an existing feature, send a PUT request with the `true` text body to the `/app/rest/buildTypes/<build-type-locator>/features/<ID>/disabled`.



<seealso>
    <category ref="external">
        <a href="https://youtu.be/fttWwJG7C38">Video tutorial: Improving your first build configuration</a>
    </category>
        
</seealso>