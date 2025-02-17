[//]: # (title: Perforce Workspace Handling in TeamCity)
[//]: # (auxiliary-id: Perforce Workspace Handling in TeamCity)

To perform Perforce-related operations, TeamCity usually operates in a "no-workspace" mode, that is it executes Perforce commands without the workspace context. For instance, workspaces are not required for tracking changes or for most server-side operations.

The cases when a workspace is created are:
* If [agent-side checkout](vcs-checkout-mode.md#agent-checkout) is enabled (it is the default checkout mode). In this case, TeamCity creates a Perforce workspace to check out the build sources.
* Using [versioned project settings](storing-project-settings-in-version-control.md) with Perforce Helix Core.
* Using [Perforce streams as feature branches](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Streams). In this case, TeamCity creates workspaces on the Perforce server to correctly process task streams.

## Perforce Workspace Name

The names of the created workspaces start with the `TC_p4_` prefix. To support [feature branches](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Streams), workspaces created on the Perforce server side have the `TC_p4_server_` prefix.

If your build configurations use the [agent-side checkout](vcs-checkout-mode.md#agent-checkout), you can modify a workspace name as follows:

* Add a workspace name prefix via the `teamcity.perforce.workspace.prefix` [configuration parameter](configuring-build-parameters.md).
* Set a custom workspace name in case your workflow requires a specific workspace name pattern. To do this, set the `vcsroot.<VCSRootExternalID>.p4client` parameter. If a custom workspace name is specified, TeamCity updates the workspace according to corresponding VCS root settings (for example, applies related stream parameters if the VCS root has the stream support enabled) before the checkout. See also: [](#Reuse+Checked+Out+Sources+on+Cloud+Agents).

> You can set properties' values during a build by sending the [setParameter Service Message](service-messages.md#set-parameter). To send these messages before the checkout, send them from a [Bootstrap step](https://youtrack.jetbrains.com/issue/TW-14646/Ability-to-run-custom-task-before-a-branch-is-checked-out-on-agent-bootstrap-steps).
> 
{style="tip"}


The workspace name also includes the build agent name and a hash value built from the checkout directory and (optionally) checkout rules.

## Perforce Workspace Parameters

With [agent-side checkout](vcs-checkout-mode.md#agent-checkout), TeamCity provides environment variables describing the Perforce workspace created during the checkout process.

If multiple Perforce VCS roots are used for the checkout, the variables are created for the __first__ VCS root in the list of the build's VCS roots:
* `P4USER` — same as `vcsroot.<VCS_root_ID>.user` [parameter](predefined-build-parameters.md#VCS+Parameters).
* `P4PORT` — same as `vcsroot.<VCS_root_ID>.port` [parameter](predefined-build-parameters.md#VCS+Parameters).
* `P4CLIENT` — same as `vcsroot.<VCS_root_ID>.p4client` [parameter](predefined-build-parameters.md#VCS+Parameters), the name of the generated P4 workspace on the agent.

These variables can be used to perform custom Perforce commands after the checkout. For example, to be able to access the port of the `PerforceTest` VCS root, define the `env.P4PORT=%\vcsRoot.PerforceTest.port%` [environment variable](configuring-build-parameters.md#Environment+Variables) in the project or build configuration settings.

## Workspace Deletion

### Deleting Workspace on Build Agent

TeamCity deletes the Perforce workspaces in the following scenarios:
* Immediately after a versioned settings commit (a workspace is created for each commit).
* For the agent-side checkout — when a [clean checkout](clean-checkout.md) is performed (TeamCity will also run `p4 sync -f` in this case, see details [below](#Perforce+Sync+-f+and+Workspace+Reuse)).
* In the background of the agent process (between builds), when it detects a non-existing workspace directory for a workspace associated with the current agent. A TeamCity agent performs a clean-up of unused [checkout directories](build-checkout-directory.md) (the default timeout is 8 days, can be changed with the `system.teamcity.build.checkoutDir.expireHours` [system property](configuring-build-parameters.md)). When a checkout directory is deleted, and this directory is associated with a Perforce workspace, this workspace is deleted as well. Cleaning Perforce workspaces can be disabled via the `teamcity.perforce.workspace.cleanup=false` setting, either in the [`buildAgent.properties`](configure-agent-installation.md) file or globally at the server level as a Root project [configuration parameter](configuring-build-parameters.md).

When deleting a Perforce workspace which contains pending changes or opened files, TeamCity tries to revert the changes and remove pending changelists, and after that repeats the operation. If the second attempt fails as well, TeamCity tries to run the `p4 client -d -f`operation (forced). All those actions are logged to `teamcity-vcs.log`.

TeamCity does not force workspace deletion when a Perforce edge/replica server is used.

### Cleaning Workspaces on Perforce Server

If you enable the [feature branches support](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Streams) in a Perforce VCS root, TeamCity will start processing your Perforce task streams. To do this correctly, it needs to create dedicated workspaces on the Perforce server. Over time, these workspaces might consume a significant amount of resources on this server's machine. You can clean no longer necessary workspaces directly from the TeamCity UI.

<anchor name="perforce-admin-access"/>

To establish direct access to your Perforce server, go to __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Connections__ in TeamCity and add a _Perforce Administrator Access_ connection. In its settings, enter the host and user credentials for accessing the Perforce server (the user must have the [`admin` permission](https://www.perforce.com/manuals/p4sag/Content/P4SAG/protections.set.html#protections.set.access_levels)), and TeamCity will connect to it.
{id="P4AdminAccessConnection" help-id="P4AdminAccessConnection"}

During every [clean-up](teamcity-data-clean-up.md), TeamCity will detect and delete workspaces that have been inactive for more than 7 days. You can also delete them anytime by clicking _Delete workspaces_ right in the connection settings. Note that workspaces are deleted only on the server — not on build agents — and only if they were created by TeamCity.

It is also possible to delete only workspaces associated with a specific stream. To do this, go to __Build Configuration Home__, open the __Actions__ menu, and click _Delete Perforce stream workspaces_. By default, this action is available to all users with the Project Developer role.  
In the opened dialog, specify a path to a stream, and TeamCity will delete the related workspaces on the Perforce server. To connect to the server, TeamCity will use the settings from the project's Perforce Administrator Access connection. If it is not available, it will use the [Perforce VCS root](perforce.md) settings instead.
{id="DeleteP4StreamWorkspaces" help-id="DeleteP4StreamWorkspaces"}

## Perforce Sync -f and Workspace Reuse

When the [agent-side checkout](vcs-checkout-mode.md#agent-checkout) is used, TeamCity creates a workspace which is bound to the checkout directory on an agent. The checkout is performed with an incremental `p4 sync` command (both for personal and non-personal builds).

When a VCS root is configured to use `p4 sync -p`, TeamCity always runs this command to check out the sources.

Usually, each [clean checkout](clean-checkout.md) build results in `p4 sync -f` command with cleaning the sources. For the Perforce agent checkout, there are exceptions described below.

### Errors During Checkout

When an error occurs during a checkout, or a build is interrupted/stopped during the checkout, or a timeout occurs, no [clean checkout](clean-checkout.md) will occur for the subsequent builds on the same build agent. Instead, TeamCity will rely on the Perforce ability to recover from the state. 

### VCS Root Client Mapping Modification
{instance="tc"}

Usually, when a project administrator modifies a VCS root client mapping specified in the VCS root, this is considered a change in the VCS root settings and results in a [clean checkout](clean-checkout.md). This clean checkout behaviour can be disabled using the `teamcity.perforce.enable-no-clean-checkout=true` [internal property](server-startup-properties.md#TeamCity+Internal+Properties).

Changing the `teamcity.perforce.enable-no-clean-checkout` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) results in one-time clean checkout for all affected build configurations.

When a VCS root is configured to use Client Name, or Stream, no clean checkout will occur when the client mapping of the corresponding client/stream is edited in Perforce.

[//]: # (Internal note. Do not delete. https://youtrack.jetbrains.com/issue/TW-25344)

### Forced Protection Against Clean Checkout

To protect a build configuration from the TeamCity-initiated clean checkout, you can set the `teamcity.agent.failBuildOnCleanCheckout` [configuration parameter](configuring-build-parameters.md) to `true`. In this case, TeamCity will fail a build instead of running a clean checkout. It will never clean the workspace, unless it is explicitly requested via the _[Enforce clean checkout](clean-checkout.md#Enforcing+Clean+Checkout)_ action or if the "_Delete all files in the checkout directory before the build_" option is enabled in the checkout options of the version control settings for a build configuration.

When using a custom checkout path, TeamCity will not clean the checkout directory when VCS settings are changed — it will fail a build instead. To ignore clean checkout and proceed with incremental checkout, use the `teamcity.agent.failBuildOnCleanCheckout=ignoreAndContinue` [parameter](configuring-build-parameters.md) for a project or build configuration. Do this only if you are absolutely sure that the sources in the checkout directory are in the correct state.

The same applies to the broken personal builds. When the sources are corrupted and this option is set, TeamCity will fail the build instead of running a clean checkout. You can clean the working copy via `p4 clean` and try to continue with the `ignoreAndContinue` value after this (to run a custom build with the specified [configuration parameter](configuring-build-parameters.md)).

[//]: # (Internal note. Do not delete. https://youtrack.jetbrains.com/issue/TW-33168)

## Reuse Checked Out Sources on Cloud Agents

To avoid clean checkouts on each new TeamCity [cloud agent](teamcity-integration-with-cloud-solutions.md), you can copy or mount a persistent storage with code sources to the agent checkout directory. However, since Perforce keeps track of workspaces, agent IP addresses and names, revisions stored on agents, and other data, running a build on a new cloud agent causes the `p4 sync` operation to ignore existing source files and result in a clean checkout.

To prevent this from happening and reuse sources from a persistent storage, do the following:

1. Add the `teamcity.agent.failBuildOnCleanCheckout=ignoreAndContinue` parameter to your build configuration to explicitly [disable clean checkouts](perforce-workspace-handling-in-teamcity.md#Forced+Protection+Against+Clean+Checkout).

2. To make adjustments to a workspace before the checkout starts, enable [Bootstrap steps](https://youtrack.jetbrains.com/issue/TW-14646/Ability-to-run-custom-task-before-a-branch-is-checked-out-on-agent-bootstrap-steps).

3. Add one or multiple command-line or script steps to your configuration and check their **Run during bootstrap** option. These bootstap steps should do the following:
    
   * ensure that the build checkout directory points to your persistent storage and this storage has all required sources checked out at the correct revision;
   * run the `p4 -c <p4_workspace_name> flush <changelist_revision>` command to tell the Perforce server a workspace already has all required sources;
   * set the custom workspace name to the required value. To do that, send the [setParameter Service Message](service-messages.md#set-parameter) that assigns a new value to the [`vcsroot.<VCSRootExternalID>.p4client`](#Perforce+Workspace+Name) property.
        
       ```Shell
       echo "##teamcity[setParameter name='vcsroot.P4_ExternalVCSRootID.p4client' value='customP4ClientName']"
       ```
       
       > Since TeamCity updates a workspace according to settings of a corresponding VCS Root, make sure the VCS Root settings have the same [Client Mapping](perforce.md#Map+Perforce+Depot+to+Client) as `customP4ClientName` workspace in your Service Message.
       > 
       {style="warning"}

With these bootstrap steps in place, TeamCity updates your P4 client specification, runs the `p4 sync` command and performs a checkout that should fetch only changes between the flushed `<changelist_revision>` and the revision associated with the current build.