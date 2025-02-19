# Server Administrator Guide

This section focuses on **server administration**: installing TeamCity server and agents, configuring multi-node setup, setting up user permissions, and more.

## Major Concepts

The TeamCity build system comprises a **server** and **build agents**. The following sections outline the key information about each of these components.

### Build Server

* TeamCity server __does not run either builds or tests:__ the server's job is to monitor all the connected build agents, distribute [queued builds](working-with-build-queue.md) to the agents based on compatibility requirements, and report the results.

* All information on the build results (build history and all the build-associated data except for artifacts and build logs), VCS changes,
agents, build queue, user accounts, user permissions, and so on, are stored in a database that can be hosted on a separate machine.

* TeamCity server can occupy one machine or include multiple distributed [nodes](multinode-setup.md) for an improved stability and performance.

* Similarly to agents, a server can run as a [Docker container](https://hub.docker.com/r/jetbrains/teamcity-server) rather than a permanently installed software.

### Build Agents
{id="Build+Agents+TC" instance="tc"}

* An agent is a piece of software that actually executes the build process.
It is installed and configured separately from the TeamCity server.

* Agents can be installed on both physical and virtual (cloud) machines.

    > TeamCity agents can be installed on the same machine as TeamCity server. However, for production purposes, we recommend installing them on different machines for a number of reasons, the server performance being the most important.
    >
    {style="note"}

* To ensure the smooth agent operation, you need to periodically update core software and tools of agents installed from executable files or archives. For example, after you upgrade a TeamCity server to a newer version, all cloud agents started from an existing VM image will need some time to update (this happens automatically but delays the moment your queued builds can start). To ensure your agents are always running the latest software, run them as [Docker containers](agent-docker-images.md) instead.

* Each agent can have a unique environment: architecture, operating system, installed tools, and so on. These properties define which builds an agent can run.

* Since builds can run inside [Docker or Podman containers](container-wrapper.md), the agent machine's OS alone does not limit the agent-project compatibility. In other words, you can run Linux-specific tasks on Windows agents and vice versa.




### Build Agents
{id="Build+Agents+TCC" instance="tcc"}

An agent is a piece of software that actually executes the build process.
It is installed and configured separately from the TeamCity server.

You get access to Cloud agents with your TeamCity Cloud subscription,
but you can also host agents on a physical machine.  
Build agents in TeamCity can have different platforms, operating systems, and preconfigured environments.
Different types of tests can be run on different platforms simultaneously,
so the developers get faster feedback and more reliable testing results.
{instance="tcc"}


## Database

<include from="set-up-external-database.md" element-id="intro"/>

See this section for more information: [](set-up-external-database.md)


## Multi-Node Setup

<include from="multinode-setup.md" element-id="intro"/>

See this article to learn more: [](multinode-setup.md)


## Server and Agent Health

<include from="teamcity-monitoring-and-diagnostics.md" element-id="intro"/>

For more information, refer to these topics: [](teamcity-monitoring-and-diagnostics.md), [](build-agents-configuration-and-maintenance.md)


## Security

A CI/CD server is the backbone of modern software development, holding the keys to your internal processes, user credentials, and integrations with external services like AWS build agents, Google Cloud artifact storage, or HashiCorp Vault. A single security lapse can expose critical infrastructure, making robust protection essential. See the following articles to learn more about basic layers of TeamCity protection:

* [](configuring-authentication-settings.md)
* [](https-server-settings.md)
* [](configuring-proxy-server.md)

> For security reasons, we always recommend installing [minor bug-fix updates](teamcity-release-cycle.md) that are typically released monthly. These updates ship vulnerability patches, preventing potential security-related accidents. See our [security bulletin](https://www.jetbrains.com/privacy-security/issues-fixed/) to learn more.
> 
{style="warning"}













