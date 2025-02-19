[//]: # (title: Reporting Issues)
[//]: # (auxiliary-id: Reporting Issues)

If you experience problems running TeamCity and believe they are related to the software, please [contact us](troubleshooting.md) with a detailed description of the issue.

To fix a problem, we may need a wide range of information about your system as well as various logs. The section below explains how to collect such information for different issues.
{instance="tc"}

In case with TeamCity Cloud, you only need to provide your server URL, and our support will be able to check the state of your server.
{instance="tcc"}

## Best Practices When Reporting Issues
{instance="tc"}

Following these guidelines will ensure timely response and effective issue resolution. Check [Feedback](troubleshooting.md) for appropriate ways to contact us. 

<note>
 
When suggesting an improvement or feature request, describe _why_ you need the feature and the __original goal__ you want to achieve. Suggestions as to _how_ you would like to see the feature implemented are also welcome.

</note>

__When submitting a request for support, include the following general information__:

* The __TeamCity version__ in use, including the build number (this can be found in the footer and the `teamcity-server.log`). It is also worth including the Operating System and Java version being used.

* Document any pattern of issue occurrence (first time, recurring, and so on), if it was mitigated in the past, whether there were any recent environment changes. __Be Specific__: note exact times, error messages, and describe both the expected and actual behavior.

* Whenever possible, include related __screenshots__ of the TeamCity UI (always include the entire page and the browser URL in the capture) and provide details on the related settings files, actual values, [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html) entity representations.

* When including an error message or other related text messages (as text, not as an image), __include the complete message__ and where it was/is being observed. If the message is more than a few lines long, attach it to the ticket as a file.

* If sending more than three files, package the files into a single archive.

* If you are sending a file larger than 10Mb, follow the guide for [Uploading Large Data Archives](#Uploading+Large+Data+Archives).

__Other helpful tips for specific issues__:

* If necessary, attach/upload an archive with TeamCity server logs (see [details](teamcity-server-logs.md), ideally, the entire `<server_home>\logs` directory with all the directories inside. If impractical, all the files updated around or later than the issue time); if related to the build time or agent behavior, attach the entire `<agent_home>\logs` directory.

* For performance/slowness/delays issues, take a set of (10+ spread across the issue time) server or agent [thread dumps](#Server+Thread+Dump) during the issue occurrence and make sure to send us the dumps with the related data.

* Consider checking if the issue exists in our [YouTrack Issue Tracker](https://youtrack.jetbrains.com/issues/TW). We identify most of the bugs and edge-conditions here. There is a chance your issue has already been fixed in a later TeamCity version.
 
* If you have an issue with your build logic, make sure it is a TeamCity-related issue (that is it does not [reproduce in the same environment](common-problems.md#Build+works+locally+but+fails+or+misbehaves+in+TeamCity) without TeamCity) and include details of your investigation.

* When reporting build procedure issues, try to [reproduce](common-problems.md#Build+works+locally+but+fails+or+misbehaves+in+TeamCity) them without TeamCity on the agent machine in the same environment and let us know the results.

* If specific builds are affected, __include the complete build logs__ (downloaded via a dedicated link from the build results' __Build Log__ tab).

* When replacing or masking sensitive data in logs, note the replacement patterns used.

* If there is an antivirus installed and/or if there is a network proxy or reverse proxy in front of the TeamCity server, include this information in your request.

* List any non-bundled plugins installed.

* Note any previous modifications of the TeamCity Data Directory or the database.

* Do not include large portions (above 10Kb) of the textual data into the email text — rather attach it in a file.

__Consider the following guidelines for posts__:

* When posting new details about the already reported issue, update previous posting on the topic instead of creating new ones. However, if you need to create a duplicate topic, make sure to note all previous postings on the same topic you have made or found.

* Post one issue per submission, the most important issue first. If you do make more than one submission, make a note of the other issues if they are related.

Check the sections below for common cases and specific information to collect and send to us.

## Slowness, Hangings and Low Performance
{instance="tc"}

If TeamCity is running slower than you would expect, please use the notes below to locate the slow process and send us all the relevant details if the process is a TeamCity one.

### Determine Which Process Is Slow

If you experience a slow TeamCity web UI response, checking for changes process, server-side sources checkout, long clean-up times or other slow server activity, your target should be the machine where the TeamCity server is installed.   
If the issue is related only to a single build, you will need to investigate the TeamCity agent machine, which is running the build, and the server. 

Investigate the system resources (CPU, memory, IO) load. If there is a high load, determine the process causing it. If it is not a TeamCity-related process, that might need addressing outside of the TeamCity scope. Also check for generic slow-down reasons like antivirus software, maintenance time, and so on.

If it is the TeamCity server that is loading the CPU/IO or there is no substantial CPU/IO load and everything runs just fine except for TeamCity, then this is to be investigated further.

Check if you have any [conflicting software](known-issues.md#Conflicting+Software) like antivirus running on the machine. Consider temporarily disabling it for test purposes, but make sure it doesn't compromise the security of your setup.

Check that the database used by TeamCity and the file storage of the TeamCity Data Directory do not have performance issues.

If you have a substantial TeamCity installation, check your [memory settings](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server) as the first step.

### Collect Data

During the slow operation, take several thread dumps of the slow process (see below for thread dump taking approaches) with 5-10 seconds interval. If the slowness continues, take several more thread dumps (for example, 3-5 within several minutes) and then repeat after some time (for example, 10 minutes) while the process is still being slow.

Then [send](troubleshooting.md) us a detailed description of the issue accompanied with the thread dumps and full server (or agent) [logs](#Logging+events) covering the issue. Unless it is undesirable for some reason, the preferred way is to file an issue into our [issue tracker](https://youtrack.jetbrains.com/issues/TW) and let us know via support email. Please include all the relevant details of investigation, including the CPU/IO load information, what specifically is slow and what is not, note affected URLs, visible effects, and so on. For large amounts of data, use [our file upload](#Uploading+Large+Data+Archives) service to share the archives with us.

### Server Thread Dump

When an operation on the server is slow, take a set of the server thread dumps (10\+) spread over the time of the slowness. TeamCity automatically saves thread dumps on super slow operations, so there might already be some saved in `logs/threadDumps-<date>` directories. It is recommended to send us an archive of the entire content of server's [`<TeamCity Home>`](teamcity-home-directory.md)`/logs/threadDumps-<date>` directories for all the recent dates.

It is recommended that you take a thread dump of the TeamCity server from the Web UI if the hanging is local and you can still open the TeamCity __Administration__ pages: go to the __Administration | Server Administration | Diagnostics__ page and click the __Save Thread Dump__ button to save a dump under the [`<TeamCity Home>`](teamcity-home-directory.md)`/logs/threadDumps-<date>` directory (where you can later download the files from "Server Logs"). If the server is fully started but the web UI is not responsive, try the [direct URL](http://YOUR_TEAMCITY_SERVER_URL/admin/diagnostic.html?actionName=threadDump&amp;save=false){nullable="true"} using the actual URL of your TeamCity server.

If the UI is not accessible (or the server is not yet fully started), you can take a server thread dump manually using the approaches described [below](#Taking+Thread+Dump).

You can also adjust the `teamcity.diagnostics.requestTime.threshold.ms=30000` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) to change the timeout after which a thread dump is automatically created in the `threadDumps-<date>` directory under TeamCity logs whenever there is a user-originated web request taking longer than timeout.

#### Taking the server thread dumps automatically

TeamCity server is able to take several thread dumps automatically with some configured interval.
For this the following [internal properties](server-startup-properties.md#TeamCity+Internal+Properties) should be added:
* `teamcity.diagnostics.periodicThreadDumps.name` - some name to use as a thread dump file name suffix
* `teamcity.diagnostics.periodicThreadDumps.count` - the number of thread dumps to save
* `teamcity.diagnostics.periodicThreadDumps.period.ms` - the time in milliseconds to wait between each thread dump

Once the properties are added, the server will start taking the thread dumps. All the thread dumps will be stored under the [`<TeamCity Home>`](teamcity-home-directory.md)`/logs/threadDumps-<date>` directory.

### Agent Thread Dump

It is recommended that you take an agent thread dump from the Web UI: go to the Agent page, __Agent Summary__ tab, and use the __Dump threads on agent__ action.

If the UI is not accessible, you can take the dump thread manually using the approaches described [below](#Taking+Thread+Dump).

> * TeamCity agent consists of two `java` processes: the launcher and agent itself. The agent is triggered by the launcher. You will usually be interested in the agent (nested) process and not the launcher one.
> * Capturing thread dumps via the `jstack` commands might require [configuring additional agent startup properties](configuring-build-agent-startup-properties.md) to deactivate the [DisableAttachMechanism](https://docs.oracle.com/cd/E15289_01/JRCLR/optionxx.htm#BABJAJBA) option. This Java option is initially enabled for security reasons and prevents external processing from attaching themselves to a JVM. It is recommended that you re-enable it after collecting required dumps.
> 
{style="note"}

### Taking Thread Dump

These can help if you are unable to take a thread dump from the TeamCity web UI.

To take a thread dump:

#### Under Windows

You have several options:
* To take a server thread dump if the server is run from the console, press __Ctrl\+Break__ (Ctrl\+Pause on some keyboards) in the console window (this will not work for an agent, since its console belongs to the launcher process). If the server is run as a service, try running it from console by logging under the same use as configured in the service and executing `<TeamCity server home>\bin\teamcity-server.bat run` command.
* Another approach is to figure out the ID of the TeamCity server process (it's the topmost "java" process with "org.apache.catalina.startup. Bootstrap start" at the end of the command line) and use of the following approaches:
  * Run `jstack <pid_of_java_process>` in the `bin` directory of the Java installation used to by the process (the Java home can be looked up in the process command line. If the installation does not have the `jstack` utility, you might need to get the Java version via the `java -version` command, download full JDK of the same version and use the `jstack` form there). You might also need to supply `-F` flag to the command.
  * Use TeamCity-bundled agent thread dump tool (can be found in the agent's plugins). Run the command:
    ```Shell
    
    <TeamCity agent>\plugins\stacktracesPlugin\bin\x86\JetBrains.TeamCity.Injector.exe <pid_of_java_process>
    ```
    
    Note that if the hanging process is run as a service, the thread dumping tool must be run from a console with elevated permissions (using Run as Administrator). If the service is run under System account, you might also need to launch the thread dumping tools via [`PsExec.exe`](http://technet.microsoft.com/en-us/sysinternals/bb897553.aspx) `-s <path to the tool>\<tool> <options>`. When the service is run under a regular user, wrapping the tool invocation in `PsExec.exe -u <user> -p <password> <path to the tool>\<tool> <options>` might also help.

If neither of these work for the server running as a service, try [running the server](start-teamcity-server.md) from console and not as a service. This way the first (Ctrl\+Break) option can be used.

#### Under Linux

* run `jstack <pid_of_java_process>` (using jstack from the Java installation as used by the process) or `kill -3 <pid_of_java_process>`. In the latter case output will appear in [`<TeamCity Home>`](teamcity-home-directory.md)`/logs/catalina.out` or [`<TeamCity agent home>`](agent-home-directory.md)`/logs/error.log`.   

See also [Server Performance](#Determine+Which+Process+Is+Slow) section above.

### Database-related Slowdowns

When the server is slow, check if the problem is caused by database operations.It is recommended to use database-specific tools.

You can also use the `debug-sql` server [logging preset](teamcity-server-logs.md#Logging-related+Diagnostics+UI). Upon enabling, all the queries which take longer 1 second will be logged into the `teamcity-sql.log` file. The time can be changed by setting the `teamcity.sqlLog.slowQuery.threshold` [internal property](server-startup-properties.md#TeamCity+Internal+Properties). The value should be set in milliseconds and is 1000 by default.

#### MySQL

Along with the server thread dump, make sure to attach the output of the `show processlist;` SQL command executed in MySQL console. Like with thread dumps, it makes sense to execute the command several times if slowness occurred and send us the output. Also, MySQL can be set up to keep a log of long queries executed with the changes in `my.ini`:


```Shell

[mysqld]
...
log-slow-queries
long_query_time=15

```

The log can also be sent to us for analysis.

## OutOfMemory Problems
{instance="tc"}

If you experience problems with TeamCity consuming too much memory or "OutOfMemoryError"/"Java heap space" errors in the log, do the following:
* Determine what process encounters the error (the actual building process, the TeamCity server, or the TeamCity agent). You can track memory and CPU usage by TeamCity with the charts on the __Administration | Server Administration | Diagnostics__ page of your TeamCity web UI.
* If the problem is on the server's side, check you have increased memory settings from the default ones for using the server in production (see the [section](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server)).
* If the problem is on the build process' side, set "JVM Command Line Parameters" settings in the build runner. Increase the value for the `-Xmx` JVM option: for instance, `-Xmx1200m`. Note that Java Inspections builds may specifically need increasing the `-Xmx` value.
* If the problem is on the TeamCity server's side and increasing the memory size does not help, please report the case for us to investigate. For this, while the server is high on memory consumption, take several server thread dumps as described [above](#Taking+Thread+Dump), get the memory dump (see below) and all the server logs including `threadDumps-*` sub-directories, archive the results, and [send them](#Uploading+Large+Data+Archives) to us for further analysis. Make sure that the `-Xmx` setting is less than 8Gb before getting the dump:
  * if a memory dump (`hprof` file) is created automatically, the `java_xxx.hprof` file is created in the process startup directory ([`<TeamCity Home>/bin`](teamcity-home-directory.md) or [`<TeamCity Agent home>/bin`](agent-home-directory.md));
  * for the server, you can also take memory dump manually when the memory usage is at its peak. Go to the __Administration | Server Administration | Diagnostics__ page of your TeamCity UI and click __Dump Memory Snapshot__.
  * another approach to take a memory dump manually is to use the `jmap` standard JVM command line utility of the full JVM installation of the same version as the Java used by the process. Example command line is:   
    
    ```Shell
    jmap -dump:file=<file_on_disk_to_save_dump_into>.hprof <pid_of_the_process>
    ```

See how to change JVM options for the [server](server-startup-properties.md#JVM+Options) and for [agents](configuring-build-agent-startup-properties.md#Agent+Properties).

## "Too many open files" Error
{instance="tc"}

1. Determine what computer it occurs on
2. Determine the process which has opened a lot of files and the files list (on Linux use `lsof`, on Windows you can use [handle](http://technet.microsoft.com/en-us/sysinternals/bb896655) or [TCPView](http://technet.microsoft.com/en-us/sysinternals/bb897437.aspx) for listing sockets)
3. If the number is under thousands, check the OS and the process limits on the file handles (on Linux use `ulimit -n`) and increase them if necessary. Note that default Linux 1024 handles per process is way too small for a server application like TeamCity. Increase the number to at least 16000. Check the actual process limits after the change as there are different settings in the OS for settings global and per-session limits (e.g. see the [post](https://stackoverflow.com/questions/13988780/too-many-open-files-ulimit-already-changed))

If the number of files is large and looks suspicious and the locking process is a TeamCity one (the TeamCity agent or server with no other web applications running), then, while the issue is still occurring, grab the list of open handles several times with several minutes interval and send the result to us for investigation together with the relevant details.

Note that you will most probably need to reboot the machine with the error after the investigation to restore normal functioning of the applications.

## Agent does not connect to the server

Please refer to [Common Problems](common-problems.md#Started+Build+Agent+is+not+available+on+the+server+to+run+builds).

## Logging events
{instance="tc"}

The TeamCity server and agent create logs that can be used to investigate issues.

<note>

Before reproducing the problem it makes sense to enable 'DEBUG' log level for TeamCity classes. On the server side, go to the __Administration | Server Administration | Diagnostics__ page and select logging preset (`debug-all`, `debug-vcs`, etc).   
After that, DEBUG messages will go to `teamcity-*.log` files ([read more](teamcity-server-logs.md)).
</note>

For detailed information, refer to the corresponding sections:

* [TeamCity Server Logs](teamcity-server-logs.md)   
* [Viewing Build Agent Logs](viewing-build-agent-logs.md)

### Version Control debug logging

<note>

To enable VCS logging on the server side, [switch logging preset](teamcity-server-logs.md) to "debug-vcs" in administration web UI and then retrieve `logs/teamcity-vcs.log` log file.
</note>

Most VCS operations occur on the TeamCity server, but if you're using the [agent-side checkout](vcs-checkout-mode.md#agent-checkout), VCS checkout occurs on the build agents.

For the agent and the server, you can change the Log4j configuration manually in [`<TeamCity Home>`](teamcity-home-directory.md)`/conf/teamcity-server-log4j.xml` or [`<BuildAgent home>`](agent-home-directory.md)`/conf/teamcity-agent-log4j.xml` files to include the following fragment:

```Shell

<category name="jetbrains.buildServer.VCS" additivity="false">
    <appender-ref ref="ROLL.VCS"/>
    <appender-ref ref="CONSOLE-ERROR"/>
    <priority value="DEBUG"/>
</category>
 
<category name="jetbrains.buildServer.buildTriggers.vcs" additivity="false">
    <appender-ref ref="ROLL.VCS"/>
    <appender-ref ref="CONSOLE-ERROR"/>
    <priority value="DEBUG"/>
</category>

```

Be sure to update the `<appender name="ROLL.VCS">` node to increase the number of the files to store:

```Shell

    <param name="maxBackupIndex" value="30"/>

```

If there are separate logging options for specific version controls, they are described below.

#### Subversion debug logging

<note>

To enable SVN logging on the server side, [switch the logging preset](teamcity-server-logs.md) to "debug\-SVN" in the administration web UI and then retrieve the `logs/teamcity-vcs.log` and `logs/teamcity-svn.log` files.
</note>

An alternative manual approach is also necessary for agent-side logging.

First, enable the generic VCS debug logging, as described [above](#Version+Control+debug+logging).

Uncomment the SVN-related parts (the `SVN.LOG` appender and `javasvn.output` category) of the Log4j configuration file on the server and on the agent (if the [agent-side checkout](vcs-checkout-mode.md#agent-checkout) is used). The log will be saved to the `logs/teamcity-svn.log` file. Generic VCS log should be also taken from `logs/teamcity-vcs.log`

#### ClearCase

Uncomment the Clearcase-related lines in the [`<TeamCity Home>`](teamcity-home-directory.md)`/conf/teamcity-server-log4j.xml` file. The log will be saved to `logs/teamcity-clearcase.log` directory.

## Patch Application Problems

In case the [server-side checkout](vcs-checkout-mode.md#server-checkout) is used, the "patch" that is passed from the server to the agent can be retrieved by:
* add the property `system.agent.save.patch=true` to the build configuration.
* trigger the build.
the build log and the agent log will contain the line "Patch is saved to file $\{file.name\}"Get the file and supply it with the problem description.

## Build triggers debug logging
{instance="tc"}

To collect all build triggers debug logs in TeamCity 2021.1 and later, switch the logging preset on the __Administration | Diagnostics__ page to `debug-triggers`, reproduce the problem and then collect all the `teamcity-triggers.log` files.

In TeamCity versions before 2021.1, it is only possible to enable debug logging for a VCS trigger defined in a specific build configuration:

1. Take a default logging preset file [`<TeamCity Home>`](teamcity-home-directory.md)`/conf/teamcity-server-log4j.xml` and save it with some other name under the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/_logging/` directory.
2. Modify the resulting file as follows:
   * add a new appender to log VCS trigger related events to a separate file: 
   ```XML
     <appender name="ROLL.VCS.TRIGGER" class="jetbrains.buildServer.util.TCRollingFileAppender">
       <param name="file" value="${teamcity_logs}/teamcity-vcs-trigger.log"/>
       <param name="maxBackupIndex" value="20"/>
       <layout class="org.apache.log4j.PatternLayout">
         <param name="ConversionPattern" value="[%d] %6p [%15.15t] - %30.30c - %\m%n"/>
        </layout>
     </appender>
    ```
    * add a new category with the name `jetbrains.buildServer.buildTriggers.vcs.AllBranchesVcsTrigger_<build configuration id>` as follows:
    ```XML
      <category name="jetbrains.buildServer.buildTriggers.vcs.AllBranchesVcsTrigger_MyBuildConfigurationId">
        <priority value="DEBUG"/>
        <appender-ref ref="ROLL.VCS.TRIGGER"/>
      </category>
   ```
   Note: in the example above, `MyBuildConfigurationId` is the ID of the build configuration with a VCS trigger which we want to debug.

3. Switch the logging preset on the __Administration | Diagnostics__ page to the newly created preset file.

No server restart is required. If the configuration is correct, TeamCity will create the `teamcity-vcs-trigger.log` log file in a few minutes. The file will contain the debug log related to the selected configuration only.

## Logging for .NET Runners

To investigate process launch issues for [.NET-related runners](supported-platforms-and-environments.md#.NET+Runners), enable debugging as described below. The detailed information will then be printed into the build log. It is recommended not to have the debug logging for a long time and revert the settings after investigation.

Aa alternative way to enable the logging is as follows:
Add the `teamcity.agent.dotnet.debug=true` [configuration parameter](configuring-build-parameters.md) in the build configuration or on the agent and run the build.
1. Open the [`<agent home>`](agent-home-directory.md)`/plugins/dotnetPlugin/bin` directory.
2. Make a backup copy of `teamcity-log4net.xml`.
3. Replace `teamcity-log4net.xml` with the content of `teamcity-log4net-debug.xml`.

<note>

After a debug log is created, it is recommended to roll back the change. The change in the `teamcity-log4net.xml` will be removed on the build agent autoupgrade.

</note>

## Remote Run Problems

The changes that are sent from the IDE to the server on a [remote run](remote-run.md) can be retrieved from the server `.BuildServer/system/changes` directory. Locate the `<change_number>.changes` file that corresponds to your change (you can pick the latest number available or deduce the URL of the change from the web UI). The file contains the patch in the binary form. Please send it with the problem description.

## Logging in IntelliJ IDEA/Platform-based IDEs

To enable debug logging for the [IntelliJ Platform-based IDE plugin](intellij-platform-plugin.md), include the following fragment into the Log4j configuration of the `<IDE home>/bin/log.xml` file.

> Since platform version 2019.3, this file is no longer loaded by default. Consider adding the configuration via __Help | Diagnostic Tools | Debug Log Settings__ instead. Alternatively, you can copy this file to some location and specify a path to it by adding `idea.log.config.file = /path/to/log.xml` in __Help | Edit Custom Properties__.
>
{style="note"}

```Shell

<appender name="TC-FILE" class="org.apache.log4j.RollingFileAppender">
  <param name="MaxFileSize" value="10Mb"/>
  <param name="MaxBackupIndex" value="10"/>
  <param name="file" value="$LOG_DIR$/idea-teamcity.log"/>
  <layout class="org.apache.log4j.PatternLayout">
    <param name="ConversionPattern" value="%d [%7r] %6p - %30.30c - %m \n"/>
  </layout>
</appender>
 
<appender name="TC-XMLRPC-FILE" class="org.apache.log4j.RollingFileAppender">
  <param name="MaxFileSize" value="10Mb"/>
  <param name="MaxBackupIndex" value="10"/>
  <param name="file" value="$LOG_DIR$/idea-teamcity-xmlrpc.log"/>
  <layout class="org.apache.log4j.PatternLayout">
    <param name="ConversionPattern" value="%d [%7r] %6p - %30.30c - %m \n"/>
  </layout>
</appender>
 
<category name="jetbrains.buildServer.XMLRPC" additivity="false">
  <priority value="DEBUG"/>
  <appender-ref ref="TC-XMLRPC-FILE"/>
</category>
 
<category name="jetbrains.buildServer" additivity="false">
  <priority value="DEBUG"/>
  <appender-ref ref="TC-FILE"/>
</category>

```

After changing this file, restart the IDE. The TeamCity plugin debug logs are saved into `idea-teamcity\*` files and will appear in the logs' directory of the [IDE settings](https://www.jetbrains.com/idea/webhelp/project-and-ide-settings.html#d1270417e197) (`<IDE settings/Data Directory>/system/log` directory).

### Open in IDE Functionality Logging

Add the following JVM option before starting IntelliJ IDEA:   
`-Dteamcity.activation.debug=true`   

The logging related to the __open in IDE__ functionality will appear in the IDE _console_.

### No Suitable Build Configurations Found for Remote Run

First of all, check that your [VCS settings in IDEA](https://www.jetbrains.com/idea/webhelp/configuring-general-vcs-settings.html) correspond to the [VCS settings](configuring-vcs-settings.md) in TeamCity. If they do not, change them and it should fix the problem.

Secondly, check that the build configurations you expect to be suitable with your IDEA project has either [server-side VCS checkout mode](vcs-checkout-mode.md#server-checkout) or [agent-side checkout](vcs-checkout-mode.md#agent-checkout) and NOT manual VCS checkout mode (it is not possible to apply a personal patch for a build with the manual checkout mode because TeamCity must apply that patch after the VCS checkout is done, but it does not know or manage the time when it is performed).

If the settings are the same and you do not use the manual checkout mode but the problem is there, do the following:
* Provide us with your IDEA VCS settings and TeamCity VCS settings (for the build configurations you expect to be suitable with your IDEA project)
* Enable debug logs for the TeamCity IntelliJ plugin (see [above](#Logging+in+IntelliJ+IDEA%2FPlatform-based+IDEs))
* Enable the TeamCity server debug logs (see [above](#Logging+events))
{instance="tc"}
* In the [TeamCity IntelliJ plugin](intellij-platform-plugin.md), try to start a remote run build
* Provide us with the debug logs from the TeamCity IntelliJ plugin and from the TeamCity server.

## TeamCity Visual Studio Add-in issues

### TeamCity Add-in logging

To capture logs from the TeamCity [Visual Studio Addin](visual-studio-addin.md):

1. Locate the Visual Studio installation directory (in the example below `c:\Program Files (x86)\Microsoft Visual Studio\2017\Common7\IDE`)

2. Run Microsoft Visual Studio executable (`INSTALLATION_DIRECTORY\Common7\IDE\devenv.exe`) from the command line with the ReSharper-related command line arguments:
   * For TeamCity VS Add-in as a part of [ReSharper Ultimate](https://www.jetbrains.com/dotnet/), use `/ReSharper.LogFile <PATH_TO_FILE>` and `/ReSharper.LogLevel <Normal|Verbose|Trace>` switches
     ```Shell 
     c:\Program Files (x86)\Microsoft Visual Studio\2017\Common7\IDE>devenv.exe /ReSharper.LogFile C:\Users\jetbrains\Desktop\vs.log /ReSharper.LogLevel Verbose
     ```
   * For the legacy version of TeamCity VS Add-in, use `/TeamCity.LogFile <PATH_TO_FILE>` and  `/TeamCity.LogLevel <Normal|Verbose|Trace>` switches

### Visual Studio logging

To troubleshoot common Visual Studio, run Microsoft Visual Studio executable `INSTALLATION_DIRECTORY\Common7\IDE\devenv.exe` with the [`/Log`](https://msdn.microsoft.com/en-us/library/ms241272.aspx) command Line switch and send us the resulting log file.

## dotCover Issues

To collect additional logs generated by [JetBrains dotCover](jetbrains-dotcover.md), add the `teamcity.agent.dotCover.log` [configuration parameter](configuring-build-parameters.md) to the build configuration. This parameter should store an absolute or relative path to an empty agent directory where you want to keep dotCover logs. If you need to publish these logs as [build artifacts](build-artifact.md), add the same directory path to the list of [artifact paths](configuring-general-settings.md#Artifact+Paths). 

<!--[//]: # (Internal note. Do not delete. "Reporting Issuesd267e1133.txt")-->

## JVM Crashes

On a rare occasion of the TeamCity server or agent process terminating unexpectedly with no apparent reason, it can happen that this is caused by a Java runtime crash.   
If this happens, the JVM regularly creates a file named `hs_err_pid*.log` in the working directory of the process. The working directory is usually [`<TeamCity server home>`](teamcity-home-directory.md)`/bin` or [`<agent home>`](agent-home-directory.md)`/bin`.   
Under Windows, when running as a service, it can be other like `C:\Windows\SysWOW64`. You can also search the disk for the recent files with "hs_err_pid" in the name. See also the related Fatal Error Log section in this [document](https://www.oracle.com/technetwork/java/javase/felog-138657.html).
{instance="tc"}

On a rare occasion of the TeamCity agent process terminating unexpectedly with no apparent reason, it can happen that this is caused by a Java runtime crash.   
If this happens, the JVM regularly creates a file named `hs_err_pid*.log` in the working directory of the process. The working directory is usually or [`agent home`](agent-home-directory.md)`>/bin`. You can also search the disk for the recent files with `hs_err_pid` in the name. See also the related Fatal Error Log section in this [document](https://www.oracle.com/technetwork/java/javase/felog-138657.html).
{instance="tcc"}

Please send this file to us for investigation and consider updating the JVM for the [server](how-to.md#Install+Non-Bundled+Version+of+Java) (or for [agents](configure-java-for-agent.md)) to the latest version available.
{instance="tc"}

Please send this file to us for investigation and consider updating the [agents'](configure-java-for-agent.md) JVM to the latest version available.
{instance="tcc"}

If you get the "There is insufficient memory for the Java Runtime Environment to continue. Native memory allocation (malloc) failed to allocate..." message with the crash or in the crash report file, make sure to [switch to 64-bit JVM](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server) or reduce the `-Xmx` setting below `1024m`, see details in the [memory configuration section](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server).
{instance="tc"}

## Build Log Issues

While investigating issues related to a build log, we might need the raw binary build log as stored by TeamCity. It can be downloaded via the Web UI from the Build Log build's tab: select "Verbose" log detail and use the "raw messages file" link in the upper right corner.

## IntelliJ IDEA Inspections

The inspections result from a TeamCity build may not match inspections from a local run in IntelliJ IDEA.

To help us investigate issues with inspections, do the following:
1. Add `system.teamcity.dont.delete.temp.result.dir=true` to the [configuration parameters](configuring-build-parameters.md)
2. Add `%\system.teamcity.build.tempDir%/inspection*result/** => inspections-reports-data-%\build.number%.zip` rule to [Artifact paths](configuring-general-settings.md#Artifact+Paths)
3. Add `%\system.teamcity.build.tempDir%/idea-logs/** => inspections-reports-idea-logs-%\build.number%.zip` rule to [Artifact paths](configuring-general-settings.md#Artifact+Paths)
4. Add `-Didea.log.path=%\system.teamcity.build.tempDir%/idea-logs/` to the runner's [JVM command line parameters](inspections.md#Java+Parameters) field.
5. Run a new build.
6. Send us `inspections-reports-*.zip` files.

## Uploading Large Data Archives

Files under 10 MB in size can be attached right into the [tracker issue](https://youtrack.jetbrains.com/issues/TW) (if you do not want the attachments to be publicly accessible, limit the attachment visibility to "teamcity-developers" user group only).  
You can also send small files (up to 2 MB) via email: [teamcity-support@jetbrains.com](mailto:teamcity-support@jetbrains.com) or via [online form](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?ticket_form_id=66621){nullable="true"} (up to 20 MB). Please do not forget to mention your TeamCity version and environment and archive the files before attaching.

<!--[//]: # (Internal note. Do not delete. "Reporting Issuesd267e1305.txt")-->    

Large files can be uploaded via [`https://uploads.jetbrains.com/`](https://uploads.jetbrains.com/). Please let us know the exact filename after the upload.

If you cannot upload a large file in one go, try splitting the file into parts and upload them separately.
