[//]: # (title: Supported Platforms and Environments)
[//]: # (auxiliary-id: Supported Platforms and Environments)

This page covers software-related environments TeamCity works with. For hardware-related notes, see [this section](system-requirements.md#Estimating+External+Database+Capacity).
{instance="tc"}

## Operating Systems

### TeamCity Server
{instance="tc"}

TeamCity Server is a web application that runs within a capable J2EE servlet container. It requires a Java SE JRE installation to run. See [notes](how-to.md#Install+Non-Bundled+Version+of+Java) on how to install Java on a TeamCity server.

#### Supported Java Versions
{id="Supported+Java+Versions+for+TeamCity+Server" help-id="Supported+Java+Versions+for+TeamCity+Server"}

Supported Java versions: __OpenJDK and Oracle Java 8 (8u16 or later) to 21 (32 or 64 bit)__. Using 64-bit Java is recommended.

The TeamCity server Windows installer and server Docker images come __bundled with [Amazon Corretto](https://aws.amazon.com/corretto/) 64-bit Java 17__.

For Apple ARM systems (for example, Apple M1 or M2), consider using a different version of Java, like [Azul OpenJDK](https://www.azul.com/downloads/?package=jdk#download-openjdk).

>Java 8 support will be discontinued in one of the future TeamCity releases. If you use a non-bundled version of Java 8, we highly recommend that you migrate your server to a newer version of Java.
> 
{style="warning"}

#### Supported Platforms
{id="Supported+Platforms+for+TeamCity+Server" help-id="Supported+Platforms+for+TeamCity+Server"}

>The core features of TeamCity Server are platform-independent. See [considerations](system-requirements.md#Choosing+Server+OS%2FPlatform) on choosing the server platform.

Generally, __all the recent versions of Windows, Linux, and macOS are supported__. If you find any compatibility issues with any of the operating systems, [let us know](feedback.md).

TeamCity Server is tested under the following operating systems:
* Linux (Ubuntu, Debian, RedHat, SUSE, and others)
* macOS
* Windows 7/7x64
* Windows Server 2008, 2012, 2016, 2019, 2022
* Server Core installation of Windows Server 2016
* Windows 10 and 11 under the Tomcat 8.5+ web application server
   
Reportedly works without known issues on:
* Windows 7+
* Windows Server 2008 R2
* Solaris
* FreeBSD
* IBM z/OS
* HP-UX
   
Windows XP/XP x64 are not supported.

Regarding architectural compatibility, TeamCity is not limited to any specific list. The verified and thoroughly tested architectures include:

* Intel x86
* AMD64 (x86_64)
* Apple Silicon (M chips)
* Amazon ARM (Graviton)

If you are using a different architecture that is not explicitly mentioned above, you may expect the TeamCity server to support it, provided that your operating system is supported and a suitable Java Virtual Machine (JVM) option exists.


### TeamCity Agent
{instance="tc"}

TeamCity Agent is a standalone Java application. It requires a Java SE JRE installation to run. See [notes](configure-java-for-agent.md) on how to configure Java on agents.

#### Supported Java Versions
{id="Supported+Java+Versions+for+TeamCity+Agent" help-id="Supported+Java+Versions+for+TeamCity+Agent"}

Supported Java versions: __OpenJDK and Oracle Java 8 (8u16 or later) to 21 (32 or 64 bit)__. Using 64-bit Java is recommended.

The TeamCity agent Windows installer and agent Docker images come __bundled with [Amazon Corretto](https://aws.amazon.com/corretto/) 64-bit Java 17__.

For Apple ARM systems (for example, Apple M1 or M2), consider using a different version of Java, like [Azul OpenJDK](https://www.azul.com/downloads/?package=jdk#download-openjdk).

> Note that Java versions specified in this section are requirements to run the agent itself. Builds can utilize [other versions](predefined-build-parameters.md#Detecting+Java+on+Agent) installed on agent machines (for example, JDK 19).
> 
{style="note"}

>Java 8 support will be discontinued in one of the future TeamCity releases. If you use a non-bundled version of Java 8, we highly recommend that you migrate your server to a newer version of Java.
>
{style="warning"}

#### Supported Platforms
{id="Supported+Platforms+for+TeamCity+Agent" help-id="Supported+Platforms+for+TeamCity+Agent"}

TeamCity Agent is tested under the following operating systems:
* Linux
* macOS
* Windows 7/7x64
* Windows 10, 11
* Windows Server 2003/2008, 2012, 2016, 2019, 2022
* Server Core installation of Windows Server 2016

Reportedly works on:
* Windows XP/XP x64
* Windows 2000 (interactive mode only)
* Solaris
* FreeBSD
* IBM z/OS
* HP-UX

Regarding architectural compatibility, TeamCity is not limited to any specific list. The verified and thoroughly tested architectures include:

* Intel x86
* AMD64 (x86_64)
* Apple Silicon (M chips)
* Amazon ARM (Graviton)

If you are using a different architecture that is not explicitly mentioned above, you may expect the TeamCity Agent to run on it, provided that your operating system is supported and a suitable Java Virtual Machine (JVM) option exists.

### TeamCity Agent
{instance="tcc"}

TeamCity Agent is a standalone Java application. TeamCity Cloud supports two types of agents:
* Hosted by JetBrains
* Hosted by a customer

You can combine agents of both types in your installation. Read more information on licensing these agents in [Subscription and Licensing](teamcity-cloud-subscription-and-licensing.md).

#### JetBrains-Hosted Agents

These agents are automatically maintained by JetBrains and don't require to be installed or configured by a customer. To learn more about available agent types and software installed on them, refer to this article: [](jetbrains-hosted-agents.md).

#### Self-Hosted Agents

You can install a build agent locally on your machine, similarly to how you would do it in [TeamCity On-Premises](https://www.jetbrains.com/help/teamcity/setting-up-and-running-additional-build-agents.html), and connect it to the TeamCity Cloud instance. Note that you need to acquire a [concurrent build slot](teamcity-cloud-subscription-and-licensing.md#Using+Build+Credits) for each self-hosted agent.

#### Supported Java Versions
{id="Supported+Java+Versions+for+TeamCity+Agent" help-id="Supported+Java+Versions+for+TeamCity+Agent"}

Since agents are Java applications, you need to install Java SE JRE on machines that will run self-hosted agents.

Supported Java versions: OpenJDK and Oracle Java 8 - 21. We recommend using the latest available version of JDK. See this article for more information: [Configure Java for Agent](configure-java-for-agent.md).

> Note that Java versions specified in this section are requirements to run the agent itself. Builds can utilize [other versions](predefined-build-parameters.md#Detecting+Java+on+Agent) installed on agent machines (for example, JDK 19).
> 
{style="note"}

#### Supported Platforms
{id="Supported+Platforms+for+TeamCity+Agent" help-id="Supported+Platforms+for+TeamCity+Agent"}

TeamCity Agent is tested under the following operating systems:
* Linux
* macOS
* Windows 7/7x64
* Windows 10
* Windows Server 2003/2008, 2012, 2016, 2019, 2022
* Server Core installation of Windows Server 2016

Reportedly works on:
* Windows XP/XP x64
* Windows 2000 (interactive mode only)
* Solaris
* FreeBSD
* IBM z/OS
* HP-UX

## Browsers

The TeamCity web interface is mostly W3C-compliant, so any modern browser should work well with TeamCity. The recent versions of the following browsers have been specifically tested and reported to work correctly:
* Google Chrome
* Mozilla Firefox 
* Safari under macOS
* Microsoft Edge
* Opera
   
## Build Runners

### Java Runners

<table>

<tr><td>Runner</td><td>Supported versions</td><td>Bundled versions</td></tr>

<tr><td>

[Ant](ant.md)

</td><td>1.6-1.10</td><td>1.10.10</td></tr>

<tr><td>

[Maven](maven.md)

</td><td>2.0.x, 2.x, 3.x</td><td>Version 3.9.6 is downloaded and installed after TeamCity server first starts. Versions 2.2.1, 3.0.5, 3.1.1, 3.2.5, 3.3.9, 3.5.4, 3.6.3, 3.8.6 are additionally installed if required by build configurations.</td></tr>

<tr><td>

[IntelliJ IDEA Project](intellij-idea-project.md)

</td><td></td><td></td></tr>

<tr><td>

[Gradle](gradle.md)

</td><td>0.9-rc-1 or later</td><td></td></tr>

<tr><td>

[Java Inspections](inspections.md)

</td><td></td><td></td></tr>

<tr><td>

[Java Duplicates](duplicates-finder-java.md)

</td><td></td><td></td></tr>

</table>

### .NET Runners

We recommend that you use the __unified [.NET](net.md) runner__ to run .NET projects in TeamCity. See its requirements [here](net.md#Requirements).

Other .NET runners:

<table>

<tr><td>Runner</td><td>Supported versions</td><td>Requirements</td></tr>

<tr><td>

[C# Script](c-script.md)

</td><td>

.NET 6.0 or later

</td><td>

.NET 6.0 or later installed on the build agent, or can be run inside a Docker container with .NET 6.0 or later

</td></tr>

<tr><td>

[MSBuild](msbuild.md)

</td><td>Microsoft Build Tools 2013, 2015, 2017, 2019</td><td>.NET Framework or Mono installed on the build agent</td></tr>

<tr><td>

[NAnt](nant.md)

</td><td>0.85 - 0.91 alpha 2</td><td>.NET Framework or Mono installed on the build agent</td></tr>

<tr><td>

[Microsoft Visual Studio Solutions](visual-studio-sln.md)

</td><td></td><td>A corresponding version of Microsoft Visual Studio installed on the build agent</td></tr>

<tr><td>

[FxCop](fxcop.md)

</td><td></td><td>FxCop installed on the build agent</td></tr>

<tr><td>

[Duplicates Finder for C# and VB.NET code](duplicates-finder-resharper.md) based on [ReSharper Command Line Tools](https://www.jetbrains.com/resharper/features/command-line.html)

</td><td>Supported languages are C# up to version 4.0 and VB.NET version 8.0 - 10.0</td><td>.NET Framework 4.6.1 or later installed on the build agent</td></tr>

<tr><td>

[Inspections for .NET](inspections-resharper.md) based on [ReSharper Command Line Tools](https://www.jetbrains.com/resharper/features/command-line.html)

</td><td></td><td>.NET Framework 4.6.1 or later installed on the build agent</td></tr>

<tr><td>

[.NET Process Runner](net-process-runner.md)

</td><td></td><td>.NET installed on the build agent</td></tr>

<tr><td>

[NuGet](nuget.md)

</td><td>NuGet 1.4 or later</td><td>

Required on the build agent:
* NuGet.exe command-line tool
* on Windows: NuGet versions prior to 2.8.6 — .NET Framework 4.0 or later; NuGet 2.8.6 or later — .NET Framework 4.5
* on Linux and macOS: [Mono](https://www.mono-project.com/docs/getting-started/install/) 4.4.2 or later and NuGet CLI 3.2 or later

</td></tr>

</table>

### Other Runners

<table>

<tr><td>Runner</td><td>Supported versions</td><td>Requirements</td></tr>

<tr><td>

[Command Line](command-line.md)

</td><td></td><td></td></tr>

<tr><td>

[Python](python.md)

</td><td>2.0 or later</td><td>Python installed on the build agent</td></tr>

<tr instance="tc"><td>

[Kotlin Script](kotlin-script.md)

</td><td></td><td></td></tr>

<tr instance="tc"><td>

[Node.js](nodejs.md)

</td><td></td><td></td></tr>

<tr><td>

[Rake](rake.md)

</td><td>0.7.3 gem or later</td><td></td></tr>

<tr><td>

[PowerShell](powershell.md)

</td><td>1.0 - 5.0</td><td></td></tr>

<tr><td>

[Xcode](xcode-project.md)

</td><td>3-13</td><td>Xcode installed on the build agent</td></tr>

</table>

## Testing Frameworks

<table>

<tr><td>Framework</td><td>Supported versions</td><td>Requirements</td></tr>

<tr><td>

JUnit

</td><td>3.8.1+, 4.x, 5.x</td><td></td></tr>

<tr><td>

[NUnit](nunit-support.md)

</td><td>2.2.10, 2.4.x, 2.5.x, 2.6.x, 3.0.x</td><td></td></tr>

<tr><td>

TestNG

</td><td>5.3 or later</td><td></td></tr>

<tr><td>

MSTest and VSTest

</td><td>

8.x-12.x, 14.x, 15.x, 19.x are supported by the [.NET](net.md) runner

</td><td>A corresponding Microsoft Visual Studio edition or Visual Studio Test Agent installed on the build agent</td></tr>

<tr><td>

[MSpec](mspec.md)

</td><td></td><td>MSpec installed on the build agent</td></tr>

</table>

[Read more](testing-frameworks.md) about the support for testing frameworks in TeamCity.

## Version Control Systems

### VCS Support on Server

<table>

<tr><td>VCS</td><td>Supported versions</td><td>Requirements</td></tr>

<tr><td>

[Git](git.md)

</td><td></td><td>

For automatic `git gc` support and maintenance of Git clones, requires a Git client installed on the server

</td></tr>

<tr><td>

[Subversion](subversion.md)

</td><td>Server versions 1.4-1.9 or later</td><td></td></tr>

<tr><td>

[Perforce](perforce.md)

</td><td></td><td>

A Perforce Helix Core client installed on the server (2017.1+ versions are supported). 

</td></tr>

<tr><td>

[Azure DevOps](team-foundation-version-control.md)

</td><td>2005, 2008, 2010, 2012, 2013, 2015, 2017</td><td></td></tr>

<tr><td>

[Mercurial](mercurial.md)

</td><td></td><td>A Mercurial "hg" client v1.5.2+ installed on the server</td></tr>

<tr instance="tc"><td>

[CVS](cvs.md) (via an external plugin)

</td><td></td><td></td></tr>

<tr instance="tc"><td>

[Borland StarTeam](starteam.md)

</td><td>6 or later</td><td>A StarTeam client application installed on the server</td></tr>

</table>

Other VCSs can be supported in TeamCity via [external plugins](https://plugins.jetbrains.com/search?products=teamcity&tags=VCS%20integration).

### VCS Support on Agent

<table>

<tr><td>VCS</td><td>Supported versions</td><td>Requirements</td></tr>

<tr><td>

[Git](git.md)

</td><td>1.6.4 or later</td><td>A Git client installed on the agent</td></tr>

<tr><td>

[Subversion](subversion.md)

</td><td>1.4-1.8</td><td></td></tr>

<tr><td>

[Perforce](perforce.md)

</td><td></td><td>A Perforce client installed on the agent</td></tr>

<tr><td>

[Azure DevOps](team-foundation-version-control.md)

</td><td>2005-2015, 2017</td><td></td></tr>

<tr><td>

[Mercurial](mercurial.md)

</td><td></td><td>A Mercurial "hg" client v1.5.2+ installed on the agent</td></tr>

<tr instance="tc"><td>

[CVS](cvs.md)

</td><td></td><td></td></tr>

</table>
   
### Labeling Build Sources

* Git
* Subversion
* Perforce
* Azure DevOps
* Mercurial
* CVS
{instance="tc"}
* Borland StarTeam
{instance="tc"}
   
### Remote Run

* Git
* Mercurial
   
### Feature Branches

* Git
* Mercurial

## VCS Hosting Services

* [GitHub.com / GitHub Enterprise](integrating-teamcity-with-vcs-hosting-services.md#Integrating+TeamCity+with+GitHub)
* [GitLab.com / GitLab CE/EE](integrating-teamcity-with-vcs-hosting-services.md#Integrating+TeamCity+with+GitLab)
* [Bitbucket (Cloud, Server, Data Center)](integrating-teamcity-with-vcs-hosting-services.md#Integrating+TeamCity+with+Bitbucket)
* [Azure DevOps Services](integrating-teamcity-with-vcs-hosting-services.md#Integrating+TeamCity+with+Azure+DevOps), or formerly Visual Studio Team Services
* [JetBrains Space](integrating-teamcity-with-vcs-hosting-services.md#Integrating+TeamCity+with+JetBrains+Space)

## Cloud Platforms
{instance="tc"}

The following cloud platforms can be used to run build agents:
* [Amazon EC2](setting-up-teamcity-for-amazon-ec2.md)
* [VMWare vSphere](setting-up-teamcity-for-vmware-vsphere-and-vcenter.md)
* [Kubernetes](setting-up-teamcity-for-kubernetes.md)

Available as non-bundled plugins:
* [Windows Azure](https://plugins.jetbrains.com/plugin/9260-azure-resource-manager-cloud-support)
* [Google Cloud](https://plugins.jetbrains.com/plugin/9704-google-cloud-agents)
* [Other plugins](https://plugins.jetbrains.com/category/102-cloud-support/teamcity).

## Issue Trackers

<table>

<tr><td>Tracker</td><td>Supported versions</td></tr>

<tr><td>

[JetBrains YouTrack](youtrack.md)

</td><td>1.0 or later</td></tr>

<tr><td>

[Atlassian Jira](jira.md)

</td><td>4.4 or later (all major features also reportedly worked for version 4.2)</td></tr>

<tr><td>

[Bugzilla](bugzilla.md)

</td><td>3.0 or later</td></tr>

<tr><td>

[GitHub](github.md)

</td><td>0.9-rc-1 or later</td></tr>

<tr><td>

[Bitbucket Cloud](bitbucket-cloud.md)

</td><td></td></tr>

<tr><td>

[Azure DevOps Server](azure-board-work-items.md) (formerly Team Foundation Server — supported version 2012 or later), and Azure DevOps Services

</td><td></td></tr>

</table>
   
See also [additional requirements](integrating-teamcity-with-issue-tracker.md).

## IDE Integration

TeamCity provides productivity plugins for the following IDEs:

<table>

<tr><td>IDE</td><td>Supported versions</td><td>Requirements</td></tr>

<tr><td>

[IntelliJ Platform Plugin](intellij-platform-plugin.md)

</td><td>

Compatible with IntelliJ IDEA 2019.3 - 2021.2.3 (Ultimate and Community editions); as well as other IDEs based on the same version of the platform, including JetBrains RubyMine 6.3+, JetBrains PyCharm 3.1+, JetBrains PhpStorm/WebStorm 7.1+, AppCode 2.1+. See [more information](intellij-platform-plugin-compatibility.md) on compatibility.

</td><td></td></tr>

<tr><td>

[Microsoft Visual Studio](visual-studio-addin.md)

</td><td>2010, 2012, 2013, 2015, 2017, 2019</td><td>.NET Framework</td></tr>

</table>

### Remote Run and Pre-tested Commit

[Remote Run](remote-run.md) and [Pre-tested commit](pre-tested-delayed-commit.md) functionality is available for the following IDEs and version control systems:

<table><tr>

<td>

IDE

</td>

<td>

Supported VCS

</td></tr><tr>

<td>

[IntelliJ IDEA Platform](intellij-platform-plugin.md)

(supported only for VCS integrations bundled with JetBrains IDEs)

</td>

<td>

* Subversion
* Perforce
* Git (remote run only)
* Azure DevOps, or formerly Team Foundation Server

</td></tr><tr>

<td>

[Microsoft Visual Studio](visual-studio-addin.md)

</td>

<td>

* Subversion 1.4-1.11 (the command-line client is required); note that 1.10-1.11 is supported since ReSharper 2018.3.
* Azure DevOps (formerly Team Foundation Server — supported version 2005 or later). Installed Team Explorer is required.
* Perforce 2008.2 or later (the command-line client is required).

</td></tr></table>

### Code Coverage

<table><tr>

<td>

IDE

</td>

<td>

Supported Coverage Tools

</td></tr><tr>

<td>

[IDEA](intellij-idea.md) and [EMMA](emma.md) code coverage

</td></tr><tr>

<td>

[IntelliJ IDEA Platform](intellij-platform-plugin.md)

</td>

<td>

[IDEA](intellij-idea.md), [EMMA](emma.md), and [JaCoCo](jacoco.md) code coverage

</td></tr><tr>

<td>

[Microsoft Visual Studio](visual-studio-addin.md)

</td>

<td>

JetBrains dotCover coverage. Requires [JetBrains dotCover](https://www.jetbrains.com/dotcover/) installed in Microsoft Visual Studio.

</td></tr></table>

## Databases
{instance="tc"}

<table>

<tr>
   <td>Database</td>
   <td>Supported versions</td>
</tr>

<tr>
   <td>HSQLDB<br/><br/>The internal HSQLDB database can be used for <b>evaluation purposes only</b>.</td>
   <td>2.7.2</td>
</tr>

<tr>
   <td>MySQL</td>
   <td>5.7.34 or later</td>
</tr>

<tr>
   <td>Microsoft SQL Server</td>
   <td>2012 or later (including Express editions), SQL Azure</td>
</tr>

<tr>
   <td>PostgreSQL</td>
   <td>9.6 or later</td>
</tr>

<tr>
   <td>Oracle</td>
   <td>10g or later (tested with the [driver](https://www.oracle.com/technetwork/database/features/jdbc/index-091264.html) version 12.1.0.1</td>
</tr>

<tr>
   <td>MariaDB</td>
   <td>10.2 or later</td>
</tr>

</table>


[Read more details](set-up-external-database.md).

## Game Engines

* Unity, by the means of the [Unity Support](https://plugins.jetbrains.com/plugin/11453-unity-support) plugin (bundled in TeamCity Cloud and can be installed on-demand in TeamCity On-Premises)
