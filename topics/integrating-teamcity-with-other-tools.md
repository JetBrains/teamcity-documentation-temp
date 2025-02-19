[//]: # (title: Integrating TeamCity with Other Tools)
[//]: # (auxiliary-id: Integrating TeamCity with Other Tools)

One of the key features of TeamCity is straightforward integration with modern software technologies and platforms. To ensure our users are able to integrate every component of their CI/CD pipeline with TeamCity, we either:
* provide smart detection and handy UI controls on the TeamCity side, or
* expose specialized [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html) endpoints for easier scripting and integration on a third-party system's side.

There are many places in the TeamCity UI where you can set up or adjust software integrations, depending on their context. This article gives an overview of third-party software and platforms supported in TeamCity out of the box. Remember that you can extend this scope by [installing additional plugins](installing-additional-plugins.md) or even [writing your own ones](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html).  
See also [versions of platforms and environments](supported-platforms-and-environments.md) currently supported in TeamCity.
{instance="tc"}

There are many places in the TeamCity UI where you can set up or adjust software integrations, depending on their context. This article gives an overview of third-party software and platforms supported in TeamCity out of the box.  
See also [versions of platforms and environments](supported-platforms-and-environments.md) currently supported in TeamCity.
{instance="tcc"}

The tables below are updated in accordance with the newly introduced integrations and whenever we have extra guides to share.

## Operating Systems and Databases
{instance="tc"}

<table>
<tr><td>Software</td><td>Available Integrations</td></tr>

<tr><td>

**Windows**

</td><td>

* [Installing TeamCity Server on a Windows machine](install-teamcity-server-on-windows.md)
* [Installing TeamCity Agent on a Windows machine](install-teamcity-agent.md#Install+from+Windows+Executable+File) and running builds on Windows

</td></tr>

<tr><td>

**Linux**

</td><td>

* [Installing TeamCity Server on a Linux machine](install-teamcity-server-on-linux-or-macos.md)
* [Installing TeamCity Agent on a Linux machine](install-teamcity-agent.md) and running builds on Linux

</td></tr>

<tr><td>

**macOS**

</td><td>

* [Installing TeamCity Server on a macOS machine](install-teamcity-server-on-linux-or-macos.md)
* [Installing TeamCity Agent on a macOS machine](install-teamcity-agent.md) and running builds on macOS

</td></tr>

<tr><td>

**MySQL**

</td><td>

* [Storing build history, users, results, and some runtime data](set-up-external-database.md#MySQL)

</td><td></td></tr>

<tr><td>

**Microsoft SQL Server**

</td><td>

* [Storing build history, users, results, and some runtime data](set-up-external-database.md#Microsoft+SQL+Server)

</td><td></td></tr>

<tr><td>

**PostgreSQL**

</td><td>

* [Storing build history, users, results, and some runtime data](set-up-external-database.md#PostgreSQL)

</td><td></td></tr>

<tr><td>

**Oracle**

</td><td>

* [Storing build history, users, results, and some runtime data](set-up-external-database.md#Oracle)

</td><td></td></tr>

</table>

## Software Development Platforms and Build Tools

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**Java**, including
* **Maven**
* **Gradle**
* **Ant**

</td><td>

* Automating build tasks with:
  * [Apache Maven](maven.md), using [special build triggers for Maven](configuring-maven-triggers.md)
  * [Gradle](gradle.md)
  * [Ant](ant.md)
* [Autodiscovery of Maven, Gradle, and Ant build steps in a source project](configuring-build-steps.md#Autodetecting+Build+Steps)
* Autodetection of [Maven](maven.md#Maven+runner+settings) and Gradle installations on a build agent
* [Running a Maven, Gradle, or Ant build step inside a Docker container](container-wrapper.md)
* [Reporting Java code inspection results for a build](inspections.md)
* [Detecting Java code duplicates in a build's source](duplicates-finder-java.md)
* [Detailed test reports for JUnit and TestNG frameworks on the fly](java-testing-frameworks-support.md)
* [Detailed code coverage](configuring-java-code-coverage.md)

</td><td>

* [Building and Testing Java with Maven in TeamCity (2021)](https://www.jetbrains.com/teamcity/tutorials/maven-build-configure-test/)
* [Incremental Building with Maven and TeamCity (2012)](https://blog.jetbrains.com/teamcity/2012/03/incremental-building-with-maven-and-teamcity/)
* [Building and Testing Java with Gradle in TeamCity (2021)](https://www.jetbrains.com/teamcity/tutorials/gradle-build-configure-test/)
* [Configuring Artifact Dependencies Using Ant Build Script](artifact-dependencies.md#Configuring+Artifact+Dependencies+Using+Ant+Build+Script)
* [Get TeamCity Artifacts Using HTTP, Ant, Gradle and Maven (2012)](https://blog.jetbrains.com/teamcity/2012/06/teamcity-ivy-gradle-maven/)

</td></tr>

<tr><td>

**.NET**, including
* **MSBuild**
* **NAnt**
* **Visual Studio Solutions**
* **FxCop**
* **C# Script**
* **C#**
* **VB.NET**
* **NuGet**

</td><td>

* [Autodiscovery of .NET build steps in a source project](configuring-build-steps.md#Autodetecting+Build+Steps)
* [Autodetection of a .NET installation on a build agent](net.md#.NET+Version+Detection+Algorithm)
* Cross-platform .NET builds
* [Running .NET CLI commands](net.md#Build+Runner+Options)
* [Running MSBuild commands](net.md#msbuild)
* [Running devenv](net.md#devenv-build-action)
* [Running any custom .NET commands](net.md#Custom+Commands)
* [Running a .NET build step inside a Docker container](container-wrapper.md)
* [Detailed and structured test reports for .NET frameworks on the fly](net-testing-frameworks-support.md)
* [Detailed code coverage](configuring-.net-code-coverage.md)
* [Reporting .NET code inspection results for a build](inspections-resharper.md)
* [Detecting .NET code duplicates in a build's source](duplicates-finder-resharper.md)
* [Uploading to and downloading packages from NuGet feeds](nuget.md):
  * Support for public, private NuGet feeds, and an ability to [use TeamCity as a private NuGet feed](using-teamcity-as-nuget-feed.md)
      {instance="tc"}
  * Automatically [installing](nuget-installer.md), [packing](nuget-pack.md), and [publishing packages](nuget-publish.md)
  * [Triggering builds on updates in a NuGet feed](nuget-dependency-trigger.md)
* [Using FxCop for inspecting .NET assemblies](fxcop.md)
* [Using NAnt to run build scripts](nant.md)
* [Using C# Scripting to run build scripts](c-script.md)

</td><td>

* [Building .NET Projects Using TeamCity (2021)](https://www.jetbrains.com/teamcity/tutorials/dotnet-build-configure-test/)
* [TeamCity integration with .NET, Part 1: New approach and demo (2020)](https://blog.jetbrains.com/teamcity/2020/12/teamcity-integration-with-net-part-1-new-approach-and-demo/)
* [TeamCity integration with .NET, Part 2: Testing and building projects (2020)](https://blog.jetbrains.com/teamcity/2020/12/teamcity-integration-with-net-part-2-testing-and-building-projects/)
* [TeamCity integration with .NET, Part 3: Deploying projects (2020)](https://blog.jetbrains.com/teamcity/2020/12/teamcity-integration-with-net-part-3-deploying-projects/)
* [How to automate CI/CD tasks with C# Scripting in TeamCity (2021)](https://blog.jetbrains.com/teamcity/2021/11/how-to-automate-ci-cd-tasks-with-c-scripting-in-teamcity/)

</td></tr>

<tr><td>

**Command Line**

</td><td>

* Performing any command-line actions across platforms with the simple [Command Line](command-line.md) runner

</td><td>

* [Video tutorial: How to run command line scripts (2022)](https://www.youtube.com/watch?v=oKNdLRrO3mA)

</td></tr>

<tr><td>

**PowerShell**

</td><td>

* [Autodetection of a PowerShell installation on a build agent](net.md#.NET+Version+Detection+Algorithm)
* [Running PowerShell scripts across platforms](powershell.md) for various build tasks
* [Running a PowerShell build step inside a Docker container](container-wrapper.md)

</td><td>

</td></tr>

<tr><td>

**Python**

</td><td>

* [Autodiscovery of Python build steps in a source project](configuring-build-steps.md#Autodetecting+Build+Steps)
* [Autodetection of a Python installation on a build agent](python.md#pythonVersion)
* [Cross-platform execution of Python scripts](python.md)
* Support for Pipenv, Poetry, Venv, and Virtualenv
* A structured build log output based on build stages and test reports
* [Running a Python build step inside a Docker container](container-wrapper.md)

</td><td>

* [Building and Testing Python Code with TeamCity (2021)](https://www.jetbrains.com/teamcity/tutorials/python-build-configure-test/)
* [Video tutorial: New in TeamCity 2020.2: Python Build Runner](https://www.youtube.com/watch?v=DOLY1K5e8hQ)

</td></tr>

<tr><td>

**Kotlin**

</td><td>

* [Cross-platform execution of build scripts written in Kotlin](kotlin-script.md)
* [Using Kotlin DSL for configuring TeamCity project and build settings as code](kotlin-dsl.md)

</td><td>

* [Kotlin DSL for Beginners: Recommended Refactorings (2021)](https://blog.jetbrains.com/teamcity/2021/04/kotlin-dsl-for-beginners-recommended-refactorings/)
* [Creating TeamCity project templates with Kotlin DSL context parameters (2020)](https://blog.jetbrains.com/teamcity/2020/09/creating-teamcity-project-templates-with-kotlin-dsl-context-parameters/)
* [Configuration as Code, Part 1: Getting Started with Kotlin DSL (2019)](https://blog.jetbrains.com/teamcity/2019/03/configuration-as-code-part-1-getting-started-with-kotlin-dsl/)
* [Configuration as Code, Part 2: Working with Kotlin Scripts (2019)](https://blog.jetbrains.com/teamcity/2019/03/configuration-as-code-part-2-working-with-kotlin-scripts/)
* [Configuration as Code, Part 3: Creating Build Configurations Dynamically (2019)](https://blog.jetbrains.com/teamcity/2019/04/configuration-as-code-part-3-creating-build-configurations-dynamically/)
* [Configuration as Code, Part 4: Extending the TeamCity DSL (2019)](https://blog.jetbrains.com/teamcity/2019/04/configuration-as-code-part-4-extending-the-teamcity-dsl/)
* [Configuration as Code, Part 5: Using DSL extensions as a library (2019)](https://blog.jetbrains.com/teamcity/2019/04/configuration-as-code-part-5-using-dsl-extensions-as-a-library/)
* [Configuration as Code, Part 6: Testing Configuration Scripts (2019)](https://blog.jetbrains.com/teamcity/2019/05/configuration-as-code-part-6-testing-configuration-scripts/)

</td></tr>

<tr><td>

**Node.js**

</td><td>

* [Autodiscovery of JavaScript build steps in a source project](nodejs.md#Autodetecting+JavaScript+Steps)
* [Running npm, yarn, and node commands within a build](nodejs.md)
* A structured build log output based on build stages and test reports
* [Running a Node.js script inside a Docker container](container-wrapper.md)
* Support for ESlint, Jest, and Mocha
* [Accessing private NPM registries](nodejs.md#Accessing+Private+NPM+Registries)

</td><td>

* [Video tutorial: New in TeamCity 2021.1: Node.js Runner](https://www.youtube.com/watch?v=8-m5GFrxUzw)

</td></tr>

<tr><td>

**Ruby**

</td><td>

* [Automating build tasks by running a Rakefile within a build](rake.md)
* Support for Test::Unit, Test-Spec, Shoulda, RSpec, and Cucumber

</td><td>

</td></tr>

<tr><td>

**SBT (Scala)**

</td><td>

* [Building and testing sources with Simple Build Tool's (Scala)](simple-build-tool-scala.md)

</td><td>

</td></tr>

<tr><td>

**Xcode**

</td><td>

* [Running an Xcode scheme within a build](xcode-project.md)
* A structured build log output based on Xcode build stages, compilation errors, and test reports
* Automatically added [agent requirements](configuring-agent-requirements.md) to a specific version of Xcode or IDE

</td><td>
</td></tr>

</table>

## Testing Frameworks and Code Coverage

<table>
<tr><td>Software</td><td>Available Integrations</td></tr>

<tr><td>

**JUnit**

</td><td>

* Detailed on-the-fly test reporting for [supported Java testing frameworks](java-testing-frameworks-support.md)
* [Running risk-group tests first](running-risk-group-tests-first.md)

</td></tr>

<tr><td>

**TestNG**

</td><td>

* Detailed on-the-fly test reporting for [supported Java testing frameworks](java-testing-frameworks-support.md)
* [Running risk-group tests first](running-risk-group-tests-first.md)

</td></tr>

<tr><td>

**NUnit**

</td><td>

* [Running NUnit tests](nunit.md) and showing detailed test results in the build overview
* [Running risk-group tests first](running-risk-group-tests-first.md)

</td></tr>

<tr><td>

**MSTest / VSTest**

</td><td>

* [Testing a project and automatically importing detailed test results](net.md#vstest)

</td></tr>

<tr><td>

**MSpec**

</td><td>

* [Running MSpec tests and code coverage](mspec.md) and showing detailed test results in the build overview

</td></tr>

<tr><td>

**Test::Unit, Test-Spec, Shoulda, RSpec, and Cucumber**

</td><td>

* [Running tests](rake.md) and showing detailed test results in the build overview

</td></tr>

<tr><td>

**Pytest**

</td><td>

* [Running tests](python.md) and showing detailed test results in the build overview

</td></tr>

<tr><td>

**Python Unittest**

</td><td>

* [Running tests](python.md) and showing detailed test results in the build overview

</td></tr>

<tr><td>

**JaCoCo**

</td><td>

* [Measuring coverage metrics and code complexity of a Java build's sources](jacoco.md)
* [Importing JaCoCo coverage](jacoco.md#Importing+JaCoCo+coverage+data+to+TeamCity)

</td></tr>

<tr><td>

**Emma**

</td><td>

* [Measuring coverage metrics and code complexity of a Java build's sources](emma.md)

</td></tr>

<tr><td>

**IntelliJ IDEA coverage**

</td><td>

* [Measuring coverage metrics and code complexity of a Java build's sources](intellij-idea.md)

</td></tr>

<tr><td>

**JetBrains dotCover**

</td><td>

* [Measuring coverage metrics and code complexity of a .NET build's sources](jetbrains-dotcover.md)

</td></tr>

<tr><td>

**NCover**

</td><td>

* [Measuring coverage metrics and code complexity of a .NET build's sources](ncover.md)

</td></tr>

<tr><td>

**PartCover**

</td><td>

* [Measuring coverage metrics and code complexity of a .NET build's sources](partcover.md)

</td></tr>

<tr><td>

**Flake8**

</td><td>

* [Parsing Python files and showing coverage reports in build results](partcover.md)

</td></tr>

<tr><td>

**Pylint**

</td><td>

* [Parsing Python files and showing coverage reports in build results](partcover.md)

</td></tr>


</table>

## Version Control Systems

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**Git**

</td><td>

* [Building projects based on Git repositories](git.md)
* [Remote run](remote-run.md), [remote debug](remote-debug.md), and [pretesting commits](pre-tested-delayed-commit.md) of Git-based projects
* Multiple checkout modes on [server](git.md#Server+Settings) and [agent](git.md#GitAgentSettings) machines
* [Support for Git LFS](git.md#LFS+and+Submodules+Support)
* [Automatically tagging build sources](vcs-labeling.md)
* [Automatically merging build sources after successful builds](automatic-merge.md)


</td><td></td></tr>

<tr><td>

**Subversion**

</td><td>

* [Building projects based on SVN repositories](subversion.md)
* [Remote run](remote-run.md), [remote debug](remote-debug.md), and [pretesting commits](pre-tested-delayed-commit.md) of SVN-based projects
* [Automatically tagging build sources](vcs-labeling.md#Subversion+Labeling+Rules)

</td><td></td></tr>

<tr><td>

**Perforce**

</td><td>

* [Building projects based on Perforce Helix Core repositories](perforce.md)
* [Using Perforce streams as feature branches](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Streams) and building their sources independently of each other
* [Pre-testing and pre-building files in shelved changelists](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Shelved+Files)
* [Reporting build statuses to code reviews in Perforce Helix Swarm](integrating-teamcity-with-perforce.md#Publishing+Build+Statuses+to+Perforce+Helix+Swarm)
* [Automatic labeling of sources](vcs-labeling.md#Labeling+in+Perforce)
* [Using post-commit hooks to avoid background polling](configuring-vcs-post-commit-hooks-for-teamcity.md#Perforce+Server)

</td><td>

* [Integrating TeamCity with Perforce (2022)](integrating-teamcity-with-perforce.md)

</td></tr>

<tr><td>

**Azure DevOps (TFVC)**

</td><td>

* [Building projects based on TFVC repositories](perforce.md) across platforms
* [Automatic labeling of sources](vcs-labeling.md)
* [Remote run](remote-run.md), [remote debug](remote-debug.md), and [pretesting commits](pre-tested-delayed-commit.md) of TFVC-based projects

</td><td></td></tr>

<tr instance="tc"><td>

**CVS**

</td><td>

* [Building projects based on CVS repositories](cvs.md) across platforms
* [Automatic labeling of sources](vcs-labeling.md)
* [Remote run](remote-run.md), [remote debug](remote-debug.md), and [pretesting commits](pre-tested-delayed-commit.md) of CVS-based projects

</td><td></td></tr>

<tr><td>

**Mercurial**

</td><td>

* [Building projects based on Mercurial repositories](mercurial.md) across platforms
* [Automatic labeling of sources](vcs-labeling.md)

</td><td></td></tr>

<tr instance="tc"><td>

**Borland StarTeam**

</td><td>

* [Building projects based on StarTeam repositories](starteam.md) across platforms
* [Automatic labeling of sources](vcs-labeling.md)

</td><td></td></tr>

</table>

## Data Transfer Protocols

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>
<tr><td>

**SSH**

</td><td>

* [Storing SSH keys in TeamCity and reusing them in builds](git.md) by means of an [SSH agent](ssh-agent.md)
* [Executing arbitrary remote commands using SSH](ssh-exec.md)
* [Uploading files/directories via SSH (using SCP or SFTP protocols)](ssh-upload.md)

</td><td>

* [Video tutorial: How to checkout SSH repositories (2022)](https://www.youtube.com/watch?v=nUTb1BjMMoE)
* [Video tutorial: How to use SSH during your builds (2022)](https://www.youtube.com/watch?v=D6JOyGd4pWI)

</td>

</tr>

<tr instance="tc"><td>

**Amazon S3** protocols

</td><td>

* [Uploading build artifacts to an Amazon S3 storage](configuring-artifacts-storage.md#Amazon+S3+Support)

</td><td></td></tr>

<tr><td>

**SMB**

</td><td>

* [Uploading files to Windows shares during builds via SMB](smb-upload.md)

</td><td></td></tr>

<tr><td>

**FTP**

</td><td>

* [Uploading files to an FTP server during builds](ftp-upload.md)

</td><td></td></tr>

</table>

## VCS Hosting Services

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**GitHub.com / GitHub Enterprise**

</td><td>

* [Creating a project or build configuration from a GitHub repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL)
* [Guessing VCS root settings from a GitHub repository URL](guess-settings-from-repository-url.md)
* [Creating a Git VCS root from a GitHub repository URL](git.md)
* [Inserting links to GitHub issues when detecting their IDs in commit messages](github.md)
* [Building sources of GitHub pull requests](pull-requests.md#GitHub+Pull+Requests)
* [Authenticating with a GitHub account in TeamCity](configuring-authentication-settings.md#GitHub)

</td><td>

* [Video tutorial: How to integrate TeamCity and GitHub (2022)](https://www.youtube.com/watch?v=hbh7sk5sGTc)
* [Video tutorial: How to use GitHub commit hooks for faster checkouts (2022)](https://www.youtube.com/watch?v=VzDI2HoiHk4)
* [Video tutorial: How to send build information to GitHub, Jira etc. (2022)](https://www.youtube.com/watch?v=o0oj7mOcNvc)
* [Video tutorial: How to work with pull requests (2022)](https://youtu.be/4yFck9PvXI4)
* [Building GitHub pull requests with TeamCity (2019)](https://blog.jetbrains.com/teamcity/2019/08/building-github-pull-requests-with-teamcity/)

</td></tr>

<tr><td>

**GitLab.com / GitLab CE/EE**

</td><td>

* [Creating a project or build configuration from a GitLab repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL)
* [Guessing VCS root settings from a GitLab repository URL](guess-settings-from-repository-url.md)
* [Inserting links to GitLab issues when detecting their IDs in commit messages](gitlab.md)
* [Building sources of GitLab merge requests](pull-requests.md#GitLab+Merge+Requests)
* [Authenticating with a GitLab account in TeamCity](configuring-authentication-settings.md#GitLab.com)

</td><td>

* [Video tutorial: New in TeamCity 2019.1 — GitLab integration and merge requests support](https://www.youtube.com/watch?v=aE150gCfQyI)
* [Video tutorial: How to work with pull requests (2022)](https://youtu.be/4yFck9PvXI4)

</td></tr>

<tr><td>

**Bitbucket Cloud / Bitbucket Server**

</td><td>

* [Creating a project or build configuration from a Bitbucket repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL)
* [Guessing VCS root settings from a Bitbucket repository URL](guess-settings-from-repository-url.md)
* [Creating a Mercurial VCS root from a Bitbucket repository URL](mercurial.md)
* [Building sources of Bitbucket pull requests](pull-requests.md#Bitbucket+Cloud+Pull+Requests)
* [Inserting links to Bitbucket Cloud issues when detecting their IDs in commit messages](bitbucket-cloud.md)
* [Authenticating with a Bitbucket Cloud account in TeamCity](configuring-authentication-settings.md#Bitbucket+Cloud)

</td><td>

* [Video tutorial: New in TeamCity 2020.2: Bitbucket Cloud Pull Request Support](https://youtu.be/M2wi6l0pZe4)
* [Run and view TeamCity builds from Bitbucket](https://blog.jetbrains.com/teamcity/2021/05/run-and-view-teamcity-builds-from-bitbucket/)
* [Video tutorial: How to work with pull requests (2022)](https://youtu.be/4yFck9PvXI4)

</td></tr>

<tr><td>

**Azure DevOps Services**

</td><td>

* [Creating a project or build configuration from an Azure DevOps repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL)
* [Guessing VCS root settings from an Azure DevOps repository URL](guess-settings-from-repository-url.md)
* [Building sources of Azure DevOps pull requests](pull-requests.md#Azure+DevOps+Pull+Requests)
* [Inserting links to Azure Board Work Items issues when detecting their IDs in commit messages](azure-board-work-items.md).
* [Authenticating with an Azure DevOps Services account in TeamCity](configuring-authentication-settings.md#Azure+DevOps+Services)

</td><td>

* [Video tutorial: How to work with pull requests (2022)](https://youtu.be/4yFck9PvXI4)

</td></tr>

<tr><td>

**JetBrains Space**

</td><td>

* [Creating a project or build configuration from a JetBrains Space repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* [Guessing VCS root settings from a JetBrains Space repository URL](guess-settings-from-repository-url.md).
* [Authenticating with a JetBrains Space account in TeamCity](configuring-authentication-settings.md#JetBrains+Space).

</td><td>

* [How to configure CI/CD for JetBrains Space](how-to-configure-cicd-for-jetbrains-space.md)

</td></tr></table>

## Virtualization Solutions

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**Docker**

</td><td>

* [Launching Docker commands and creating a Docker image during a build](docker.md)
* [Running some build steps inside a Docker container, across platforms](container-wrapper.md)
* [Automatically signing in to Docker registry before each build](docker-support.md)
* [Displaying Docker-related information in build results](docker-support.md)
* [Running multiple containers in a build with Docker Compose](docker-compose.md)
* [Cleaning obsolete Docker images on build agents](integrating-teamcity-with-container-managers.md#Docker+Disk+Space+Cleaner)

</td><td>

* [Integrating TeamCity with Docker](integrating-teamcity-with-container-managers.md)

</td>

</tr>

</table>

## Cloud Hosting and Orchestration Solutions
{instance="tc"}

<table>
<tr><td>Software</td><td>Available Integrations</td></tr>

<tr><td>

**Amazon EC2**

</td><td>

* [Running builds on cloud agents](setting-up-teamcity-for-amazon-ec2.md)

</td></tr>

<tr><td>

**VMWare vSphere and vCenter**

</td><td>

* [Running builds on cloud agents](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md)

</td></tr>

<tr><td>

**Kubernetes**

</td><td>

* [Running builds on cloud agents](setting-up-teamcity-for-kubernetes.md)

</td></tr>
<tr><td>

**Microsoft Azure**  
(via additional plugin)

</td><td>

* [Running builds on cloud agents](https://plugins.jetbrains.com/plugin/9260-azure-resource-manager-cloud-support)

Read [how to install a plugin in TeamCity](installing-additional-plugins.md).

</td></tr>
<tr><td>

**Google Cloud**  
(via additional plugin)

</td><td>

* [Running builds on cloud agents](https://plugins.jetbrains.com/plugin/9704-google-cloud-agents)

Read [how to install a plugin in TeamCity](installing-additional-plugins.md).

</td></tr>
</table>

## Issue Trackers

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**JetBrains YouTrack**

</td><td>

* [Providing links to related YouTrack issues from the TeamCity build overview](youtrack.md)

</td><td></td></tr>

<tr><td>

**Atlassian Jira**

</td><td>

* [Providing links to related Jira issues from the TeamCity build overview](jira.md)
* [Reporting TeamCity build statuses to Jira Cloud](jira-cloud-integration.md)

</td><td>

* [Video tutorial: How to integrate TeamCity and JIRA (Cloud) (2022)](https://www.youtube.com/watch?v=rK7faWbCh0Q)

</td></tr>

<tr><td>

**Bugzilla**

</td><td>

* [Providing links to related Bugzilla issues from the TeamCity build overview](bugzilla.md)

</td><td></td></tr></table>

__Integration with issue trackers is also represented in terms of integration with [VCS hosting services](#VCS+Hosting+Services)__.

## IDEs

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**IntelliJ Platform**

</td><td>

* [Generating code coverage within a build](intellij-idea.md)
* [Building a project created in IntelliJ IDEA](intellij-idea-project.md)
* [Remote run](remote-run.md), [remote debug](remote-debug.md), and [pretesting commits](pre-tested-delayed-commit.md)
* [Other numerous features](intellij-platform-plugin.md)

</td><td>

* [TeamCity Integration with IntelliJ-based IDEs (2017)](https://blog.jetbrains.com/teamcity/2017/10/teamcity-integration-with-intellij-based-ides/)

</td></tr>

<tr><td>

**Microsoft Visual Studio**

</td><td>

* [Running Visual Studio in command-line mode within a build](net.md#Visual+Studio+Command-Line+Mode)
* [Remote run](remote-run.md) and [pretesting commits](pre-tested-delayed-commit.md)

</td><td>

</td></tr>

</table>

## Notification Services

<table>
<tr><td>Software</td><td>Available Integrations</td><td>Extra Guides and Tutorials</td></tr>

<tr><td>

**Slack**

</td><td>

* [Sending customizable notifications on different events in TeamCity to a Slack channel](notifications.md#Slack+Notifier)

</td><td>

* [Video tutorial: How to integrate TeamCity and Slack (2022)](https://www.youtube.com/watch?v=d_Xuw7kkp4c)

</td></tr>

<tr><td>

**Web browsers**

</td><td>

* [Showing customizable notifications on different TeamCity events in Google Chrome, Microsoft Edge, Opera, and Mozilla Firefox](browser-notifier.md)

</td><td>

* [New TeamCity Notifier browser extension (2020)](https://blog.jetbrains.com/teamcity/2020/05/new-teamcity-notifier-browser-extension/)

</td></tr>

</table>

__TeamCity can also [send notifications via email](notifications.md#Email+Notifier).__


