[//]: # (title: Maven)
[//]: # (auxiliary-id: Maven)

The _Maven_ build runner allows using [Apache Maven](https://maven.apache.org/) for automating builds.

Note that you can create a new Maven-based build configuration [automatically from URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL), and set up a [dependency build trigger](configuring-maven-triggers.md#Maven+Artifact+Dependency+Trigger), if a specific Maven artifact has changed.

>This build runner has some [Remote Run limitations](#Remote+Run+limitations).
> 
{style="note"}

>Note that TeamCity Cloud currently doesn't support automatic delivery of tools to [build agents](build-agent.md). To be able to use this runner, you need to download and install the required version of NuGet on the agent. You can do this manually (only on self-hosted agents) or via any convenient utility step at the beginning of the build (for example, [Command Line](command-line.md)). When configuring a NuGet build step, you will need to specify the path to NuGet relatively to the [build checkout directory](build-checkout-directory.md).
>
{type="warning" instance="tcc"}

This **video guide** explains [how to get the best from platform-specific build runners](https://www.youtube.com/watch?v=wLmLgh5OK5o): it uses Maven as an example.

## Maven runner settings

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Goals

</td>

<td>

Specify the sequence of space-separated Maven goals that you want TeamCity to execute. Some Maven goals can use version control systems, thus they may become incompatible with some [VCS checkout modes](configuring-vcs-settings.md#Checkout+Settings).  
To execute such a goal, select "_Automatically on agent_" in the __[VCS Checkout Mode](vcs-checkout-mode.md)__ drop-down menu on the __Version Control Settings__ page. This makes the version control system available to the goal execution software. On how to use the `release:prepare` goal with Perforce Helix Core, see [this section](#Using+Maven+Release+with+Perforce).

</td></tr><tr>

<td>

Path to POM file

</td>

<td>

Specify the path to the POM file relative to the [build working directory](build-working-directory.md).   
By default, the property contains a `pom.xml` file. If you leave this field empty, the same value is put in this field. The path may also point to a subdirectory, and as such `<subdirectory>/pom.xml` is used.

</td></tr><tr>

<td>

Additional Maven command line parameters

</td>

<td>

Specify the list of command-line parameters.

>The following parameters are ignored: `-q`, `-f`, `-s` (if __User settings path__ is provided).
> 
{style="note"}

</td></tr><tr>

<td>

Working directory

</td>

<td>

Specify the [build working directory](build-working-directory.md) if it differs from the [build checkout directory](build-checkout-directory.md).

</td></tr></table>

>When TeamCity discovers a Maven build step automatically, it sets the goal automatically to `clean test` and an additional Maven command-line parameter `-Dmaven.test.failure.ignore` to `true` to ignore failed tests. This parameter is helpful when the `test` goal is used for a Maven project with multiple modules. With this property set to `true`, even if the tests fail in some module, all the following modules will be tested as well.   
If you change the goal from `test` to `deploy` (or any other sequential goal from the Maven build lifecycle: `package`, `verify`, or `install`), make sure to set `-Dmaven.test.failure.ignore` to `false` so the failed tests are not ignored.

### Maven Settings

Choose the Maven version you want to use. You can also [manage the installed versions](installing-agent-tools.md).
{instance="tc"}

Choose the Maven version you want to use.
{instance="tcc"}

<table>

<tr>

<td>

Setting

</td>

<td>

Description

</td>

</tr>

<tr>

<td>

&lt;Auto&gt;

</td>

<td>

The path to the Maven installation is taken from the `M2_HOME` environment variable, otherwise the current default version is used.

</td></tr><tr>

<td>

&lt;Default&gt;

</td>

<td>

The bundled version is used as default. See how to [change the defaults](installing-agent-tools.md).
{instance="tc"}

The bundled version is used as default.
{instance="tcc"}

</td></tr><tr>

<td>

&lt;Custom&gt;

</td>

<td>

Provide a path to a custom Maven version.

</td></tr></table>

### User Settings

Specify what kind of user settings to use here. This is equivalent to the Maven command-line option `-s` or `--settings`. The available options are:

<table>

<tr>

<td>

Option

</td>

<td>

Description

</td>

</tr>

<tr>

<td>

&lt;Default&gt;

</td>

<td>

Settings are taken from the default Maven locations on the agent. For the server logic, see [Maven Server-Side Settings](maven-server-side-settings.md).

</td></tr><tr>

<td>

&lt;Custom&gt;


</td>

<td>

Enter the path to an alternative user settings file. The path should be valid on the agent and also on the server, see [Maven Server-Side Settings](maven-server-side-settings.md).

</td></tr><tr>

<td>

Predefined settings

</td>

<td>

If there are settings files uploaded to the TeamCity server via the administration UI, you can select one of the available options here. To upload settings file to TeamCity, click _Manage settings files_.  Maven settings are defined on the project level. You can see the settings files defined in the current project or upload files on the [project settings](project-administrator-guide.md#Edit+and+View+Modes) page using __Maven Settings__. The files will be available in the project and its subprojects. The uploaded files are stored in the `<TeamCity Data Directory>/config/projects/%\projectID%/pluginData/mavenSettings` directory. If necessary, they can be edited right there. The uploaded files are used both for the agent and server-side Maven functionality.   
If Custom or Predefined settings are used, the path to the effective user settings file is available inside the maven process as the `teamcity.maven.userSettings.path` system property.

</td></tr></table>

### Java Parameters

<include from="java-parameters.md" element-id="java-param"/>

### Local Artifact Repository Settings


Select one of the following options:

<table id="Maven-LocalArtifactRepositorySettings">

<tr>
<td>

Option

</td>
<td>

Description

</td>
</tr>

<tr>
<td>

Per agent (default)

</td>
<td>

Use a separate repository to store artifacts, produced by all builds run by an agent, under the agent system directory.

</td>
</tr>

<tr>
<td>

Per build configuration

</td>
<td>

Use a separate repository to store artifacts, produced by all builds of the current build configuration.

</td>
</tr>

<tr>
<td>

Maven default

</td>
<td>

Use the default Maven repository location. The repository is shared between all build configurations and all agents on the machine.

The runner will use the location specified in the additional command-line parameter `-Dmaven.repo.local`. If the parameter is not specified, it will search for values set in `settings.xml`.

</td>
</tr>

</table>

### Incremental Building

Select the _Build only modules affected by changes_ checkbox to enable incremental building of Maven modules.   
The general idea of incremental building is to process only changed modules without spending time on reprocessing unchanged modules they are connected with. TeamCity utilizes this method to run tests only for changed Maven modules thus saving time when rerunning a build or a build chain.

Since Maven itself has very limited support for incremental builds, TeamCity uses its own change impact analysis algorithm for determining the set of affected modules and uses a special preliminary phase for making dependencies of the affected modules.

First TeamCity performs own change impact analysis taking into account parent relationship and different dependency scopes and determines affected modules. Then the build is split into two sequential Maven executions.

The first Maven execution called preparation phase is intended for building the dependencies of the affected modules. The preparation phase is to assure there will be no compiler or other errors during the second execution caused by the absence or inconsistency of dependency classes.

The second Maven execution called main phase executes the main goal (for example, `test`), thus performing only those tests affected by the change.

Also, check the related [blog post](https://blog.jetbrains.com/teamcity/2012/03/incremental-building-with-maven-and-teamcity/) on the topic.

### Docker Settings

In this section, you can specify a Docker image which will be [used to run the build step](container-wrapper.md).

### Code Coverage

The Maven build runner supports code coverage based on the IDEA coverage engine. To learn about configuring code coverage options, refer to the [Configuring Java Code Coverage](configuring-java-code-coverage.md) page.

>Only Surefire version 2.4 or later is supported.
>
{style="note"}

If you have several build agents installed on the same machine, by default they use the same local repository. However, there are two ways to allocate a custom local repository to each build agent:

* Specify the following property in `teamcity-agent/conf/buildAgent.properties`:

    ```
    system.maven.repo.local=%\system.agent.work.dir%/<subdirectory_name>
    
    ```   
    For instance, `%\system.agent.work.dir%/m2-repository`.

* Run each build agent under different user account.

## Maven Release with Different VCSs

To run the `release:prepare` Maven task with different VCSs supported by TeamCity, make sure you are using at least 2.0 version of the [Maven Release plugin](https://maven.apache.org/maven-release/maven-release-plugin/).

### Using Maven Release with Perforce

The Maven Release plugin needs a [ticket](https://www.perforce.com/manuals/p4sag/Content/P4SAG/superuser.basic.auth.tickets.html) to authenticate in Perforce.

In the [Perforce VCS root](perforce.md) settings of your build configuration in TeamCity:
1. Enable the [checkout on agent](perforce.md#Agent+Checkout+Settings).
2. Enable [Use ticket-based authentication](perforce.md#P4+Connection+Settings) in Perforce VCS root settings.
3. Make sure your build agent environment doesn't have any occasional P4 variables which can interfere with the execution of Maven Release Plugin.
4. Specify `release:prepare` in the _Goals_ field of the Maven build step and run the build.

### Using Maven Release with Git VCS

To use this plugin with Git, set a Git SSH URL as [SCM URL](https://maven.apache.org/scm/git.html) in your `pom.xml`.

On the TeamCity agent:
1. Make sure the agent has Git installed and added to the agent's `$PATH` on Unix-like OS's and to the `%\PATH%` environment variable on Windows.
2. On the agent, set your account's identity by executing
    ```Shell
    git config --system user.email "buildserver@example.com"
    git config --system user.name "TeamCity Server"
    ```
3. Make sure your Git VCS is added to the known hosts database on the agent.

On the TeamCity server:
1. Upload a [Git SSH key](ssh-keys-management.md) to your TeamCity server.
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Version Control Settings"/></include>
3. Enable the checkout on the agent.
4. In your Git VCS root, enable _Private Key_ authentication.
5. Add the [SSH Agent](ssh-agent.md) build feature to your configuration.
6. Specify `release:prepare` in the __Goals__ field of the Maven build step and run the build.

## Remote Run limitations

Remote Run Limitations related to the Maven runner:

As a rule, a personal build in TeamCity doesn't affect any "regular" builds run on the TeamCity server, and its results are visible to its initiator only. However, in case of using Maven runner, this behavior may differ.

TeamCity doesn't interfere anyhow with the Maven dependencies model. Hence, if your Maven configuration deploys artifacts to a remote repository, __they will be deployed there even if you run a personal build__. Thereby, a personal build may affect builds that depend on your configuration.   
For example, you have a configuration A that deploys artifacts to a remote repository, and these artifacts are used by configuration B. When a personal build for A has finished, your personal artifacts will appear in B. This can be especially injurious, if configuration A is to produce release-version artifacts, because proper artifacts will be replaced with developer's ones, which will be hard to investigate because of Maven versioning model. Plus, these artifacts will become available to all dependent builds, not only to those managed by TeamCity.  
To avoid this, we recommend not using remote run for build configurations which perform deployment of artifacts.

<seealso>
        <category ref="concepts">
            <a href="build-runner.md">Build Runner</a>
        </category>
        <category ref="admin-guide">
            <a href="configuring-maven-triggers.md">Maven Artifact Dependency Trigger</a>
            <a href="creating-and-editing-build-configurations.md">Creating Maven Build Configuration</a>
        </category>
</seealso>