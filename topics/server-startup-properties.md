[//]: # (title: Configuring TeamCity Server Startup Properties)
[//]: # (auxiliary-id: Server Startup Properties;Configuring TeamCity Server Startup Properties)

Various aspects of a TeamCity server behavior can be customized through options passed on its start:
* Internal properties affecting TeamCity itself
* Java Virtual Machine (JVM) properties

>Be cautious when altering the TeamCity behavior. If you are not sure in the consequences, contact the [TeamCity support](troubleshooting.md) first.
>
{style="warning"}

## TeamCity Internal Properties 

TeamCity has internal configuration properties which affect various aspects of the internal logic. These are normally meant for debugging, changing internal constants, or enabling experimental behavior.

__Please do not change internal properties unless asked by the TeamCity support team.__ If you have internal properties customized, make sure to note this when you contact the TeamCity support.

Server administrators can review and edit internal properties in the TeamCity UI. To do this, go to __Administration | Server Administration | Diagnostics | Internal Properties__ and click __Edit internal properties__.   
Many properties do not require the server restart, but some do. When the restart is required, it is usually specifically noted.

The properties are stored in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/internal.properties` file. This is a Java [properties file](https://en.wikipedia.org/wiki/.properties). You can edit the file manually and add each `<property_name>=<property_value>` on a separate line.

An alternative but obsolete method of adding an internal property is to pass it as a `-D<name>=<value>` JVM option (see the [section below](#JVM+Options)).

>If you are installing TeamCity from a Docker image, see [this reference](https://hub.docker.com/r/jetbrains/teamcity-server/) for more information on properties.

## JVM Options

If you need to pass additional JVM options to a TeamCity server (for example, `-D` or `-X...`), the approach will depend on the way the server is run. You will need to [restart](start-teamcity-server.md) the server for the changes to take effect.

>For general notes on the memory settings, refer to [this article](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server).

If you run the server using the `runAll` or `teamcity-server` scripts or as a Windows service, you need to set the options via the OS [environment variables](https://en.wikipedia.org/wiki/Environment_variable) passed to the TeamCity server process:
* `TEAMCITY_SERVER_MEM_OPTS` — server JVM memory options (for example, `-Xmx750m`).
* `TEAMCITY_SERVER_OPTS` — additional server JVM options (for example, `-Dteamcity.git.fetch.separate.process=false`).

>We recommend removing the `-XX:MaxPermSize` JVM option from the `TEAMCITY_SERVER_MEM_OPTS` environment variable, if previously configured, since it is ignored in Java 8.

The process of setting an environment variable depends on your operating system. For example, in Windows, go to `Control Panel\System and Security\System` and open __Advanced system settings | Environment Variables__.

Make sure the environment variables are set for the user whose account is used to run TeamCity or as global environment variables. You might need to restart the OS then — for the changes to take effect.

<seealso>
        <category ref="concepts">
            <a href="teamcity-data-directory.md">TeamCity Data Directory</a>
        </category>
        <category ref="admin-guide">
            <a href="configuring-build-agent-startup-properties.md">Configuring Build Agent Startup Properties</a>
        </category>
</seealso>

