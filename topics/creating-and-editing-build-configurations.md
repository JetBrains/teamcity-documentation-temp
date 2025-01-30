[//]: # (title: Creating and Editing Build Configurations)
[//]: # (auxiliary-id: Creating and Editing Build Configurations)

This section contains articles on how to create and configure build configurations via the TeamCity UI.

A _build configuration_ is a collection of settings used to start a build and group the sequence of the builds in the UI. Examples of build configurations are _distribution_, _integration tests_, _prepare release distribution_, _"nightly" build_.  
A build configuration belongs to a [project](project.md) and contains builds. You can explore details of a build configuration on its [Home page](build-configuration-home-page.md) and modify its settings on the [Settings page](creating-and-editing-build-configurations.md).

<img src="create-build-configuration-button.png" alt="Create a build configuration" width="750"/>

It is recommended to have a separate build configuration for each sequence of builds (that is performing a specified task in a dedicated environment). This allows for proper features functioning, like detection of new problems/failed tests, first failed in/fixed in tests status, automatically removed investigations, and so on.

To tackle an increased number of build configurations you can use [build configuration templates](build-configuration-template.md) and project-level [parameters](configuring-build-parameters.md).

This video tutorial illustrates how to work with a build configuration in TeamCity and gives a few extra tips:

<video src="https://youtu.be/fttWwJG7C38"
title="Improving your first build configuration"/>

>Build configurations can also be created via [TeamCity REST API](https://www.jetbrains.com/help/teamcity/rest/get-build-details.html#Get+Specific+Builds) and as a versioned [Kotlin DSL code](storing-project-settings-in-version-control.md).

## Where to Create Build Configuration

To find the build configuration creation wizard:
1. Go to __Administration | Projects__ and open the required project.  
   Alternatively, open the project using the __Projects__ pop-up menu and click __Edit Project Settings__.  
   The __Project Settings__ page will open.
2. On the __Project Settings__ page, click __Create build configuration__ under the __Build Configurations__ section.
3. Follow the instructions specific to your preferred method:
   * [manually](#Creating+Build+Configuration+Manually)
   * [pointing to a repository URL](#Creating+Build+Configuration+from+URL)
   * pointing to a repository in GitHub.com, GitLab CE/EE, Bitbucket Cloud, Azure DevOps Services, and JetBrains Space
   * based on a [build configuration template](#Creating+Build+Configuration+Template)

<img src="create-build-configuration.png" alt="Create a build configuration from a repository URL" width="828"/>

After build configurations are created, you can:
* [change their order](#Ordering+Build+Configurations)
* [edit their settings](#Configuring+Settings)
* control them via the [Actions](#Actions+in+Build+Configuration+Settings) menu

## Creating Build Configuration from URL

In the [build configuration creation wizard](#Where+to+Create+Build+Configuration):
1. Click __From a repository URL__.
2. Enter your VCS repository URL and, if authentication is required, VCS credentials (username and password/token).
3. Click __Create__.

TeamCity will suggest the build configuration name and will configure the rest of the settings for you:
* It will determine the type of the VCS repository and create a [VCS root](vcs-root.md). For a Git repository, it will autodetect the default branch. You have an option to change it now or later, in the [VCS root](vcs-root.md) settings. You can also change the branch specification: by default, TeamCity monitors all branches of the repository, but you can choose what exact branches to monitor by entering [custom rules](working-with-feature-branches.md#Common+Specification+Syntax).
* It will attempt to autodetect build steps: Ant, NAnt, Gradle, Maven, MSBuild, Visual Studio solution files, PowerShell, Xcode project files, Rake, and IntelliJ IDEA projects. You can then always add and edit them [manually](configuring-build-steps.md).

Next, TeamCity will display suggestion icons with prompts to create [build triggers](configuring-build-triggers.md), [failure conditions](build-failure-conditions.md), and [build features](adding-build-features.md). Depending on the build configuration settings, it may suggest some additional configuration options.

After the build configuration is created, you can run a build and/or tweak the settings.

## Creating Build Configuration Pointing to Specific VCS

In the [build configuration creation wizard](#Where+to+Create+Build+Configuration):
1. Click the button respective to your VCS: from GitHub.com, GitLab CE/EE, Bitbucket Cloud, Azure DevOps Services, or JetBrains Space.  
   Note that to be able to access the selected VCS, TeamCity needs to always have the connection parameters at its disposal. You can configure the connection preset in advance and select the target VCS server in the wizard, or let TeamCity redirect you to the __Connections__ page right from the wizard. The connections settings of supported VCS providers are described [here](configuring-connections.md).
2. Verify the connection to the VCS.
3. TeamCity will propose the build configuration name. You can change it if needed.  
   For a Git repository, it will also autodetect the default branch. You have an option to change it now or later, in the [VCS root](vcs-root.md) settings. You can also change the branch specification: by default, TeamCity monitors all branches of the repository, but you can choose what exact branches to monitor by entering [custom rules](working-with-feature-branches.md#Common+Specification+Syntax).  
   Click __Proceed__.
4. TeamCity will add a VCS build trigger and attempt to [autodetect build steps](configuring-build-steps.md#Autodetecting+Build+Steps): Ant, NAnt, Gradle, Maven, MSBuild, Visual Studio solution files, PowerShell, Xcode project files, Rake, and IntelliJ IDEA projects. On the __Auto-detected Build Steps__ page, select the step(s) to use in your build configuration. Click __Use selected__. You can then always add or edit steps [manually](configuring-build-steps.md).  
  Depending on the build configuration settings, TeamCity can suggest some additional configuration options. Review the suggested settings ![suggestedSettings.PNG](suggestedSettings.PNG) and configure the required ones.

The build configuration is created. Click __Run__ to start its first build.

## Creating Build Configuration Manually

In the build configuration creation wizard:
1. Click __Manually__
2. In the _Create Build Configuration_ dialog, specify the name, ID and (optionally) description for the build configuration, click __Create__.
3. Proceed with creating other settings:
   * [Create/edit VCS roots and specify VCS-specific settings](configuring-vcs-settings.md)
   * On the __Build Steps__ page, configure build steps discovered by the automatic detection. To create them manually by selecting a desired build runner from the drop-down menu. Click __Save__. You can add as many build steps as you need within one build configuration. Note that they will be executed sequentially. In the end, the build gets the merged status and the output goes into the same build log. If some step fails, the rest is executed or not, depending on their [step execution policy](configuring-build-steps.md#Execution+Policy).
   * Additionally, configure [build triggering options](configuring-build-triggers.md), [dependencies](configuring-dependencies.md), [properties and variables](configuring-build-parameters.md), and [agent requirements](configuring-agent-requirements.md).

## Creating Build Configuration Template

Creating a new [build configuration template](build-configuration-template.md) is similar to creating a new configuration:
1. On the __Administration | Projects__ page, select the target project in the list.  
  Alternatively, open the project using the __Projects__ pop-up menu and click __Edit Project Settings__.  
  The __Project Settings__ page opens.
2. On the __Project Settings__ page, click __Create template__ under the __Build Configuration Templates__ section.
3. Proceed with configuring [general settings](configuring-general-settings.md), [VCS settings](configuring-vcs-settings.md), and [build steps](configuring-build-steps.md).

Alternatively, you can create a build configuration template from an existing build configuration:
1. Open the existing build configuration settings page, click __Actions__ in the upper right corner of the screen, and select the __Extract Template__ option.
2. Specify the required settings and click __Create__.

## Creating Build Configuration from Template

> When you create a template from scratch or extract it from an existing build configuration, this template is available only for this project (and its subprojects). This means you cannot create new configurations in "&lt;Root project&gt;/ProjectA" using the "&lt;Root project&gt;/ProjectB/BuildConfTemplate" template.
> 
{style="tip"}

You can create a templated build configuration in two ways: from the template settings page, or by creating a regular configuration and choosing a template it should utilize.

Option #1:

1. Navigate to **Administration | Project** and select a project that owns a template.

2. Click a required template under the **Build configuration templates** section.
    
    <img src="dk-buildConfTemplates.png" width="706" alt="Choose the required template"/>

3. Click **Actions | Create build configuration from this template...**.

    <img src="dk-createConfFromTemplate.png" width="706" alt="Create new configuration from a template"/>

4. Specify the required settings for the new configuration. Do not click any tiles other than **Manually**; otherwise, you will create a new configuration where the selected template is not used.

Option #2:

1. Navigate to **Administration | Project** and select a project that owns a template.
2. Click __Create build configuration__ under the __Build Configurations__ section.
    
    <img src="dk-createNewBuildConf.png" width="706" alt="Create a regular configuration"/>
   
3. Click the **Manually** tile and choose the required template in the __Based on template__ drop-down menu.

    <img src="dk-confBasedOnTemplate.png" width="706" alt="Create a regular configuration"/>

Option #2 is helpful when the **Build configuration templates** section does not show the required template since it is owned by another (sub)project.

<note>

The settings specified in the template cannot be edited in a configuration created from this template. However, some of them can be [redefined in a build configuration](build-configuration-template.md#Redefining+settings+inherited+from+template). Note that modifying the settings of the template itself will affect __all configurations__ based on this template.
</note>

## Ordering Build Configurations

You can view all build configurations of a project on the __Project Overview__ page. By default, they are listed in the alphabetical order, but administrators can [customize this order](ordering-projects-and-build-configurations.md).

## Build Configuration Settings

Build configuration settings include:
* [General settings](configuring-general-settings.md)
* [Version control settings](vcs-root.md), defining how the source code is retrieved from VCS, where it is checked out to, and so on
* [Build steps](configuring-build-steps.md), that are sequentially run actions: for example, running msbuild, a script, or unit tests
* [Triggers](configuring-build-triggers.md), which are rules defining when to start a new build
* [Failure conditions](build-failure-conditions.md) specifying when a build will be marked as failed
* Additional [build features](adding-build-features.md)
* Dependencies:
   * for [snapshot dependencies](snapshot-dependencies.md), TeamCity will run all dependent builds on the sources taken at the moment the build they depend on starts
   * for [artifact dependencies](artifact-dependencies.md), before a build is started, all artifacts this build depends on will be downloaded and placed in their configured target locations and then will be used by the build
* [Parameters](configuring-build-parameters.md) which allow sharing settings
* Agent requirements specifying whether a [build configuration](managing-builds.md) can run on a particular [build agent](build-agent.md).

<note>

Build configuration settings and build behavior may vary depending on the type of build configuration.
</note>

### Configuring Settings

When you select a build configuration from the list of build configurations, TeamCity displays the __Build Configuration Home__ page where you can preview its recent build results. To access the build configuration's settings, click __Edit Configuration Settings__ in the upper right corner of the screen.

Different build configuration settings are described in the respective articles inside this section.

>Note that editing via the TeamCity UI will be disabled for build configurations created via the [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html).

## Permissions to Edit Build Configuration

While only users with _Project Administrator's_ permissions can change project and build configuration settings, there are several ways how contributors to the source code can also affect the build settings and environment.

The default _Project Developer_ [role](managing-users-and-roles.md) grants users two permissions:
* _Customize build parameters_ allows changing the values of [build configuration parameters](configuring-build-parameters.md) thus potentially affecting how the source code is executed.
* _Change build source code with a custom patch_ allows running a [custom build](running-custom-build.md) based on a user's local sources, not yet committed to the repository.

Besides, all the users who author the source code or/and can write to the repository with project settings stored in [Kotlin DSL](kotlin-dsl.md), could potentially execute their arbitrary code on common build agents.

We recommend considering this aspect when granting users the permissions mentioned above or writing access to the projects' repositories. If necessary, you can adjust the set of [permissions granted to each role](managing-roles-and-permissions.md).

## Actions in Build Configuration Settings

Use the __Actions__ menu, located in the upper right corner of the settings screen, to:
* [Pause/activate a build configuration](changing-build-configuration-status.md).
* [Copy/move/delete a build configuration](copy-move-delete-build-configuration.md).
* [Attach a build configuration to a template](build-configuration-template.md#Associating+build+configurations+with+templates).
* [Extract a template from a build configuration](build-configuration-template.md#Creating+build+configuration+template).
* [Extract a meta-runner from a build configuration](working-with-meta-runner.md#Extracting+and+Using+Meta-Runner).
* [Attach a history to a build configuration](kotlin-dsl.md#Restore+Build+History+After+ID+Change).

 
 <seealso>
        <category ref="admin-guide">
            <a href="configuring-dependencies.md">Configuring Dependencies</a>
            <a href="configuring-build-parameters.md">Configuring Build Parameters</a>
            <a href="configuring-vcs-settings.md">Configuring VCS Settings</a>
        </category>
        <category ref="examples">
            <a href="how-to-configure-cicd-for-jetbrains-space.md">How to Configure CI/CD for JetBrains Space</a>
        </category>
</seealso>