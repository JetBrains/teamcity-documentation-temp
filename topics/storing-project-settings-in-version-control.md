[//]: # (title: Storing Project Settings in Version Control)
[//]: # (auxiliary-id: Storing Project Settings in Version Control)

TeamCity allows synchronizing project settings with the version control repository (VCS). Supported VCSs are Git, Mercurial, Perforce, Subversion, and Azure DevOps Server (formerly TFS).

You can store project settings in the XML format or in the [Kotlin language](https://kotlinlang.org/) and define settings programmatically using the [Kotlin-based DSL](kotlin-dsl.md).

> If your build configuration targets a repository where non-trusted users can push commits or create pull (merge) requests, do not configure [VCS triggers](configuring-vcs-triggers.md) and/or [Pull Requests build features](pull-requests.md) that automatically run builds with these changes. Instead, start new builds manually after you inspect and verify incoming changes. Otherwise, TeamCity can execute malicious code introduced in these changes (for example, handle a [service message](service-messages.md) sent from the source code or apply altered [project settings](storing-project-settings-in-version-control.md) from modified [project settings directory](#Custom+Settings+Path) files).
> 
> See this section for more information about potential damage caused by users who can modify repository code: [](security-notes.md#manage-permissions).
> 
{style="warning"}



## Key Takeways

* You can store settings of individual projects in an XML or Kotlin formats in remote repositories.
* By default, versioned settings are stored in the `.teamcity` directory in the root of the VCS repository, in the same format as in the [TeamCity Data Directory](teamcity-data-directory.md). You can [change this default path](#Choosing+the+Settings+Location) to any custom location within a remote repository.
* Project settings can be saved to the same repo that hosts application sources, or a [completely separate repository](#Separate+VCS+Root).
* You can choose whether a project [can be edited](#SynchronizingSettingswithVCS) via TeamCity UI (in this case TeamCity synchronizes edits made in the UI with remotely stored settings) or only by modifying settings files on the VCS side.
* Different repository branches can store [different project settings](#Example%3A+Branch-Specific+Settings).
* Global server-wide settings can be stored and managed via HashiCorp Terraform using the [dedicated TeamCity Provider](#Storing+and+Managing+Global+Server+Settings).

<anchor name="StoringProjectSettingsinVersionControl-SynchronizingSettingswithVCS"/>

## Synchronizing Settings with VCS
{id="SynchronizingSettingswithVCS" help-id="SynchronizingSettingswithVCS"}

By default, the synchronization of the project settings with the version control system is disabled.

To enable it, go to __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Versioned Settings | Configuration__. The "_Enable/disable versioned settings_" permission is required (default for the [System Administrator](managing-roles-and-permissions.md#Per-Project+Authorization+Mode) role).
{instance="tc"}

To enable it, go to __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Versioned Settings | Configuration__. The "_Enable/disable versioned settings_" permission is required (default for the [System Administrator](managing-roles-and-permissions.md#Default+User+Roles) role).
{instance="tcc"}

<img src="dk-versioned-settings-main.png" width="706" alt="Main versioned settings page"/>

You can choose one of the following options on this page:

* Use the same settings as in the parent project (default).
* Disable synchronization.
* Enable synchronization. In this case, you can also define which settings to use when the build starts. See details [below](#Defining+Settings+to+Apply+to+Builds).

If synchronization is enabled, it can work in either _two-way_ or _one-way_ mode.

<anchor name="two-way-sync"/>

The default mode is a _two-way synchronization_. This mode is enabled when the _Allow editing project settings via UI_ option is checked.

<img src="dk-vs-allowUIedits.png" width="706" alt="Enable editing in UI"/>

The two-way synchronization works as follows:

* Each administrative change made to the [project settings](project-administrator-guide.md#Edit+and+View+Modes) in the TeamCity UI is committed to the version control system. The author of the commited changes matches the TeamCity user who made related project edits.

* If the changes are applied on the VCS side (if a Kotlin or XML settings file is edited), the TeamCity server detects them and modifies the project on the fly.
  
   Before applying the newly checked-in settings, TeamCity validates them. If the validation fails (for example, when a build configuration references a non-existent VCS root or has duplicate ID), the current project settings are left intact and an error is shown in the UI.

If you disable the _Allow editing project settings via UI_ option, the project settings become read-only in the UI and only reflect changes made on the VCS side. This is convenient if you prefer defining project settings' [as code](kotlin-dsl.md) or load settings from a read-only VCS branch.

Enabling synchronization for a project also enables it for all its subprojects with the default "_Use settings from a parent project_" option selected. TeamCity synchronizes all changes to the project settings (including modifications of [build configurations](managing-builds.md), [templates](build-configuration-template.md), [VCS roots](vcs-root.md), and so on) except [SSH keys](ssh-keys-management.md). To exclude individual subprojects from the synchronization, switch them to **Synchronization disabled** mode.

As soon as you enable settings synchronization, TeamCity commits the current project tree and server settings to the remote repository. If the target location already stores project settings, a warning pops up. This warning allows you to choose whether TeamCity should:
* overwrite the settings in the VCS with the current project settings on the TeamCity server (only if the two-way [synchronization](#two-way-sync) is enabled); or
* import the settings from the VCS replacing the current project settings on the TeamCity server with those from version control.

>If you choose to generate [portable DSL scripts](kotlin-dsl.md#Getting+Started+with+Kotlin+DSL) and the current project comprises less than 20 entities, TeamCity will create one `settings.kts` file to define them all. Individual entities include _projects_, _build configurations_, _templates_, and _VCS roots_.  
>If there are more than 20 entities in the project, TeamCity will create a hierarchy of separate `.kt` files to define and group these entities.

## Choosing the Settings Location

The default location for TeamCity project settings is the `.teamcity` folder in the root of the same repository that stores the target project. Depending on your workflow specifics and business needs, this default setup might not be optimal for your team.

For example, teams working with monorepos where stand-alone microservices and external libraries are hosted in adjacent directories would likely want to set up individual TeamCity projects that store their settings in separate directories. Using custom settings directories for each project ensures that projects targeting the same monorepo do not constantly override each other's settings.

Another scenario you might want to implement is moving TeamCity-specific files away from the sources. This approach obscures the specifics of your CI/CD ecosystem, hiding them from external parties. In addition, having a dedicated VCS repository that stores settings of your entire TeamCity server (each project has its own repository folder to store its settings) can also prove beneficial for settings maintenance and testing.

To implement these or similar tasks, use the **Project settings VCS root** and **Settings path in VCS** settings.

### Separate VCS Root


The **Project settings VCS root** selector allows you to choose which [](vcs-root.md) TeamCity should use to obtain and commit project settings. You can choose any root owned by either this project directly, or by any of its parent projects.

<img src="dk-versioned-settings-chooseroot.png" width="706" alt="Choose a VCS root"/>


Since a VCS root defines a connection to a remote repository, choosing a root means choosing a repository that should store project settings.

* If you use the same root as the one that retrieves code changes and checks out source files, project settings will be stored in the same repository as your application. Use this setup whenever you want to keep TeamCity settings side-by-side with source files.
* Using a separate root for settings synchronization allows you to move project settings to a separate repository. This approach can be beneficial for public repositories with 3rd-party contributors, since storing TeamCity project settings in a separate repository ensures your project cannot be altered by untrusted parties. You can make this repository private to furthermore conceal the inner workings of your CI/CD setup.

Note that the **Project settings VCS root** combo-box does not allow you to create new roots. You need to navigate to the **VCS Roots** tab of your project settings and set up a required root before you can start using it on the **Versioned Settings** page.


### Custom Settings Path


The **Settings path in VCS** option allows you to manually specify a path to the directory that stores project settings.

<img src="dk-vcs-settings-custompath.png" width="706" alt="Custom settings path"/>

You can change the default `.teamcity` value to any custom path (for example, `.teamcity-settings/accounting`) or a period (`.`). The latter allows you to save settings directly to the repository root.

> Before committing a new revision of project settings, TeamCity clears the target settings directory. To prevent TeamCity from wiping important files, make sure this directory is not used for anything but project settings. Be extra careful when specifying "." as the settings directory: this value should only be used if you want a dedicated repository that stores TeamCity project settings and nothing else.
> 
{style="warning"}


To prevent unexpected errors caused by ambiguous settings source, TeamCity does not allow you to change the settings path when the synchronization is already active. To modify this path, disable the synchronization and save the settings, then re-enable it and specify the required directory.

> When you create new projects, TeamCity currently detects existing project settings only if they are stored in the default `.teamcity` folder. If project settings are stored in a custom repository directory, TeamCity does not offer options to import these settings or start a new project from scratch. As a workaround, do the following:
> 1. Create a new project from a remote repository.
> 2. Navigate to [project settings](project-administrator-guide.md#Edit+and+View+Modes) and enable synchronization on the **Versioned Settings** page.
> 3. Specify the path to existing project settings in the **Settings path in VCS** field.
> 4. Click **Apply** to save your settings, then **Load project settings from VCS...** at the bottom of the page.
>
{style="note"}



<anchor name="StoringProjectSettingsinVersionControl-DefiningSettingstoApplytoBuilds"/>

## Defining Settings to Apply to Builds

When TeamCity needs to start a build, it can apply either of the two possible settings:

* Current settings on the TeamCity server. These are settings that include all latest changes applied to the server either via TeamCity UI or via a commit to the [project settings directory](#Custom+Settings+Path) in the VCS.

* Custom settings stored in the VCS. These are settings from the [project settings directory](#Custom+Settings+Path) stored in a non-default branch or in the specific revision selected for a build.

An ability to choose which of these two settings to apply grants you the following options:

* Have [multiple branches](working-with-feature-branches.md) with different settings in the [project settings directory](#Custom+Settings+Path). This means your branch A can have parameters, steps, build features, artifact publishing rules and chain settings that differ from those in branch B.

* Start [personal builds](personal-build.md) with changes made in the [project settings directory](#Custom+Settings+Path), and these changes will affect the build behavior.

* Add more flexibility to your [history builds](history-build.md). TeamCity initially attempts to use the settings corresponding to the moment of the selected change. Otherwise, the current project settings will be used.

To specify which settings TeamCity should apply when a build starts, choose a required option on the **Administration | &lt;Project&gt; Versioned Settings** page.

<img src="dk-chooseSettingsOnStart.png" width="706" alt="Choosing start-up settings"/>

* **always use current settings** — all builds use current project settings from the TeamCity server. Settings' changes in branches, history, and personal builds are ignored. Users cannot run a build with custom project settings.

* **use current settings by default** — regular builds use the latest project settings from the TeamCity server. Users can run [custom builds](build-results-page.md#Changes+Tab) with settings imported from a VCS.
  
  <img src="dk-customRunWithVcsSettings.png" width="706" alt="Custom run with VCS settings"/>

* **use settings from VCS** — all branch builds and history builds, which use settings from VCS, load settings from the versioned settings' revision calculated for the build. Users can change configuration settings in [personal builds from IDE](remote-run.md) or run a build with current project settings on the TeamCity server via the [custom build dialog](build-results-page.md#Changes+Tab).

<anchor name="general-rules"/>

> Certain settings cannot be loaded from a VCS. When TeamCity detects edits in these settings, it ignores them and applies current settings stored on the server instead. These settings are:
>
> * build triggers
> * build configuration-level options, such as hanging builds detection, personal builds' availability, build configuration type, and so on
> * clean-up rules
> * agent requirements in [custom runs](running-custom-build.md)
> * settings of certain build features (for example, [Commit Status Publisher](commit-status-publisher.md) uses VCS settings, while [Pull Requests](pull-requests.md) always use the current server settings)
> * edits to existing artifact rules (you can still add new and remove existing rules)
> * (only if the [Apply Changes in Snapshot Dependencies and Version Control Settings](#Apply+Changes+in+Snapshot+Dependencies+and+Version+Control+Settings) setting is disabled) creating new and editing existing snapshot dependencies, checkout rules, and VCS roots.
> 
{style="warning"}

> Before starting a build, TeamCity stores a configuration for this build as a [hidden artifact](build-artifact.md#Hidden+Artifacts) under the [`<project_settings_directory>`](#Custom+Settings+Path)`/settings` directory. You can inspect these configuration files to determine what settings were actually used by the build.
> 
{style="tip"}

### Example: Branch-Specific Settings

1. Create and init an empty repository in your VCS of choice. In this example, a repository on GitHub.com is used.
2. Create a new TeamCity project from this repository.
3. Add a [](python.md) runner to your build configuration.

    ```Kotlin
    import jetbrains.buildServer.configs.kotlin.*
    import jetbrains.buildServer.configs.kotlin.buildSteps.python
    
    object Build : BuildType({
        name = "Build"
        // ...
        steps {
            python {
                id = "python_runner"
                command = script {
                    content = """print ("Running a Python script...")"""
                }
            }
        }
        // ...
    })
    ```

4. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Versioned Settings"/></include>
5. Choose the following options:

   * **Synchronization enabled**: On
   * **Project settings VCS root**: Choose your project's VCS root
   * **Settings format**: Kotlin
   * **When build starts**: Select "use settings from VCS"
   * **Apply changes in snapshot dependencies and version control settings**: On (see the [](#Apply+Changes+in+Snapshot+Dependencies+and+Version+Control+Settings) section)
   
6. Click **Apply** to save your new settings. TeamCity will verify the validity of your build configuration and push the [project settings directory](#Custom+Settings+Path) with your settings to the related VCS repository.
7. Clone TeamCity settings from a remote repository to local storage.
    
    ```Shell
    git clone <Clone_URL> .
    ```

8. Create a new repository branch.

    ```Shell
    git checkout -b custom-branch
    ```
9. Edit the local copy of your [`<project settings directory>`](#Custom+Settings+Path)`/settings.kts` (`.teamcity/settings.kts` by default) file as follows:

    ```Kotlin
    import jetbrains.buildServer.configs.kotlin.*
    import jetbrains.buildServer.configs.kotlin.buildSteps.csharpScript
    
    object Build : BuildType({
        name = "Build"
        // ...
        steps {
            csharpScript {
                id = "csharpScript"
                content = """Console.WriteLine("Running a CSharp script...");"""
                tool = "%\teamcity.tool.TeamCity.csi.DEFAULT%"
            }
        }
        // ...
    })
    
    ```

10. Push your new branch to the remote repository.

    ```Shell
    git add *
    git commit -a -m "Modify custom-branch settings"
    git push --set-upstream origin custom-branch
    ```
   
11. TeamCity should collect your new changes shortly. You can manually trigger this process by clicking **Load project settings from VCS...** on the **Versioned Settings** tab of your project settings page.

12. If you have a default [trigger](configuring-build-triggers.md) set up to launch new builds whenever a remote repository changes, a new build for your new custom-branch will start automatically. Otherwise, choose the required branch on the build configuration page and run a new build manually.
    
    <img src="dk-versionedSettingsBranch.png" width="706" alt="Trigger custom branch build"/>

As a result, your build configuration now performs different actions depending on which branch build runs: a [Python Script](python.md) for the main branch and a [](c-script.md) for a custom branch. You can experiment by adding more differences between branch settings and running branch builds to see how TeamCity handles different versions of your `settings.kts` file.

### Apply Changes in Snapshot Dependencies and Version Control Settings

If the corresponding option is disabled on the project's **Versioned Settings** page, TeamCity ignores edits made to:

* snapshot dependencies
* checkout rules
* VCS roots

This applies to both editing existing and creating new dependencies, checkout rules, and roots.

For example, the following Kotlin sample illustrates two [versions of settings](#Example%3A+Branch-Specific+Settings) for the same project.

* The main (default) branch defines three build configurations linked in a single BC0 &rarr; BC3 &rarr; BC5 [build chain](build-chain.md). Each configuration edits the "output.txt" file and passes it to the next configuration.
    
    <img src="dk-vcsSettings-3step.png" width="706" alt="3-Step Setup"/>

* The custom branch extends the original chain with two new configurations: BC0 &rarr; BC2 &rarr; BC3 &rarr; BC4 &rarr; BC5.

<tabs>

<tab title="Main branch settings">

```Kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.script

project {
    buildType(BC0)
    buildType(BC3)
    buildType(BC5)
}

// 1st build configuration
object BC0 : BuildType({
    name = "BC0"
    artifactRules = "output.txt"
    steps {
        script {
            name = "Prepare File"
            id = "Prepare_File"
            scriptContent = """
                rm output.txt
                touch output.txt
            """.trimIndent()
        }
        script {
            name = "WriteLine"
            id = "WriteLine0"
            scriptContent = """echo "Running BC0 config" > output.txt"""
        }
    }
    // ...
})

// 2nd build configuration
object BC3 : BuildType({
    name = "BC3"
    artifactRules = "output.txt"
    steps {
        script {
            name = "WriteLine"
            id = "WriteLine3"
            scriptContent = """echo "Running BC3 config" >> output.txt"""
        }
    }
    dependencies {
        dependency(BC0) {
            snapshot { reuseBuilds = ReuseBuilds.NO }
            artifacts { artifactRules = "output.txt" }
        }
    }
    // ...
})

// 3rd build configuration
object BC5 : BuildType({
    name = "BC5"
    artifactRules = "output.txt"
    steps {
        script {
            name = "WriteLine"
            id = "WriteLine5"
            scriptContent = """echo "Running BC5 config" >> output.txt"""
        }
    }
    dependencies {
        dependency(BC3) {
            snapshot { reuseBuilds = ReuseBuilds.NO }
            artifacts { artifactRules = "output.txt" }
        }
    }
    // ...
})
```

</tab><tab title="Custom branch settings">

```Kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.script

project {
    buildType(BC0)
    buildType(BC2)
    buildType(BC3)
    buildType(BC4)
    buildType(BC5)
}

// 1st build config
object BC0 : BuildType({
    name = "BC0"
    artifactRules = "output.txt"
    steps {
        script {
            name = "Prepare File"
            id = "Prepare_File"
            scriptContent = """
                rm output.txt
                touch output.txt
            """.trimIndent()
        }
        script {
            name = "WriteLine"
            id = "WriteLine0"
            scriptContent = """echo "Running BC5 config" > output.txt"""
        }
    }
    // ...
})

// 2nd build config
object BC2 : BuildType({
    name = "BC2"
    artifactRules = "output.txt"
    steps {
        script {
            name = "WriteLine"
            id = "WriteLine2"
            scriptContent = """echo "Running virtual BC2 config" >> output.txt"""
        }
    }
    dependencies {
        dependency(BC0) {
            snapshot { reuseBuilds = ReuseBuilds.NO }
            artifacts { artifactRules = "output.txt" }
        }
    }
    // ...
})

// 3rd build config
object BC3 : BuildType({
    name = "BC3"
    artifactRules = "output.txt"
    steps {
        script {
            name = "WriteLine"
            id = "WriteLine3"
            scriptContent = """echo "Running BC3 config" >> output.txt"""
        }
    }
    dependencies {
        dependency(BC2) {
            snapshot { reuseBuilds = ReuseBuilds.NO }
            artifacts { artifactRules = "output.txt" }
        }
    }
    // ...
})

// 4th build config
object BC4 : BuildType({
    name = "BC4"
    artifactRules = "output.txt"
    steps {
        script {
            name = "WriteLine"
            id = "WriteLine4"
            scriptContent = """echo "Running virtual BC4 config" >> output.txt"""
        }
    }
    dependencies {
        dependency(BC3) {
            snapshot { reuseBuilds = ReuseBuilds.NO }
            artifacts { artifactRules = "output.txt" }
        }
    }
    // ...
})

// 5th build config
object BC5 : BuildType({
    name = "BC5"
    artifactRules = "output.txt"
    steps {
        script {
            name = "WriteLine"
            id = "WriteLine5"
            scriptContent = """echo "Running BC5 config" >> output.txt"""
        }
    }
    dependencies {
        dependency(BC4) {
            snapshot { reuseBuilds = ReuseBuilds.NO }
            artifacts { artifactRules = "output.txt" }
        }
    }
    // ...
})
```

</tab></tabs>

If you disable the **Apply changes in snapshot dependencies and version control settings** option and try to run a build for the custom branch, it will fail with the "Failed to resolve artifact dependency" error. This happens because edits made to snapshot dependencies are ignored. From the TeamCity's point of view, this setup is invalid since it requires non-existing configurations.

<img src="dk-vcsSettings-failing.png" width="706" alt="Failing chain from custom branch"/>

The aforementioned setting allows TeamCity to automatically generate missing configurations and resolve updated dependencies, which in turn makes it possible to apply these custom branch settings.

<img src="dk-vcsSettings-5step.png" width="706" alt="5-Step Setup"/>

> Auto-generated build configurations are "service" configurations that ensure that all required settings are successfully applied. These configurations are not designed to be manually run, and should only be triggered by TeamCity when running chains with custom branch-specific settings.
> 
> If you merge settings from a custom branch into the default one, the history of auto-generated configurations' builds will be preserved.
> 
{style="note"}

## Storing Secure Settings 

It is recommended to store security data outside of VCS. The _Store passwords and API tokens outside of VCS_ option is available on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Versioned Settings | Configuration__ page if the [two-way synchronization](#two-way-sync) is enabled. By default, this option is enabled if versioned settings are enabled for a project for the first time, and disabled for projects already storing their settings in the VCS.

If this option is enabled, TeamCity stores randomly generated IDs in XML configuration files instead of the scrambled passwords. Actual passwords are stored on the disk under the [TeamCity Data Directory](teamcity-data-directory.md) and are not checked into the version control system.

<warning>

If this option is disabled, the [security implications](#Implications+of+Storing+Security+Data+in+VCS) listed below should be taken into account before committing security data to the VCS.

</warning>






<anchor name="tokensGen"/>

### Managing Tokens
[//]: # (AltHead: tokensGen)

If you need to add a password (or other secure value) to the versioned settings not via the TeamCity UI (for example, via Kotlin DSL), you can generate a token to be used in the settings instead of this password.   

In the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)s__, select _Generate token for a secure value_ in the __Actions__ drop-down menu. Enter the password and click __Generate Token__. The generated token will be stored on the server. You can copy and [use it in the project configuration files](kotlin-dsl.md#Working+with+secure+values+in+DSL) instead of the password.

```Kotlin
params {
    password("<parameter_name>", "credentialsJSON:<token>")
}
```

> You can use tokens to reference secure values only in configuration files. If you insert "credentialsJSON:&lt;token&gt;" as a value of a [password parameter](typed-parameters.md) in TeamCity UI, TeamCity will not retrieve the token's underlying secure value. Instead, the "credentialsJSON:&lt;token&gt;" string itself will be used as an actual password value.
>
{style="note"}

You can also generate new secure tokens on the __Tokens__ tab of the project __Versioned Settings__ section. The tab is available for projects with the enabled "_[Store secure values outside of VCS](#Storing+Secure+Settings)_" option.

The __Tokens__ tab allows viewing all the project tokens, including unused tokens.  
When secure data of a project is stored outside of the version control system, it could potentially be detached from the project: for example, if a project with tokens is moved to another place in the hierarchy or is created from DSL on the new TeamCity server. In such case, you can specify the values for the project tokens on this tab, so the project can continue using them.

Moreover, if the required tokens are available in other projects you are permitted to edit, TeamCity will automatically find them. You can copy the secure values used in any of these projects to your current project.

<img src="tokens-tab.png" alt="Versioned Settings| Tokens" width="800"/>

When there is one or more secure values available for a token, the <img src="magic-wand.png" alt="Switch to the Sakura UI" height="20" width="20"/> button appears opposite this token. Click it to review available projects and choose a project to copy a value from, and then click __Copy__ to confirm your choice.

Secure values can be inherited by project hierarchy. If a setting in a project (VCS root, OAuth connection, cloud profile) requires a password, the token generated for this password can be used in this project and in any of its subprojects. To be able to use the inherited password, the subproject must have versioned settings enabled and store settings in the same VCS as its parent project.  
Alternatively, you can add a [password parameter](typed-parameters.md) with the secure value and use a [reference](using-build-parameters.md) to the parameter in the nested projects.

### Implications of Storing Security Data in VCS 

If you decide to store secure settings in VCS, it is recommended to carefully consider the following implications:
* If the projects or build configurations with settings in a VCS have __password fields defined__, the values appear in the settings committed into the VCS (though, in scrambled form).
   * If the project settings are stored in __the same repository as the source code__, anyone with access to the repository will be able to see these scrambled passwords.
   * If the project settings are stored __separately from the source code in a dedicated repository__ and the "_Show settings changes in builds_" option is enabled, any user with the "_View VCS file content_" permission will be able to see all the changes in the TeamCity UI using the [changes difference viewer](difference-viewer.md).
* Being able to change the settings in an arbitrary manner via a VCS, it is possible to trigger builds of any build configurations and obtain settings of any build configurations irrespective of the build configurations permissions configured in TeamCity.
* By committing wrong or malicious settings, a user can affect the entire server performance or server presentation to other users.

It is recommended to store passwords, API tokens, and other secure settings outside of VCS using the corresponding option [described above](#Storing+Secure+Settings).

Note that SSH keys will not be stored in the VCS repository.

## Storing and Managing Global Server Settings

<snippet id="iac-terraform">

Storing [](kotlin-dsl.md) or XML settings in a VCS allows you to utilize the configuration-as-code approach for projects and build configurations. You can specify the hierarchy of projects, manage individual configurations, dynamically change step settings and parameters, and more.

However, this approach does not allow you to manage server-wide settings (such as user and user group settings, clean-up rules, notifications, authentication modules, licenses, and more). To automate your server administration, define required settings in the HCL language format and use [HashiCorp Terraform](https://www.terraform.io) to manage them. To allow Terraform to communicate with your TeamCity server instance, add the dedicated **TeamCity Terraform Provider** to your Terraform configuration.

<video src="https://youtu.be/gXteNQIWkwU"
title="TeamCity Terraform Provider"/>

**Learn more:**

* [TeamCity Terraform Provider on Terraform Registry](https://registry.terraform.io/providers/JetBrains/teamcity/0.0.68)
* [TeamCity Terraform Provider on GitHub (source code and examples)](https://github.com/JetBrains/terraform-provider-teamcity)
* [TeamCity Terraform Provider Documentation](https://registry.terraform.io/providers/JetBrains/teamcity/latest/docs)
* [Configuration as Code for TeamCity Using Terraform | The TeamCity Blog](https://blog.jetbrains.com/teamcity/2024/02/configuration-as-code-terraform-teamcity/#managing-github-resources-via-terraform)

</snippet>


<anchor name="settingsFormat"/>

## Settings Format
[//]: # (AltHead: settingsFormat)

To be able to select the settings format, click __Show advanced options__ in __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Versioned Settings | Configuration__. 

TeamCity stores project settings:
* in the XML format
* in the Kotlin-based DSL format (see a [dedicated page](kotlin-dsl.md))

<note>

To test Kotlin-based settings in a sandbox project, you can download the settings as a ZIP archive without disabling the UI: on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page, click the __Actions__ menu and select __Download settings in Kotlin format.__ A ZIP archive will be downloaded.

</note>

<anchor name="ForcingSynch"/>

## Committing Current Project Settings to VCS
[//]: # (AltHead: settingsFormat)

<warning>

Before committing settings to the VCS, consider the recommended approach to storing security settings [described above](#Storing+Secure+Settings).

</warning>

If you want to commit the current configuration to the VCS (for example, earlier you committed misconfigured settings to the repository and TeamCity was unable to load it displaying errors and warnings), you can use the _Commit current project settings_ option on the __Versioned Settings | Configuration__ page.

When TeamCity commits settings into a VCS, it uses the standard commit message noting the TeamCity user as the committer and the project whose settings have changed. It is possible to add a fixed custom prefix to each settings' change committed by TeamCity via the `teamcity.versionedSettings.commitMessagePrefix` [internal property](server-startup-properties.md), for example, `teamcity.versionedSettings.commitMessagePrefix=TC Change\n\n`.
{instance="tc"}

When TeamCity commits settings into a VCS, it uses the standard commit message noting the TeamCity user as the committer and the project whose settings have changed.
{instance="tcc"}

## Displaying Changes

TeamCity will not only synchronize the settings, but will also automatically display changes to the project settings the same way it is done for regular changes in the version control. You can configure the changes to be displayed for the affected build configurations: on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Versioned Settings | Configuration__ tab, click __Show advanced options__ and check the _Show settings changes in builds_ box. This option is only effective if you store versioned settings in a VCS root that is not directly attached to the current build configuration. If a VCS root is attached to this configuration, its changes will be displayed by default, unless specific [build triggering rules](configuring-build-triggers.md) are configured.

If you store versioned settings in a separate VCS root, not the one which is directly attached to your build configurations, then when turned ON this checkbox starts showing changes made to these VCS roots. If VCS root is already attached to your configuration, then it does nothing.

By default, the VCS trigger will ignore such changes. To enable build triggering on a settings' commit, add a trigger rule in the following format: `+:root=Settings_root_id;:*`.

All changes in the VCS root, where project settings are stored, are listed on the __Versioned Settings | Change log__ tab.

<anchor name="enableAfterUpgrade"/>

<anchor name="StoringProjectSettingsinVersionControl-enableAfterUpgrade"/>

## Enabling Versioned Settings after TeamCity Upgrade
[//]: # (AltHead: enableAfterUpgrade)

The format of the XML settings files changes from one TeamCity version to another to accommodate the new features and improvements. Generally, the format is not changed within bugfix releases and is changed in major releases. When a TeamCity server is upgraded, the current settings on the TeamCity server are changed from the earlier to the current format.

It is a common practice to upgrade a TeamCity test server with production data before upgrading the production server. In order to avoid accidentally changing the format of the settings which are used on a production server of an older version, versioned settings are disabled after a TeamCity upgrade and the corresponding health item is displayed. System administrators have permissions to enable versioned settings (__Administration | Server Health | Versioned settings disabled__, click __Enable__). When enabled, the converted settings in the format of the current TeamCity version will be checked in the version control. Note that the new settings will be committed to the default branch of the VCS root; the settings stored in other branches will have to be updated manually.

## FAQs

**Q.** Can I apply the settings from a TeamCity server of a different version?<br/>  
**A.** No, because just like with the TeamCity Data Directory, the format of the settings differs from one TeamCity version to another.

**Q.** Where are the settings stored?<br/>
**A.** You can manually [configure a directory](#Custom+Settings+Path) where project settings should be stored. The default directory is `.teamcity` directory in the root of the VCS root-configured repository. The default branch configured in the VCS root is used with [Git](git.md) and [Mercurial](mercurial.md). You can create a dedicated VCS root to change the repository or a branch (or a repository path in case of Perforce, Subversion, or Azure DevOps — formerly TFS).

**Q.** Why is there a delay before a build is run after I changed to the settings in the UI?<br/>
**A.** When the settings are changed via the UI, TeamCity waits for the changes to be completed with a commit to the VCS before running a build with the latest changes.

**Q.** Who are the changes authored by?<br/>
**A.** If the settings are changed via the user interface, in Git and Mercurial a commit in the VCS will be performed on behalf of the user who actually made the change via the UI. For Perforce as well as Azure DevOps Server (formerly TFS), the name of the user specified in the VCS root is used, and in Subversion the commit message will also contain the username of the TeamCity user who actually made the change via the UI.

<seealso>
        <category ref="admin-guide">
            <a href="kotlin-dsl.md">Kotlin DSL</a>
        </category>
</seealso>
