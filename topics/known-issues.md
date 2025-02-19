[//]: # (title: Known Issues)
[//]: # (auxiliary-id: Known Issues)

This page contains a list of workarounds for known issues in TeamCity.

To see issues specific to particular versions of TeamCity, go to the respective section of [upgrade notes](upgrade-notes.md).
{instance="tc"}

<anchor name="jdk8_240"/>

## Incompatibility with JDK 8 update 242+
{instance="tc"}

TeamCity versions up to 2019.2.1 running under JDK 8u242+ can report `java.lang.NoClassDefFoundError: Could not initialize class XXX` errors, for example, on Git operations or Windows domain authentication operations.

Until TeamCity 2019.2.2 is released, it is recommended to use Java 8u232 version for TeamCity server.

## Agent running as Windows Service Limitations

When a TeamCity build agent is installed as a Windows service, there may appear various "Permission denied" or "Access denied" errors during the build process, see details below.

### Windows service limitations

As a Windows service, the TeamCity agent and the build processes are not able to access network shares and mapped drives.

To overcome these restrictions, run TeamCity agent [via console](start-teamcity-agent.md).

#### Issues with automated GUI and browser testing

These problems include errors running tests headless, issues with the interaction of the TeamCity agent with the Windows desktop, and so on.

<tip>

Note that there is a Windows limitation to accessing a remote computer via mstsc: the desktop of the remote machine will be locked on RDP disconnect, which will cause issues running tests. The VNC protocol allows you to remote control another machine without locking it.To run GUI tests and be able to use RDP, see the [workaround](#Running+automated+GUI+tests+and+using+RDP) below.

</tip>

To resolve / avoid these:
1. Run TeamCity agent [via console](start-teamcity-agent.md).
2. Configure the build agent machine not to launch a screensaver locking the desktop.
3. Configure the TeamCity agent to start [automatically](start-teamcity-agent.md) (for example, configure an automatic user logon on Windows start and then configure the TeamCity agent start (via `agent.bat start`) on the user logon).
    
For graphical tests the build agent cannot be started as a service and it is recommended to configure the build agent launch with a 1 minute delay after the user auto-logon, e.g. using the `bin\agent.bat start` command in the task scheduler and configuring the delay there.

#### Running automated GUI tests and using RDP 

RDP uses its own video driver overriding the one from the machine's video card for the session. Redirecting the session to console will unload the Windows graphical drivers. This can be done  by adding the following step to your build configuration prior to your tests (the example below is for PowerShell, but other languages (DOS, Python) can be used too):


```Shell
$sessionInfo=((quser $env:USERNAME | select -Skip 1) -split '\s+')
if ($sessionInfo[1] -like "rdp-tcp*") { tscon $sessionInfo[2] /dest:console }
```



where `quser [current username]` lists all the connections to that machine for the user, either console or graphical.  
The one listed as `rdp-tcp#*` is the remote desktop connection which can be redirected to the console using `tscon [connection id] /dest:console`.

 

<note>

An unsupervised computer with a running desktop permanently logged into a user session might be considered a network security threat, as access to it can be difficult to trace. Therefore, it is recommended to run automated GUI and browser tests on a virtual machine isolated from sensitive corporate network resources, e.g. on a machine not included in a Windows domain.

</note>

#### Issues with .NET Selenium 

When a TeamCity agent is started as a Windows service and automated tests for .NET applications use Selenium WebDriver, the tests may fail due to browser drivers limitations. As a solution, consider starting the agent [manually](start-teamcity-agent.md).


### Early start of the service before other resources are initialized

To handle this, consider using the __Automatic (Delayed Start)__ option of the service settings or configure [service dependencies](https://youtrack.jetbrains.com/issue/TW-32987#comment=27-608269).

 For more investigation steps, see the [Common Problems](common-problems.md#Build+works+locally+but+fails+or+misbehaves+in+TeamCity) page.


## java.lang.OutOfMemoryError: unable to create new native thread error

If your TeamCity server is running on SUSE® Linux Enterprise (or using systemd Daemon), the __java.lang.OutOfMemoryError: unable to create new native thread error__ may be caused by the [cgroup process number controller](https://www.kernel.org/doc/Documentation/cgroup-v1/pids.txt) feature limiting the number of processes and the amount of threads in a cgroup to 512 by default.

Increasing the limit (for example, to 4096) on the TeamCity server should solve the issue.

See also this [external posting](https://www.elastic.co/blog/we-are-out-of-memory-systemd-process-limits).

## Clearing Browser Caches

There is a web UI-related issue which some users have encountered (and it cannot be reproduced on other computers) which is tied to the cached versions of content. If you have come across such problem, make sure your browser does not use cached versions of content by [clearing browser caches](https://en.wikipedia.org/wiki/Wikipedia:Bypass_your_cache).

## Logging with Log4j in Your Tests

If you use Log4j logging in your tests, in some cases you may miss the Log4j output from your logs. In such cases do the following:
* Use Log4j 1.2.12
* For Log4j 1.2.13\+, add the `Follow=true` parameter for the console appender used in Log4j configuration:

    ```Shell
    <appender name="CONSOLE" class="org.apache.log4j.ConsoleAppender">
      <param name="Follow" value="true"/>
    </appender>
    ```

## Agent Service Can Exit on User Logout under Windows x64

The used version of [Java Service Wrapper](https://wrapper.tanukisoftware.com/) does not fully support Windows 64 and this causes agent launcher process to be killed on user logout. The agent itself will be function until the next restart (server upgrade or agent properties change).


## Failed Build Can be Reported as a Successful One With Maven 2.0.7

This is a known bug in this version of Maven. Consider using any later version.  
In case it's not possible, you can patch `mvn.bat` yourself by replacing the fragment at line 148 of `mvn.bat`:


```Shell
:error
set ERROR_CODE=1

```

with the following one:

```Shell
:error
if "%\OS%"=="Windows_NT" @endlocal
set ERROR_CODE=1

```

## Conflicting Software

Most common indicators of conflicting software are errors like "Access is denied", "Permission denied" or _java.io.FileNotFoundException_ mentioning the file that is present and is writable by the user the agent/build runs under. Also, certain software running in the background (like antivirus) can significantly slow down such build agent operations as sources checkout, artifact publishing or even build running.

Certain antivirus software like Kaspersky Internet Security can result in Java process crashes or other misbehavior like inability to access files. For example, see [this issue](https://youtrack.jetbrains.com/issue/TW-7138).

ESET antivirus can also slow down Ant/IntelliJ IDEA project builds a great deal (slowing down TCP connections to localhost on an agent).

If you run antivirus on the TeamCity server or agent machines and get disk access errors or experience degraded performance, you may consider temporarily disabling the antivirus software before investigating the issue and reporting it to JetBrains. Note that disabling the antivirus can make your setup more vulnerable to potential external attacks — ensure you take proper security measures before doing this.
{instance="tc"}

If you run antivirus on the TeamCity agent machines and get disk access errors or experience degraded performance, you may consider temporarily disabling the antivirus software before investigating the issue and reporting it to JetBrains. Note that disabling the antivirus can make your setup more vulnerable to potential external attacks — ensure you take proper security measures before doing this.
{instance="tcc"}

It is recommended to exclude entire TeamCity server home and [TeamCity Data Directory](teamcity-data-directory.md) from the background checks and perform periodical checks there in the well-known maintenance window so that those do not affect server performance much. On TeamCity agent, it is recommended to exclude TeamCity agent home from the background checks.
{instance="tc"}

There might be problems with the Windows Indexing Service, so disable various indexing services. See [the related issue](https://youtrack.jetbrains.com/issue/TW-10033#comment=27-82484) for more details. Windows System Restore Feature might also need disabling.

Do not install software with background indexing like WinCVS, TortoiseCVS, TortoiseSVN, and other Tortoise\* products. This applies to server and also to agents if you use agent-side checkout.

Skype software is known to corrupt layout of pages displayed in Internet Explorer. ([TW-13052](https://youtrack.jetbrains.com/issue/TW-13052)).

## Subversion issues

### svn: E175002: Received fatal alert: bad_record_mac
{instance="tc"}

Add a system property `-Dsvnkit.http.sslProtocols=SSLv3,TLS` on the build server (see [Configuring TeamCity Server Startup Properties](server-startup-properties.md)).   
If you use checkout on agent, add this property [on build agent](configuring-build-agent-startup-properties.md) as well.

### Subversion-related JVM Crashes

If JVM crashes while executing SVN-related code (e.g. under org.tmatesoft.svn package), you can try to disable it using one of the options:
* Passing `-Dsvnkit.useJNA=false` JVM option to the crashing process (server or agent)
* Making NTLM support less prioritative by passing the `-Dsvnkit.http.methods=Basic,Digest,NTLM` JVM option.  
Anyway, upgrading the JVM used to the [latest available version](https://java.sun.com/javase/downloads) is recommended.


## NUnit 2.4.6 Performance
{instance="tc"}

Due to an issue in NUnit 2.4.6, its performance may be slower than NUnit 2.4.1. For additional information, refer to the corresponding issue in our issue tracker: [TW-4709](https://youtrack.jetbrains.com/issue/TW-4709)

## StarTeam Performance
{instance="tc"}

Using StarTeam SDK 9.0 instead of StarTeam SDK 9.3 on the TeamCity server can significantly improve VCS performance when there is a slow connection between TeamCity and StarTeam servers.

## Perforce 2009.2 Performance on Windows

If you run Perforce 2009.2 on Windows you may experience significant slow down. This is an issue with P4 server running on Windows. Refer to corresponding [section](https://www.perforce.com/manuals/p4sag/Content/P4SAG/chapter.performance.html) in Perforce documentation.

## Wrong times for build scheduled triggering (Timezone issues)
{instance="tc"}

Make sure you use the latest update for Java 8 installation available for your platform (e.g. OpenJDK 8 by [Amazon Corretto](https://aws.amazon.com/corretto/)).


## Upgrading IntelliJ IDEA May Affect Active Pre-Tested Commits

Before you upgrade to IntelliJ IDEA X (or other IntelliJ X platform products), make sure you do not have active pre-tested commits, otherwise they will not be able to be committed after upgrade. This is only relevant if you use directory-based IDEA project (project files are stored under `.idea` directory).

## Other Java Applications Running on the Same Server
{instance="tc"}

If other web applications are available via the same hostname, a session cookie conflict can occur. This usually is visible via random user logouts or losing session-level data. (e.g. [TW-12654](https://youtrack.jetbrains.com/issue/TW-12654)). To resolve this, you can use different host names when accessing the applications.

## The Server Does Not Start Claiming the Database is in Use
{instance="tc"}

Only a single TeamCity server can work with one database, which is checked on the TeamCity server start.
 
"`The Database is in Use`" error on the start\-up is reported in either of the following cases:
* An attempt to start more than one TeamCity server connected to the same database
* A second TeamCity instance detected
* The internal HSQL database is being used by another application

The error is most probably caused by the fact that there is another running TeamCity installation which is connected to the same database. Сheck that the [database properties](set-up-external-database.md) are correct and there is no other TeamCity server using the same database.

## Slow download from TeamCity server
{instance="tc"}

If you experience slow speed when downloading artifacts from TeamCity, try checking the speed on the server machine, downloading from localhost. 
If the speed is OK for the localhost, the issue can be in the network configuration or OS/hardware settings when combined with TeamCity (Tomcat) settings.

Make sure \<[TeamCity Home](teamcity-home-directory.md)\>\conf\server.xml file corresponds to the file included in TeamCity distribution (can be checked in .tar.gz distribution). 
If you have the following "Connector" node (ports numbers can be different):


```Shell
<Connector port="8111" protocol="HTTP/1.1"
               connectionTimeout="60000"
               redirectPort="8543"
               useBodyEncodingForURI="true"
    />
```



change it to:


```Shell
<Connector port="8111" protocol="org.apache.coyote.http11.Http11NioProtocol"
                connectionTimeout="60000"
                redirectPort="8543"
                useBodyEncodingForURI="true"
                socket.txBufSize="64000"
                socket.rxBufSize="64000"
                tcpNoDelay="1"
                                />
```



and restart TeamCity server.

If this does not help with the download speed, to investigate the case, you might need to find an administrator with appropriate network\-related issues investigation skills to look into the case.

## IIS-Related Issues

### Failure to publish artifacts to server behind IIS reverse proxy

This problem is only relevant to configurations that involve an IIS reverse proxy between the build server and agents. Sometimes a build agent can be found in infinite loop trying to publish build artifacts to the server. The build log looks like this:


```Shell
[11:15:05]Publishing artifacts
[11:15:05][Publishing artifacts] Collecting files to publish: [toZip/** => artifact.zip]
[11:15:06][Publishing artifacts] Creating archive artifact.zip (9s)
[11:15:06][Creating archive artifact.zip] Creating C:\BuildAgent\temp\buildTmp\ZipPreprocessor2847146024236637664\artifact.zip
[11:15:15][Creating archive artifact.zip] Archive was created, file size 32142324 bytes
[11:15:15][Publishing artifacts] Sending toZip/**
[11:15:25][Publishing artifacts] Sending toZip/**
[11:15:39][Publishing artifacts] Sending toZip/**
[11:15:48][Publishing artifacts] Sending toZip/**
[11:16:01][Publishing artifacts] Sending toZip/**
[11:16:16][Publishing artifacts] Sending toZip/**

```
meanwhile, `teamcity-agent.log` is filled with 404 responses from IIS:

```Shell
[2012-08-01 12:04:55,514]   WARN -    jetbrains.buildServer.AGENT - <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1"/>
<title>404 - File or directory not found.</title>
<style type="text/css">
<!--
body{margin:0;font-size:.7em;font-family:Verdana, Arial, Helvetica, sans-serif;background:#EEEEEE;}
fieldset{padding:0 15px 10px 15px;}

```

The most common cause for this is `maxAllowedContentLength` setting (in IIS) is either

* set to too small value, or
* left unconfigured and so defaults to 30000000 bytes (&lt;30 Mb)

So any artifact larger than `maxAllowedContentLength` is discarded by IISCheck the settings value and try to rerun your build

### Artifacts Domain Isolation in setups with IIS causes authorization error on accessing artifacts
{instance="tc"}

If [Artifacts Domain Isolation](teamcity-configuration-and-maintenance.md#artifacts-domain-isolation) is enabled on your TeamCity server, trying to access build artifacts might result in the 401 Unauthorized error. This issue is most likely caused by the default behavior of your IIS web server: it rewrites the response headers targeting them back from the artifacts' domain to the main server domain. To disable this behavior, go to __Application Request Routing | Server Proxy Settings__ in the IIS interface and disable the "_Reverse rewrite host in response headers_" option.


### Content Missing from TeamCity UI

When loading a TeamCity page, you might notice some content missing. Most commonly, build configurations that have finished builds do not display their build histories and appear empty. At the same time, a web browser logs multiple 404 (Not Found) errors.

If this happens for a TeamCity instance running on an IIS server, check the values of the IIS `maxQueryStringLength` and/or `maxQueryString` values. These settings limit the maximum length of query strings. If these limits are too low, some of REST requests TeamCity sends to the `<server-url>/app/rest/...` endpoints may fail.

To resolve this issue, increase these limits in the `web.config` file of your IIS server. The exact value should depend on your current configuration and the length of your project/configuration IDs. As a general recommendation, set this limit to 4000 characters and gradually raise it if the issue persists.

See also: [Request Limits](https://learn.microsoft.com/en-us/iis/configuration/system.webServer/security/requestFiltering/requestLimits/) | [httpRuntime Element](https://learn.microsoft.com/en-us/previous-versions/dotnet/netframework-4.0/e1f13641(v=vs.100)?redirectedfrom=MSDN)



## SSL problems when connecting to HTTPS from TeamCity (handshake alert: unrecognized_name)
{instance="tc"}

This problem may happen when changing JVM from 1.6 to 1.7 and connecting some incorrectly configured HTTPS servers. 
The problem and workaround for it are described in [this issue](https://youtrack.jetbrains.com/issue/TW-30210).

<!--[//]: # (Internal note. Do not delete. "Known Issuesd193e619.txt")-->

## Distorted Configuration Window During Agent Reinstallation

When installing a TeamCity agent via a [Windows agent installer](install-teamcity-agent.md#Install+from+Windows+Executable+File) on top of the already installed agent with a different version of Java, the "_Configure Build Agent Properties_" installation window might appear distorted.

__This issue has been fixed in TeamCity 2019.1.5__.

To work around this issue without upgrading to 2019.1.5, uninstall the previously installed agent version before installing a new agent into the same directory.   
To uninstall the agent, invoke `Uninstall.exe` in the [Agent Home Directory](agent-home-directory.md), clear all the "_Remove..._" checkboxes to keep the agent logs and configuration, and click __Uninstall__. After the successful uninstallation, you can proceed with installing the new agent version via the agent installer.


## Incorrect TeamCity Server Version in Windows

Automatic TeamCity updates may fail to write new version numbers to the related Windows registry key. This causes the standard Windows **Add or Remove Programs** application to show the version of the initial TeamCity installation that never changes after updates.

To manually update the registry key and set a correct version, follow the instructions on the TeamCity **Administration | Updates** page to run a PowerShell script shipped with your TeamCity installation:

```Shell
runas /user: Administrator "powershell -File C:\TeamCity\bin\update-registry-version.ps1"
```

## Windows Docker Containers
{instance="tc"}

Problems common to TeamCity Docker container images.

* Since Windows 10 version 1803 with [KB4340917](https://support.microsoft.com/en-us/help/4340917/windows-10-update-kb4340917) it's possible to use port mapping from containers to localhost. For previous Windows versions it works for the non\-localhost IP address associated with this machine and you can access a running application via the machine's hostname or determine the IP address via the `ipconfig` command. Note that the `netstat -an` command may not show that the port is open on any IP address, while in fact it can work. This is also a known problem of Docker on Windows.

* On Windows 10, the memory allocated per container is 1GB by default. To increase this value, use the following memory options:

    ```Shell
        docker run ... -m 6GB --cpus=4 -e TEAMCITY_SERVER_MEM_OPTS="-Xmx3g -XX:ReservedCodeCacheSize=640m"
        
    ```

* On Windows 10 containers work via Hyper-V and may experience problems with network and other subsystems. To diagnose these problems, execute the following PowerShell script:

    ```Shell
        Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
        
    ```

* When starting a TeamCity server from a Windows Docker image, make sure to grant "_Authenticated Users_" the _Full Control_ over the directories used as volumes. [See the related issue](https://github.com/docker/for-win/issues/1058).

* When starting a Windows Docker container with the directory `C:/BuildAgent/work` mapped as a volume to the container host, Git for Windows fails with a following error: "_Invalid path '/ContainerMappedDirectories': No such file or directory_". The workaround is not to add `C:/BuildAgent/work` as a volume.

To analyze the script output, refer to the [following documents](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-server-container-tools/Debug-ContainerHost). If it shows that there are problems with the container network subsystem, try resetting it using the [clean-up scripts](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-server-container-tools/CleanupContainerHostNetworking).

More details on troubleshooting Docker for Windows are available in the [Docker](https://docs.docker.com/docker-for-windows/troubleshoot/) and [Microsoft](https://docs.microsoft.com/en-us/virtualization/windowscontainers/troubleshooting) documentation.

### Information about installed Docker server OS on Windows missing on Agent

On Windows 10, the Docker server depends on Hyper-V service and its start may take a significant amount of time. To resolve the issue, configure the TCBuildAgent Windows service to depend on the Docker for Windows Service, `com.docker.service` by default.

### Linux Docker Containers under Windows

The [](container-wrapper.md) works on Windows when Windows-based containers are started.

If a Linux container is started on a Windows machine, TeamCity displays the error message "Starting Linux Docker containers under Windows is not supported. To avoid this problem, add the [`teamcity.agent.jvm.os.name`](integrating-teamcity-with-container-managers.md#Build+Parameters+and+Agent+Requirements) does not contain Windows [agent requirement](configuring-agent-requirements.md).

If you need to support a use case when the [](container-wrapper.md) runs Linux containers under Windows platform, vote for the related comment in [TW-51820](https://youtrack.jetbrains.com/issue/TW-51820).

### Problems with a local time in Windows containers

When using Windows Docker containers, there is an issue with [incorrect time in Windows containers](https://youtrack.jetbrains.com/issue/TW-53474) when the system time in a container goes out of sync with the time on the host machine. It could cause problems in integrations where response time is significant (for example, OAuth tokens).

To address it, upgrade your host machine to Windows Server 2019 / Windows 10 1809 and use TeamCity docker images [compatible with Windows containers 1809](https://youtrack.jetbrains.com/issue/TW-58161).

### "Access is denied" or "Access to the path is denied" problem on container start

When Docker is starting Windows containers with __process isolation__, it uses a Windows user account which lacks the write access to the directory with Docker volumes. In this case, build agents may fail to start due to the "_Access to the path is denied_" or "_Access is denied_" error.

To resolve this issue, specify a dedicated volume mapping for the `..\TeamCity\temp` directory in the `docker run` command. We also suggest ensuring that a sufficient amount of resources is allocated to this process.

```Shell
docker run --memory="6g" --cpus=4 -e TEAMCITY_SERVER_MEM_OPTS="-Xmx3g -XX:MaxPermSize=270m -XX:ReservedCodeCacheSize=640m" --name teamcity-server-instance
    -v <path-to-data-directory>:C:/ProgramData/JetBrains/TeamCity 
    -v <path-to-logs-directory>:C:/TeamCity/logs  
    -v <path-to-temp-directory>:C:/TeamCity/temp  
    -p <port-on-host>:8111 jetbrains/teamcity-server

```

Alternatively, you can grant the "Full control" permission to the "Authenticated Users" group for the `%\PROGRAMDATA%\docker\volumes` directory.

## dotCover known issues

### dotCover does not support Windows Nano Server

If you try to run dotCover on an agent with the Nano Server OS, the build will fail with an exit error "_Process exited with code -1073741515_". Instead of Nano Server, consider using [Server Core](https://docs.microsoft.com/en-us/windows-server/administration/server-core/what-is-server-core) which is an alternative minimal installation option of Windows Server.

### dotCover does not support coverage statistics for msbuild

dotCover does not support collection of coverage statistics for the `dotnet msbuild /t:vstest` command — use `dotnet test` instead.

### Code coverage configuration using Test Settings is deprecated
{instance="tc"}

Code coverage configuration using Test Settings is deprecated in dotCover. [Read more](https://docs.microsoft.com/en-us/previous-versions/dd504821(v=vs.140)) in Microsoft documentation.

If you use `.testsettings` files, TeamCity will display a warning message in the log. To prevent this issue, we suggest that you use [`.runsettings`](https://docs.microsoft.com/en-us/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file?view=vs-2019) files instead. If, for some reason, you have to continue using `.testsettings`, install dotCover version 2019.1.x or earlier on TeamCity agents, as described [here](installing-agent-tools.md).

## Xcode 10 is unable to clean artifacts in custom output directory

If you use a custom output directory for Xcode, note that on upgrading to Xcode 10, TeamCity builds with the Xcode Project build runner might fail with the error: "_Could not delete \<directory\> because it was not created by the build system and it is not a subfolder of derived data."_

This is caused by the following Xcode 10 known issue:   
When performing `xcodebuild clean` on a project that uses a customized build location outside the derived data directory, and that has older build products produced prior to Xcode 10, Xcode might report an error indicating that it won't delete directories not created by the new build system. (40427159)   
See [Xcode documentation](https://developer.apple.com/documentation/xcode_release_notes/xcode_10_release_notes/build_system_release_notes_for_xcode_10) for details.

To resolve this issue, we suggest that you use Xcode 11 instead. To work around this issue in Xcode 10, you can either clean the output directory manually or try using the previous build system by passing `-UseNewBuildSystem=NO` to command line parameters.

## Cross-server Projects pop-up menu may not work in latest browsers
{instance="tc"}

Due to the recent updates in the [SameSite cookie](https://web.dev/samesite-cookies-explained/) support, the __Projects__ pop-up menu may not display [cross-server projects](configuring-cross-server-projects-pop-up-menu.md) in some latest web browsers (see [more details](https://www.chromestatus.com/feature/5088147346030592) for Chrome Platform).
 
If you cannot access the cross-server __Projects__ menu, you can try to temporarily work around this issue by changing your browser settings:

* In Google Chrome, open the `chrome://flags` page and disable the "_SameSite by default cookies_" option.
* In Mozilla Firefox, open the `about:config` page and disable the `network.cookie.sameSite.laxByDefault` option.
* In Safari, go to __Preferences | Privacy__ and disable the "_Prevent cross-site tracking_" option.

>By disabling this setting, you reduce the overall security of your browser. Use this workaround only if necessary. We highly recommend that you wait until this issue is fixed in the next version of TeamCity.
>
{style="note"}
 
This problem will be resolved in the TeamCity 2020.1.5 update. See the [related issue](https://youtrack.jetbrains.com/issue/TW-67644) for more details.

## .NET runner known issues

### .NET runner is not compatible with obsolete external .NET CLI Support plugin
{instance="tc"}

The reworked [.NET build runner](net.md) is not compatible with the obsolete external plugin [.NET CLI Support](https://plugins.jetbrains.com/plugin/9190--net-cli-support) (used in versions 2017.1 and earlier). If the obsolete plugin is installed on your server, you will get the _Error creating bean with name "jetbrains.buildServer.dotnet.DotnetRunnerRunType"_ after updating to TeamCity 2020.1. To solve this issue, please [uninstall the obsolete plugin](installing-additional-plugins.md#Uninstalling+Plugin+via+Web+UI) from your server.

Note that all the existing .NET CLI Support build steps are automatically switched to the new .NET runner on updating to TeamCity 2019.2.3 or later. For more information, refer to our [upgrade notes](upgrade-notes.md#Reworked+.NET+build+runner).

### Cannot create an instance of the logger when using early Visual Studio 2017 build

In rare cases, if an early build of Visual Studio 2017 is installed on your build agent, you might get the "Cannot create an instance of the logger" error when running the .NET build step. This might be caused by the logger's incompatibility with the framework version being used.

To work around this problem, you can upgrade Visual Studio 2017 to the latest build or, alternatively, install any later version of .NET Framework.

### Parsing .rsp files (Error MSB1006: Property is not valid)

Due to the [breaking change](https://github.com/dotnet/command-line-api/pull/1714) introduced by Microsoft in the core `System.CommandLine` library, .rsp files with commas and (or) semicolons in property values are parsed incorrectly. To ensure this issue does not occur, enclose values of [system properties](configuring-build-parameters.md#System+Properties) in quotes. If this solution is not applicable to your project, add the `teamcity.internal.dotnet.msbuild.parameters.escape` configuration parameter and set its value to _"true"_. With this setting enabled, TeamCity automatically wraps values of user-defined properties in quotes.

## NuGet known issues

### Prerelease packages are not visible in the TeamCity NuGet feed

__Problem__: Prerelease packages are not visible in the TeamCity NuGet feed.

__Cause__: NuGet clients __prior to version 3__ fail to list prerelease packages if the package version violates [the required format](http://docs.nuget.org/create/versioning#user-content-prerelease-versions).

__Solution__: Delete build artifacts whose versions violate  [the required format](http://docs.nuget.org/create/versioning#user-content-prerelease-versions).

### Packages indexing is slow in TeamCity NuGet feed
{instance="tc"}

__Problem__: After TeamCity server host machine move or upgrade, build metadata can be reset.

__Cause__: The TeamCity NuGet feed relies on build metadata, and packages reindexing can take a lot of time depending on the number of packages and the idle time of the TeamCity server.

__Solution__: To speed up build metadata reindexing, specify the following [internal properties](server-startup-properties.md):


```Shell
teamcity.buildIndexer.indexPackSize=1000
teamcity.buildIndexer.packSleepDurationMs=10

```

To check the metadata indexing progress, look for lines similar to the ones below in the [teamcity-server.log](teamcity-server-logs.md) file:

```Shell
INFO - .index.BuildIndexer (metadata) - Enqueued next 100 builds for indexing, builds left: 7064, last build id: 8142

```

After reindexing is complete, remove these internal properties.

### NuGet does not pull recent package version

__Problem__: NuGet can sometimes skip the recent versions of packages when pulling these packages from the feed.

__Cause__: NuGet caches the server responses, thus pulling does not detect the most recent changes in the feed.

__Solution__: To send a new request directly to the server instead of the cache, use the `--no-cache` parameter in your request.

### Packages are not found in NuGet feed
{instance="tc"}

__Problem__: Not all packages are found in a NuGet feed, though artifacts are present on the disk and in the UI and <path>teamcity-nuget.log</path> does not indicate that indexing of packages is in progress.

__Cause__: One of the possible causes could be that TeamCity was restored from backup or moved to a new server.

__Solution__: Clear the [buildsMetadata](teamcity-monitoring-and-diagnostics.md#Caches) cache and wait until reindexing of packages is finished on the server. Note that it could take a lot of time for a large server with a long build history.

## Cannot use multiline parameters in PowerShell

Earlier versions of the [PowerShell](powershell.md) runner don't support passing multiline arguments. Since version 2020.1.4, you can enable this support by setting the `teamcity.powershell.arguments.multiline=true` [configuration parameter](configuring-build-parameters.md).

## Error while loading VCS changes

_Error while loading VCS changes_ on the TeamCity server startup might be associated with the enabled cursor-based streaming in MySQL. To prevent this error, try setting the `connectionProperties.useCursorFetch` property to `false` in `database.properties` and restart the server.  
See [this issue](https://youtrack.jetbrains.com/issue/TW-71781) for more details.



## Known issues of Pull Requests build feature

* In some cases, TeamCity could start builds on old open merge requests in __JetBrains Space__ and pull requests in __Bitbucket Cloud__. This is reproduced with the following sequence of steps:
  1. Create a new VCS root that connects to a repository with open merge requests. Leave the default branch specification.
  2. Create a build configuration with this VCS root and a VCS trigger.
  3. Remove the branch specification from the VCS root.
  4. Add the Pull Requests build feature to the build configuration.
  5. Disable the Pull Requests feature and then enable it again.  

  See the [related issue](https://youtrack.jetbrains.com/issue/TW-74379) for more details.
* The [Pull Requests](pull-requests.md) build feature could post a wrong build step number to the merge request timeline in __JetBrains Space__. See the [related issue](https://youtrack.jetbrains.com/issue/TW-74374).

## Issues related to Git Repository Ownership Checks

Due to [more strict repository ownership checks](https://github.blog/2022-04-18-highlights-from-git-2-36/#stricter-repository-ownership-checks) introduced in Git 2.35.2, updating the TeamCity server's Git to this or a newer version may result in malfunctions (for instance, the _"fatal: could not read from remote repository"_ message). Try the following workarounds as temporary solutions:

* If your TeamCity server runs on Linux, ensure the volume with the &lt;TeamCity data dir&gt;/system/caches/git directory is mounted under the same user that TeamCity utilizes to access this directory.

* If you experience related issues on a TeamCity Server installed or updated via a Windows Docker image, disable the [native Git](https://www.jetbrains.com/help/teamcity/git.html#Native+Git+for+VCS-related+operations+on+the+server) to force the TeamCity server to use JGit instead.


## Known issues of native Git checkout

These issues concern the use of native [Git](git.md) for checking out sources to the TeamCity server, effective since [version 2022.04](https://www.jetbrains.com/help/teamcity/2022.04/what-s-new-in-teamcity.html#Native+Git).

### SSH DSA keys do not work with native Git

Switching the server to native Git results in builds failing to authorize in repositories via SSH DSA keys. See the related issue [TW-74580](https://youtrack.jetbrains.com/issue/TW-74580).

To work around this issue, please set the `teamcity.git.sshCommandOptions` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) to `-o "PubkeyAcceptedKeyTypes=+ssh-dss"`.
{instance="tc"}

### Native Git via OpenSSH may fail

Native Git via OpenSSH may fail on Windows if the server/agent installation path contains spaces. See [the related issue](https://youtrack.jetbrains.com/issue/TW-76041/).

### nonProxyHosts option for ssh proxy is not available

TeamCity uses an ssh proxy when connecting via native Git. See [the related issue](https://youtrack.jetbrains.com/issue/TW-27672/Add-a-nonProxyHosts-option-for-ssh-proxy-in-git-plugin)

### Custom ssl certificates are not supported for native Git 
{instance="tc"}

Custom ssl certificates support for native Git on the server is not available. See [the related issue](https://youtrack.jetbrains.com/issue/TW-75507/Custom-ssl-certificate-support-for-native-git-on-server)

__This issue was fixed in TeamCity 2022.10.1__.

### Publishing artifacts to third party S3-compatibles storages may fail 
{instance="tc"}

TeamCity 2022.04 changes [the ACL setting related to the Amazon S3 artifact storage](https://youtrack.jetbrains.com/issue/TW-75512), 
which may cause publishing artifacts to third party S3-compatibles storages, such as Backblaze, to fail. 

To fix this problem after updating to TeamCity 2022.04, download the plugin version from [the related issue](https://youtrack.jetbrains.com/issue/TW-76119/Artifact-upload-to-Backblaze-B2-s-S3-compatible-API-fails-with-T#focus=Comments-27-6054071.0-0) 
and add the `storage.s3.acl` configuration parameter with the required `acl` value. 
All the values listed [here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html#canned-acl) are supported.

## Artifact Decompression Issues

The regular ZIP format supports archives up to 2^32 bytes (4 GB) in size. Since builds can produce artifacts that exceed this value, TeamCity compresses artifacts with the ZIP64 format that supports archives as large as 2^64 bytes (16 exabytes).

If tools that you utilize to unzip TeamCity build artifacts do not support ZIP64 (for example, you may observe header error warnings), add the following [configuration parameters](configuring-build-parameters.md) to a project (either a &lt;Root project&gt; or a specific project that produces unsupported archives):

```
teamcity.internal.artifacts.useZip64=Never
teamcity.internal.artifacts.useByteChannelForZip=false
```