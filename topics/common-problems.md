[//]: # (title: Common Problems)
[//]: # (auxiliary-id: Common Problems)

Most user issues are related to the following topics. Before reporting your problem, check if any of these Help pages contains the solution already:
{instance="tc"}

* [Configuring server memory settings](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server)
{instance="tc"}
* [Reporting server slowness issues](reporting-issues.md#Collect+Data)
{instance="tc"}

## Build works locally but fails or misbehaves in TeamCity

If a build fails or otherwise misbehaves in TeamCity but you believe it should not, the first thing to do is to check whether the issue is related to TeamCity or not. 

To do that, follow this procedure:

1. Find a way to run the task from a command prompt. Make sure it works on the TeamCity agent machine, under the same user as the TeamCity agent runs under, with the same environment the agent receives. If necessary, run the TeamCity agent under a different user or tweak its environment.
2. When the command runs OK, configure the same command in a TeamCity build using the [Command Line](command-line.md) runner with the custom script setting. 
3. If that works, try another runner if that feels applicable.

Here are details on the approach:

Check that the build runs fine from the command prompt when run on the same machine as the TeamCity agent and under the same user that the agent is running, with the same environment variables and the same working directory, same architecture (32/64 bit) command line.

If the TeamCity build agent is run as a service (for example, it is installed as a Windows service), try running the TeamCity agent under a regular user with administrative permissions [from the command line](start-teamcity-agent.md). See also [Windows Service limitations](known-issues.md#Agent+running+as+Windows+Service+Limitations).

If this fixes the issue, you can try to figure out why running under the service is a problem for the build. Most often this is service-specific and is not related to TeamCity directly. Also, you can set up the TeamCity agent to be run from the console all the time (for example, [configure](start-teamcity-agent.md#Automatic+Start) an automatic user logon and run the agent on the user logon).

Here are the detailed steps you can use to run a build from the command line.

Assuming you have a configured build in TeamCity which is failing, do the following:

* run the build in TeamCity and see it misbehaving
* disable the agent so that no other builds run on it. This can be done while the build is still in progress
* log in to the agent machine using the same user as the one running the TeamCity agent (check the right user in the machine processes list)
* stop the agent
* in a command line console, `cd` to the checkout directory of the build in question (the directory can be looked up in the beginning of the build log in TeamCity)
* run the build with a command line as you would do on a developer machine. This is runner-dependent. (For some runners you can look up the command line used by TeamCity in the build log, see also the `logs\teamcity-agent.log` file for the command line used by TeamCity)
* if the build fails — investigate the reason as the issue is probably not TeamCity-related and should be investigated on the machine.
* if it runs OK, continue
* in the same console window `cd` to the [`<TeamCity agent home>`](agent-home-directory.md)`/bin` directory and start TeamCity agent from there with the `agent start` command
* ensure the runner settings in TeamCity are appropriate and should generate the same command line as you used manually. For example, use the _Command Line_ build step with the _Custom script_ option and the same command which can be saved in a `.sh` or `.bat` file and run from the command prompt
* run the build in TeamCity selecting the agent in the Run custom build dialog
* when finished, enable the agent

If the build succeeds from the console but still fails in TeamCity, use a command line runner in TeamCity to launch the same command as in the console. 

If it still behaves differently in TeamCity, most probably this is an environment or a tool issue. 

If the command line runner works but the dedicated runner does not while the options are all the same, create a new issue in our [tracker](https://youtrack.jetbrains.com/issues/TW) detailing the case. Please attach all the build step settings, the build log, all agent logs covering the build, the command you used in the console to run the build and the full console output of the build.

<!--[//]: # (Internal note. Do not delete. "Common Problemsd63e112.txt")-->

## Build is slow under TeamCity

If you experience slow builds, the first thing to do is to check the build log to see if there are some long operations or the time is just spread over the entire process. You can compare build logs of slower and faster builds to figure out what the difference is. You can also run the build from the console on the same machine as detailed [above](#Build+works+locally+but+fails+or+misbehaves+in+TeamCity) to see if there is any difference between the build run from the console and the build in TeamCity.

If the slowness is spread over all the operations, the agent machine resources (CPU, disk, memory, network) are to be analyzed during the build to see if there is a bottleneck in any of those. The [Performance Monitor](performance-monitor.md) build feature lets you see how much CPU, memory and disk I/O is used by the build and at which stage.

If build process is hanging, then "View thread dump" link on the running build results page can help with understanding why it happens.

If there is some long operation and it is a TeamCity-related one (before the start or after the end of the actual build process), the TeamCity agent and server are to be analyzed (logs and thread dumps).

If you want to [turn to us](troubleshooting.md) with the issue, make sure to describe the visible effects, detail the process of investigation and attach the build log, full agent logs and other data collected.

## Started Build Agent is not available on the server to run builds

First start of agent after installation or TeamCity server upgrade/plugin installation can take time as agent downloads updates form the server and autoupgrades. 

Regularly, agent should become connected in 1 to 10 minutes, depending on the agent/server network connection speed.

If the agent is not connected within that time, check the name of the agent (as configured in `conf\buildAgent.properties` file) and check the tabs under the __Agents__ page:
* the agent is under Connected — the agent is ready to run builds
* the agent is under Disconnected — the agent was connected to the server, but became disconnected. Check the "Inactivity reason" in the table. If the reason is "Agent has unregistered (will upgrade)", then wait for several more minutes
* the agent is under Unauthorized — all the agents connected to the server for the first time should be authorized by a server administrator

If the agent stays in the state for more than 10 minutes and you have a fast network connection between the agent and the server, do the following:
* check the related agent machine to ensure that the agent process is running and `serverURL` in `conf\buildAgent.properties` is correct (and that the server is reachable by that URL from the machine);
* check that all the related environment [requirements](system-requirements.md#Common+Requirements) are met;
* check [agent logs](viewing-build-agent-logs.md) (`teamcity-agent.log`, `launcher.log`, `upgrade.log`) for any related messages/errors;
* check [server logs](teamcity-server-logs.md) (`teamcity-server.log`) for any messages/errors mentioning agent name or IP.
{instance="tc"}

If you cannot find the cause of the delayed agent upgrade in the logs, [contact us](troubleshooting.md) and provide the full agent and server logs. Be sure to check/include the state of the agent processes (java ones) on the agent machine.

## Artifacts of a build are not cleaned

If you encounter a case when artifacts are preserved while they should have been removed by the server clean-up process, check the following:
* the clean-up rules of the build configuration, artifacts section
* presence of the icon "This build is used by other builds" in the build history line (prior to Pin action/icon on Build History)
* build's Dependencies tab, "Delivered Artifacts" section. For every build configuration, check whether "Prevent dependency artifacts clean-up" is turned ON (this is default value). If it is, then the build's artifacts are not cleaned because of the setting. 

Read more on [clean-up settings](teamcity-data-clean-up.md#Base+Rule+Behavior+for+Dependency+Builds).

## Database-related issues
{instance="tc"}

### "out of memory" error with internal (HSQLDB) database

If during the TeamCity server start-up you encounter errors like: _"error in script file line: ... out of memory"_, _"java.sql.SQLException: out of memory"_, perform the following:

* try [increasing server memory](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server). If this does not help, most probably this means that you have encountered __internal database corruption__. You can try to deal with this corruption using the [notes](https://www.hsqldb.org/doc/1.8/guide/apc.html) based on the HSQLDB documentation.

Here is a way to attempt a manual database restore:
* stop the TeamCity server
* back up the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/system/buildserver.data` file
* remove the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/system/buildserver.data` file and replace it with zero-size file of the same name
* start the TeamCity server

However, if the database does not recover automatically, chances that it can be fixed manually are minimal.

The internal (HSQL) database is not stable enough for production use and we highly recommend using an [external database](set-up-external-database.md) for TeamCity non-evaluation usage.  
If you encountered database corruption, you can restore the last good backup or drop builds history and users, but preserve the settings, see [Migrating to an External Database](migrating-to-external-database.md#Switch+with+No+Data+Migration).

### The transaction... log is full

This error can occur with an MS SQL or Sybase database. In this case we recommend increasing the transaction log for the TeamCity database. The log size can be 1 - 16 GB depending on the number of build agents in the system and the number of tests all agents report daily.

### The table 'table_name' is full

This error can occur with a MySQL database. The error indicates that the database has run out of free space either on the disk where the database files are located or in the temp directory.

### Unable to extend ... segment ... in tablespace ...

This error can occur with an Oracle database. The error indicates that Oracle could not obtain more space for a table or an index as the database has run out of space on the disk or the specified quotas are insufficient.

### NOCOUNT is enabled on MS SQL Server

`teamcity-server.log` reports that the unsupported `NOCOUNT` option is enabled on the MS SQL database server. Contact your DBA to disable the setting.

### MySQL JDBC driver error: PacketTooBigException

The `ER_NET_PACKET_TOO_LARGE` error (PacketTooBigException / Packet for query is too large) is caused by the server-side `max_allowed_packet` [configuration variable](https://dev.mysql.com/doc/refman/8.0/en/packet-too-large.html) set to a low value or left at the default one.

 The variable controls the maximum size of MySQL communication buffer with 4MB being the default for Windows builds of MySQL 5.6, whereas in popular Linux distributions (e. g. Debian and Fedora Core), this variable defaults to 16MB, for both i686 and amd64 architectures.
 
1. Check the value of the `max_allowed_packet` [configuration variable](https://dev.mysql.com/doc/refman/5.7/en/packet-too-large.html) by either examining `my.cnf / my.ini` , or via 
         
    ```Shell
    mysql> show variables like 'max_allowed_packet';
    ```

2. Increase (or explicitly set) the value of the `max_allowed_packet` [configuration variable](https://dev.mysql.com/doc/refman/5.7/en/packet-too-large.html) in `my.cnf` or `my.ini`: 
             
    ```Shell
    [mysqld]
    max_allowed_packet=16M

    ```
    

<note>

Setting a client-side value (via `maxAllowedPacket` [connection property](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html) ) in the JDBC URL will have no effect,  as this value cannot exceed the server-side limit.
</note>

### Database character set/collation-related problems

<anchor name="CommonProblems-Characterset%2Fcollationmismatch"/>

#### Character set/collation mismatch

TeamCity reports character set/collation mismatch error: database tables/columns have a character set or collation that is not the same as the default character set or collation in your database schema. You may see this message if you are using a non-unicode character set as default for your database as TeamCity enforces unicode charset for some of the `varchar` fields. Make sure you configured the database according to our [recommendations](set-up-external-database.md).

#### TeamCity displays ???? instead of national symbols

If you want to allow your local characters in texts in TeamCity (for example, VCS messages, test names, usernames), you need to migrate to a database with the appropriate character set.

#### "Unique key violations" or "Duplicates found" error on restore from backup

TeamCity server restoration from a backup may fail with errors like `"Unique key violation"` or `"Duplicates found"` if the character sets (or collations) of the source and destination databases are not the same, and the cardinality of the destination character set is less than that of the source database.

To resolve the problem, select and set up the proper character set (and collation) for the destination database.

As for case sensitivity, the possible transitions are: 
- CS &gt; CS 
- CI &gt; CS 
- CI &gt; CI 

However, it is recommended to always use CS.

If the source character set is Unicode or UTF, the destination one must also be Unicode or UTF. If the source character set is 8-bit non-UTF, the destination one can be the same or Unicode/UTF.

#### Resolve character set/collation-related problems

To fix a problem, perform the following steps:

1. Create a new database with the appropriate character set and collation. We recommend using a __unicode case-sensitive__ collation: see instructions for [PostgreSQL](set-up-external-database.md#On+PostgreSQL+Server+Side) and [MySQL](set-up-external-database.md#On+MySQL+Server+Side). For MySQL, `utf8_bin` or `utf8mb4_bin` is preferred.  
    See also [PostgreSQL](https://www.postgresql.org/docs/9.3/static/multibyte.html), [MySQL](http://dev.mysql.com/doc/refman/5.0/en/charset-mysql.html), [MS SQL](http://technet.microsoft.com/en-us/library/ms180175(v=sql.105).aspx) documentation for details on character set.
    
2. Copy the current [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/database.properties` file, and change the database references in the copy to the newly created database.
3. Stop the TeamCity server.
4. Use the `maintainDB` tool to migrate to the new database:

    ```Shell
    maintainDB migrate [-A <path-to-data-dir>] -T <new-database-properties-file>

    ```
    Depending on the size of your database, the migration may take from several minutes to several hours. For more information on the `maintainDB tool`, see [this section](migrating-to-external-database.md#Full+Migration).

5. Upon the successful completion of the database migration, the `maintainDB` tool should update the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/database.properties` file with references to the new database. Ensure that the file has been updated. Edit the file manually if the tool fails to do it automatically.
6. Start the TeamCity server.

#### MySQL exception: Specified key was too long; max key length is 767 bytes 

#### Index column size too large. The maximum column size is 767 bytes

Check if the character set of your MySQL database. It is recommended to use the `utf8` character set.

If your database uses the `utf8mb4` character set (available since MySQL 5.5), set the following InnoDB configuration options (under `[mysqld]` section in `my.cnf` or `my.ini`) to run:
* `innodb_large_prefix=1` for index key prefixes longer than 767 bytes (up to 3072 bytes) to be allowed for InnoDB tables that use DYNAMIC row format (deprecated in MySQL 5.7.7).
* `innodb_file_format=Barracuda` to enable the DYNAMIC row format.
* `innodb_file_per_table=1` to enable the Barracuda file format

The parameters above have the following default values:

<table><tr>

<td>

</td>

<td>

MySQL 5.5

</td>

<td>

MySQL 5.6

</td>

<td>

MySQL 5.7

</td>

<td>

MariaDB 5.5

</td>

<td>

 MariaDB 10.0

</td>

<td>

 MariaDB 10.1

</td>

<td>

MariaDB 10.2

</td></tr><tr>

<td>

innodb_large_prefix 

</td>

<td>

0

</td>

<td>

0

</td>

<td>

1

</td>

<td>

0

</td>

<td>

0

</td>

<td>

0

</td>

<td>

1

</td></tr><tr>

<td>

innodb_file_format

</td>

<td>

Antelope 

</td>

<td>

Antelope 

</td>

<td>

Barracuda

</td>

<td>

Antelope 

</td>

<td>

Antelope 

</td>

<td>

Antelope 

</td>

<td>

Barracuda

</td></tr><tr>

<td>

innodb_file_per_table

</td>

<td>

0

</td>

<td>

1

</td>

<td>

1

</td>

<td>

1

</td>

<td>

1

</td>

<td>

1

</td>

<td>

1

</td></tr></table>

Depending on the database server version, the following configuration changes need to be made:

__MySQL 5.5:__

* `innodb_large_prefix=1`
* `innodb_file_format=Barracuda`
* `innodb_file_per_table=1`


__MySQL 5.6 and MariaDB from 5.5 to 10.1:__
* `innodb_file_format=Barracuda`
* `innodb_file_per_table=1`


For __MySQL 5.7\+ and MariaDB 10.2\+__ use the defaults, no changes are required.


### 'Incorrect string value' error in MySQL

1. Make sure the database uses the `utf8mb4` character set and `utf8mb4_bin` collation. If that's not the case, [change the DB character set and collation](#Resolve+character+set%2Fcollation-related+problems).
2. If the character set and collation are set properly, but the new errors still occur, the problem is that the default charset in MySQL is not utf8mb4. Set up the MySQL server to use `utf8mb4` by default by specifying the following options in the `[mysqld]` section of `my.cnf`:  
    `character_set_server = utf8mb4`   
    `collation_server = utf8mb4_bin`
3. Restart MySQL for the changes in `my.cnf` to take effect.

### 'This driver is not configured for integrated authentication' error with MS SQL database

During TeamCity installation, the following error might occur when connecting and creating the MS SQL database with Windows integrated security: "SQL error when doing: Taking a connection from the data source: This driver is not configured for integrated authentication."

The most common reason for the problem is the different bitness of the `sqljdbc_auth.dll` MS SQL shared library and the JRE used by TeamCity.

To solve the problem, do the following:

1. Make sure you use the MS SQL native driver (downloadable from the [Microsoft Download Center](http://www.microsoft.com/download/en/details.aspx?displaylang=en&amp;id=11774)). 
2. Use the right JRE bitness: ensure that you are running TeamCity using Java with the same bitness as your `sqljdbc_auth.dll` MS SQL shared library. By default, TeamCity uses the 64-bit Java. However, both 32-bit and 64-bit Java versions [can be used](how-to.md#Install+Non-Bundled+Version+of+Java).

To run TeamCity with the required JRE, do one of the following:
    * either set the `TEAMCITY_JRE` environment variable
    * or remove the JRE bundled with TeamCity from [`<TeamCity home>`](teamcity-home-directory.md)`\jre` and set `JAVA_HOME`.  
    
<note>

Note that on upgrade, TeamCity will overwrite the existing JRE with the default 64-bit version. If you switched to the 32-bit version, you will need to re-apply your custom changes after every upgrade.

</note>

See also: [](how-to.md#Update+from+32-bit+to+64-bit+Java).

### Protocol violation error (Oracle only)

This error can occur when the Oracle JDBC driver is not compatible with the Oracle server. For example, Oracle JDBC driver version 11.1 is not compatible with Oracle server version 10.2.

In order to resolve the problem, use the Oracle JDBC driver from your Oracle server installation, or [download the driver](https://www.oracle.com/technetwork/database/features/jdbc/index-091264.html) of the same version as the Oracle server.

### MySQL data directory size

MySQL data directory may include large `<hostname>-bin.<index>` files that may consume significant disk space. These files are not related to TeamCity: they are [MySQL Server binary log files](https://dev.mysql.com/doc/refman/8.0/en/binary-log.html) that contain a sequential log of changes to MySQL databases. These files are used for MySQL Server replication and data recovery.

By default, the binary log files are set to expire automatically after 30 days.

## Common Maven issues

There are two kinds of Maven-related issues commonly seen in the TeamCity build configurations:
* Error message on "Maven" tab of build configuration: "An error occurred during collecting Maven project information ... "
* Error message in build configuration with Maven dependencies trigger activated: "Unable to check for Maven dependency Update ..."
If the build configuration produces successful builds despite displaying such error messages, these errors are likely to be caused by the __server-side Maven misconfiguration__.

To collect information for the __Maven__ tab, or to perform Maven dependencies check (for the trigger), TeamCity runs the embedded Maven. The execution is performed on the _server_ machine, and any _agent-side_ maven settings are __not accessible__. TeamCity resolves the `settings.xml` files on the server-side separately, as described in [Maven Server-Side Settings](maven-server-side-settings.md).

It makes sense to check if the `server-side settings.xml` files contain correct information about remote repositories, proxies, mirrors, profiles, credentials, and so on.

<anchor name="%22Criticalerrorinconfigurationfile%22errors"/>

## "Critical error in configuration file" errors

If you encounter the error, it means the settings stored in the TeamCity Data Directory are in an inconsistent state. This can occur after manual change of the files or if newer version of TeamCity starts to report the inconsistencies.  
To resolve the issue, you can edit the file noted in the message on the server file system. (Make sure to create backup copy of the file before any manual edits). Usually server restart is not necessary for the changes to take effect.

__VCS root with id "XXX" does not exist__ 
  
The build configuration or template reference a VCS root which is not defined in the system. Remedy actions: Restore the VCS root or create a new VCS root with the id noted or edit the file noted in the message to remove the reference to the VCS root.

## TeamCity installation problems
{instance="tc"}

If the TeamCity web UI cannot be accessed after installation, you might be running TeamCity on a port that is already in use by another program. [Check and configure](configure-server-installation.md) your TeamCity installation.

## Problems with TeamCity NuGet Feed
{instance="tc"}

If you are experiencing issues with partial TeamCity NuGet Feed (for example, missing NuGet packages), you might have to reindex the TeamCity NuGet Feed.

To force TeamCity to reindex all available packages and reset the NuGet package list, navigate to the server __Administration | Diagnostics | Caches__ and use the [buildsMetadata Reset](teamcity-monitoring-and-diagnostics.md#Caches) link.

## Problems with .NET-related TeamCity Tools

### Startup performance issues

.NET Framework below version 4.0 installed on TeamCity agents may cause performance issues of .NET-related TeamCity tools due to Code access security (CAS) policy imposed by Microsoft.

To solve the issue, use one of the options:

1. Add the following setting described in the [Microsoft documentation](https://msdn.microsoft.com/en-us/library/bb629393%28v=vs.110%29.aspx) to the `machine.config` file on all agents:


    ```Shell
    <configuration>
	    <runtime>
		    <generatePublisherEvidence enabled="false"/>
	    </runtime>
    </configuration>
    
    ```
    
    You can modify the machine.config file as described in this [external blog post](http://blogs.msdn.com/b/amolravande/archive/2008/07/20/startup-performance-disable-the-generatepublisherevidence-property.aspx) and pass this config file to all agents, e.g. using a custom script.

2. Alternatively, upgrade .NET Framework on the TeamCity agents to version 4.0 and above. Details are available in the [Microsoft documentation](https://msdn.microsoft.com/en-us/library/dd233103%28v=vs.100%29.aspx#simplification).

## Using tools requiring manual input, in particular Extended Validation code signing

You might want to use tools which require some manual interaction during the build procedure executed on the TeamCity agent. This is not a TeamCity-specific problem, so it should be approached using generic means.

Under Windows, you might want to configure TeamCity agent to run not as a Service, but with access to the desktop by configuring automatic user logon, [related details](start-teamcity-agent.md#Automatic+Agent+Start+Under+Windows).

There is no simple solution for Extended Validation (EV) code signing as the feature is built in for a reason. There is some discussion on the issue on [stack overflow](https://stackoverflow.com/questions/17927895/automate-extended-validation-ev-code-signing=). The appropriate solution seems to implement a dedicated service with own authorization approach and sign the binaries through it.

## Tomcat error: Request Header too large

By default, Tomcat sets a 4096 bytes (4KB) limit to the request and response HTTP headers. If a header includes too many cookies (for example, from the proxy servers between an end user and TeamCity) or their size is too large, users might stumble on the `400: Request Header too large` or `502: Bad Gateway` errors when browsing TeamCity. To identify the cause of this problem, we recommend that you inspect the affected header with the browser developer console.

To resolve this issue, increase the Tomcat header limit by editing the `maxHttpHeaderSize` parameter value for all relevant connectors in the `$TOMCAT_HOME/conf/server.xml` file. For example, the following sample sets the parameter value to `65536` to increase the limit to 64KB.

```XML
<Connector port="8111" protocol="org.apache.coyote.http11.Http11NioProtocol"
           connectionTimeout="60000"
           redirectPort="8543"
           useBodyEncodingForURI="true"
           tcpNoDelay="1"
           maxHttpHeaderSize="65536"
/>
```
