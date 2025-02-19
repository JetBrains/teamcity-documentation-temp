[//]: # (title: Inspections)
[//]: # (auxiliary-id: Inspections)

The _Inspections (IntelliJ IDEA)_ build runner is intended to run code analysis based on [IntelliJ IDEA inspections](https://www.jetbrains.com/help/idea/code-inspection.html) for your project. In addition to the version that TeamCity automatically downloads and installs on its first startup, it is possible to install another version of JetBrains IntelliJ Inspections and Duplicates Engine and/or change the defaults using the __[Administration | Tools](installing-agent-tools.md)__ page.
{instance="tc"}

The _Inspections (IntelliJ IDEA)_ build runner is intended to run code analysis based on [IntelliJ IDEA inspections](https://www.jetbrains.com/help/idea/2016.3/code-inspection.html) for your project. In addition to the version that TeamCity automatically downloads and installs on its first startup, it is possible to install another version of JetBrains IntelliJ Inspections and Duplicates Engine.
{instance="tcc"}

IntelliJ IDEA's code analysis engine is capable of inspecting your Java, JavaScript, HTML, XML and other code and allows you to
* find probable bugs;
* locate "dead" code;
* detect performance issues;
* improve the code structure and maintainability;
* ensure the code conforms to guidelines, standards, and specifications.

Refer to [IntelliJ IDEA documentation](https://www.jetbrains.com/idea/documentation/inspections.jsp) for more details.

<note>

To run inspections for your project, you must have IntelliJ IDEA project directory (`.idea`) or project file (`.ipr`) checked into your version control.   
Note that compilation errors are not reported by inspections, so it's a good idea to run compilation step in the same build.
</note>

<note>

__Maven projects__

The runner also supports Maven2 or above: to use `pom.xml`, you need to open it in IntelliJ IDEA and configure inspection profiles as described in the [IntelliJ IDEA documentation](https://www.jetbrains.com/help/idea/customizing-profiles.html). IntelliJ IDEA will save your inspection profiles in the [corresponding directory](https://www.jetbrains.com/help/idea/tuning-the-ide.html#default-dirs). Make sure you have it checked into your version control. Then specify the paths to the inspection profiles while configuring this runner.

It is a good idea to execute `mvn install` as the step preceding the Inspections step in order to allow projects with dependencies to be resolved successfully.
</note>

This page contains reference information about the __Inspections (IntelliJ IDEA)__ build runner fields.

## IntelliJ IDEA Project Settings

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Project file type

</td>

<td>

To be able to run IntelliJ IDEA inspections on your code, TeamCity requires either an IntelliJ IDEA project file/directory, Maven `pom.xml` or Gradle `build.gradle` to be specified here.

</td></tr><tr>

<td>

Path to the project

</td>

<td>

Depending on the type of project selected in the __Project file type__, specify here:

* __For IntelliJ IDEA project__: the path to the project file (`.ipr`) or the path to the project directory the root directory of the project containing the `.idea` directory).
* __For Maven project__: the path to the `pom.xml` file. 
* __For Gradle project__: the path to the `.gradle` or `.gradle.kts` file.    
This information is required by this build runner to understand the structure of the project.

>The specified path should be relative to the checkout directory.

</td></tr><tr>

<td>

Detect global libraries and module-based JDK in the `*.iml` files

</td>

<td>

_This option is available if you use an IntelliJ IDEA project to run the inspections._ In IntelliJ IDEA, the module settings are stored in `*.iml` files, thus, if this option is checked, all the module files will be automatically scanned for references to the global libraries and module JDKs when saved. This helps ensure that all references will be properly resolved.

<warning>
When this option is selected, the process of opening and saving the build runner settings may become time-consuming, because it involves loading and parsing all project module files.
</warning>

</td></tr><tr>

<td>

Check/Reparse Project

</td>

<td>

_This option is available if you use an IntelliJ IDEA project to run the inspections._ Click this button to reparse your IntelliJ IDEA project and import the build settings right from the project, for example the list of JDKs.

<note>

If you update your project settings in IntelliJ IDEA (for example, add new JDKs, libraries), remember to update the build runner settings by clicking __Check/Reparse Project__.
</note>

</td></tr><tr>

<td>

Working directory

</td>

<td>

Enter a path to a [Build Working Directory](build-working-directory.md) if it differs from the [Build Checkout Directory](build-checkout-directory.md).

<!--[//]: # (Internal note. Do not delete. "Inspectionsd166e196.txt")-->    


Optional, specify if differs from the checkout directory.

</td></tr></table>

## Unresolved Project Modules and Path Variables

This section is displayed when an IntelliJ IDEA module file (`.iml`) referenced from an IntelliJ IDEA project file:
* cannot be found
* allows you to enter the values of path variables used in the IPR-file.

To refresh the values in this section, click __Check/Reparse Project__.

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

&lt;path\_variable\_name&gt;

</td>

<td>

This field appears if the project file contains path macros, defined in the _Path Variables_ dialog of the IntelliJ IDEA settings. In __Set value to field__, specify a path to the project resources to be used on different build agents.

</td></tr></table>

## Project SDKs

This section provides the list of SDKs detected in the project.

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

Use this field to specify the JDK home for the project.

<note>

When building with the __Ipr__ runner, this JDK will be used to compile the sources of corresponding IDEA modules. For __Inspections__ and __Duplicate Finder__ builds, this JDK will be used internally to resolve the Java API used in your project.    
To run the build process, the JDK specified in the `JAVA_HOME` environment variable will be used.
</note>

</td></tr><tr>

<td>

JDK Jar File Patterns

</td>

<td>

Click this link to open a text area where you can define templates for the jar files of the project JDK. Use Ant rules to define the jar file patterns. The default value is used for Linux and Windows operating systems:

```
jre/lib/*.jar

```

For macOS, use the following lines:

```
lib/*.jar

../Classes/*.jar

```

</td></tr><tr>

<td>

IDEA Home

</td>

<td>

If your project uses the IDEA JDK, specify the location of the IDEA home directory

</td></tr><tr>

<td>

<anchor name="Inspections-IdeaPatterns"/>

IDEA Jar Files Patterns

</td>

<td>

Click this link to open a text area, where you can define templates for the jar files of the IDEA JDK.

</td></tr></table>

<note>

You can use references to external properties when defining the values, like `%\system.idea_home%` or `%\env.JDK_1_3%`. This will add a [requirement](configuring-agent-requirements.md) for the corresponding property.
</note>

### Java Parameters

<include from="java-parameters.md" element-id="java-param"/>

## Inspection Parameters

In IntelliJ IDEA-based IDEs, the code inspections reported are configured by an [inspection profile](https://www.jetbrains.com/idea/webhelp/code-inspection.html#d559109e437).    
When running the inspections in TeamCity, you can specify the inspection profile to use: first you need to configure the inspection profile in IntelliJ IDEA-based IDE and then specify it in TeamCity.

Follow these rules when preparing inspection profiles:
* if your inspection profile uses scopes, make sure the scopes are shared;
* lock the profile (this ensures that inspections present in TeamCity but not enabled in your IDEA installation will not be run by TeamCity);
* ensure the profile does not have inspections provided by plugins not included into the default IntelliJ IDEA Ultimate distribution (otherwise they will just be ignored by TeamCity);
* for best results, edit the inspection profile in the IntelliJ IDEA of the same version as used by TeamCity (can be looked up in the inspection build log).

The logic of selecting an inspection profile is as follows:
* if the path to the inspection profile is specified, then the profile will be loaded from the file. If the loading fails, the inspection runner will fail too.
* if the name of the inspection profile is specified, the profile is searched for in the project's shared profiles. If there is no such profile, the inspection runner will fail.
* if neither the name nor path is specified, the default profile of the project is used.

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Inspections profile path

</td>

<td>

Use this text field to specify the path to inspections profiles file relative to the project root directory. Use this field only if you do not want to use the shared project profile specified in "Inspections profile name".

</td></tr><tr>

<td>

Inspections profile name

</td>

<td>

Enter the name of the desired shared project profile. If the field is left blank and no profile path is specified, the default project profile will be used.

</td></tr><tr>

<td>

Include / exclude patterns:

</td>

<td>

Optional, specify to restrict the sources scope to run Inspections on.

</td></tr></table>

<!--[//]: # (Internal note. Do not delete. "Inspectionsd166e394.txt")-->

Include/exclude patterns are newline-delimited set of rules of the form:

```
[+:|-:]pattern

```

where the pattern must satisfy the following rules:
* must end with either `**` or `*` (this effectively limits the patterns to only the directories level, they do not support file-level patterns);
* references to modules can be included as `[<IDEA_module_name>]/<path_within_module>`. If you have a Maven project configured, you can use Maven module's artifactId as `<IDEA_module_name>`;
* the configured paths are treated as relative paths within content roots of the IDEA project modules. That is, the paths should be relative to the module's roots.

Some notes on patterns processing:
* excludes have precedence over includes
* if include patterns are specified, only directories matching these patterns will be included, all other directories will be excluded
* the include pattern has a special behavior (due to underlying limitations): it includes the directory specified and all the files residing directly in the directories above the one specified.

Example:

```
+:testData/tables/**
-:testData/**
-:testdata/**
-:[testData]/**

```

>For the file paths to be reported correctly, the "_References to resources outside project/module file directory_" option for the project and all modules should be set to "Relative" in IDEA project.

For Maven inspections run, to ensure correct Java is used for the project JDK, define `env.JAVA_HOME` configuration parameter pointing to the JDK to be used as the project JDK.

## Getting the same results in IntelliJ IDEA and TeamCity Inspections Build

The code inspections reported by IntelliJ IDEA and TeamCity Java Code Inspections build depend on a number of factors. You would need to ensure equal settings in IntelliJ IDEA and the build to get the same reports. The relevant settings include:
* [inspections profile](#Inspection+Parameters) used in IntelliJ IDEA and TeamCity build;
* environment-specific project dependencies (files not in version control, and so on);
* IDE-level settings, like defined SDKs, path variables, and so on;
* generated files: should be present in the TeamCity agent if they are present when working with the project in IntelliJ IDEA;
* IntelliJ IDEA version. It is recommended to use the same IntelliJ IDEA version that is used in the TeamCity build. The version is written in the Inspections build log;
* the set and versions of the IntelliJ IDEA plugins that the project relies on.
