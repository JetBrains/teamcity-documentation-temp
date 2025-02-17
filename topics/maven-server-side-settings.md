[//]: # (title: Maven Server-Side Settings)
[//]: # (auxiliary-id: Maven Server-Side Settings)

## Maven Settings Resolution on the Server Side

The TeamCity server invokes Maven on the server side for functionality like Maven dependency triggers and the Maven model display on the __Maven__ build configuration tab.

You can upload Maven settings using the **Maven Settings** tab of [project settings](project-administrator-guide.md#Edit+and+View+Modes), and then select one of the uploaded settings on the [Maven step](maven.md) settings.

During the process, TeamCity uses the usual Maven logic for finding the `settings.xml` files with several differences (see below). 

### Global Settings

Maven _global-level_ settings are used from the `.xml` file in the default Maven location for the TeamCity server process: `${env.M2_HOME}/conf/settings.xml` or `${system.maven.home}/conf/settings.xml`.   
The global values of `M2_HOME` environment variable and `maven.home` JVM option are used set for the TeamCity server process.

### User-Level Settings

Maven _user-level_ settings are defined in the __User settings selection__ [section](maven.md#User+Settings) of the Maven build step of the build configuration (if there are several Maven steps, settings from the first one are used).

The following options are available: 

<table><tr>

<td>
Value

</td>

<td>
Description

</td></tr><tr>

<td>

&lt;Default&gt;

</td>

<td>


TeamCity searches the following locations for the `settings.xml` file (listed in order of priority):

1. [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/system/pluginData/maven/settings.xml`
2. `<User Home>/.m2/settings.xml` (The home directory of the user under whom the TeamCity server process runs is used)


</td></tr><tr>

<td>

&lt;Custom&gt;

</td>

<td>

The path to the file is provided by user. The file should be available both on the server and all the agents where the build will be run.

</td></tr><tr>

<td>

Uploaded settings name

</td>

<td>

TeamCity automatically uses the specified file content both on the server and agents. Maven settings are defined on the project level: the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Maven Settings__ tab. The settings are stored in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/projects/<projectID>/pluginData/mavenSettings` directory.

<note>

The settings are available in the current project and its subprojects. To override the inherited settings, in a subproject create a new settings file with the same name as the inherited one.
</note>

</td></tr></table>

For the logic of Maven settings, refer to the related Maven [documentation](https://maven.apache.org/settings.html).

User-level settings can be configured in the [Maven Artifact Dependency Trigger](configuring-maven-triggers.md#Maven+Artifact+Dependency+Trigger).

 <seealso>
        <category ref="admin-guide">
            <a href="maven.md">Maven</a>
            <a href="configuring-maven-triggers.md">Maven Artifact Dependency Trigger</a>
        </category>
</seealso>
