[//]: # (title: Snapshot Dependencies)
[//]: # (auxiliary-id: Snapshot Dependencies)

By setting a [snapshot dependency](dependent-build.md#Snapshot+Dependency) of a build (for example, build B) on another build's (build A) sources, you can ensure that build B will start only after build A is run and finished. We call build A a _dependency_ build, whereas build B is a _dependent_ build.

The __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes) | Dependencies__ page displays the configured dependencies; the __Snapshot Dependencies__ section of this page allows previewing the build chain and its configuration. The preview shows builds of the chain; the builds with configured automatic triggers are marked with ![v.png](v.png) icon.

A snapshot dependency has the following options:

<anchor name="EnforceRevisionsSynchronization"/>

<table><tr>

<td width="150">

Option


</td>

<td>

Description


</td></tr><tr>

<td id="dependency-build">

Depend on


</td>

<td>

Specify the build configuration to depend on.

_In our example, select build A as a dependency of the current build B._

</td></tr><tr>

<td id="enforce-rev-sync">

Enforce revisions synchronization

</td>

<td>

For all builds in a chain, which are linked by snapshot dependencies with this option __enabled__ (by default), TeamCity will use the same sources' snapshot (for example, the same revision of the same VCS root). This is a recommended setting for configurations that need to use the same state of the sources to build successfully.

If you __disable__ this option for a snapshot dependency, then, when the [dependency build](#dependency-build) is promoted to the current build configuration, the build of the current build configuration will use the most recent revision of the sources instead of the revision corresponding to the promoted dependency. This is useful when the builds do not have strict sources' dependencies (for example, as with package and deploy steps).

_In our example, if the snapshot dependency of build B has this option disabled, the behavior is following: Build A launches on revision 1.2 and, after finishing, is promoted to build B. TeamCity will find the latest revision for build B (let's say 1.3) at the moment of starting B._  
_Otherwise, if this option is enabled, TeamCity will start build B on the same 1.2 revision as A_.

Note that the sources' snapshot rule is only applied to the [parts of the builds chain](build-chain.md#Disabling+Revisions+Synchronization+Between+Chain+Parts)) linked via the snapshot dependencies with the option enabled.

</td></tr><tr>

<td>

<anchor name="do-not-run-new-build-if-there-is-a-suitable-one"/>
 
Do not run new build if there is a suitable one


</td>

<td>

If this option is enabled, TeamCity will not run a new dependency build, if another running or finished dependency build with the appropriate sources' revision already exists. See also [Suitable Builds](#Suitable+Builds).  
In this case, when a dependent build is triggered, the dependency build will also be put into the queue. Then, when the changes for the build chain are collected, this dependency build will be removed from the queue and the dependency will be set to a suitable finished build.

<note>

If there is more than one snapshot dependency on some build configuration, all of them must have the "_Do not run new build if there is a suitable one_" option enabled if you want to ensure that the build reusage works properly for all of them.

</note>

<!--[//]: # (Internal note. Do not delete. "Snapshot Dependenciesd292e62.txt")-->    

</td></tr><tr>

<td>

Only use successful builds from suitable ones


</td>

<td>

A new triggered build will only use successfully finished [suitable builds](#Suitable+Builds) as dependencies. If the latest finished suitable build fails, it will be rerun.

</td></tr><tr>

<td id="RunOnTheSameAgent">

<anchor name="SnapshotDependencies-RunOnTheSameAgent"/>

Run build on the same agent

</td>

<td>

When enabled and B snapshot-depends on A, each build B is run on the same agent where build A from the same build chain was run.

<note>

Before starting a build chain having _run on the same agent_ dependencies, TeamCity forms groups of builds combined by this dependency. For each starting build participating in such a group, TeamCity chooses agents which can be used by any build of the group. Thus, this option makes sense for [composite builds](composite-build-configuration.md) too, even though a composite build does not occupy an agent, it still can form a group of builds combined by such dependencies. For instance, composite build B having _run on the same agent_ dependencies on A and C will cause both A and C using the same agent.

</note>

</td></tr><tr>

<td id="on-failed-dependency">

On failed dependency/ On failed to start/canceled dependency

</td>

<td>


If a dependency fails, you can manage the status of the dependent build by selecting one of the following options:
* __Run build, but add problem__: the dependent build will be run and the problem will be added to it, changing its status to failed (if the problem was not muted earlier).
* __Run build, but do not add problem__: the dependent build will be run and no problem will be added.
* __Mark build as failed to start__: the dependent build will not run and will be marked as "_Failed to start_".
* __Cancel build__: the dependent build will not run and will be marked as "_Canceled_".


</td></tr></table>

<anchor name="SnapshotDependencies-SuitableBuilds"/>

### Suitable Builds
<!--[//]: # (Internal note. Do not delete. "Snapshot Dependenciesd292e145.txt")-->    

In terms of snapshot dependencies, a _suitable_ build is a build which can be used instead a queued dependency build within a [build chain](build-chain.md). That is, a queued build which is a part of a build chain can be dropped — and the builds depending on it can be made dependent on another queued, running, or already finished "suitable" build. This behavior only works when the "_Do not run new build if there is a suitable one_" option of a corresponding snapshot dependency is enabled.

For a build to be considered "suitable", it should comply with all these conditions:
* It must belong to the same or the default branch.
* It must use _the same sources' snapshot_ as the entire queued build chain being processed. If the build configurations have the same VCS settings, this means _the same sources' revision_. If the VCS settings are different (VCS roots or checkout rules), this means _the revisions taken simultaneously at some moment in time_.
* It must be successful (if the "_Only use successful builds from suitable ones_" snapshot dependency option is enabled).
* It must be a usual, not a [personal build](personal-build.md).
* It must have no customized parameters, including those set via `reverse.dep.` parameters (related feature request: [TW-23700](https://youtrack.jetbrains.com/issue/TW-23700)).
* The original build must not be triggered by the "Rebuild" action for the dependency build configuration.
* It must have no VCS settings preventing effective revision calculation, see [more  details](#VCS+Settings+Disabling+Builds+Reuse).
* There is no other build configuration snapshot-depending on the current one with the disabled "_Do not run new build if there is a suitable one_" option.
* The running build is not "hanging".
* The settings of the build configuration have not changed since the build (that is, the build was run with the current build configuration settings). This also includes no changes to the parameters of all the parent projects of the build configuration. You can check if the settings were changed between several builds by comparing `.teamcity/settings/digest.txt` file in the [hidden build's artifacts](build-artifact.md#Hidden+Artifacts).
* If there is also an artifact dependency in addition to the snapshot one, the suitable build should have artifacts. Otherwise, a build with no published artifacts and a build whose artifacts were removed during a [cleanup](teamcity-data-clean-up.md) are both equally suitable.
* All the dependency builds (the builds the current one depends on) are "suitable" and are appropriately merged.

#### VCS Settings Disabling Builds Reuse

Some settings in VCS roots can effectively disable builds reuse. These settings are:
 
* Subversion: __Checkout, but ignore changes__ mode
* CVS: __Checkout by tag__ mode
* Perforce: __Stream__ or __Client__ connection settings, or label is specified as the __Label/revision to checkout__
* Starteam: checkout mode option set to __view label__ or __promotion date__
 
<seealso>
        <category ref="concepts">
            <a href="dependent-build.md">Dependent Build</a>
        </category>
</seealso>
