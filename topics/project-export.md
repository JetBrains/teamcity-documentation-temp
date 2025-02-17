[//]: # (title: Project Export)
[//]: # (auxiliary-id: Project Export)

It is possible to export settings of a project with its children to an archive to later import it to a different TeamCity server. Export includes settings (basically everything configured in the project administration area), but does not include builds or any other data visible in the user area. To export a project with all the related data, use [server backup](teamcity-data-backup.md).

## Exporting Project Settings

To export the project settings, perform the following: 
1. Go to __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__, from the __Actions__ menu in the upper right corner select __Export Project__: <img src="export1.png" width="1322" alt="Export a project"/>
2. The __Settings Export__ page is displayed allowing exporting the project and viewing all its dependencies. Click __Export__ to download a ZIP archive containing project settings. <img src="export2.png" width="1307" alt="Project export settings"/>

The user exporting the project settings must have the "_View build configuration settings_" permission granted to the project developer role by default. External dependencies are exported only if the user has the required permission there, otherwise a warning will be shown before export.

## Export Scope

The export scope of a project contains only this project's settings, including [private SSH keys](ssh-keys-management.md) uploaded by the project administrator.

External dependencies for build configurations are exported as well. A build configuration defined in one project can depend on other projects in a number of ways. It can:
* be associated with a [template](build-configuration-template.md) defined in the [parent projects](project.md#Settings+Propagation);
* use [VCS roots](vcs-root.md) or an [SSH key](ssh-keys-management.md) defined higher in the [project hierarchy](project.md#Project+Hierarchy);
* use some external build configuration as a snapshot dependency.

The `report.log` file included in the archive details reasons for exporting external entities. 

## Export Limitations

* Builds, changes, and other project-related data cannot be exported.
* External build configurations that are used in artifact dependencies only are not exported.