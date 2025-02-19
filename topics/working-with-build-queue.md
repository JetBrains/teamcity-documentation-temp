[//]: # (title: Working with Build Queue)
[//]: # (auxiliary-id: Working with Build Queue;Ordering Build Queue;Build Queue)

In TeamCity, _build queue_ is a list of builds that were [triggered](configuring-build-triggers.md), or launched manually, and are waiting to be started. TeamCity distributes them to [compatible](configuring-agent-requirements.md) build agents as soon as they become idle. A queued build is assigned to an agent at the moment it is started on the agent; no preassignment is made while the build is waiting in the build queue.

## Queue Page

Access the __Queue__ page from the top navigation bar.
This page shows the list of builds waiting to be run and displays the following information for each build:

* The position in the queue, which also serves as a link to the __Build Results__ page.
* The source branch name (if available).
* A linkable path in the hierarchy: all parent subprojects and the build configuration.
* Time to start: the estimated wait duration. Hovering over the estimated time value shows a tooltip with the following information:
    * Expected start/finish time.
    * The link to the planned agent.
    * If the current build is a part of a <tooltip term="build-chain">_build chain_</tooltip> and the builds it depends on are not finished yet, a corresponding note will be displayed. For some builds, like the builds that have never been run before, TeamCity cannot estimate possible duration, so the respective message will be displayed in the tooltip.
* A brief description of the [event that triggered the build](configuring-build-triggers.md).
* The number of agents compatible with this build configuration. You can click an agent's name link to open the __[Agents](viewing-build-agent-details.md)__ page, or use the down arrow to quickly view the list of compatible agents in the pop-up menu.

## Build Queue Optimization by TeamCity

By default, TeamCity optimizes the build queue as follows:
* If a similar build exists in the queue, a new build (on the same change set and with the same custom properties) will not be added.
* If an automatically triggered build chain has more changes than a build chain that is already queued, the latter will be replaced with the automatically triggered build chain, if such replacement will not delay obtaining the build chain results (based on the [estimated duration](#Queue+Page)).
* While a build chain is in the queue, TeamCity tries to replace the queued builds with equivalent started builds.
* Builds that have been staying in a queue for longer than 15 days are canceled automatically (for example, if there are no compatible agents).

## Agent Selection for Queued Build

When there are several idle agents that can run a queued build, TeamCity tries to select the fastest one as follows:
1. If no builds have previously run on agents, the [CPU rank](viewing-build-agent-details.md#Agent+Summary) is used to select an agent.
2. If builds have previously run on agents, the estimated build duration for the given build configuration is used to select an agent. The estimate is made based on heuristics of the latest builds in the history of the build configuration; for estimating, the execution time of the more recent builds has more weight than that of the earlier builds. [Personal](personal-build.md) and [canceled](build-state.md#Canceled%2FStopped+build) builds are not taken into account, neither are any individual builds whose duration differs significantly from the rest of the builds for this build configuration.

## Ordering Build Queue

You can:
* Reorder the builds in the queue manually.
* Remove build configurations or personal builds from the queue.
* With System Administrator permissions, assign different priorities to build configurations, which will affect their position in the queue.

### Manually Reordering Builds in Queue

To reorder builds in the build queue, you need to drag them to the desired position.

### Removing Builds from Build Queue

To remove build(s) from the queue, check the __Remove__ box next to the selected build and confirm the deletion. If a build to be removed from the queue is a part of a build chain, TeamCity shows the respective message below the comment field. Refer to the [Build Chain](build-chain.md#Stopping%2FRemoving+From+Queue+Builds+from+Build+Chain) article for details.

Additionally, you can:
* Remove all your personal builds from the queue at once from the __Actions__ menu.
* Remove several builds of [paused build configurations](changing-build-configuration-status.md#Pausing+Several+Build+Configurations+in+Project) from the queue.

### Moving Builds to Top

To move a build to the top spot in the queue, do one of the following:
* On the **Queue** page, click the arrow button next to the build sequence number.

  <img src="dk-movetotop.png" width="706" alt="Move to top"/>

* Click the build number or build status link anywhere in the UI, and, on the **[Build Results](working-with-build-results.md)** page of the queued build, click the __Actions__ menu in the upper right corner. Select the __Move to top__ action.

For a [composite build](composite-build-configuration.md), the whole build chain will be moved to the top of the queue. If a running composite build has dependency builds that have not yet started, click the build number or build status link anywhere in the UI, and, on the **[Build Results](working-with-build-results.md)** page of the running build, click the __Actions__ menu in the upper right corner. Select the __Move queued dependencies to top__ action. All queued dependencies of this build will be moved to the top of the queue.

## Managing Build Priorities
{instance="tc"}

By default, builds are placed in the build queue in the order they are triggered: the most recently triggered build is added to the bottom of the queue. It is possible to change the build's priorities so that builds are inserted into the build queue at a position depending on their defined priority and the wait time of the currently queued builds.

You can control build priorities by creating _Priority Classes_. A priority class is a set of build configurations with a specified priority (the higher the number, the higher the priority; for example, `priority=2` is higher than `priority=1`). The higher priority a configuration has, the higher place it gets when added to the build queue.

To access these settings, click __Priorities__ in the upper right corner on the __Queue__ page. Note that this action is only available to system administrators.

There are two predefined priority classes: _Personal_ and _Default_, both with `priority=0`:
* All personal builds (launched via [Remote Run](remote-run.md) or [Pre-tested Commit](pre-tested-delayed-commit.md)) are assigned to the _Personal_ priority class once they are added to the build queue. You can change the priority for personal builds.
* The _Default_ class includes all the builds not associated with any other class. This allows creating a class with priority lower than default and place some builds to the bottom of the queue.

To create a new priority class:
1. Click __Create new priority class__.
2. Specify its name, priority (in the range `-100..100`), and additional description. Click __Create__.
3. Click __Add configurations__ and specify which build configurations should have priority defined in this class.

This setting is taken into account only when a build is added to the queue. To ensure that builds with lower priority always have a chance to run, TeamCity also considers how long each build is staying in the queue. 
This allows running a build that has been waiting a long time with a lower priority before the recently added builds with higher priority. 
See the detailed explanation of the algorithm below.

### Build priority algorithm 

Every time a new build is added to the queue, the priorities of all the builds are recalculated,
and the new build is placed in the position `i`, such that
`priority of build at position i-1` \>= `priority of new build` \> `priority of build at position i+1` (`i = 0` at the top of the queue).

We use the following formula to recalculate priorities of builds in the queue:

<!--
`buildPriority = a * timeSpentInTheQueue / estimatedBuildDuration + b * buildConfigurationPriority`

where `a` and `b` are configurable coefficients ([internal properties](server-startup-properties.md#TeamCity+Internal+Properties) `teamcity.buildqueue.waitWeight` and `teamcity.buildqueue.priorityWeight` respectively) 
with the default values of `1.0`. Changing internal properties requires the server restart.
-->

`buildPriority = (timeSpentInTheQueue / estimatedBuildDuration) + buildConfigurationPriority`

So when the build waits in the queue for the amount of time that equals to the estimated build duration, 
its priority is increased by one. 
This helps the builds with low priority to start eventually.

## Pausing and Resuming Build Queue

The build queue can be paused manually or automatically. In this case, the builds are still going to be added to the queue, but they will not be assigned to agents until the queue is unpaused.

Users with the _Enable/disable agent_ permission (included in the [Agent Manager](managing-roles-and-permissions.md#Per-Project+Authorization+Mode) role by default) can manually pause/resume the build queue (since pausing the queue is equivalent to disabling all agents on the server). This action is available in the upper right corner of the __Queue__ page.
{instance="tc"}

Users with the _Enable/disable agent_ permission (included in the [Agent Manager](managing-roles-and-permissions.md#Default+User+Roles) role by default) can manually pause/resume the build queue (since pausing the queue is equivalent to disabling all agents on the server). This action is available in the upper right corner of the __Queue__ page.
{instance="tcc"}

The build queue can be paused automatically [if the TeamCity server runs out of disk space](teamcity-disk-space-watcher.md). The queue will be automatically resumed when sufficient space is available.
{instance="tc"}

When the queue is paused, every page in TeamCity will contain a message with information on the reasons for pausing.

## Limiting Maximum Size of Build Queue
{instance="tc"}

It is possible to limit the maximum number of builds in the queue. By default, the limit is 6000 builds. The default value can be changed by configuring the `teamcity.buildTriggersChecker.queueSizeLimit` [internal property](server-startup-properties.md#TeamCity+Internal+Properties).

When the queue size reaches the limit, TeamCity will pause [automatic build triggering](configuring-build-triggers.md). It will be reenabled once the queue size gets below limit. While triggering is paused, a warning message is displayed to all the users.  
However, even if the queue reached the limit and automatic triggering is paused, it is still possible to add builds to the queue [manually](running-custom-build.md).

 <seealso>
        <category ref="concepts">
            <a href="build-chain.md">Build Chain</a>
        </category>
</seealso>
