[//]: # (title: How to Configure CI/CD for JetBrains Space)
[//]: # (auxiliary-id: How to Configure CI/CD for JetBrains Space)

[JetBrains Space](https://www.jetbrains.com/space/) is a full-cycle collaboration solution for software development teams. This guide explains how to achieve continuous integration and delivery of JetBrains Space projects by integrating them with TeamCity.



Integration with TeamCity brings the following advantages to the JetBrains Space users:

* Compiling, testing, and deploying projects within the same environment.
* Building source code of merge requests and merging them automatically after a successful build.
* Extensive build overview: diffs and artifacts, detailed test reports on the fly, code coverage, inspections, and various other metrics. Statuses of builds and code reviews are cross-shared between systems for easier monitoring.
* Flexible pipelines where builds depend on one another and share settings and results.
* Ability to configure builds as code, in [Kotlin DSL](kotlin-dsl.md).
* Authentication with a single account in both systems: VCS (JetBrains Space) and CI/CD (TeamCity).



This guide consists of the common [preliminary setup](#Preliminary+Setup) and optional procedures for enabling each component of the integration.

## Prerequisites

To perform all the steps described in this guide, you need to have:
* _Project administration rights_ in the JetBrains Space instance of your organization.
* _Project administration rights_ in either the [TeamCity Cloud instance](https://www.jetbrains.com/teamcity/signup/) or [TeamCity On-Premises server](https://www.jetbrains.com/teamcity/download/) of your organization.
* To enable authentication via JetBrains Space in TeamCity, _system administration rights_ in TeamCity.

## Preliminary Setup

You can connect TeamCity to JetBrains Space using two different techniques. The traditional approach includes two steps:

1. creating a service application for TeamCity authentication in your Space instance;
2. creating a preset of connection to Space.

Starting with version 2023.07, you can utilize semi-automatic Space connections that allow you to skip setting up Space applications manually. Instead, specify the organization URL (for Space On-Premises) or choose the required Cloud instance (for Space Cloud) and TeamCity will automatically create and install Space applications with all required permissions.
{instance="tcc"}

Starting with version 2023.11, you can utilize semi-automatic Space connections that allow you to skip setting up Space applications manually. Instead, specify the organization URL (for Space On-Premises) or choose the required Cloud instance (for Space Cloud) and TeamCity will automatically create and install Space applications with all required permissions.
{instance="tc"}


This tutorial explains how to manually setup the TeamCity-Space integration. For the detailed instructions on how to implement the Space integration using the newer semi-automatic approach, see this section instead: [Configuring Connections, JetBrains Space](configuring-connections.md#jetbrains-space-connection).

### Step 1: Create Application in JetBrains Space

In your JetBrains Space instance:
1. On the navigation bar, click __Extensions__ and choose __Installed to organization__.
2. Click __New application__.
3. Enter a convenient name (for example, `Space-to-TeamCity`), save the application, and click __Go to application settings__.
4. Configure _In-context Authorization_:
   1. On the __Authorization__ tab, click __Authorize in new context__.
   2. Enter the name of the Space project you are about to access from TeamCity and click __Authorize__.
   >When you create a project in JetBrains Space, it does not automatically add you to this project as a member — this needs to be done manually. TeamCity will be able to see only those projects where you are listed as a member.
   3. (_Optional_) If you want TeamCity to be able to publish commit statuses to Space, you will need to add a respective permission.  
      Click __Configure__ and enable _Git Repositories | Report external check status_. This request has to be accepted by the project administrator.
5. Configure _Global Authorization_:
   1. On the __Authorization__ tab, click __Configure__.
   2. To establish general access from TeamCity to Space, enable the _Members | View member profile_ permission and click __Save__. This request has to be accepted by the server administrator.
6. Configure _Authentication Mode_:
   1. Go back to the app's __Overview__ and open the __Authentication__ tab.
   2. Enable _Client Credentials Flow_.
   
      <anchor name="redirect-uri"/>
   3. To be able to use authentication via Space in TeamCity or/and to create projects/configurations from Space repositories, enable _Authorization Code Flow_ as well. Enter the redirect URI (`https://<server>:<port>/oauth/space/accessToken.html`) of your TeamCity Server.
      >To ensure that your TeamCity server can always connect to JetBrains Space, it is important to specify all the other possible endpoint addresses of the server. In most cases, it would be enough to specify the _Server URL_ set in __Global Settings__ in TeamCity. However, if you use a [proxy](configuring-proxy-server.md) for your TeamCity server but access this server directly, the authentication might not work unless the server's IP address is also specified here.
      >
      {instance="tc"}
   4. Copy the app's _Client ID_ and _Client secret_. You will need them for configuration on the TeamCity side.

Now, you can return to TeamCity and add a connection to JetBrains Space.

### Step 2: Establish Connection to JetBrains Space

>TeamCity allows you to configure all settings of your _connection_ to a service in one place and then reuse these settings in different projects and build configurations. If you add such a _connection_ on the <tooltip term="root-project">_Root project_</tooltip> level, this will allow using its settings to connect any other project on the server. To make a connection available only in a certain project, you need to add it in this project.

To create a connection to your JetBrains Space instance:
1. Go to __Project Settings | Connections__ and click __Add Сonnection__.
2. Choose the _JetBrains Space_ connection type.  
   <img src="connection-to-space.png" width="460" alt="Create a connection to Space"/>
4. Enter the settings as follows:
   * _Space URL_: the URL of your Space instance (`<company_name>.jetbrains.space`).
   * _Client ID_: the client ID value copied from the Space app's __Authentication__ tab.
   * _Client secret_: the client secret value copied from the Space app's __Authentication__ tab.
5. Save the connection.

At this stage, you are free to access your JetBrains Space instance from the current project or any of its subprojects whenever necessary.

## Connecting TeamCity Projects with JetBrains Space Projects

There are three ways to integrate a VCS repository with TeamCity:
* Create a TeamCity _project_ based on a repository.
* Create a _build configuration_ based on a repository in an existing TeamCity project.
* Create a <tooltip term="vcs-root">_VCS root_</tooltip> based on a repository in an existing TeamCity project.

We will describe the first approach as it's the most popular and self-sufficient. However, you can always add one more Space [root](configuring-vcs-roots.md) or [build configuration](creating-and-editing-build-configurations.md#Creating+Build+Configuration+from+URL) to an existing project — the procedure is similar.

Let's create a subproject of the project where you added the Space connection during the [preliminary setup](#Step+2%3A+Establish+Connection+to+JetBrains+Space). If it is the <tooltip term="root-project">_Root project_</tooltip>, go to __Administration | Projects__ and click __Create project__ there.

You will notice the new button: __From JetBrains Space__. Its name depends on the _Display name_ you gave to the connection, but you can always distinguish Space connections from others by the Space logo. To create a new project:

1. Click __From JetBrains Space__.
2. As it is the first time you connect this server to your Space instance, you have to authenticate in Space via your user profile. Click __Sign in to Space__ and accept the access request. Next time, you won't have to confirm it again, unless you sign out or change your password.  
   <img src="create-project-from-space.png" width="706" alt="Create a project from a Space repository" style="block"/>
   >If you get the _OAuth 2.0 Error_, this might mean that the _Redirect URI_ has not been configured properly in Step 1 of the preliminary setup. Make sure to [revise it](#redirect-uri). Note that Space supports only HTTPS connection.
3. The project creation wizard will display a list of all Space projects your user has access to. Choose a repository and wait until TeamCity verifies the connection settings.
4. Now it's time to configure the main settings of the new project and its [VCS root](vcs-root.md). You can always adjust them later.
   * _Project name_ and the name of its first [build configuration](managing-builds.md).
   * _VCS root_: (read-only) matches the URL of a repository you choose in Step 3.
   * _Default branch_: `refs/heads/main`, or another branch of your choice. Remember to keep `refs/heads` and change only the branch name.
   * _Branch specification_: if you want to run builds on other branches apart from the default one, this setting allows extending the scope of the VCS root to monitor these branches as well. TeamCity uses a [special format or branch specification rules](working-with-feature-branches.md) for this. To monitor the whole repository, use the `refs/heads/*` rule.
     <img src="space-project-basic-settings.png" width="706" alt="Configure project settings"/>
   >Our sample project contains project settings specified in a TeamCity [Kotlin DSL](kotlin-dsl.md) format. If TeamCity detects them in your repository, it can automatically apply the specification to the new project.
5. Click __Proceed__.

TeamCity will attempt to [autodetect build steps](configuring-build-steps.md#Autodetecting+Build+Steps) in your project. You can confirm or reject the proposed steps and explore the project settings further.

If you create a project from a repository automatically, like we just did, TeamCity automatically adds a [VCS trigger](vcs-root.md). This trigger will be watching your Space repository and run builds on each new commit. You can edit its settings or [add other types of triggers](configuring-build-triggers.md).

TeamCity shows the commits that got into a build on the build's **Overview** tab. Click the Space icon opposite a commit to open its details in JetBrains Space:

<img src="space-commit-details.png" width="706" alt="Create a connection to Space" border-effect="line"/>

If the commit mentions a Space code review tag (for example, `JETBRAINS-DEMO-CR-3`) or a Space merge request tag, you can click the tag to jump directly to the corresponding code review or merge request in Space. In particular, a merge commit from Space is always accompanied by a link back to the corresponding merge request.

After this basic setup, you can advance the Space integration by following the instructions below, or learn how to [create more sophisticated build configurations](configuring-general-settings.md) and utilize the power of TeamCity to the fullest.

## Building Sources from Merge Requests

A build's checkout scope usually consists of the following items: the default branch of [VCS root(s)](vcs-root.md) + the project's [branch specification](working-with-feature-branches.md) + [checkout rules](vcs-checkout-rules.md) of the build configuration. By adding the [Pull Request](pull-requests.md) build feature to this build configuration, you can add one more item to this formula — branches of merge requests. This will allow TeamCity to monitor changes in merge requests and run builds on them. The most common use case for this is prebuilding and pretesting sources of feature branches before they are merged into the default branch.

To add this feature:

1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Build Features"/></include>
3. Click __Add build feature__ and choose _Pull Requests_.
4. Choose the recently created VCS root.
5. Select _JetBrains Space_ as the VCS hosting type and specify the settings as follows:
   * _Connection_: choose the [connection to Space](#Step+2%3A+Establish+Connection+to+JetBrains+Space).
   * _By target branch_: define the [branch filter](branch-filter.md) to monitor merge requests only on branches that match the specified criteria. If left empty, no filters apply.
6. Save the settings.

>Note that the scope of branches you define in this feature should not overlap with the branch specification of the VCS root. This measure will ensure that no conflicts occur when starting builds on merge requests.
>
{style="warning"}

Now, TeamCity will monitor merge requests submitted from the source branches of your repository. If a build is run on a merge request, TeamCity will display the details of the request and report the build status to the code review in Space:

<img src="space-timeline.png" width="460" alt="Reporting build statuses to Space"/>

>To protect a JetBrains Space branch from unverified merge requests, you can also configure [Quality Gates](https://www.jetbrains.com/help/space/branch-and-merge-restrictions.html#quality-gates-for-merge-requests) in your repository settings. If you set a TeamCity build as an external check, JetBrains Space will require the build on a merge request to finish successfully before allowing this request to be merged.

Read more about the Pull Requests build feature in [this article](pull-requests.md).

### Automatically Merging Request if Build is Successful

TeamCity can automatically merge a request into a target branch if the respective build finishes successfully. To achieve this:
1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Build Features"/></include>
3. Click __Add build feature__ and choose _Automatic Merge_.
4. Specify what branches to monitor and to merge into.
5. Choose a merge policy. You can find more information about advanced settings of this feature [here](automatic-merge.md).
6. Save the settings.

Each time a build satisfies the conditions of the selected merge policy, TeamCity will merge it to the specified target branch.

## Reporting Build Statuses to JetBrains Space

TeamCity can report statuses of builds to JetBrains Space. To achieve this:
1. <include from="common-templates.md" element-id="open-configuration-settings"/>
2. <include from="common-templates.md" element-id="open-configuration-settings-tab"><var name="configuration-tab-name" value="Build Features"/></include>
3. Click __Add build feature__ and choose _Commit Status Publisher_.
4. Select the _JetBrains Space_ publisher and the [connection to Space](#Step+2%3A+Establish+Connection+to+JetBrains+Space).
5. Specify the name that will be displayed for this service in Space.
6. Save the settings.

Now, whenever you run a build in this configuration, TeamCity will report the build status to JetBrains Space.

<img src="space-csp.png" width="296" alt="Display build status in JetBrains Space"/>

## Authenticating in TeamCity with JetBrains Space Account

Users of your JetBrains Space instance can authenticate in TeamCity with their Space accounts. Note that this functionality requires adding the [JetBrains Space connection](#Step+2%3A+Establish+Connection+to+JetBrains+Space) on the Root project level.

To configure it:
1. Go to __Administration | Authentication__.
2. Click __Add module__ and choose the _JetBrains Space_ type.  
   <img src="add-space-auth-module.png" width="460" alt="Add a JetBrains Space authentication module"/>
3. Choose if you want to allow creating new users on the first login, if their emails are not recognized by TeamCity. Disabling this option could be helpful if you use a publicly available TeamCity server and want to limit access to it.
4. Save the module.

To sign in to TeamCity, click the JetBrains Space icon above the TeamCity login form and, after the redirect, approve the TeamCity request.

<img src="spaceauth.png" width="296" alt="Sign in to TeamCity with JetBrains Space account"/>

<seealso>
        <category ref="admin-guide">
            <a href="configuring-vcs-roots.md">Configuring VCS Roots</a>
            <a href="configuring-connections.md">Configuring Connections</a>
            <a href="creating-and-editing-projects.md">Creating and Editing Projects</a>
            <a href="creating-and-editing-build-configurations.md">Creating and Editing Build Configurations</a>
        </category>
</seealso>