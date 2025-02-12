[//]: # (title: Solve Build Problems)
[//]: # (auxiliary-id: Solve Build Problems)

Whenever you see a failed test or another problem in __[Build Results](view-build-results.md)__, you can assign it to any member of your team for further investigation. This guide explains how to:
* Assign and resolve investigations.
* Mute and unmute failed tests and build problems.
* Browse the history of investigations.

You can perform these actions from any place the problem or test is visible in the TeamCity UI. For example, if you expand the failed build in the list of builds:

<img src="problemactions.png" alt="Investigate or mute a problem"/>

Or, in the __Overview__ and __Tests__ tabs of __Build Results__. Besides, the __Tests__ tab allows you to filter and select multiple tests, so you can apply a certain action to the whole selection.

<img src="groupactions.png" width="500" alt="Investigate or mute a problem"/>

An investigator will be able to explore the build results, view the related stacktrace in the build __Overview__, and identify the cause of the problem.

## Assign Investigations and Mute Problems

>If you want to try this on your server, make sure there is at least one failed build available. You can use the sample project from the [pipeline tutorial](create-build-chain.md#Import+Sample+Project).

Let's begin with opening __[Build Results](view-build-results.md)__ of our failed build. Find the failure on the __Overview__ tab and open its context menu, as shown on the above screenshot. Select either _Assign investigation_ or _Mute problem/test_.

The investigation/mute menu is equal for both tests (from your source project) and build problems (other failures, mostly related to the build environment). It consists of four parts:
* __Project scope__: select in what projects this problem should be investigated.
* __Investigation options__: select an investigator from users of your TeamCity instance (including yourself). You can choose if this failure should be resolved automatically (that is, as soon as it's no longer reproduced during any of the next builds), or wait until the responsible developer solves its cause and marks it as fixed in this very menu.  
  This is also where you can remove the existing investigation without fixing — by selecting _No investigation_.
* __Mute options__: control how to mute this failure. A muted problem or test has a special mark in the build results and does not affect the build status. This allows temporarily taking out of the picture any complex problem that isn't critical to your builds.  
  To remove the "mute" status from a failure, open this menu and choose _Not muted_.
* __Comment__: leave any hints related to the failure.

## View Your Investigations

To see all investigations that have been assigned to you, open your user profile menu in the header and select __Investigations__. If there are any active investigations, you will see their number to the left of your username. In this case, you can simply click it, with no need to open the user menu.

<img src="accessmyinvestigations.png" alt="Open My investigations" width="300"/>

The __My Investigations__ page shows a tree of all your investigations. Here, you can manage any failure manually, or mark them all as fixed.

See how to set up [notifications](set-up-notifications.md) about new investigations, so you can learn about them as soon as possible.

## View Investigation History

You can see the investigation history of a recurring test or problem anytime. This helps identify who and when changed its status and how it could have affected the build statuses.

To see the investigation history, open the context menu of a failure and select __Show investigation history__.

Read more about investigating and muting failures in [this article](investigating-and-muting-build-failures.md).