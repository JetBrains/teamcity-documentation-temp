[//]: # (title: Quick Setup Guide)
[//]: # (auxiliary-id: Quick Setup Guide)

This article describes the __evaluation setup__ of a TeamCity server and one build agent on the same machine, for the most popular operating systems.

To get a general idea of how to install TeamCity for evaluation, you can watch this video tutorial, or read the full guide below.

<video src="https://youtu.be/5Akqy-vEFr0"
       title="TeamCity Installation and initial setup"/>

To install TeamCity for the production setup, read [this section](install-and-start-teamcity-server.md).

>You can also use the TeamCity [Server](https://hub.docker.com/r/jetbrains/teamcity-server/) and [Build Agent](https://hub.docker.com/r/jetbrains/teamcity-agent/) Docker images.

## Download TeamCity

__[Download](https://www.jetbrains.com/teamcity/download/) the free Professional Edition of TeamCity__, which is a full-featured TeamCity bundled with 3 build agents with a limit of 100 build configurations.  
After trial, you can __switch to the Enterprise Edition__. Read the details of our [licensing policy](licensing-policy.md) and [pricing](https://www.jetbrains.com/teamcity/buy/).

## Install and run on Windows

Run the downloaded `.exe` file and follow the instructions of the TeamCity Setup wizard. The TeamCity web server and one build agent will be installed on the same machine.

During the __installation__, you can:
* set the [TeamCity Home Directory](teamcity-home-directory.md) where TeamCity will be installed.
* choose whether the TeamCity server and agent will run as Windows services    
   <img src="installAsWinServicepng.png" alt="TeamCity server and agent as Windows services"/>
* configure ports:      
   * Server port: `8111` is the default port. Note that it can be already occupied by other applications on your machine. If this port is already in use, you can change it during the installation. In the example below, we set the port to `7777`.
   * Agent port: `9090` is the default for incoming connections from the server. If this port is already in use, you can set the `ownPort` property to a different value to change the port.   
   <img src="configure-agent-port.png" alt="Configure a build agent port" width="450"/>
     
If you have installed TeamCity as a Windows service, follow the [usual procedure](https://bit.ly/2yJF87R) of starting and stopping services.  
Otherwise, to start or stop the TeamCity server and one default agent at the same time, launch the `runAll` script via a command line. The script is located in the `<TeamCity Home>\bin` directory.

* to __start__ the server and the default agent:
    ```Shell
    .\runAll.bat start
    ```
* to __stop__ the server and the default agent:
    ```Shell
    .\runAll.bat stop
    ```

If you did not change the default port (`8111`) during the installation, the TeamCity web UI can be accessed at [`http://localhost`](http://localhost/){nullable="true"} via a web browser running on the same machine. Otherwise, use `http://localhost:<port>` (`http://localhost:7777` in our example).

## Install and run on Linux and macOS

1. Make sure you have Java 8 or 11 installed (for example, use the bundled [Amazon Corretto](https://aws.amazon.com/corretto/)).   
   >Java 8 support will be discontinued in TeamCity Server 2023.04 (to be released in April 2023). If you use a non-bundled version of Java 8, we highly recommend that you migrate your server to Java 11.
   
   Open a command-line terminal and run the following command:   
    ```Shell
    java -version
    ```
2. Make sure the `JAVA_HOME` environment variable is pointing to the Java installation directory. Run the following command in the command-line terminal:   
    ```Shell
    echo $JAVA_HOME
    ```
3. Use the `TeamCity<version number>.tar.gz` archive to unpack TeamCity bundled with the Tomcat servlet container.   
   Unpack the archive. For example, under Linux use:   
   ```Shell
   tar xfz TeamCity<version number>.tar.gz
   ```
   >The archive can be used on Windows as well.
4. After the archive is unpacked, the TeamCity web server and one build agent will be available on the current machine.

To start/stop the TeamCity server and one default agent at the same time, run the `runAll` script via a terminal. The script is located in the `<TeamCity Home>/bin` directory.

* to __start__ the server and the default agent:
    ```Shell
    ./runAll.sh start
    ```
* to __stop__ the server and the default agent:
    ```Shell
    ./runAll.sh stop
    ```

By default, TeamCity runs on [`http://localhost:8111/`](http://localhost:8111/){nullable="true"} and has one registered build agent that runs on the same machine. If another application already uses this port, the TeamCity server (Tomcat server) will not start with the "_Address already in use_" errors in the server logs or server console.

To change the server port, locate the `<TeamCity Home>/conf/server.xml` and modify the port in the `<Connector>` XML node, for example:

```Shell
<Connector port="8111" ...>

```

## TeamCity First Start

When you launch TeamCity in a browser for the first time, it will prompt you to complete the installation:
1. Review the [location of the TeamCity Data Directory](teamcity-data-directory.md#Configuring+Location), where all the configuration information is stored. Click __Proceed__.
2. TeamCity stores the build history, users, build results, and some runtime data in an SQL database and allows you to select the database type.   
   For now, keep the default internal database. Click __Proceed__.   
   <img src="default-DB.png" dark-src="default-DB_dark.png" alt="Select the database type" width="460" border-effect="line"/>
   It will take some time for TeamCity to configure the necessary components.
3. On the next screen, accept the License Agreement to proceed with the launch. Click __Continue__.
4. TeamCity displays the __Create Administrator Account__ page. Specify the administrator credentials and click __Create Account__.  

You are signed in to TeamCity: now you can configure your user settings and [create and run your first build](configure-and-run-your-first-build.md).


## Activating TeamCity

TeamCity provides a wide range of [licensing options](https://www.jetbrains.com/teamcity/buy/#on-premises) that suite teams of any size. In addition to multiple available server licenses, you can purchase additional agents to scale the performance of your build server.

Server and agent licenses can be activated on the **Administration | Licenses** page. We recommend activating your server license, even for the free Professional version, to ensure system administrators receive timely email notifications about critical TeamCity server security updates.

Learn more: [](licensing-policy.md), [](manage-teamcity-license.md)


## What's Next

After your server is up and running, you can proceed to creating your first project.

* [](project-administrator-guide.md) — learn about basic TeamCity concepts, such as build configurations, build features, triggers, and VCS roots.
* [](configure-and-run-your-first-build.md) — a step-by-step tutorial that walks you through the main stages of configuring a project.
* [](create-build-chain.md) — learn how to link separate build configurations in a single chain where multiple builds can run in parallel.
* [](user-guide.md) — explains how regular TeamCity users can run builds, inspect build results, and investigate problems.