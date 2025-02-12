[//]: # (title: Multinode Setup for High Availability)
[//]: # (auxiliary-id: Multinode Setup for High Availability;Multinode Setup)

<snippet id="intro">
The TeamCity server can be configured to use multiple nodes (or servers) for high availability and flexible load distribution. It is possible to set up a cluster of TeamCity nodes, where each node is responsible for different tasks, like processing data from builds or collecting changes from VCS repositories. Or, to keep one main node that does all the work and a secondary node that provides a read-only interface. In case the main node goes down, all data processing can be switched to the secondary node with minimum downtime.
</snippet>

As the main use case of a multinode setup is to achieve high availability (HA), this article focuses on configuring a HA cluster. However, you can use the same methods for any setup with multiple TeamCity nodes: for example, to distribute load between several machines.

>The instructions are provided for TeamCity 2021.1 or later. If you use an earlier version, please refer to its [respective documentation](documentation-for-previous-versions.md).
>
{style="note"}

## Main vs. Secondary Node

A TeamCity cluster can have one _main node_ and multiple _secondary nodes_. The main node is the preferred one and, by default, it receives all incoming HTTP requests. It also performs critical background tasks, such as starting builds. A secondary node mostly serves as a backup server, which is needed for failover. For better load distribution and performance optimization, it can also be granted [additional responsibilities](#Responsibilities).

<img src="multinode-setup-2022.png" width="600" alt="TeamCity setup with two nodes"/>

## High-Availability Setup

### Prerequisites

A basic HA setup must include the following components:
* __Dedicated database server__: an external database server from the list of [supported databases](supported-platforms-and-environments.md#Databases).
* __Dedicated server for the TeamCity data directory__: the [data directory](#Shared+Data+Directory) should be shared with nodes by network, for example using NFS or SMB.
* __At least two servers for TeamCity nodes__ with the mounted [shared data directory](#Shared+Data+Directory): both servers should have the same or comparable hardware. Otherwise, if a secondary node is less performant, you can experience a significant performance drop in case of a failover.
* __Local storage on the TeamCity nodes__: necessary for the TeamCity installation and storing logs and local caches. To estimate the size of caches, see the size of the`<TeamCity data directory>/system/caches` directory in your current  installation.
* __Dedicated reverse HTTP proxy server__.

The minimum number of server machines necessary for a high-availability setup is 5: a database, a network storage server, two TeamCity nodes, and one server for the reverse HTTP proxy. A simpler load-balancing solution might be achieved with fewer machines.

>All nodes that use the same data directory share the same license key. If you own a TeamCity Enterprise license, no additional purchases are required.
>
{style="note"}

### Shared Data Directory

The main TeamCity node and secondary nodes must be able to access and share the same [TeamCity Data Directory](teamcity-data-directory.md).

Here are the main recommendations on setting up the shared Data Directory:
* For a high-availability setup, store it on a separate and well-performing machine, so it is accessible even when the main node goes down.
* All TeamCity nodes’ machines should be able to access it in the read\/write modes.
* The typical Data Directory mounting options are SMB and NFS. TeamCity uses the Data Directory as a regular file system so all basic file system operations should be supported.
* The I/O operations count or I/O volume limits should not be restricted by the storage or mounting option.
* Make sure to review performance guidelines for your storage solution. For example, increasing MTU for the network connection between the server and the storage usually increases the artifact transfer speed.
<anchor name="Disable-Network-Client-Caches-on-Data-Directory-Mounts"/>
* **Disable Network Client Caches on Data Directory Mounts**
  * It is important that all the nodes see the current state of the shared Data Directory without delay. If this is not the case, it is likely to result in unstable behavior and frequent build log corruptions.
  * If TeamCity nodes run on Windows with Data Directory shared via SMB protocol, make sure all the registry keys mentioned in [this article](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-7/ff686200(v=ws.10)) are set to 0 on all the TeamCity nodes.
  * If the Data Directory is shared via NFS, make sure all nodes have the following option in their mount settings: `lookupcache=positive`.

### Required JVM Options

TeamCity server nodes require you to configure the following JVM options:

<dl>
<dt>teamcity.server.nodeId=&lt;node_ID&gt;
</dt>
<dd>Unique identifier of the TeamCity node, which is used in the UI and configuration files.
</dd>

<dt>teamcity.server.rootURL=&lt;node_root_URL&gt;
</dt>
<dd>Root URL for the TeamCity node, which usually has the form <code>http://&lt;node_hostname&gt;</code>. This URL is required for inter-node communication and a firewall should be configured to allow connections to this URL from one node to another.
</dd>

<dt>teamcity.data.path=&lt;TeamCity Data Directory&gt;
</dt>
<dd><p><i>(Optional)</i> Path to the mount point of a shared <a href="teamcity-data-directory.md">TeamCity Data Directory</a>.</p>

> You can choose between setting this option or setting the <code>TEAMCITY_DATA_PATH</code> environment variable.
>
{style="note"}
</dd>

<dt>teamcity.node.data.path=&lt;Node-specific Data Directory&gt;
</dt>
<dd>Path to the node-specific TeamCity Data Directory, which is the local storage on each node for node-specific configuration and caches.
</dd>

<dt>teamcity.server.responsibilities=&lt;responsibilities_list&gt;
</dt>
<dd><p><i>(Optional)</i> Comma-separated list of <a href="#Setting+Responsibilities+on+the+Command+Line">responsibilities</a> to assign to this node.</p>

> You can choose between setting this option or manually updating the responsibilities from the UI or REST API.
>
{style="note"}
</dd>

</dl>

The JVM options can be configured using the `TEAMCITY_SERVER_OPTS` environment variable, whose value is a space-separated list of settings in the format `-D<JVMOption>=<Value>`.

>Paths like `<TeamCity Data Directory>` or `<Node-specific data directory>` can be different on each node. To keep the setup simple, we recommend that you keep them the same.
> 
{style="tip"}

### Configuring HA Setup

This section relies on the following assumptions:
* All the [system requirements](system-requirements.md) are met.
* [TeamCity is installed](install-and-start-teamcity-server.md) on both nodes.
* [TeamCity Data Directory](teamcity-data-directory.md) and database are already initialized.

Now, we can proceed with the transition from a single server setup to a high-availability cluster setup.

To configure a TeamCity cluster consisting of two nodes, follow these steps:
1. Check that the version of the TeamCity server installed on both nodes is the same and corresponds to the version of the Data Directory.
2. Select an ID for each of the nodes: for example, a short ID based on a hostname.
3. Create the `TEAMCITY_SERVER_OPTS` environment variable on each node. This variable should have the following arguments:
    ```
    -Dteamcity.server.nodeId=<node_ID> -Dteamcity.server.rootURL=<node_root_URL> -Dteamcity.data.path=<TeamCity Data Directory> -Dteamcity.node.data.path=<Node-specific Data Directory>
    ```
    >You can also specify a path to the shared TeamCity Data Directory via the `TEAMCITY_DATA_PATH` environment variable.
    > 
    {style="tip"}
    
    >Make sure the database is configured to accept enough parallel connections to handle requests from both nodes. By default, each node requires 50 connections to the database.
    >
    {style="tip"}
4. Start both nodes using [regular TeamCity scripts](start-teamcity-server.md) or via the TeamCity service.
5. Open the TeamCity __Administration | Nodes Configuration__ page on any of the two servers and enable the _[Main TeamCity node](#Main+Node+Responsibility)_ responsibility for a node you want to make main.
6. Proceed with [configuring the reverse HTTP proxy](#Proxy+Configuration).

<anchor name="MultinodeSetup-ProxyConfiguration"/>

## Proxy Configuration
{id="Proxy+Configuration" help-id="Proxy+Configuration" instance="tc"}

The reverse HTTP proxy serves as a single endpoint for TeamCity users and for [build agents](build-agent.md). This is also a good place to configure HTTPS connection settings for the entire cluster.

When used with TeamCity, a proxy server also acts as a load balancer for incoming requests. It can determine where the request should be sent and route it to the corresponding upstream server. In this article, we provide examples of the proxy configuration for the most popular proxy servers: NGINX, NGINX Plus and HAProxy.

<tabs>

<tab title="HAProxy">

```
defaults
    mode http
    timeout connect 240s
    timeout client 1200s
    timeout server 1200s

frontend http-in
    bind *:80

    stats enable
    stats uri /healthz
    
    default_backend web_endpoint
    option httplog
    log /dev/log local0 info

    # Uncomment if logging to stdout is desired (e.g. when running in a containerized environment)
    #log stdout local0  info

    option http-buffer-request
    declare capture request len 40000000
    http-request capture req.body id 0
    capture request header user-agent len 150
    capture request header Host len 15

    capture cookie X-TeamCity-Node-Id-Cookie= len 100

    http-request add-header X-TeamCity-Proxy "type=haproxy; version=2023.05"
    http-request set-header X-Forwarded-Host %[req.hdr(Host)]

    acl node_id_cookie_found req.cook(X-TeamCity-Node-Id-Cookie) -m found
    acl browser req.hdr(User-Agent) -m sub Mozilla

    default_backend clients_not_supporting_cookies
    use_backend clients_with_node_id_cookie if node_id_cookie_found
    use_backend clients_supporting_cookies if browser

backend clients_with_node_id_cookie
    # this backend handles the clients that provided the "X-TeamCity-Node-Id-Cookie" cookie
    # clients that do so are TeamCity agents and browsers handling HTTP requests asking to switch to a specific node 
    cookie X-TeamCity-Node-Id-Cookie
  
    http-request disable-l7-retry if METH_POST METH_PUT METH_DELETE
    retry-on empty-response conn-failure response-timeout 502 503 504
    retries 5
   
    option httpchk GET /healthCheck/ready
  
    default-server check fall 6 inter 10000 downinter 5000
   
    server NODE1 {node1_hostname} cookie {node1_id}
    server NODE2 {node2_hostname} cookie {node2_id}

backend clients_supporting_cookies
    # this backend is for the browsers without "X-TeamCity-Node-Id-Cookie"
    # these requests will be served in a round-robin manner to a healthy server
    balance roundrobin
    option redispatch
    cookie TCSESSIONID prefix nocache

    http-request disable-l7-retry if METH_POST METH_PUT METH_DELETE
    
    option httpchk

    http-check connect
    http-check send meth GET uri /healthCheck/preferredNodeStatus
    http-check expect status 200

    default-server check fall 6 inter 10000 downinter 5000 on-marked-down shutdown-sessions

    server NODE1 {node1_hostname} cookie n1 weight 50
    server NODE2 {node1_hostname} cookie n2 weight 50

backend clients_not_supporting_cookies
    # for compatibiity reasons requests from non browser clients are always 
    # routed to a single node (the first healthy) 
    balance first
    option redispatch

    http-request disable-l7-retry if METH_POST METH_PUT METH_DELETE

    option httpchk

    http-check connect
    http-check send meth GET uri /healthCheck/preferredNodeStatus
    http-check expect status 200

    default-server check fall 6 inter 10000 downinter 5000 on-marked-down shutdown-sessions

    server NODE1 {node1_hostname} 
    server NODE2 {node2_hostname} 
```
</tab>

<tab title="NGINX Plus">

```
events {
    worker_connections 10000;
}

http {

   upstream round_robin {
       zone round_robin 1m;
       server {node1_hostname};
       server {node2_hostname};
       sticky cookie X-TeamCity-RoundRobin-Cookie path=/;
   }
   
   upstream first_available {
       zone first_available 1m;
       server {node1_hostname} weight=100;
       server {node2_hostname} weight=1;
   }
   
   upstream sticky_route {
       zone sticky_route 1m;
       server {node1_hostname} route={node1_id};
       server {node2_hostname} route={node2_id};
       sticky route $node_id;
   }
   
   map $http_user_agent $browser {
       default 0;
       "~*Mozilla*" 1;
   }
   
   map $http_cookie $node_id_cookie {
       default 0;
       "~*X-TeamCity-Node-Id-Cookie" 1;
   }
   
   map "$browser$node_id_cookie" $backend {
       00 @clients_not_supporting_cookies;
       10 @clients_supporting_cookies;
       01 @clients_with_node_id_cookie;
       11 @clients_with_node_id_cookie;
   }
   
   map $http_cookie $node_id {
       default '';
       "~*X-TeamCity-Node-Id-Cookie=(?<node_name>[^;]+)" $node_name;
   }
   
   map $http_upgrade $connection_upgrade { # WebSocket support
       default upgrade;
       '' '';
   }

   proxy_read_timeout     1200;
   proxy_connect_timeout  240;
   client_max_body_size   0;    # maximum size of an HTTP request. 0 allows uploading large artifacts to TeamCity

   server {
      listen        80;
      server_name   {proxy_server_hostname};
      status_zone   status_page;
      
      set $proxy_header_host $host;
      set $proxy_descr "type=nginx_plus; version=2023.05";
      
      location / {
         try_files /dev/null $backend;
      }
      
      location @clients_with_node_id_cookie {
         # this backend handles the clients which provided the cookie with name "X-TeamCity-Node-Id-Cookie"
         # such clients are TeamCity agents and browsers handling HTTP requests asking to switch to a specific node 
         proxy_pass http://sticky_route;
         health_check uri=/healthCheck/ready;
         
         proxy_next_upstream error timeout http_503 http_502 non_idempotent;
         proxy_intercept_errors on;
         proxy_set_header Host $host:$server_port;
         proxy_redirect off;
         proxy_set_header X-TeamCity-Proxy $proxy_descr;
         proxy_set_header X-Forwarded-Host $http_host; # necessary for proper absolute redirects and TeamCity CSRF check
         proxy_set_header X-Forwarded-Proto $scheme;
         proxy_set_header X-Forwarded-For $remote_addr;
         proxy_set_header Upgrade $http_upgrade; # WebSocket support
         proxy_set_header Connection $connection_upgrade; # WebSocket support
      }
      
      location @clients_supporting_cookies {
         # this backend is for the browsers without "X-TeamCity-Node-Id-Cookie"
         # these requests will be served in a round-robin manner to a healthy server
         proxy_pass http://round_robin;
         health_check uri=/healthCheck/preferredNodeStatus;
         
         proxy_next_upstream error timeout http_503 http_502 non_idempotent;
         proxy_intercept_errors on;
         proxy_set_header Host $host:$server_port;
         proxy_redirect off;
         proxy_set_header X-TeamCity-Proxy $proxy_descr;
         proxy_set_header X-Forwarded-Host $http_host; # necessary for proper absolute redirects and TeamCity CSRF check
         proxy_set_header X-Forwarded-Proto $scheme;
         proxy_set_header X-Forwarded-For $remote_addr;
         proxy_set_header Upgrade $http_upgrade; # WebSocket support
         proxy_set_header Connection $connection_upgrade; # WebSocket support
      }
      
      location @clients_not_supporting_cookies {
         # for compatibiity reasons requests from non browser clients are always 
         # routed to a single node (the first healthy) 
         proxy_pass http://first_available;
         health_check uri=/healthCheck/preferredNodeStatus;
         
         proxy_next_upstream error timeout http_503 http_502 non_idempotent;
         proxy_intercept_errors on;
         proxy_set_header Host $host:$server_port;
         proxy_redirect off;
         proxy_set_header X-TeamCity-Proxy $proxy_descr;
         proxy_set_header X-Forwarded-Host $http_host; # necessary for proper absolute redirects and TeamCity CSRF check
         proxy_set_header X-Forwarded-Proto $scheme;
         proxy_set_header X-Forwarded-For $remote_addr;
         proxy_set_header Upgrade $http_upgrade; # WebSocket support
         proxy_set_header Connection $connection_upgrade; # WebSocket support
      }
   }
}
```
</tab>

<tab title="NGINX">

```
events {
    worker_connections 10000;
}

http {
   upstream {main_node_id} {
       server {main_node_hostname};
       server {secondary_node_hostname} backup;
   }
   
   upstream {secondary_node_id} {
       server {secondary_node_hostname};
       server {main_node_hostname} backup;
   }
   
   upstream web_requests {
       server {main_node_hostname};
       server {secondary_node_hostname} backup;
   }
   
   map $http_cookie $backend_cookie {
       default "{main_node_id}";
       "~*X-TeamCity-Node-Id-Cookie=(?<node_name>[^;]+)" $node_name;
   }
   
   map $http_user_agent $is_agent {
       default @users;
       "~*TeamCity Agent*" @agents;
   }
      
   map $http_upgrade $connection_upgrade { # WebSocket support
      default upgrade;
      '' '';
   }   
   
   proxy_read_timeout     1200;
   proxy_connect_timeout  240;
   client_max_body_size   0;    # maximum size of an HTTP request. 0 allows uploading large artifacts to TeamCity
   
   server {
     listen        80;
     server_name   {proxy_server_hostname};
   
     set $proxy_header_host $host; 
     set $proxy_descr "type=nginx; version=2023.05";

     location / {
        try_files /dev/null $is_agent;
     }
       
     location @agents {
        proxy_pass http://$backend_cookie;
        proxy_next_upstream error timeout http_503 non_idempotent;
        proxy_intercept_errors on;
        proxy_set_header Host $host:$server_port;
        proxy_redirect off;
        proxy_set_header X-TeamCity-Proxy $proxy_descr;
        proxy_set_header X-Forwarded-Host $http_host; # necessary for proper absolute redirects and TeamCity CSRF check
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Upgrade $http_upgrade; # WebSocket support
        proxy_set_header Connection $connection_upgrade; # WebSocket support
     }
   
     location @users {
        proxy_pass http://web_requests;
        proxy_next_upstream error timeout http_503 non_idempotent;
        proxy_intercept_errors on;
        proxy_set_header Host $host:$server_port;
        proxy_redirect off;
        proxy_set_header X-TeamCity-Proxy $proxy_descr;
        proxy_set_header X-Forwarded-Host $http_host; # necessary for proper absolute redirects and TeamCity CSRF check
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Upgrade $http_upgrade; # WebSocket support
        proxy_set_header Connection $connection_upgrade; # WebSocket support
     }
   }
}

```
</tab>
</tabs>

> The sample configs above have multiple placeholders (for instance, `{node1_hostname}`). These placeholders must be replaced with your actual values.
>
{style="warning"}



> * Since users will use the proxy server URL to access the TeamCity UI, this URL must be set as the **Server URL** on the __Administration | Global Settings__ page and inside [build agents' configurations](configure-agent-installation.md).
> * The configurations above are designed for the two-node TeamCity setup. You can add more nodes if necessary.
> * Nodes without the *Handling UI actions and load balancing user requests* [responsibility](#Responsibilities) will report HTTP error 503 to health check requests sent from your reverse proxy server. TeamCity categorizes these proxy requests as user operations, which these nodes cannot handle. This behavior is normal, does not affect node functionality, and does not indicate any malfunction.
>
{style="note"}


#### Choosing the Right Proxy Server

TeamCity can work with different types of proxy servers. However, **HAProxy** and **NGINX Plus** are preferable since these servers support active health checks and sticky sessions. These features are essential for the [round-robin](#Round-Robin) of user requests among different nodes (supported by TeamCity 2023.05+).

In comparison, the regular **NGINX** proxy with standard modules lacks these features and cannot be used to configure the round-robin.

Additionally, the regular NGINX requires you to explicitly distinguish main and secondary nodes in the configuration file (the main node should be first in the list). This requirement forces you to manually update node roles when a main node transfers its responsibilities to a secondary node (for example, in case of a failover). HAProxy and NGINX Plus servers do not require you to update configuration files in the same scenario manually.

Finally, configuration files of **HAProxy** and **NGINX Plus** proxy servers are easier to maintain, especially when adding new nodes to the cluster. 

#### Matching Proxy Version with Server

Sample [proxy configurations](#Proxy+Configuration) above set a special `X-TeamCity-Proxy` header. This header notifies TeamCity that a request comes through a properly configured proxy.

The `X-TeamCity-Proxy` header also defines a version of the proxy config. If this version is incompatible with the current TeamCity version, TeamCity shows the _Proxy configuration version mismatch_ [health report](server-health.md). Check your proxy settings and ensure they are similar to the configuration illustrated in the [](#Proxy+Configuration) section.

### Round-Robin

Starting with version 2023.05, the main TeamCity node and every secondary node configured with the [Handling UI actions and load balancing user requests](#Handling+UI+Actions+and+Load+Balancing+User+Requests) responsibility participate in a round-robin.


The proxy randomly routes the first browser request to any of participating nodes. Subsequent HTTP requests are sent to the same node (sticky session). This behavior distributes the load produced by user requests among different nodes, potentially allowing TeamCity to handle more simultaneous HTTP requests. In addition, round-robins improve user experience in case of a [failover](#Failover) or a planned node restart. For instance, if a single node should be restarted, only users assigned to this node will be affected. And as soon as the proxy server detects that the node is no longer available, all these users will be distributed among other nodes.


If a request must be handled by one specific node rather than randomly assigned to any node, use the node selector in the TeamCity UI footer or add the `__nodeId=<id of a node>` request parameter to the request query string.

To add or remove a node to/from the round-robin list, change the state of the
[Handling UI actions and load balancing user requests](#Handling+UI+Actions+and+Load+Balancing+User+Requests) responsibility. No changes in the proxy configuration are required.

> Since the round-robin distribution logic utilizes sticky sessions, we recommend HAProxy and NGINX Plus reverse proxy servers that support this feature. See this section to learn more: [](#Choosing+the+Right+Proxy+Server).
> 
{style="note"}



### Domain Isolation Proxy Configuration

The [domain isolation](teamcity-configuration-and-maintenance.md#artifacts-domain-isolation) mode requires configuring a dedicated domain for serving artifacts. Make sure to add a new record in your DNS pointing from this domain to the proxy address: for example, it could be a `CNAME` record pointing to the proxy URL.

## Failover

In the case of a failover, when the main node is no longer available either because of a crash or during maintenance, a secondary node can be granted with the _Main TeamCity node_ responsibility and act as a main node temporarily or permanently.

Each secondary node tracks the activity of the current main node and, if the main node is inactive for several (3 by default) minutes, shows the corresponding health report.
The _Main TeamCity Node_ responsibility can be reassigned to another node via the TeamCity UI or REST API. However, if this inactivity has not been planned (that is, the main node has crashed), it is important to verify that no TeamCity server processes are left running on the inactive node. If you detect such processes, you need to stop them before reassigning the _Main TeamCity node_ responsibility.

If you use the standard NGINX proxy server, update node IDs and hostnames in the [reverse proxy configuration](#Proxy+Configuration) and reload the proxy server configuration after switching the _Main TeamCity Node_ responsibility. HAProxy and NGINX Plus proxy servers do not require manual configuration updates.

To sum up, follow these steps in case of a failover:
1. Wait for a server health report about the main node unavailability on the secondary node.
2. Make sure the main node is stopped.
3. Switch the _Main node_ responsibility on the secondary node.
4. (if regular NGINX is used) Update the main node ID in the reverse proxy configuration and reload the config.

### Monitoring and Managing Nodes via REST API

Starting from TeamCity 2022.10, you can use the REST API to check the node status and reassign node responsibilities.

To get the list of all online nodes, use:

```Shell
GET /app/rest/server/nodes?locator=state:online
```

To assign the _Main node_ responsibility to a secondary node, use:

```Shell
PUT /app/rest/server/nodes/id:<node id>/enabledResponsibilities/MAIN_NODE
```

The same example with curl:

```Shell
curl \
   -X PUT \
   -H "Content-Type: text/plain" \
   -H "Origin: <host>:<port>" \
   --data-raw "true" \
   --header “Authorization: Bearer <token-value>” \
   "https://<host>:<port>/app/rest/server/nodes/id:<node id>/enabledResponsibilities/MAIN_NODE"
```

> The MAIN_NODE responsibility can be reassigned only if the current main node is offline. Otherwise the request will fail. Note that the responsibility change will not be immediate. To get effective responsibilities of the node, use the following REST API call:

```Shell
GET /app/rest/server/nodes/id:<node id>/effectiveResponsibilities
```

<anchor name="Nodes+Configuration+and+Usage"/>

## Node Configuration

### Authentication and License

The main and secondary nodes operate under the same license.

Secondary nodes use the same authentication settings as the main node. However, users might be asked to login again after they are routed to a secondary node. This happens in two cases: (1) a user did not select the Remember Me option on the login screen and (2) [SSO authentication](configuring-authentication-settings.md) is not configured.

### Global Settings

Secondary nodes use the same global settings (path to artifacts, version control settings, and so on) as the main node.

### Responsibilities

By default, a newly started secondary node provides a read-only user interface and does not perform any background activity. You can assign the following additional responsibilities to each secondary node in __Administration | Server Administration | Nodes Configuration__:

* [Processing data produced by builds](#Processing+Data+Produced+by+Builds+on+Secondary+Node)
* [VCS repositories polling](#VCS+Repositories+Polling+on+Secondary+Node)
* [Processing build triggers](#Processing+Triggers+on+Secondary+Node)
* [Handling UI actions and load balancing user requests](#Handling+UI+Actions+and+Load+Balancing+User+Requests)
* [Main TeamCity node](#Main+Node+Responsibility)

To allow users to perform the most common actions on builds and agents, enable the [Handling UI actions and load balancing user requests](#Handling+UI+Actions+and+Load+Balancing+User+Requests) responsibility.

You can enable and disable responsibilities for nodes at any moment.

#### Processing Data Produced by Builds on Secondary Node

It is possible to use one or more secondary nodes to process traffic from the TeamCity agents. This allows moving the related load to a separate machine from the main TeamCity node which will improve the TeamCity performance when handling hundreds of concurrent and actively logging builds.

In general, you do not need a separate node for running builds unless you have more than 400 agents connected to a single node. Using a secondary node allows you to significantly increase the number of agents which the setup can handle.

Once you assign a secondary node to the _Processing data produced by builds_ responsibility for the first time, all newly started builds will be routed to this node. The existing running builds will continue being executed on the main node. When you disable the responsibility, only the newly started builds will be switched to the main node. The builds that were already running on the secondary node will continue running there.  
If you assign more than one secondary nodes to this responsibility, builds will be distributed equally between these nodes.

> You can control how many builds can be run by each node.  
>To do this, find the required node in the list of available nodes and click __Edit__  next to its _Processing data produced by running builds_ responsibility. The _Limit builds_ dialog will open. Here, you can enter a relative limit of builds allowed to run on this node. We suggest that you choose this limit according to the node's hardware capabilities.
>
{style="tip"}

> If a main TeamCity node is down, agents automatically reconnect to a secondary node. However, to avoid excessive configuration updates when the main node is only temporarily unavailable (for instance, due to the node restart or a rare network issue), builds assigned to this main node are not immediately relayed to a secondary node. A secondary node waits for 10 minutes before taking over these builds.
> 
{style="note"}


#### VCS Repositories Polling on Secondary Node

Initially, only the main TeamCity node polls VCS repositories for new commits. Enabling the _VCS Repositories Polling_ responsibility on multiple nodes allows you to distribute this potentially slow activity and reduce the latency for starting new builds.

Commit hooks configured on your main node do not require any changes and remain functional after you delegate the VCS polling to secondary node(s).


#### Processing Triggers on Secondary Node

In setups with many build agents, a significant amount of the main node's CPU is allocated to constant processing of build triggers. By enabling the _Processing build trigger_ responsibility for one or more secondary nodes, you can distribute the trigger processing tasks and CPU load between the main node and the responsible secondary ones. TeamCity distributes the triggers automatically but you can see what triggers are currently assigned to each node.

#### Handling UI Actions and Load Balancing User Requests

This responsibility is responsible for allowing user actions on a secondary node. It is especially useful when the main node is down or goes through maintenance.

> Nodes without this responsibility will report HTTP error 503 to health check requests sent from your reverse proxy server. TeamCity categorizes these proxy requests as user operations, which these nodes cannot handle. This behavior is normal, does not affect node functionality, and does not indicate any malfunction.
> 
{style="note"}

Enabling of this responsibility also adds the node to the list of the nodes participating in [round-robin](#Round-Robin). 

#### Main Node Responsibility

You can assign a secondary node to the _Main TeamCity node_ responsibility. This responsibility by default belongs to the current main node, but gets vacant if this node becomes unavailable. After you assign any secondary node to this responsibility, it becomes the main node and in addition receives the _Handling UI Actions and Load Balancing User Requests_ responsibility. All the running builds will continue their operations without interruption. If a [proxy is configured](#Proxy+Configuration) in your setup, build agents will seamlessly reconnect to the new main node.  
When the previous main node starts again, it becomes a secondary node, as the _Main TeamCity node_ responsibility is already occupied by another node. If necessary, you can repeat the procedure above to switch roles between these nodes.

### Setting Responsibilities on the Command Line

You can define a server's responsibilities with the following JVM option:

```
teamcity.server.responsibilities=<responsibilities_list>
```

In the server's environment, add the JVM option to the `TEAMCITY_SERVER_OPTS` environment variable, specifying the responsibilities as a comma-separated list. For example:

```Shell
-Dteamcity.server.responsibilities=CAN_PROCESS_BUILD_MESSAGES,CAN_CHECK_FOR_CHANGES,CAN_PROCESS_BUILD_TRIGGERS,CAN_PROCESS_USER_DATA_MODIFICATION_REQUESTS

```

The following responsibilities can be enabled:

<table>
<tr>
<th>Identifier
</th>
<th>Responsibility
</th>
</tr>

<tr>
<td><code>CAN_PROCESS_BUILD_MESSAGES</code>
</td>
<td><a href="#Processing+Data+Produced+by+Builds+on+Secondary+Node">Processing data produced by builds</a>
</td>
</tr>

<tr>
<td><code>CAN_CHECK_FOR_CHANGES</code>
</td>
<td><a href="#VCS+Repositories+Polling+on+Secondary+Node">VCS repositories polling</a>
</td>
</tr>

<tr>
<td><code>CAN_PROCESS_BUILD_TRIGGERS</code>
</td>
<td><a href="#Processing+Triggers+on+Secondary+Node">Processing build triggers</a>
</td>
</tr>

<tr>
<td>

<code>CAN_PROCESS_USER_DATA_MODIFICATION_REQUESTS</code>

</td>
<td><a href="#Handling+UI+Actions+and+Load+Balancing+User+Requests">Handling UI actions and load balancing user requests</a>
</td>
</tr>

<tr>
<td><code>MAIN_NODE</code>
</td>
<td><a href="#Main+Node+Responsibility">Main TeamCity node</a>
</td>
</tr>


</table>

Note the following limitations that apply when a server's responsibilities are set on the command line:

* It is not possible to specify the *max percent of builds to process* or the *triggers assigned to each node* from the command line. These responsibility settings are available only from the UI or the REST API.
* You can edit the server's responsibilities using the UI or the REST API if the `CAN_PROCESS_USER_DATA_MODIFICATION_REQUESTS` responsibility is enabled, but only when you are directly connected to the server. Moreover, these changes are only temporary (they will be reset when the node restarts).

> When a server's responsibilities are set on the command line, it is generally recommended not to edit the responsibilities using the UI, because any manual changes will be reset when the node restarts.
>
{style="note"}


### Internal Properties

All TeamCity nodes rely both on common internal properties, stored in the [shared data directory](#Shared+Data+Directory), and specific properties, stored in the node-specific data directory. The node-specific properties have a higher priority and overwrite the common values in case of a conflict.

To disable any common property on a given secondary node, pass it using the following syntax: `-<property_name>`.

### Secondary Node Memory Settings

The secondary node requires the same memory settings as the main node. If you have already configured the `TEAMCITY_SERVER_MEM_OPTS` [environment variable](server-startup-properties.md) for the main node, make sure to use the same variable for the secondary node. If your main node uses a 64-bit JVM, the secondary node must use a 64-bit JVM as well.

### Environment Variables

The following environment variables can be used to configure a TeamCity server node:

<table>
<tr>
<th>
Variable
</th>
<th>
Description
</th>
</tr>

<tr>
<td>
<code>TEAMCITY_DATA_PATH</code>
</td>
<td>
Path to the shared <a href="teamcity-data-directory.md">TeamCity Data Directory</a>.
</td>
</tr>

<tr>
<td>
<code>TEAMCITY_SERVER_OPTS</code>
</td>
<td>
<i>(Required)</i> Server JVM options (see <a href="#Required+JVM+Options">Required JVM Options</a>).
</td>
</tr>

<tr>
<td>
<code>TEAMCITY_SERVER_MEM_OPTS</code>
</td>
<td>
Server JVM memory options (see <a href="server-startup-properties.md#JVM+Options">JVM Options</a>).
</td>
</tr>

</table>


## Managing the Nodes

### Project Import

You can import projects to the main node only. Secondary nodes will detect the imported data in the runtime, without restarting.

### Clean-up

The TeamCity clean-up task can be scheduled on any TeamCity node, but it executes on the main node only. In a multi-node configuration, as well as in a single node configuration, the task can run while secondary nodes are handling their operations.

### Using Plugins

A secondary node has access to all plugins enabled on the main node. It also watches for the newly uploaded plugins. If a secondary node detects that a plugin has been uploaded to the main node, it will show the respective notification on its __Administration | Plugins__ page. If the plugin supports it, it can be reloaded in runtime with no need to restart the secondary node — the respective hint will be displayed in the UI.

>Secondary nodes can load an external/non-bundled plugin if this plugin supports secondary nodes. See the [Plugins F.A.Q.](https://plugins.jetbrains.com/docs/teamcity/plugin-development-faq.html#How+to+adapt+plugin+for+secondary+node) for details.
>
{style="note"}

<anchor name="Installing+Additional+Secondary+Node"/>

### Automated Addition of a Secondary Node

You can spin up an instance of a secondary node with a minimum of manual intervention (for example, in a script), by taking advantage of the TeamCity server [environment variables](#Environment+Variables), which provide the basic configuration of the secondary node.

To create a secondary node in a manner that is suitable for automation, follow these steps:

1. Prepare the TeamCity server software, either by downloading and installing a distribution package, or by pulling a suitable Docker image. Make sure that the version of TeamCity software is the same as the version running on the other nodes.
2. Mount the shared TeamCity Data Directory (for example, using NFS or SMB).
3. Provide the path to the shared TeamCity Data Directory by setting the `TEAMCITY_DATA_PATH` environment variable.
4. Provide the JVM options for the node by setting the `TEAMCITY_SERVER_OPTS` environment variable with a space-separated list of JVM options, as follows:
   ```
   TEAMCITY_SERVER_OPTS = -Dteamcity.server.nodeId=<node_ID> -Dteamcity.server.rootURL=<node_URL> -Dteamcity.node.data.path=<Node-specific Data Directory> -Dteamcity.server.responsibilities=<responsibilities_list>
   ```
   Where:
   * `<node_ID>` is the unique ID of the node, which should be generated specifically for this node.
   * `<node_URL>` is the secondary node root URL, which should be accessible from the main node.
   * `<Node-specific Data Directory>` is the path to the node-specific TeamCity Data Directory.
   * `<responsibilities_list>` is the comma-separated [list of responsibilities](#Setting+Responsibilities+on+the+Command+Line) to enable on this node.
   > In the case of automated creation, all of the secondary nodes should be configured with the same responsiblities.
   > The `MAIN_NODE` responsibility must not be included for a secondary node.
   >
   {style="tip"}
5. If required, specify the JVM memory options by setting the `TEAMCITY_SERVER_MEM_OPTS` environment variable.
6. Start the TeamCity server using the [regular TeamCity scripts](start-teamcity-server.md) or using the TeamCity service or by running the TeamCity server docker container.


### Upgrade/Downgrade

It is recommended that the main TeamCity node and all secondary nodes have the same version. In certain cases, the main node and secondary nodes can be running different versions for a short period, for example, during the minor upgrade of the main node. When the versions of a secondary node and the main node are different, the corresponding health report will be displayed on both nodes.

When __upgrading to a minor version__ (a bugfix release), the main and the secondary nodes should be running without issues as the TeamCity data format stays the same. You can upgrade the main TeamCity node and then the secondary nodes [manually](upgrading-teamcity-server-and-agents.md), or with the [automatic update](upgrading-teamcity-server-and-agents.md#Automatic+Update).

When __upgrading the main node to a major version__, its TeamCity data format changes. Because of that, as soon as the main node of a new version starts, the secondary nodes detect the main node has a different data format and switch to the read-only mode. It may take some time for the secondary nodes to become read-only. During this time the main node will wait until it ensures that all the other nodes do not change data anymore. 

To __upgrade__ nodes in a multinode setup to a major version of TeamCity, follow these steps:
1. [Upgrade](upgrading-teamcity-server-and-agents.md) the main TeamCity node. The procedure of ugrading notes is identical to updating standalone servers.
2. Verify that everything works properly and agents are connecting to the main node (the agents will reroute the data that was supposed to be routed to the secondary nodes to the main node).
3. Perform upgrade of TeamCity on the secondary nodes to the same version.

To __downgrade__ nodes in a multinode setup, follow these steps:
1. Shutdown the main node and the secondary nodes.
2. [Restore the data](restoring-teamcity-data-from-backup.md) from backup (only if the data format has been changed during the upgrade).
3. Downgrade the TeamCity software on the main node.
4. Start the main TeamCity node and verify that everything works properly.
5. Downgrade the TeamCity software on the secondary nodes to the same version as the main node.
6. Start the secondary nodes.

TeamCity agents will perform upgrade/downgrade automatically.

### Start/Stop

Any TeamCity node can be started/stopped using regular TeamCity scripts (`teamcity-server.bat` or `teamcity-server.sh`) or Windows services. The environment variables [`TEAMCITY_DATA_PATH`], [`TEAMCITY_SERVER_MEM_OPTS`], and [`TEAMCITY_SERVER_OPTS`] are supported by all types of nodes.

All nodes use the same approach to logging events. You can check the state of the startup in the `<TeamCity Home Directory>/logs/teamcity-server.log` file. Or, you can open `<node root URL>` in your browser to see the TeamCity startup screens.

A secondary node, as well as the main node, can be stopped or restarted while the builds are running. They will continue running on agents and either will be reassigned to another node by the proxy, or will wait until their appointed node starts again.

### Backup/Restore

You can start backup on any node [from the TeamCity UI](creating-backup-from-teamcity-web-ui.md) that has the [Handling UI actions and load balancing user requests](#Handling+UI+Actions+and+Load+Balancing+User+Requests) responsibility. You can also start backup on any node [from a command line](creating-backup-via-maintaindb-command-line-tool.md). While a backup runs, all other nodes do not allow scheduling cleanup, importing projects, or starting another backup process.

The [restore operation](restoring-teamcity-data-from-backup.md) can be done on either of the nodes, but only if all nodes using the TeamCity database and data directory are stopped.

>Currently, the contents of the `<Node-specific data directory>` are not included in the backup.
>
{style="note"}


## Multinode Setup Health Reports

See the related reports in the [dedicated article](server-health.md#Multinode+Setup+Misconfiguration).

## Common Issues

### Trouble Accessing Data Directory from Windows Service

Note that if TeamCity is running as a service on Windows, it might not be able to access the TeamCity Data Directory via a mapped network drive. This happens because Windows services cannot work with mapped network drives, and TeamCity does not support the UNC format (`\\host\directory`) for the Data Directory path. To work around this problem, you can use [`mklink`](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mklink) as it allows making a symbolic link on a network drive:

```Console

mklink /d "C:\<path to mount point>" "\\<host>\<shared directory name>\"

```

Make sure remote-to-local symbolic link evaluations are enabled in your OS:

```Console

fsutil behavior query SymlinkEvaluation

```

To enable them, use the following command:

```Console

fsutil behavior set SymlinkEvaluation R2L:1

```
