[//]: # (title: Duplicates Finder \(Java\))
[//]: # (auxiliary-id: viewpage.actionpageId113084078;Duplicates Finder \(Java\))

The _Duplicates Finder (Java)_ build runner is intended for catching similar code fragments and providing a report on discovered repetitive blocks of Java code. This runner is based on IntelliJ IDEA capabilities, so an IntelliJ IDEA project file (`.ipr`) or directory (`.idea`) is required to configure the runner.

In addition to the bundled version, it is possible to install another version of JetBrains IntelliJ Inspections and Duplicates Engine and/or change the defaults using the __[Administration | Tools](installing-agent-tools.md)__ page.
{instance="tc"}

The Duplicates Finder (Java) runner can also find Java duplicates in projects built by Maven2 or above.

<note>

In order to run inspections for your project you should have either an IntelliJ IDEA project file (`.ipr`)/project directory (`.idea`), or Maven2 or above `pom.xml` of your project checked into your version control.
</note>

This page contains reference information about the following Duplicates Finder (Java) build runner's fields.

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
 * __For Gradle project__: the path to the `.gradle` file.

This information is required by this build runner to understand the structure of the project.

>The specified path should be relative to the checkout directory.

</td></tr><tr>

<td>

Detect global libraries and module-based JDK in the `*.iml` files

</td>

<td>

_This option is available if you use an IntelliJ IDEA project._ In IntelliJ IDEA, the module settings are stored in `*.iml` files, thus, if this option is checked, all the module files will be automatically scanned for references to the global libraries and module JDKs when saved. This helps ensure that all references will be properly resolved.

<warning>

When this option is selected, the process of opening and saving the build runner settings may become time-consuming, because it involves loading and parsing all project module files.
</warning>

</td></tr><tr>

<td>

Check/Reparse Project

</td>

<td>

_This option is available if you use an IntelliJ IDEA project._ Click this button to reparse your IntelliJ IDEA project and import the build settings right from the project, for example the list of JDKs.

<note>

If you update your project settings in IntelliJ IDEA (for example, add new JDKs, libraries), remember to update the build runner settings by clicking __Check/Reparse Project__.
</note>

</td></tr><tr>

<td>

Working directory

</td>

<td>

Enter a path to the [Build Working Directory](build-working-directory.md) if it differs from the [Build Checkout Directory](build-checkout-directory.md).

<!--[//]: # (Internal note. Do not delete. "Duplicates Finder \(Java\)d129e140.txt")-->

Optional, specify if differs from the checkout directory.

</td></tr></table>

## Unresolved Project Modules and Path Variables

This section is displayed, when an IntelliJ IDEA module file (`.iml`) referenced from an IPR-file:
* cannot be found;
* allows you to enter the values of path variables used in the IPR-file.   

To refresh values in this section, click __Check/Reparse Project__.

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

<anchor name="projectJdk"/>

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

Use this field to specify the JDK home for the project.

<note>

When building with the __Ipr__ runner, this JDK will be used to compile the sources of corresponding IDEA modules. For __Inspections__ and __Duplicate Finder__ builds, this JDK will be used internally to resolve the Java API used in your project. To run the build process, the JDK specified in the `JAVA_HOME` environment variable will be used.
</note>

</td></tr><tr>

<td>

JDK Jar File Patterns

</td>

<td>

Click this link to open a text area where you can define templates for the jar files of the project JDK. Use Ant rules to define the jar file patterns. The default value is used for Linux and Windows operating systems:

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

If your project uses the IDEA JDK, specify the location of the IDEA home directory

</td></tr><tr>

<td>

IDEA Jar Files Patterns

</td>

<td>

Click this link to open a text area, where you can define templates for the jar files of the IDEA JDK.

</td></tr></table>

<note>

You can use references to external properties when defining the values, like `%\system.idea_home%` or `%\env.JDK_1_3%`. This will add a [requirement](configuring-agent-requirements.md) for the corresponding property.
</note>

<include from="java-parameters.md" element-id="java-param"/>

### Java Parameters

<anchor name="duplicatorSearcherSettingsOptionDescription"/>

## Duplicate Finder Settings

<table><tr>

<td>

Option

</td>

<td>

Description

</td></tr><tr>

<td id="duplicatorTestSourcesOptionDescription">

Test sources

</td>

<td>

If this option is checked, the test sources will be included in the duplicates analysis.

>Tests may contain the data which is duplicated intentionally, and verifying tests for duplicates may yield a lot of results creating long builds and "spamming" your reports. We recommend you not select this option.

</td></tr><tr>

<td>

Include / exclude pattern
 
</td>

<td>

Optional, specify to restrict the sources scope to run duplicates analysis on. For details, refer to the section [below](#IdeaPatterns).

</td></tr><tr>

<td id="duplicatorDetalizationLevelOptionDescription">

Detalization level

</td>

<td>

Use these options to define which elements of the source code should be distinguished when searching for repetitive code fragments. Code fragments can be considered duplicated if they are structurally similar, but contain different variables, fields, methods, types or literals. Refer to the samples below:

</td></tr><tr>

<td id="duplicatorDistinguishVariablesOptionDescription">

Distinguish variables

</td>

<td>

If this option is checked, the similar contents with different variable names will be recognized as different. If this option is not checked, such contents will be recognized as duplicated:

```Java

public static void main(String[] args) {
            int i = 0;
            int j = 0;
            if (i == j) {
                System.out.println("sum of " + i + " and " + j + " = " + i + j);
            }
 
            long k = 0;
            long n = 0;
            if (k == n) {
                System.out.println("sum of " + k + " and " + n + " = " + k + n);
            }
    }

```

</td></tr><tr>

<td id="duplicatorDistinguishFieldsOptionDescription">

Distinguish fields

</td>

<td>

If this option is checked, the similar contents with different field names will be recognized as different. If this option is not checked, such contents will be recognized as duplicated:

```Java

myTable.addSelectionListener(new SelectionListener() {
            public void widgetDefaultSelected(SelectionEvent e) {
            }
            /*.....**/
 
        });
 
myTree.addSelectionListener(new SelectionListener() {
            public void widgetDefaultSelected(SelectionEvent e) {
            }
           /*.....**/
 
        });

```

</td></tr><tr>

<td id="duplicatorDistinguishMethodsOptionDescription">

Distinguish methods

</td>

<td>

If this option is checked, the methods of similar structure will be recognized as different. If this option is not checked, such methods will be recognized as duplicated. In this case, they can be extracted and reused. Initial version:

```Java

public void buildCanceled(Build build, SessionData data) {
        /*  ... **/
        for (IListener listener : getListeners()) {
            listener.buildCanceled(build, data);
        }
    }
 
    public void buildFinished(Build build, SessionData data) {
         /*  ... **/
        for (IListener listener : getListeners()) {
            listener.buildFinished(build, data);
        }
    }

```

After analyzing code for duplicates without distinguishing methods, the duplicated fragments can be extracted:

```Java

public void buildCanceled(final Build build, final SessionData data) {
    enumerateListeners(new Processor() {
      public void process(final IListener listener) {
        listener.buildCanceled(build, data);
      }
    });
  }
 
  public void buildFinished(final Build build, final SessionData data) {
    enumerateListeners(new Processor() {
      public void process(final IListener listener) {
        listener.buildFinished(build, data);
      }
    });
  }
 
  private void enumerateListeners(Processor processor) {/*  ... **/
 
  for (IListener listener : getListeners()) {
      processor.process(listener);
    }
  }
 
  private interface Processor {
    void process(IListener listener);
  }

```

</td></tr><tr>

<td id="duplicatorDistinguishTypeOptionDescription">

Distinguish types

</td>

<td>

If this option is checked, the similar code fragments with different type names will be recognized as different. If this option is not checked, such code fragments will be recognized as duplicates.

```Java

new MyIDE().updateStatus()

new TheirIDE().updateStatus()

```

</td></tr><tr>

<td id="duplicatorDistinguishLiteralesOptionDescription">

Distinguish literals

</td>

<td>

If this option is checked, similar line of code with different literals will be considered different. If this option is not checked, such lines will be recognized as duplicates.

```Java
myWatchedLabel.setToolTipText("Not Logged In");

```

```Java
myWatchedLabel.setToolTipText("Logging In...");

```

</td></tr><tr>

<td id="duplicatorIgnoreComplexityOptionDescription">

Ignore duplicates with complexity lower than

</td>

<td>

Complexity of the source code is defined by the amount of statements, expressions, declarations and method calls. Complexity of each of them is defined by its cost. Summarized costs of all these elements of the source code fragment yields the total complexity. Use this field to specify the lowest level of complexity of the source code to be taken into consideration when detecting duplicates. For meaningful results start with value 10.

</td></tr><tr>

<td id="duplicatorDistinguishSubexprOptionDescription">

Ignore duplicate subexpressions with complexity lower than

</td>

<td>

Use this field to specify the lowest level of complexity of subexpressions to be taken into consideration when detecting duplicates.

</td></tr><tr>

<td>

Check if Subexpression Can be Extracted

</td>

<td>

If this option is checked, the duplicated subexpressions can be extracted.

</td></tr></table>

<anchor name="IdeaPatterns"/>

Include / exclude patterns are newline-delimited set of rules of the form:

```Shell

[+:|-:]pattern

```

Where the pattern must satisfy these rules:
* must end with either `**` or `*` (this effectively limits the patterns to only the directories level, they do not support file-level patterns)
* references to modules can be included as `[module_name]/<path_within_module>`
Some notes on patterns processing:
* excludes have precedence over includes
* if include patterns are specified, only directories matching these patterns will be included, all other directories will be excluded
* "include" pattern has a special behavior (due to underlying limitations): it includes the directory specified and all the files residing directly in the directories above the one specified.

Example:

```Shell

+:testData/tables/**   
-:testData/**   
-:testdata/**   
-:[testData]/**

```

>For the file paths to be reported correctly, the "_References to resources outside project/module file directory_" option for the project and all modules should be set to "Relative" in IDEA project.