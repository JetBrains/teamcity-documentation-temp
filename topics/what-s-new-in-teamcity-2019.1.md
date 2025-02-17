[//]: # (title: What's New in TeamCity 2019.1)
[//]: # (auxiliary-id: What's New in TeamCity 2019.1)

## New TeamCity documentation website

We have reworked the documentation for TeamCity 2019.1 to create a better user experience and ensure a common look and feel across the documentation of all company products.  

The main product documentation is accessible on the [new documentation website](https://www.jetbrains.com/help/teamcity/teamcity-documentation.html).
 
The Plugin Development Help is now in a [separate location](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html) and its source files have been moved to the public [GitHub repository](https://github.com/JetBrains/teamcity-sdk-docs) so our community can contribute to it. 

See the [documentation for previous versions](documentation-for-previous-versions.md).


## Revamped experimental TeamCity UI

The new TeamCity version comes with the reworked UI aiming at improving your experience with the product.

<img src="SakuraUI.png" width="1000"/>

* __New Sidebar__: You can now easily access and search all your projects and build configurations from the sidebar, tag build configurations or whole projects as your favorites to see them at the top of the sidebar. New changes, build status, new tests, and running builds: all are now visible. 
* The reworked __Project Home__ page provides a dashboard-style view over your build configurations. Each configuration gets its own card which displays a histogram with up to 14 of the latest builds. The status, the build time as well as the time in the queue are also available. Clicking an individual card takes you to the overview of this build configuration.
* Revamped __Branches__ Tab: Branches are now split into categories, and you can expand and collapse them as needed. This helps you keep an overview of your branches and browse them conveniently.
  <img src="SakuraBranches.png" width="600"/>
* Expandable __build row__: now, TeamCity can show more about builds on a page, and if you need details on a particular build, clicking it will expand the row allowing more information.
* The new UI is in the __experimental__ stage, and you can switch to it using the icon in the upper right-hand corner of the screen. There is also a new setting in your profile enabling the experimental UI by default. As not all the features are supported by the new UI, it is easy to go back to the classic TeamCity style when needed.


## Support for GitLab

TamCity 2019.1 supports GitLab. It allows creating [connections](configuring-connections.md#GitLab) to [GitLab.com](https://about.gitlab.com/) and [GitLab CE/EE](https://about.gitlab.com/install/ce-or-ee/) so you could easily select a predefined GitLab repository when creating a new project or a build configuration.   
To be able to authenticate to GitLab during the connection, register an [OAuth application](https://docs.gitlab.com/ee/integration/oauth_provider.html) in GitLab with the `api` and `read_repository` scopes and generate a secret and an application ID.   
Enter the secret and application ID, along with the GitLab server URL, when adding a new connection.

<img src="GitLabConnection.png" width="600"/>
 

### Support for GitLab merge requests

We've also added support for [GitLab merge requests](pull-requests.md#GitLab+Merge+Requests), so you can now set up TeamCity to automatically run a build on each merge request and approve it automatically if the build is successful.

## Support for Bitbucket Server pull requests

Now the [Pull Requests](pull-requests.md) build feature can detect pull requests created in Bitbucket Server.   
To add a Bitbucket VCS root, select _Bitbucket Server_ as a _VSC hosting type_ and configure the connection parameters:
* Authentication Type: VCS root credentials or username/password
* Filtration by target PR branches
* Bitbucket Server's base URL

<img src="PR_BitbucketServer.png" width="600"/>


## TeamCity multinode setup improvements

To improve scalability of TeamCity, we're working on the ability to set up a cluster, where the main node distributes different responsibilities among other nodes and also handles such tasks as upgrading, licensing, diagnostics, and server configuration. Such multinode setup implies that all secondary nodes are uniform and can perform all tasks in an interchangeable way.     
In addition to the tasks that can already be performed by a secondary node, in this version secondary nodes can be assigned the "processing the build lifecycle" responsibility, which releases the main server from the build-related tasks, such as processing builds messages coming from agents, and allows you to significantly increase the number of agents.      
As a result, it is possible to start a single secondary node and assign several responsibilities to it or distribute these responsibilities between several secondary nodes:
* VCS changes collecting
* running builds processing
* serving as a read-only backup node: providing the user interface in the read-only mode

### Cached configuration files on nodes

In a multinode setup, the TeamCity Data Directory is shared among secondary nodes via network. When a secondary node launches, it reads configuration files located in the shared Data Directory. In large setups with thousands of projects and build configurations, downloading lots of files from network storage can take a lot of time.      
In TeamCity 2019.1, we have optimized this operation by creating a cache of configuration files stored under the node local Data Directory. The files are cached on the first node launch and then updated in runtime which makes the next launches faster.   

## Improvements for Amazon spot instances

### AWS Spot Fleet requests
   
You can now configure [spot fleets](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet.html) (collections of instances) to run images, which allows cutting costs by always utilizing just enough instances.  
To configure an allocation strategy for a spot fleet, open the AWS Management Console and go to __EC2 | Spot Requests | Request Spot Instances__. Here you can select the strategy, set the target capacity, add tags, and so on. Refer to [Spot Fleet Requests](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet-requests.html) for more information on the available parameters. When you are done, download the configuration as a JSON file.
Now you can add this configuration to the settings of an Amazon image in TeamCity. Select '_Spot Instance Fleet_' in the '_Source_' drop-down list, insert the JSON configuration into the text area below, and save the new image configuration.

<img src="SpotFleetConfig.png" width="600"/>

You can monitor the state of all running instances on the __Agents | Cloud__ tab.

### Other improvements

In accordance with how EC2 services manage instances, we have made the 'Max price' value optional for [spot instances](setting-up-teamcity-for-amazon-ec2.md) so they can be launched exactly like the on-demand ones. 


## Build artifacts publishing options

Build artifacts may comprise distribution packages, log files, reports, and so on, taking up a lot of storage space. When a build fails often, you may want to restrict artifacts publishing to successful builds only in order to save disk space.   
In other cases, you may need to investigate artifacts even if a build has been interrupted.    
To support all these scenarios, we have added two new options for publishing artifacts to the General Settings of a build configuration.   

Now you can choose when to publish artifacts:
* _Even if build fails_ (default): as in the previous versions of TeamCity, artifacts are published at the last step of a build if all previous steps have been completed, successfully or not. If the 'stop' command is issued, artifacts are not published.
* _Only if build status is successful_ (__new__): artifacts are published at the last step of a build if all previous steps have been completed successfully. TeamCity checks the current build status on the server before publishing artifacts.
* _Always, even if build stop command was issued_ (__new__): artifacts are published for all builds, even for interrupted ones (for example, after the 'stop' command was issued or after the time-out, specified in the build failure conditions).


## Go language support

TeamCity now supports the Go language: 

To build Go projects:
1. Make sure a Go compiler is installed on an agent.   
2. Add the '_Golang_' build feature to a build configuration.

To see Go tests in the TeamCity UI, run them with the  `-json` flag added to the [Command Line](command-line.md) build runner's script: `go test -json` or add the `env.GOFLAGS = -json` parameter to the build configuration.

<img src="GoEnvParameter.png" width="600"/>


<note>

Make sure you use only one of the methods to set the `-json` flag. Setting it in both build parameters and the runner's script will make Go fail to build.
</note>

## Using snapshot dependencies without synchronizing revisions

In a TeamCity build chain, all builds linked via snapshot dependencies use the synchronized revision of the source code, but in some cases this may be undesirable.   
In this version you can disable revision synchronization in a snapshot dependency via the _Enforce revision synchronization_  option, for example, when promoting an older build to a deployment build configuration. The build will be run using the latest deployment scripts.

<img src="EnforceRevisionSync.png" width="600"/>

## Branch filter for VCS of build configuration

We have added a branch filter to the [version control settings](configuring-vcs-settings.md) of a build configuration, similarly to filters in build triggers or test details. In the previous versions of TeamCity, VCS settings allowed disabling builds in the default branch only, but the branch filter offers a more flexible approach. To filter branches, use the syntax described in [Configuring branches](working-with-feature-branches.md).   

The VCS branch filter is applied before any other branch filter and limits branches shown in the custom build dialog, branches visible to triggers, and changes from snapshot dependencies.

## Log out of all sessions

If a user's password is compromised and Built-in Authentication is enabled on the TeamCity server, changing a user password does not mean that the user is logged out of all current sessions in progress, which is a potential security risk.    
Now there is a new option in your user profile, __Log out of all sessions__, forcing TeamCity to invalidate all user sessions with the user set, including the current one. The administrator account also has an option to force the user to log out of all sessions. It is also possible to log off all users using the corresponding option on the __Administration | Authentication__ page.

## Token-based authentication

In addition to basic authentication via credentials, TeamCity now supports authentication based on permanent access tokens. With tokens, you don't need to expose a user login and password in scripts. Tokens are also useful for the REST API authentication.   
You can enable/disable the Token-Based Authentication module in the advanced mode of the __Authentication__ page:

<img src="TokenAuth.png" width="1000"/>

 
## Permanent token for authentication in TeamCity-YouTrack integration

If you are using the TeamCity integration with YouTrack, you should know that the login-password authentication is deprecated and soon will no longer be supported by YouTrack.   

The TeamCity-YouTrack integration now supports for token-based authorization in REST API calls. It is recommended that you obtain a token and specify it in the connection settings to your issue tracker on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Issue Trackers__ page. 

## Canceling build via service message

When you need to cancel a build from a script, for example, a build cannot proceed normally due to the environment, or a build should be canceled form a subprocess, it is possible to do it via the TeamCity REST API passing the build ID, the server URL, as well as the authentication parameters (username and password) to the script.   
Now TeamCity offers a more elegant way of using a service message to do that just add a line to your build script telling TeamCity to cancel the build. If required, you can re-add the build to the queue after canceling it:

```Shell
echo ##teamcity[buildStop comment='canceling comment' readdToQueue='true']
```

Read more about [TeamCity service messages](service-messages.md).


## Faster agents upgrade

Previously, build agents downloaded all available tools from the server right after the server upgrade, which could significantly load and slow down the server. To reduce the agents' requests impact on the server and speed up the agents' upgrade, agents now download tools from the server only when starting the first build that requests these tools. The downloaded tools are stored on agents so builds don't spend time on downloading them again.

## Separate Maven artifact repository for all builds on agent

We have changed the [local artifact repository settings](maven.md#Local+Artifact+Repository+Settings) selection for Maven and added an option to create a separate repository for all builds run by an agent.

Here is what changed:


<table>

<tr>
<td>
Old option
</td>
<td>

New option

</td>
<td>

Description

</td>
</tr>

<tr>
<td>
-
</td>
<td>

Per agent (default)

</td>
<td>

Use a separate repository to store artifacts, produced by all builds run by an agent, under the agent system directory.

</td>
</tr>

<tr>
<td>
Enabled 'Use own local repository for this build configuration'
</td>
<td>

Per build configuration

</td>
<td>

Use a separate repository to store artifacts, produced by all builds of the current build configuration.

</td>
</tr>

<tr>
<td>
Disabled 'Use own local repository for this build configuration'
</td>
<td>

Maven default

</td>
<td>

Use the default Maven repository location. The repository is shared between all build configurations and all agents on the machine.

</td>
</tr>

</table>


## Other improvements
{id="other-improvements-1"}

* If the '_Docker Support_' feature is added to a build configuration, the TeamCity server will automatically add an agent requirement for an installed Docker, so the builds will run on compatible agents only. 
* Starting from this version, TeamCity supports Visual Studio 2019 in the related runners: [Visual Studio(sln)](visual-studio-sln.md), [MSBuild](msbuild.md), [Visual Studio Test](visual-studio-tests.md), [Inspections(ReSharper)](inspections-resharper.md), [Duplicates Finder(ReSharper)](duplicates-finder-resharper.md).
* Investigations Auto Assigner has got a new option, '_On second failure_', which prevents user assignment for the flaky tests/problems. A user is assigned to investigate a failure only when the failure repeats itself the second time in a row.
* New API for setting agent requirements based on build features.
* The former _GitHub Pull Requests_ plugin to _Pull Requests_.
* The server startup performance has been improved.
* The __[My Investigations](investigating-and-muting-build-failures.md#Investigations)__ page now displays failures in the currently running builds.
* [Build runners](supported-platforms-and-environments.md#.NET+Runners) now support .NET Framework 4.8.

## Known issues

* When you use .NET CLI for running a build that tries to pass credentials via some NuGet command, you may receive one of the following errors.
    * If .NET Core SDK 2.x is installed but its version is earlier than 2.1.400:   
    "_Could not load file or assembly 'System.Runtime, Version=\<version\>, Culture=neutral, PublicKeyToken=\<key\> or one of its dependencies. The system cannot find the file specified._"
    * If only .NET Core SDK 3.0 is installed on your server:   
    "_A plugin was not found at path \<path\> if only dotnet version 3.0 is installed on an agent._"
    
    To fix any of these problems, we recommend installing .NET Core SDK 1.x and/or 2.1.400 or later additionally to your current SDK version.
* If you use Docker images and Windows Server 2019 with process isolation, build agents may fail to start. To work around the issue, use the `hyper-v` isolation for Docker/LXC containers:

    ```Shell 
    
    docker run --isolation=hyperv ...
    # or
    podman run --isolation=hyperv ...
    ```



## Fixed issues

[Full list of fixed issues](https://confluence.jetbrains.com/display/TW/TeamCity+2019.1+Release+Notes)

## Previous releases

[What's New in TeamCity 2018.2](https://confluence.jetbrains.com/display/TCD18/What's+New+in+TeamCity+2018.2)
