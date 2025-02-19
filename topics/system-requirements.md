[//]: # (title: System Requirements)
[//]: # (auxiliary-id: System Requirements)

This article contains general recommendations on choosing and configuring the environment for TeamCity Server and Agents, as well as the network connection between them and a dedicated external database. If you have specific questions that are not covered here, please contact our support via any convenient [feedback channel](troubleshooting.md).
{instance="tc"}

This article contains general recommendations on choosing and configuring the environment for TeamCity Agents. If you have specific questions that are not covered here, please contact our support via any convenient [feedback channel](troubleshooting.md).
{instance="tcc"}

## TeamCity Server Requirements
{instance="tc"}

### Choosing Server OS/Platform

TeamCity Server can run on any recent version of Windows, Linux, or macOS. Requirements for the server's operating system are listed [here](supported-platforms-and-environments.md#TeamCity+Server).

We also recommend that you review the [requirements](supported-platforms-and-environments.md) for the integrations you plan to use. For example, the following functionalities may require or work better when TeamCity Server is installed under Windows:
* VCS integration with Azure DevOps (TFS)
* VCS integration with VSS
* Windows domain logins (can work under Linux, but might be less stable), especially NTLM HTTP authentication
* NuGet feed on the server (can work under Linux, but might be less stable)
* [Agent push](install-teamcity-agent.md#Install+via+Agent+Push) to Windows machines

If you have no specific preferences, Linux platforms may be more preferable in general. They are more effective in terms of file system operations and maintenance. The final decision on OS should depend on your company's available resources and established practices.

### Estimating Hardware Requirements for Server

The server's hardware requirements depend on the server load, which in its turn significantly depends on the type of your builds and server usage patterns. This section contains notes on various hardware-related aspects.

#### CPU

TeamCity Server can utilize multiple CPU cores. It makes sense to use at least 4 CPU cores for production purposes.

#### Memory

TeamCity usually launches multiple Java processes. The main process is responsible for displaying the UI and performing most of the background operations. It can launch other processes required for specific tasks, like compiling and executing Kotlin DSL. When defining the `-Xmx` memory value for the main process, remember to leave enough memory for other potential processes. See [notes on configuring the main process memory](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server).

For estimation, 16 GB of memory is usually enough to run up to 100 concurrent builds (agents), support up to 200 online users, and work with medium-sized repositories. If your server is considerably bigger, we suggest that you scale the memory amount respectively.

TeamCity will scale to utilize whatever amount of memory there is: for example, the JetBrains build server consists of two nodes and uses 80 GB of memory in total to support 2000 build agents and many committers.

#### Disk

The performance of a TeamCity server highly depends on the disk performance. As TeamCity stores large amounts of data under [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/system` (most notably, VCS caches and build results), it is important to ensure that the access to the disk is fast (in particular, reading/writing files in multiple threads and listing files with attributes).

If you plan to store the Data Directory on a network drive, you need to ensure that it has good performance. However, we highly recommend using local storage for the `<TeamCity Data Directory>/system/caches` directory. See also: notes on [choosing the Data Directory location](teamcity-data-directory.md#Recommendations+as+to+choosing+Data+Directory+Location).

The requirements for free disk space are mainly determined by the number of builds stored on the server and the artifacts / build log size in each of them. When processing Git or Mercurial projects, TeamCity creates mirrors of repositories and puts them under the `system/caches` directory. If the server-side checkout is used by your build configurations, you need to allocate an extra space under `system/caches` to store the repositories’ sources. As a result, the required disk space can be twice as large as the size of the repositories used by all configured VCS roots.

If builds generate large amounts of data (artifacts/build logs/test data), we suggest that you use external storage with fast access for storing the `.BuildServer/system` directory.

See also the example configurations [below](#Example+Server+Configurations).

#### Network

To support a loaded server with many agents, a fast network connection is required.

Network traffic is mostly utilized by:
* Agents:
  * Downloading updates, tools, artifact dependencies, and source code (in case of server side checkout) from the server.
  * Sending build log messages back to the server and uploading artifacts.
* Server:
  * Fetching new commits from VCS repositories and checking their status.
  * Communicating with cloud providers.
  * Communicating with an external database.
* Browsers using the server web interface.
* Custom scripts/dashboards using the server REST API to fetch data.

### Server Load Factors

The load on the server depends on:
* number of build configurations
* number of builds in the history
* number of builds running daily
* amount of data consumed and produced by the builds (size of the used sources and artifacts, size of the build log, number and output size of unit tests, number of inspections and duplicates' hits, size and number of produced artifacts, and so on)
* clean-up rules configured
* number of agents and their utilization
* number of users having TeamCity web pages open
* number of users logged in from IDE plugin
* number and type of VCS roots as well as the configured interval of checking for changes. VCS checkout mode is relevant too: the server checkout mode generates a greater server load. Specific types of VCS also affect server load, but they can be roughly estimated based on the native VCS client performance.
* number of changes detected by TeamCity per day in all the VCS roots
* size of the repositories TeamCity works with
* total size of the sources checked out by TeamCity daily

### Example Server Configurations

The following hardware configuration is capable of handling up to 100 concurrently running builds. It is implied that this machine hosts only TeamCity server — not agents or database.
* Server-suitable modern multicore CPU (8 or more)
* 16 GB of memory
* Fast network connection
* Fast and reliable HDD
* Fast external database access

__Case 1__

Based on our experience, hardware like _Intel 3.2 GHz dual-core CPU, 8 GB memory under Windows, 1 GB network adapter, and single HDD_ can provide acceptable performance for the following setup:
* 60 projects and 300 build configurations (with one forth being active and running regularly)
* more than 300 builds a day
* about 2 MB log per build
* 50 build agents
* 50 web users and 30 IDE users
* 100 VCS roots (mainly Perforce and Subversion using server checkout), average checking for changes interval is 120 seconds
* more than 150 changes per day
* Kotlin DSL is not used
* the database (MySQL) is running on the same machine
* TeamCity server process has `-Xmx1100m` JVM setting

__Case 2__

The following configuration can provide acceptable performance for a more loaded TeamCity server: _Intel Xeon E5520 2.2 GHz CPU (4 cores, 8 threads), 16 GB memory under Windows Server 2008 R2 x64, 1 GB network adapter, 3 HDD RAID1 disks (one general, one for artifacts, logs, and caches, and one for the database storage)_. Tested for the following setup:
* 150 projects and 1500 build configurations (with one third being active and running regularly)
* more than 1500 builds a day
* about 4 MB log per build
* 100 build agents
* 150 web users and 40 IDE users
* 250 VCS roots (mainly Git, Hg, Perforce, and Subversion using agent-side checkout), average checking for changes interval is 180 seconds
* more than 1000 changes per day
* the database (MySQL) is running on the same machine
* TeamCity server process has `-Xmx3700m` x64 JVM setting

However, to ensure the peak load can be handled well, more powerful hardware is recommended.

__If you install TeamCity on a virtual machine__, make sure that you run daily [clean-ups](teamcity-data-clean-up.md) to remove obsolete build logs and artifacts and save disk space. Based on our rough estimations, the **Case 1** installation might require 7 GB of disk space and **Case 2** — 15 GB, given that the space is utilized optimally and all peak loads are taken into consideration.

The general recommendation for deploying a large-scale TeamCity installation is to start with reasonable hardware while considering potential hardware upgrades. You can increase the load on the server gradually (for example, add more projects) while monitoring the performance stats, and then decide on necessary hardware or software improvements. There is also a [benchmark plugin](https://plugins.jetbrains.com/plugin/9127-benchmark) that can be used to estimate the number of simultaneous builds the current server installation can handle.

We also recommend that you stick to the best practices of server administration, like keeping disk defragmentation on a reasonable level.

If you need to increase the number of concurrently running builds (agents) by some factor, be prepared to increase the CPU, memory, database, and HDD access speeds by the same factor to achieve the same performance. If you increase the number of builds per day, be prepared to increase the disk size respectively.

### Scaling Server Depending on Agents Number

One instance of TeamCity Server can stably work with 1000+ build agents (1000 concurrently running builds that are actively logging build runtime data).

The server load produced by each build depends on the amount of this build's data (logs, tests, and failure details, inspections/duplicates issues number, and so on). Keeping the amount of data reasonably constrained (publishing large outputs as build artifacts, not printing those into standard output, tweaking inspection profiles to report a limited set of the most important inspection hits, and so on) will help scale the server to handle more concurrent builds.

**If you need many more agents/parallel builds, we highly recommend using a [multinode setup](multinode-setup.md) and distribute the projects between the nodes**.

We are constantly improving the TeamCity performance and work closely with organizations running large TeamCity installations. This way, we can study performance issues and improve TeamCity to handle larger loads.
{style="note"}

### Configuring TeamCity Server for Performance

This section contains a checklist of recommendations on tweaking the TeamCity server setup for better performance. It assumes that the server is already [configured for production use](configure-server-installation.md#Configuring+Server+for+Production+Use).

* Regularly review [Server Health](server-health.md) reports (including hidden ones).
* Use a separate [reverse proxy server](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server) (for example, NGINX) to handle HTTPS.
{instance="tc"}
* Use a separate server for the external database. Monitor the database performance.
* Monitor the server's CPU and I/O performance. Increase hardware resources as necessary.
* Make sure [clean-up](teamcity-data-clean-up.md) is configured for all the projects with a due retention policy. Check __Administration | Clean-Up__ to make sure that the clean-up is performed regularly.
* Consider ensuring good I/O performance for the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/system/caches` directory: for example, move it to a separate local drive.
* Regularly [archive](archiving-projects.md) obsolete projects.
* Regularly review the installed not bundled plugins and remove those not essential for the server operation.
* Consider using [agent-side checkout](vcs-checkout-mode.md) whenever possible.
* Make sure the build logs occupy a reasonable amount of space (tens of megabytes at most, but better less than 10 MB).
* If lots of VCS roots are configured on the server, consider configuring [repository commit hooks](configuring-vcs-post-commit-hooks-for-teamcity.md) instead of using polling for changes; or, increase [VCS polling interval](configuring-vcs-roots.md#Common+VCS+Root+Properties) to 300 seconds or more.
* If the server is used by more than 1000 users, consider reducing the frequency of background UI requests by increasing the [UI refresh intervals](teamcity-tweaks.md#Web+Page+Refresh+Interval).
* When regularly exceeding 500 concurrently running builds which log a lot of data, consider switching to a [multinode setup](multinode-setup.md).
* If you have a lot of projects or build configurations, we recommend you avoid using the __Default agent__ in order to free up the TeamCity server resources. The TeamCity Administrator can [disable](build-agents-configuration-and-maintenance.md#Enabling%2FDisabling+Agents+via+UI) the default agent on the __Agents__ page of the web UI.

## TeamCity Agent Requirements

This section lists requirements for the environment and OS user suitable for running a TeamCity build agent process.

### Choosing Agent OS/Platform

TeamCity Agent can run on any recent version of Windows, Linux, or macOS. Requirements for the agent's operating system are listed [here](supported-platforms-and-environments.md#TeamCity+Agent).

### Common Requirements

The agent Java process has to:
* be able to open outbound HTTP connections to the server URL that is configured via the `serverUrl` property in the [`buildAgent.properties`](configure-agent-installation.md) file (typically the same address you use in the browser to view the TeamCity UI).  
Sending requests to the paths under the configured URL should not be limited. See also the recommended [reverse proxy settings](how-to.md#Set+Up+TeamCity+behind+a+Proxy+Server). Ensure that any firewalls installed on the agent or server machines, network configuration, and proxies (if any) comply with these requirements.
{instance="tc"}
* have full permissions (read/write/delete) to the following directories recursively: `<Agent Home Directory>` (necessary for automatic agent upgrade and agent tools support), `<Agent Work Directory>`, `<Agent Temp Directory>`, and agent system directory (set by `workDir`, `tempDir`, and `systemDir` parameters in the `buildAgent.properties` file).
* be able to launch processes (to run builds).
* be able to launch nested processes with the following parent process exit (used during the agent upgrade).

### Requirements for Windows-based Agents

The Windows user who runs an agent process must:
* be able to start/stop service (to run as Windows service, necessary for the agent upgrade to work, see also [this article](https://support.microsoft.com/en-us/help/325349/how-to-grant-users-rights-to-manage-services-in-windows-server-2003)).
* be able to debug programs (required for taking process dump functionality).
* be able to reboot the machine (required for agent reboot functionality).
* be a member of the _Performance Monitor Users_ group (to be able to [monitor the performance](performance-monitor.md) of a build agent running as a [Windows service](start-teamcity-agent.md#Build+Agent+as+Windows+Service)).
* be able to run as a Windows service, only if the agent runs as a service (see also [this article](https://support.microsoft.com/en-us/help/325349/how-to-grant-users-rights-to-manage-services-in-windows-server-2003)).

Note on granting rights:

To learn how to grant users necessary rights, see [this article](https://support.microsoft.com/en-us/kb/325349). You can assign rights to manage services with the Microsoft [SubInACL](https://social.technet.microsoft.com/wiki/contents/articles/51625.subinacl-a-complete-solution-to-configure-security-permission.aspx#Scenario_5_Set_Windows_Service_Permission) utility, a command-line tool enabling administrators to directly edit security information. The tool uses the following syntax:

```Shell
SUBINACL /SERVICE \\MachineName\ServiceName /GRANT=[DomainName]UserName[=Access]

```

For example, to grant start/stop rights, you might need to execute the command:

```Shell
subinacl.exe /service TCBuildAgent /grant=<user login name>=PTO
```

### Requirements for Linux-based Agents

The Linux user who runs an agent process must be able to run the `shutdown` command (for the agent machine reboot and shutdown functionality when running in a cloud environment). If you are using `systemd`, it should not kill the processes on the main process exit (use [`RemainAfterExit=yes`](https://serverfault.com/questions/660063/teamcity-build-agent-gets-killed-by-systemd-when-upgrading)). See also: [how to set up an automatic agent start under Linux](start-teamcity-agent.md#Automatic+Agent+Start+Under+Linux).

### Build-related Permissions

Every build process is launched by a TeamCity agent. It shares the environment and is executed under the OS user used by the TeamCity agent. Ensure that the TeamCity agent is configured accordingly. See [Known Issues](known-issues.md#Windows+service+limitations) for related Windows Service Limitations.

### Estimating Hardware Requirements for Agent

The agents’ hardware requirements are determined by the builds they run. Running a TeamCity agent software introduces a requirement for additional CPU time (but it can usually be neglected compared to the build process CPU requirements) and additional memory: about 500 MB. The disk space required corresponds to the disk usage by the builds running on the agent (sources checkout, downloaded artifacts, the disk space consumed during the build; all that combined for the regularly occurring builds).

Although you can run a build agent on the same machine as the TeamCity server, the recommended approach is to use a separate machine (it can be virtual) for each build agent. If you chose to install several agents on the [same machine](install-multiple-agents-on-one-machine.md), consider the potential CPU, disk, memory, or network bottlenecks that might occur. The [Performance Monitor](performance-monitor.md) build feature can help analyze live data.

If you consider cloud deployment for TeamCity agents (for example, on Amazon EC2), also see [this article](setting-up-teamcity-for-amazon-ec2.md).
{instance="tc"}

## Estimating Network Traffic Between Server and Agents
{instance="tc"}

The network traffic mostly depends on your settings as some of them imply transferring binaries between the agent and the server.

The most important flows of traffic between a TeamCity agent and the TeamCity server are:
* The agent retrieves commands from the server: these are typically build start tasks, including a dump of the build configuration settings and the full set of build parameters. These parameters can be reviewed on the build's [Parameters](build-results-page.md#Parameters+Tab) tab.
* The agent periodically sends current status data to the server (this includes all the agents’ parameters which can be reviewed on the agent's [Parameters](viewing-build-agent-details.md#Parameters) tab).
* During the build, the agent sends build log messages and parameters data back to the server. These can be reviewed on the [Build Log](build-results-page.md#Build+Log+Tab) and [Parameters](build-results-page.md#Parameters+Tab) tabs of the build.
* (when the server-side checkout mode is used) The agent downloads the sources before the build (as a full or incremental patch) from the server.
* (when an [artifact dependency](artifact-dependencies.md) is configured) The agent downloads build artifacts of other builds from the server before starting a build, unless an external artifact storage is used instead.
* (when artifacts are configured for a build) The agent uploads build artifacts to the server, unless an external artifact storage is used instead.
* Some runners (like coverage or code analysis) include automatic uploading of their results' reports to the server.

## Estimating External Database Capacity
{instance="tc"}

If a TeamCity server is used extensively, the database performance starts to play a greater role. To ensure production-level performance and reliability, an [external database](set-up-external-database.md) must be used. Its size and performance are crucial aspects to consider.

If you decide to run an external database on the same machine as the server (not recommended), choose the server's hardware with the database engine requirements in mind.

It is hard to estimate exact requirements when setting up or migrating to an external database, as the required capacity greatly depends on how TeamCity is used. The database size requirements naturally vary based on the amount of stored data (number of builds, number of tests, and so on). The active database usage can be estimated at several gigabytes of data per year.

### Database Size

The size of the database depends on:
* how many builds are started daily
* how many tests are reported from builds
* [clean-up](teamcity-data-clean-up.md) rules (retention policy) and schedule

The recommended initial size is 4 GB. When migrating from the internal database, we suggest at least doubling the size of the current internal database. For example, the size of the external database (without the redo log files) of the internal TeamCity server in JetBrains is about 1 TB. Setting your database to grow automatically helps increase file sizes to a predetermined limit when necessary, which minimizes the effort to monitor the disk space.

Allocating 1 GB for the redo log (see the table below) and undo files is sufficient in most cases.

### Database Performance

The following factors can affect the database performance:
* type of database (RDBMS)
* number of agents (number of builds running in parallel)
* number of web pages opened by all users
* [clean-up](teamcity-data-clean-up.md) rules (retention policy)

It is advised to place the [TeamCity Data Directory](teamcity-data-directory.md) and database data files on physically different hard disks (even when both the TeamCity server and RDBMS share the same host).

Placing redo logs on a separate physical disk is also recommended — especially if there are 50 or more agents.

### Database-Specific Notes

The redo log (or a similar entity) naming for different RDBMS:
* RDBMS: Log name
* Oracle: Redo Log
* MS SQL Server: Transaction Log
* PostgreSQL: WAL (write-ahead log)
* MySQL + InnoDB and Percona: Redo Log

PostgreSQL: it is recommended to use version 9.2+, which has a lot of query optimization features. See the information on the write-ahead-log (WAL) in the [PostgreSQL documentation](https://www.postgresql.org/docs/9.2/static/wal-internals.html).

Oracle: it is recommended to keep statistics on — all automatically gathered statistics should be enabled (since Oracle 10.0, this is the default setup). See the information on redo log files in the [Oracle documentation](https://docs.oracle.com/cd/B14117_01/server.101/b10752/iodesign.htm#26022).

MS SQL Server: it is NOT recommended using the jTDS driver — it does not work with `nchar/nvarchar`. To preserve Unicode streams, it may cause queries to take a long time and consume many I/O operations. See the information on redo log in the [Microsoft Knowledge base](https://support.microsoft.com/kb/2033523). If you use jTDS, [consider migrating](set-up-external-database.md#jTDS+Driver).

MySQL: the query optimizer might be inefficient: some queries may get a wrong execution plan causing them to take a long time and consume many I/O operations.

