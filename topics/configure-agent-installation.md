[//]: # (title: Configure Agent Installation)
[//]: # (auxiliary-id: Configure Agent Installation;Build Agent Configuration)

>This page is only relevant for [self-hosted build agents](teamcity-cloud-subscription-and-licensing.md#cloud-self-hosted-agents).
>
{type="note" instance="tcc"}

A build agent can be configured by adjusting in the `<TeamCity Agent Home>/conf/buildAgent.properties` file.

## General Agent Configuration

This [Java properties](https://java.sun.com/j2se/1.5.0/docs/api/java/util/Properties.html#load(java.io.InputStream)) configuration file can store properties that will be published on the server as _agent properties_ and can participate in the [Agent Requirements](configuring-agent-requirements.md) expressions. All [system and environment properties](predefined-build-parameters.md#Predefined+Agent+Build+Parameters) defined in the file will be passed to every build run on the agent.

Syntax reference:
* Use `property_name=value<newline>` syntax.
* Use `#` in the first position of the line for a comment.
* Use `/` instead of `\` as the path separator. If you need to include `\`, escape it with another `\`.
* Whitespaces are processed as any other symbol.

Example agent configuration file:

```Shell
## The address of the TeamCity server. The same as is used to open the TeamCity web interface in the browser.
## Must include the protocol specification (https:// is recommended).
serverUrl=http://localhost:8111/

## The unique name of the agent used to identify this agent on the TeamCity server
## Use blank name to let server generate it.
## By default, this name would be created from the build agent's host name
name=Default agent

## Container directory to create default checkout directories for the build configurations.
workDir=../work

## Container directory for the temporary directories.
## Please note that the directory may be cleaned between the builds.
tempDir=../temp
 
## Container directory for agent state files and caches.
## TeamCity agent assumes ownership of the directory and can delete the content inside.
systemDir=../system

 
######################################
#   Optional Agent Properties        #
######################################
## A token which is used to identify this agent on the TeamCity server for agent authorization purposes.
## It is automatically generated and saved back on the first agent connection to the server.
authorizationToken=1234567890abcdefghijklml

```

Make sure the file is writable for the build agent process itself. For example, that the file is updated to store its authorization token generated on the server-side.

If you install multiple TeamCity nodes [behind a reverse proxy](multinode-setup.md#Proxy+Configuration), `serverUrl` should be set to the proxy URL.
{instance="tc"}

If the `name` property is not specified, the server will generate a build agent name automatically. By default, this name will be created from the build agent's host name.

The file can be edited while the agent is running: the agent detects the change and (upon finishing a running build, if any) restarts automatically loading the new settings.

## Optional Properties

### Build Agent Port

The port where the TeamCity build agent starts and where it listens for the incoming data from the server is determined via the `ownPort` property (9090 by default). If the firewall is configured, make sure that the incoming connections for this port are allowed on the agent machine.

```Shell
ownPort=9090

```

If more than one build agent is hosted on the same machine, different ports must be assigned to them via the `ownPort` property in the `buildAgent.properties` file of every agent.

### Build Agent IP Address

When an agent connects to the TeamCity server for the first time, the agent automatically determines its IP address to use for connection, unless the `ownAddress` property is defined. If the agent machine has several network interfaces, automatic detection may fail. In this case, it is recommended to specify the `ownAddress` property:

```Shell
ownAddress=<own IP address or server-accessible domain name>

```

### Alternative Fetch URL

If you have a self-updating Git repository proxy that is significantly closer than the original Git repository for certain agents, you can allow these agents to download sources from this mirror. To do so, add the `teamcity.git.fetchUrlMapping.<name>=<original URL> => <proxy URL>` setting to the agent configuration file. See this section for more information: [Git VCS Root | General Settings](git.md#General+Settings).

## The Configure Command

<snippet id="agent-configure-command">

To set up the [buildAgent.properties](configure-agent-installation.md#General+Agent+Configuration) file, you can run the `<Agent_folder>/bin/agent.bat` or `<Agent_folder>/bin/agent.sh` script with the `configure` command. This command allows you to set up core agent properties and save them to the target configuration file. If no configuration file path is specified, the changes will be saved to the default `<Agent_folder>/conf/buildAgent.properties` properties.

To view the list of available parameters, run this command with the `--help` flag.


```Shell
./agent.sh configure --help
Java executable is found: '/Library/Java/JavaVirtualMachines/amazon-corretto-11.jdk/Contents/Home/bin/java'
Configuring TeamCity build agent...
TeamCity Agent configurator
Usage: 
	configure PROPERTIES_FILE
	configure -f PROPERTIES_FILE
	configure [PROPERTIES]
	configure --usage
	status
	status short
Where PROPERTIES format is --key=value or --key value
Supported properties:
	* --agent-config-file=PATH - path to build agent properties file (<Agent Root>/conf/buildAgent.properties by default)
	* All default properties that can be configured in buildAgent.properties file
	  Also there are aliases for some of them:
		address = ownAddress
		auth-token = authorizationToken
		logs-dir = logsDir
		name = name
		port = ownPort
		server-url = serverUrl
		system-dir = systemDir
		temp-dir = tempDir
		work-dir = workDir


```

Sample command that writes the server URL and sets up the custom agent name:

```Shell
./agent.sh configure --server-url http://localhost:8111/ --name AG2
```

> * The `configure` command writes the required parameters to `.properties` file, but does not automatically create it if no such file is present. Make sure the target properties file exists when specifying a custom file location.
> * Adding and changing custom properties (for example, `./agent.sh configure --myCustomProperty=newValue`) is not supported.
> 
{style="note"}

</snippet>

## Set up Agent Behind Proxy

It is possible to configure a forward proxy server for agent-to-server connections.

<include from="configuring-proxy-server.md" element-id="agent-proxy-server"/>

<seealso>
        <category ref="concepts">
            <a href="build-agent.md">Build Agent</a>
        </category>
        <category ref="admin-guide">
            <a href="predefined-build-parameters.md">Predefined Build Parameters</a>
            <a href="configuring-agent-requirements.md">Configuring Agent Requirements</a>
            <a href="configuring-agent-pools.md">Configuring Agent Pools</a>
            <a href="configuring-build-parameters.md">Configuring Build Parameters</a>
        </category>
</seealso>
