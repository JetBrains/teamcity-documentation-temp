[//]: # (title: IntelliJ IDEA Project)
[//]: # (auxiliary-id: IntelliJ IDEA Project)

The _IntelliJ IDEA Project_ build runner allows you to build a project created in IntelliJ IDEA.

## Supported IntelliJ IDEA features

TeamCity IntelliJ IDEA runner supports the subset of IntelliJ IDEA features:

<table><tr>

<td>

Feature

</td>

<td>

Status

</td>

<td>

Notes, limitations

</td></tr><tr>

<td>

Java

</td>

<td>

![plus.png](plus.png)

</td>

<td>

Runner is able to compile Java projects

</td></tr><tr>

<td>

JUnit 3.x/4.x

</td>

<td>


![plus.png](plus.png), with limitations 

</td>
 
<td>

* Test runner parameters are not supported
* running Maven before tests start is not supported
* alternative JRE is not supported

</td></tr><tr>

<td>

TestNG

</td>

<td>


![plus.png](plus.png), with limitations 

</td>

<td>

* Test runner parameters are not supported
* running Maven before tests start is not supported
* running of the tests from group is not supported
* alternative JRE is not supported

</td></tr><tr>

<td>

Application run configuration

</td>

<td>


![plus.png](plus.png), with limitations 

</td>

<td>

* running Maven before tests start is not supported
* alternative JRE is not supported

</td></tr><tr>

<td>

J2EE integration

</td>

<td>

![plus.png](plus.png)

</td>

<td>

Runner is able to produce WAR and EAR archives with necessary descriptors

</td></tr><tr>

<td>

JPA

</td>

<td>

![plus.png](plus.png)

</td>

<td>

Runner adds necessary descriptors in produced artifacts

</td></tr><tr>

<td>

GWT

</td>

<td>

![plus.png](plus.png)

</td>

<td>

Runner can invoke GWT compiler and add compiler result to artifacts

</td></tr><tr>

<td>

Groovy

</td>

<td>


![plus.png](plus.png), with limitations 

</td>

<td>

Runner is able to compile projects with Groovy code and run tests written in Groovy, Groovy script run configurations are not supported

</td></tr><tr>

<td>

Android

</td>

<td>

![plus.png](plus.png)

</td>

<td>

</td></tr><tr>

<td>

Flex

</td>

<td>

![minus.png](minus.png)

</td>

<td>

</td></tr><tr>

<td>

Coverage

</td>

<td>

![minus.png](minus.png), if specified in run configurations

</td>

<td>

IntelliJ IDEA based coverage can be configured separately on the runner settings page

</td></tr><tr>

<td>

Profiling plugins

</td>

<td>

![minus.png](minus.png)

</td>

<td>

</td></tr></table>

## IntelliJ IDEA Project Settings

<snippet id="idea-project-settings">

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

<anchor name="IntelliJIDEAProject-projectPath"/>

Path to the project

</td>

<td>

Use this field to specify the path to the project file (`.ipr`) or path to the project directory (root directory of the project that contains `.idea` directory). This information is required by this build runner to understand the structure of the project.

The path should be relative to the checkout directory.

</td></tr><tr>

<td>

Detect global libraries and module-based JDK in the `*.iml` files

</td>

<td>

If this option is checked, all the module files will be automatically scanned for references to the global libraries and module JDKs when saved. This helps you ensure all references will be properly resolved.

<warning>

When this option is selected, the process of opening and saving the build runner settings may become time-consuming, because it involves loading and parsing all project module files.
</warning>

</td></tr><tr>

<td>

Check/Reparse Project

</td>

<td>

Click to reparse the project and import build settings right from the IDEA project, for example the list of JDKs.

<note>

If you update your project settings in IntelliJ IDEA — add new JDKs, libraries, don't forget to update build runner settings by clicking __Check/Reparse Project__.
</note>

</td></tr><tr>

<td>

Working directory

</td>

<td>

Enter a path to a [Build Working Directory](build-working-directory.md), if it differs from the [Build Checkout Directory](build-checkout-directory.md).

<!--[//]: # (Internal note. Do not delete. "IntelliJ IDEA Projectd178e242.txt")-->

Optional, specify if differs from the checkout directory.

</td></tr></table>

## Unresolved Project Modules and Path Variables

This section is displayed, when an IntelliJ IDEA module file (`.iml`) referenced from IPR-file:
* cannot be found
* allows you to enter the values of path variables used in the IPR-file

To refresh values in this section click __Check/Reparse Project__.

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

`<path_variable_name>`

</td>

<td>

This field appears, if the project file contains path macros, defined in the Path Variables dialog of IntelliJ IDEA's Settings dialog. In the __Set value to field__, specify a path to project resources, to be used on different build agents.

</td></tr></table>

## Project JDKs

This section provides the list of JDKs detected in the project.

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

JDK Home

</td>

<td>

Use this field to specify JDK home for the project.

<note>

For __Inspections__ and __Duplicate Finder__ builds, this JDK will be used internally to resolve the Java API used in your project.   
To run the build process itself the JDK specified in the `JAVA_HOME` environment variable will be used.
</note>

</td></tr><tr>

<td>

JDK Jar File Patterns

</td>

<td>

Click this link to open a text area, where you can define templates for the `.jar` files of the project JDK. Use Ant rules to define the `.jar` file patterns.   
The default value is used for Linux and Windows operating systems:

```Shell

jre/lib/*.jar
```

For macOS, use the following lines:

```Shell
lib/*.jar

../Classes/*.jar

```

</td></tr><tr>

<td>

IDEA Home

</td>

<td>

If your project uses the IDEA JDK, specify the location of IDEA home directory

</td></tr><tr>

<td>

IDEA Jar Files Patterns

</td>

<td>

Click this link to open a text area, where you can define templates for the `.jar` files of the IDEA JDK.

</td></tr></table>

<note>

You can use references to external properties when defining the values, like `%\system.idea_home%` or `%\env.JDK_1_3%`. This will add a [requirement](configuring-agent-requirements.md) for the corresponding property.
</note>

</snippet>

### Java Parameters

<include from="java-parameters.md" element-id="java-param"/>

## Compilation settings

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Only compile classes required to build artifacts and execute run configurations

</td>

<td>

Select whether to compile all classes in the project or only those classes which are required by run configurations or for building artifacts.

</td></tr></table>

## Artifacts

<anchor name="IntelliJIDEAProject-Artifacts"/>

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Artifacts to Build

</td>

<td>

Specify here names of the artifacts to be built that are configured in the IntelliJ IDEA project.

</td></tr></table>

## Run configurations

<anchor name="IntelliJIDEAProject-Runconfigurations"/>

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Run configurations to execute

</td>

<td>

Specify here names of IntelliJ IDEA run configurations configured in the project to execute inside TeamCity build. Supported configuration types are: JUnit, TestNG, and Application. Note that run configurations specified here should be _shared_ (via "Share" checkbox in IntelliJ IDEA [Run/Debug Configurations dialog](https://www.jetbrains.com/idea/webhelp/run-debug-configuration-application.html#common)) and checked in to the version control.

Ant and Build Artifacts tasks specified in the [Before launch](https://www.jetbrains.com/help/idea/run-debug-configuration-application.html#common) list of IDEA run configurations are supported.

</td></tr></table>

## Test Parameters

* To learn more about __Run recently failed tests first__ and __Run new and modified tests first__ options, refer to the [Running Risk Group Tests First](running-risk-group-tests-first.md) page.
* The  __Run affected tests only (dependency based)__ option will take build changes into account. With this option enabled runner will compute modules affected by current build changes and will execute only those run configurations which depend on affected modules directly or indirectly.

## Code Coverage

Specify code coverage options, for the details, refer to [IntelliJ IDEA Code Coverage](intellij-idea.md) page.

<seealso>
        <category ref="admin-guide">
            <a href="intellij-idea.md">IntelliJ IDEA Code Coverage</a>
        </category>
</seealso>