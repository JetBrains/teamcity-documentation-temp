[//]: # (title: Create Build Chain)
[//]: # (auxiliary-id: Create Pipeline)

> This tutorial assumes that you have already installed and started your trial TeamCity instance as described [here](quick-setup-guide.md). We also suggest that you learn how to [run a simple build](configure-and-run-your-first-build.md).
> 
{type= "note" instance="tc"}

This tutorial assumes that you have already started your TeamCity instance. We also suggest that you learn how to [run a simple build](configure-and-run-your-first-build.md).
{type= "note" instance="tcc"}

A _build chain_ or a _pipeline_ is a sequence of consecutively run [build configurations](creating-and-editing-build-configurations.md). In TeamCity, these configurations can belong to different projects, and pass files (artifacts) from one build to another.

This tutorial walks you through setting up the following project:

<img src="buildChainSimple.png" width="674" alt="Simple build chain in TeamCity"/>

The pipeline above cycles through the following stages:

1 — Building the sample Spring Boot application
2 — Creating a Docker image for this app
3 and 4 — Running two sets of parallel tests


>You can also watch our **video guide** on how to [compose a pipeline in TeamCity](https://www.youtube.com/watch?v=p4kCMOehrqs). It shows a different example than the one described in this article.

## Import Sample Project

This tutorial uses a sample project with five separate build configurations which we are about to connect. To follow the tutorial, you can use the [sample repository](https://github.com/JetBrains/Gradle-Docker-Pipeline-TeamCity-Samples) and repeat the steps below on your TeamCity server.

To import the sample project:
1. Go to __Administration | Projects__ and click __Create project__.
2. In the _Repository URL_ field, enter the [sample repo URL](https://github.com/JetBrains/Gradle-Docker-Pipeline-TeamCity-Samples) and click __Proceed__.
3. TeamCity will detect the `.teamcity/settings.kts` file, which corresponds to a TeamCity project's settings saved in [Kotlin format](kotlin-dsl.md). Leave the default settings and proceed.

   > Choose import without synchronization when prompted. Otherwise, your project can become [non-editable via TeamCity UI](storing-project-settings-in-version-control.md#SynchronizingSettingswithVCS).
   > 
   > <img src="dk-import-wo-sync.png" width="706" alt="Import with no sync"/>
   >
   {style="note"}

4. TeamCity will import the sample project's settings and redirect you to its __General Settings__ page. Here, you can scroll a bit and see the _TodoBackend_ subproject. Click it to view all the created build configurations.

Now we can start chaining them!

## Configure Snapshot Dependency

The _TodoApp_ build configuration compiles a `.jar` application and publishes it to the `build/libs/` directory. The _TodoImage_ configuration has to build a Docker image out of this `.jar`.

To create a synchronized pipeline, or chain, you need to connect these builds with a _snapshot dependency_. We use the word _snapshot_ to describe a specific state of the project's sources, or basically a specific commit. If you connect multiple builds with snapshot dependencies, they are guaranteed to process the same sources.

>In special cases, you can create a chain where revision synchronization is disabled. Read more about advanced features of snapshot dependencies [here](snapshot-dependencies.md).

A dependency determines how one build depends on another, and thus is created in the settings of the dependent build. In our case, it's _TodoImage_. Let's go to its settings and add a snapshot dependency:
1. Open the __Dependencies__ settings tab (you might need to click __Show more__ to display this item) and click __Add new snapshot dependency__.
2. Select _TodoApp_ as a build config to depend on.
3. Leave the default settings and save the dependency.

## Configure Artifact Dependency

To pass the `.jar` from one configuration to another, we need to create an _artifact dependency_ between them. This way, when each new _TodoApp_ build finishes and produces an artifact, TeamCity will use this artifact in the following _TodoImage_ build.

To add an artifact dependency in _TodoImage_:
1. Open the __Dependencies__ settings tab and click __Add new artifact dependency__.
2. Select _TodoApp_ as a build configuration to depend on.
3. Choose to get artifacts from the build from the same chain.
4. In _Artifacts rules_, specify that we want to import the specific artifact as `todo.jar` — enter `todo.jar => build/libs/todo.jar`.  
   You can read about patterns of artifact rules and other details related to artifact dependencies [here](artifact-dependencies.md).
   <img src="chaindemo1.png" width="539" alt="Simple build chain in TeamCity"/>

>To simplify step 4 in the future, you can use the artifact browser (![popup-artifacts-tree.png](popup-artifacts-tree.png)). When there is at least one finished dependent build that already produced some artifacts, TeamCity can show them in a tree, so you can choose them in a handy way.

It is important to remember that a _build chain is a sequence of builds connected with snapshot dependencies_. Some of the builds may also be connected with artifact dependencies, but this is not a mandatory condition.

## Run Simple Chain

At this point, the first two builds are already chained together, and you can run your first chain. Note that to compose a Docker image, a [TeamCity agent](build-agent.md) needs to have [Docker](https://www.docker.com/) installed and running on its machine, so make sure to install it in advance.

When you run any build from a chain, whether it's the last one or medium one, TeamCity gathers all the other chained builds into a sequence, according to their dependencies. As you saw on our sample chain's scheme, _TodoImage_ always runs after _TodoApp_; _Test1_ and _Test2_ start only after _TodoImage_ finishes and run in parallel to each other.

Let's run the _TodoImage_ build with the __Run__ button. Go to __Project Home__ to see how TeamCity automatically runs a new _TodoApp_ build first and, after its finish, launches the following _TodoImage_ build. As a result of this chain, TeamCity will produce a Docker image according to the `Dockerfile`.

>To speed up a chain, TeamCity can reuse already finished builds instead of running new ones. Read about this optimization mechanism [here](snapshot-dependencies.md#Suitable+Builds).

To view the statuses of all chained builds, go to __Build Configuration Home__ of any chained build and open the __Dependencies__ tab of __Build Results__:

<img src="simpleBuildChain.png" width="1217" alt="Simple build chain in TeamCity"/>

The Sakura UI offers three modes of representing dependencies: _timeline_, _list_, and _chain_. When you create more advanced chains, try monitoring them with each of these modes and choose the most convenient one for your tasks.

## Configure Trigger and Checkout Rules

You already know the basics of creating chains, or pipelines, in TeamCity. However, to become effective and production-ready, a chain needs to be automated further.

### Add VCS Trigger

As we explained in the [first build guide](configure-and-run-your-first-build.md), TeamCity offers a variety of build triggers. Triggers run builds automatically if certain conditions are satisfied. The most popular type of trigger is a [VCS trigger](configuring-vcs-triggers.md), and that's the trigger we will use in this tutorial.

A VCS trigger starts a new build whenever it detects changes in the project's sources. You can define what repository and even the exact files it will monitor. Let's add a trigger in the _TodoImage_ settings:

1. Open the __Triggers__ page and click __Add new trigger__.
2. Open advanced options and then enable the option to _trigger a build on changes in snapshot dependencies_. This way, this trigger will also react to the changes relevant to the _TodoApp_ config.  
   It's often convenient to add a single trigger at the end of the chain and enable this option to consider the previous builds. Whenever you want to change the triggering settings, you will be able to do this in one place.
3. Leave other settings default and save the trigger.

<img src="vcs-trigger-settings.png" width="778" alt="Simple build chain in TeamCity"/>

Now, if you change the sample project's code, TeamCity will detect it and run the chain.

### Restrict Checkout Scope

Every chain stage is responsible for its own task. And in some cases, different build configurations need to monitor different parts of the source project. You can configure custom _checkout rules_ for a configuration, and its builds will only be triggered by changes that satisfy these rules.

For example, let's exclude `Dockerfile` from the checkout scope of _TodoApp_. This way, when you change the Docker settings, only _TodoImage_ will be triggered. Without such restrictions, TeamCity would start both of these builds per any change in the source repo, which will waste resources and could cause a mess.

You can define the scope of monitored sources in a build configuration's __Version Control Settings__:
1. Opposite our only VCS root, click __Edit checkout rules__.
2. Enter the `-:docker` rule to exclude the `docker` directory from the checkout scope. In the future, use [this syntax](vcs-checkout-rules.md) to specify these rules.
3. Save the rules.

## Complete Chain with Tests

Builds in a chain can run in parallel. Let's explore this on the example of tests.

The project's __General Settings__ list three other build configurations: _Test1_, _Test2_, and _TestReport_. According to our target scheme, _Test1_ and _Test2_ should depend on _TodoImage_, which means you need to create a snapshot dependency on it in both of these builds. If there are at least two suitable build agents on your server, TeamCity will be able to run these builds in parallel to each other; otherwise, it will start one after another.

As you might remember, our VCS trigger in _TodoImage_ considers only preceding builds (that is _TodoApp_) and won't be able to launch tests. We can add triggers in both test builds, but TeamCity provides a more straightforward option — creating an extra [composite build](composite-build-configuration.md), that is _TestReport_. A composite build can run without an agent and accumulate results of the preceding builds in a chain. Moreover, it will aggregate and report the results of _Test1_ and _Test2_ in one place. Just what we need.

So, to complete this tutorial:
1. Add snapshot dependencies from _Test1_ and _Test2_ on _TodoImage_ and from _TestReport_ on _Test1_ and _Test2_.
2. Add a VCS trigger in _TestReport_, similarly to [how we did it](#Configure+Trigger+and+Checkout+Rules) for _TodoImage_. After that, you can safely remove the trigger from _TodoImage_, as the new one will trigger the whole chain.

As _Test2_ contains a failing test, you will see that _TestReport_ will fail as well. Expand any test or build problem to quickly preview the related part of the build log.

<img src="chaindemo_failedtest.png" alt="Failed composite build"/>

The build chain mechanism in TeamCity is very flexible and designed to satisfy the needs of every project. You will also notice that build chains are much easier to monitor than scattered builds. Detailed statuses of all chained builds are displayed in the __Dependencies__ tab of __Build Results__.

Proceed with our getting started tutorials to learn about the other type of build configuration — _[deployment](deploy-build.md)_.

## Takeaway

* A build chain is a sequence of builds connected with snapshot dependencies. A snapshot corresponds to a certain commit in the source code.
* Builds in a chain can pass artifacts to each other if you configure artifact dependencies between them.
* Builds in a chain can run sequentially or in parallel. You can create chains with dozens of builds, and only the number of available build agents limits how many of them can run simultaneously.
* When any chained build is triggered, TeamCity composes and runs the whole chain from start to finish. As triggers can only consider preceding builds, it is convenient to add one VCS trigger in the very last build of a chain.
* You can limit what scopes of the source projects are relevant to each build configuration. This prevents excessive build runs.
* You can create a logical _composite_ configuration to gather the results of multiple dependency builds. Such a configuration doesn't require a build agent and only serves as an aggregator.


