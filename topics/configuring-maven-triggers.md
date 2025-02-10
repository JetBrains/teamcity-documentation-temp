[//]: # (title: Configuring Maven Triggers)
[//]: # (auxiliary-id: Configuring Maven Triggers)
<!--[//]: # (Internal note. Do not delete. "Configuring Maven Triggersd81e3.txt")-->   

The __Triggers__ page of __Build Configuration Settings__ allows you to add the following Maven-related triggers:

* [Maven Snapshot Dependency Trigger](#Maven+Snapshot+Dependency+Trigger)
* [Maven Artifact Dependency Trigger](#Maven+Artifact+Dependency+Trigger)

## Checksum Based Triggering

The trigger checks if the content of the dependency has actually changed by verifying its checksum from the repository against the locally stored version. Before triggering a build, TeamCity tries to determine the checksum of the required dependency by downloading the file digest (MD5/SHA-1) associated with that artifact.

If the checksum can be retrieved, and it matches a locally stored one, no build is triggered. If the checksum is different, a build is triggered.

If the checksum cannot be retrieved from the remote server, the dependency will be downloaded, TeamCity will calculate its checksum and follow the build triggering mechanism described above.

<anchor name="ConfiguringMavenTriggers-MavenSnapshotDependencyTrigger"/>

## Maven Snapshot Dependency Trigger

Maven snapshot dependency trigger adds a new build to the queue when there is a real modification of the snapshot dependency content in the remote repository which is detected by the checksum change.

Dependency artifacts are resolved according to the POM and the server-side [Maven Settings](maven-server-side-settings.md).

<note>

Note that since Maven deploys artifacts to remote repositories sequentially during a build, not all artifacts may be up-to-date at the moment the snapshot dependency trigger detects the first updated artifact. To avoid inconsistency, select the __Do not trigger a build if currently running builds can produce snapshot dependencies__ check box when adding this trigger, which will ensure the build won't start while builds producing snapshot dependencies are still running.
</note>


<!--[//]: # (Internal note. Do not delete. "Configuring Maven Triggersd81e53.txt")-->    

<note>

__Simultaneous usage of snapshot dependency and dependency trigger for a build__
 Assume build A depends on build B by both snapshot and trigger dependency. Then, after the build B finishes, build A will be added into the queue, only if build B is not a part of the build chain containing A.
</note>

<anchor name="ConfiguringMavenTriggers-MavenArtifactDependencyTrigger"/>

## Maven Artifact Dependency Trigger

Maven artifact dependency trigger adds build to the queue when there is a real modification of the dependency content which is detected by the checksum change.

To add a trigger, specify the following parameters in the __Add New Trigger__ dialog:

<table><tr>

<td>

Parameter

</td>

<td>

Description

</td></tr><tr>

<td>

Group ID

</td>

<td>

Specify an identifier of a group the desired Maven artifact belongs to.

</td></tr><tr>

<td>

Artifact ID

</td>

<td>

Specify the artifact's identifier.

</td></tr><tr>

<td>

Version or Version range

</td>

<td>

Specify a version or version range of the artifact. The version range syntax is described in the [section](#Version+Ranges) below. SNAPSHOT versions can also be used.

<snippet id="range-note">

<note>Maven artifact trigger focuses on the latest available artifact version that fits the range, not the entire range. This means for <code>1.3 ~ 1.5</code> version range (or the open-end <code>[1.3,)</code> range), once fired for <code>v1.5</code>, it will track further changes to this "latest" version only, ignoring changes to "older" <code>v1.4</code> and <code>v1.3</code> versions.</note>

</snippet>

</td></tr><tr>

<td>

Type


</td>

<td>

Define explicitly the type of the specified artifact. By default, the type is `jar`.

</td></tr><tr>

<td>

Classifier

</td>

<td>

(Optional) Specify the classifier of an artifact.

</td></tr><tr>

<td>

Maven repository URL

</td>

<td>

Specify a URL to the Maven repository. Note that this parameter is optional. If the URL is not specified, then:

* For a Maven project, the repository URL is determined from the POM and the server-side [Maven Settings](maven-server-side-settings.md#Maven+Settings+Resolution+on+the+Server+Side).
* For a non-Maven project, the repository URL is determined from the server-side [Maven Settings](maven-server-side-settings.md#Maven+Settings+Resolution+on+the+Server+Side) only.

</td></tr><tr>

<td>

Do not trigger a build if currently running builds can produce this artifact

</td>

<td>

Select this option to trigger a build only after the build that produces artifacts used here is finished.

</td></tr></table>

### Advanced Options

<table><tr>

<td>

Parameter

</td>

<td>

Description

</td></tr><tr>

<td>

Repository ID

</td>

<td>

Allows using authorization from the effective Maven settings

</td></tr><tr>

<td>

User settings selection

</td>

<td>

Allows selecting effective settings. The same as [User Settings](maven.md#User+Settings) of the Maven runner.

</td></tr></table>

TeamCity determines the __effective repository__ to be checked for the artifact updates and to trigger builds if changes are detected as follows:
* if a URL and Repository ID are set, authentication will be chosen from the effective settings (see below)
* if only a URL is set, the old behavior is preserved: a temporary repository ID is used (`_tc_temp_remote_repo`)
* if URL is not set (regardless of the Repository ID), the artifact will be looked up in a repository available according to the effective settings.

TeamCity determines __effective settings__ as follows:
* in the trigger settings a user can choose among the default, custom or uploaded Maven settings. See [Maven Server-Side Settings](maven-server-side-settings.md) for details.
* if no specific settings are configured for the trigger, [Maven](maven.md) build step settings are used
* if no settings for the trigger are configured and there are no Maven build steps, the `default` [server Maven settings](maven-server-side-settings.md) will be used.

### Version Ranges

For specifying version ranges use the following syntax, as [proposed in the Maven documentation](https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html).

Note that Maven Artifact Dependency Trigger can be used not only for fixed-version artifacts but also for snapshots as a fine-grained alternative to the Maven Snapshots Dependency Trigger.

<include from="configuring-maven-triggers.md" element-id="range-note"/>

<table><tr>

<td>

Range

</td>

<td>

Meaning

</td></tr><tr>

<td>

`(,1.0]`

</td>

<td>

x &lt;= 1.0

</td></tr><tr>

<td>

`1.0`

</td>

<td>

"Soft" requirement on 1.0 (just a recommendation — helps select the correct version if it matches all ranges)

</td></tr><tr>

<td>

`[1.0]`

</td>

<td>

Hard requirement on 1.0

</td></tr><tr>

<td>

`[1.2,1.3]`

</td>

<td>

1.2 &lt;= x &lt;= 1.3

</td></tr><tr>

<td>

`[1.0,2.0)`

</td>

<td>

1.0 &lt;= x &lt; 2.0

</td></tr><tr>

<td>

`[1.5,)`

</td>

<td>

x &gt;= 1.5

</td></tr><tr>

<td>

`(,1.0],[1.2,)`

</td>

<td>

x &lt;= 1.0 or x &gt;= 1.2. Multiple sets are comma-separated

</td></tr><tr>

<td>

`(,1.1),(1.1,)`

</td>

<td>

This excludes 1.1, if it is known not to work in combination with this library

</td></tr><tr>

<td>

1.0-SNAPSHOT

</td>

<td>

The trigger will check the latest snapshot version for updates

</td></tr></table>

<!--[//]: # (Internal note. Do not delete. "Configuring Maven Triggersd81e322.txt")-->

## Triggered Build Customization

<include from="configuring-vcs-triggers.md" element-id="triggered-build-customization"/>
