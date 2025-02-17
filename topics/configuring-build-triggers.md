[//]: # (title: Configuring Build Triggers)
[//]: # (auxiliary-id: Configuring Build Triggers)

Once a build configuration is created, builds can be triggered manually by clicking the [Run](running-custom-build.md) button or initiated automatically with the help of _triggers_.

A _build trigger_ is a rule which initiates a new build on certain events. The build is put into the [build queue](working-with-build-queue.md) and is started when there are agents available to run it.

While creating/editing a build configuration, you can configure triggers using the __Triggers__ sections of the __[Build Configuration Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page: click __Add new trigger__ and specify the trigger settings. For configuration details on each trigger, refer to the corresponding sections. It is possible to disable a configured build trigger temporarily or permanently using the option in the last column of the trigger list.

For each build configuration the following triggers can be configured:
* [VCS trigger](configuring-vcs-triggers.md): the build is triggered when changes are detected in the version control system roots attached to the build configuration.
* [](github-checks-trigger.md): the build is triggered for every single commit pushed to the source GitHub repository. In addition, posts detailed build result info on a GitHub page without a [](commit-status-publisher.md) configured.
* [Schedule trigger](configuring-schedule-triggers.md): the build is triggered at a specified time.
* [Finish build trigger](configuring-finish-build-trigger.md): the build is triggered after a build of the selected configuration is finished.
* [Maven artifact dependency trigger](configuring-maven-triggers.md#Maven+Artifact+Dependency+Trigger): the build is triggered if there is a content modification of the specified Maven artifact which is detected by the checksum change.
* [Maven snapshot dependency trigger](configuring-maven-triggers.md#Maven+Snapshot+Dependency+Trigger): the build is triggered if there is a modification of the snapshot dependency content in the remote repository which is detected by the checksum change.
* [Retry build trigger](configuring-retry-build-trigger.md): the build is triggered if the last build failed or failed to start.
* [Branch remote run trigger](branch-remote-run-trigger.md): personal build is triggered automatically each time TeamCity detects new changes in particular branches of the VCS roots of the build configuration. Supports Git and Mercurial.
* [NuGet dependency trigger](nuget-dependency-trigger.md): starts a build if there is a NuGet package update detected in the NuGet repository.
* [Perforce shelve trigger](perforce-shelve-trigger.md): starts a build on a change in shelved Perforce files.

>Note that if you create a build configuration from a template, it inherits build triggers defined in the template, and they cannot be edited or deleted. However, you can specify additional triggers or disable a trigger permanently or temporarily.

You can also manage build triggers using [TeamCity REST API](https://www.jetbrains.com/help/teamcity/rest/manage-build-configuration-details.html#Manage+Build+Triggers).

<seealso>
    <category ref="external">
        <a href="https://youtu.be/fttWwJG7C38">Video tutorial: Improving your first build configuration</a>
    </category>
        
</seealso>

