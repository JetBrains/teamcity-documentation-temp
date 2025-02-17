[//]: # (title: Configuring Schedule Triggers)
[//]: # (auxiliary-id: Configuring Schedule Triggers)

The _schedule trigger_ allows defining a schedule for automatically running builds in a given configuration. Multiple schedule triggers can be added to a single build configuration.

The __[Builds Schedule](builds-schedule.md)__ section of the current [project settings](project-administrator-guide.md#Edit+and+View+Modes) displays the configured build times.

## Triggering Conditions

This section describes the schedule trigger's parameters such as timing and VCS rules.

<anchor name="ConfiguringScheduleTriggers"/>

### Date and Time

You can schedule a recurring build or set a specific date and time for it.

In addition to triggering builds __daily__ or __weekly__ at a specified time for a particular time zone, you can specify advanced time settings using [cron](cron-expressions-in-teamcity.md)-like expressions. This format provides more flexible scheduling options.

TeamCity uses [Quartz](https://www.quartz-scheduler.org/) for working with cron expressions. See [these examples](cron-expressions-in-teamcity.md#Examples) or consider using the [CronMaker](http://www.cronmaker.com/) utility to generate expressions based on the Quartz cron format.

### VCS Changes

You can restrict a schedule trigger to start builds only if there are pending changes in your version control by enabling the _Trigger only if there are pending changes_ option. This option considers only newly detected pending changes: if there were pending changes before the trigger was created, the build is not triggered.

If the "_[Show changes from snapshot dependencies](configuring-vcs-settings.md#show-changes-from-snapshot-dependencies)_" option is enabled in the build configuration Version Control Settings, a schedule trigger will also [consider changes from dependency configurations](build-dependencies-setup.md#show-changes-from-dependencies)).

<anchor name="ConfiguringScheduleTriggers-buildTriggerRules"/>

### VCS Trigger Rules

<include from="configuring-vcs-triggers.md" element-id="vcs-trigger-rules"/>

#### General Syntax
{id="general-syntax-1"}

<include from="configuring-vcs-triggers.md" element-id="general-syntax"/>

<anchor name="ConfiguringScheduleTriggers-WatchedBuild"/>

#### Trigger Rules Examples

<include from="configuring-vcs-triggers.md" element-id="trigger-rules-examples"/>

<anchor name="ConfiguringScheduleTriggers-BuildChanges"/>

### Build Changes

A schedule trigger can watch a build in any specified build configuration and trigger a build only if the watched build has changed since the previous triggering. You can select which build to watch:

* Last finished build
* Last successful build
* Last [pinned build](build-actions.md#Pin+Build)
* Last finished build with a specified [build tag](build-actions.md#Add+Tags+to+Build)

If the trigger detects a new build that satisfies the selected characteristic in the watched configuration, it queues a new build in its own configuration.
 
The trigger watches only regular (not [personal](personal-build.md) or [history](history-build.md)) builds in the target branch. If the watched configuration has finished builds in multiple branches, the trigger settings also include the **Build branch filter** field that allows you to watch builds in the specific branch only. This setting is initially set to `+:<default>`, which means the trigger will monitor builds only in the default branch of the watched configuration.
 
If the triggered build depends on the watched build via a snapshot or artifact dependency, select the "_Promote watched build_" option so TeamCity can automatically [promote](running-custom-build.md#Promoting+Build) the detected build to the triggered build. Otherwise, the build will be triggered as usual and will have no relation to the detected build.

## Additional Options

### Enforce Clean Checkout 

Enable the _Delete all files in the checkout directory before the build_ option to force TeamCity to clean all files in the checkout directory before running a build.   
This option can also be applied to snapshot dependencies. In this case, all the builds of the build chain will be forced to use [clean checkout](clean-checkout.md). The option also enables rebuilding all dependencies (unless custom dependencies are provided via the custom build dialog or a schedule trigger promotes a build).

### Trigger Build on All Enabled and Compatible Agents

Use this option to run a build simultaneously on all agents that are enabled and compatible with the build configuration. This option may be useful in the following cases:

* run a build for agent maintenance purposes (for example, you can create a configuration to check whether agents function properly after an environment upgrade/update)
* run a build on different platforms (for example, you can set up a configuration and specify for it a number of compatible build agents with different environments installed)

<snippet id="queue-optimization">

### Build Queue Optimization Settings

By default, TeamCity [optimizes the build queue](working-with-build-queue.md#Build+Queue+Optimization+by+TeamCity): already queued build can be replaced with an already started build or a more recent queued build. You can disable this default behavior by unchecking the corresponding box.

</snippet>

### Branch Filter

By default, a schedule trigger works for all branches.

Read more in [Branch Filter](branch-filter.md).

### Trigger Rules and Branch Filter Combined

Trigger rules and branch filter are combined by __AND__, which means that the build is triggered only __when both conditions are satisfied__.

For example, if you specify a comment text in the trigger rules field and provide the branch specification, the build will be triggered only if a commit has the specified text and is also in a branch matched by the branch filter.

## Triggered Build Customization

<include from="configuring-vcs-triggers.md" element-id="triggered-build-customization"/>
