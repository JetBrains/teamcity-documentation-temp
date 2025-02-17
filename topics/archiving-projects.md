[//]: # (title: Archiving Projects)
[//]: # (auxiliary-id: Archiving Projects)

If a project is not in an active development state, you can _archive_ it using the __Actions__ menu in the upper right corner of the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page. On the __Archive project__ screen, you can choose whether to cancel the builds which are already in the queue using the corresponding option (enabled by default).

When _archived_:
* All the __subprojects__ of the current project __are archived__ as well.
* All project's __build configurations are__ automatically __[paused](changing-build-configuration-status.md)__.
* Automatic checking for changes in the project's [VCS roots](configuring-vcs-roots.md) is not performed if the VCS roots are not used in other non-archived projects.
* Automatic build triggering is disabled. However, builds of the project can be triggered manually or automatically as a part of a build chain.
* All data (settings, build results, artifacts, build logs, and so on) of the project's build configurations are preserved — you can still edit settings of the archived project or its build configurations.
* Archived projects do not appear in most user-facing projects lists and in IDEs including the list of build configurations for remote run.

The settings of archived projects are stored in the same [Data Directory subfolder](teamcity-data-directory.md) as settings of active projects.

By default, permissions to archive projects are given to project and system administrators.

If the parent project of an archived subproject is displayed in the project list (for example, on the __[Projects](ordering-projects-and-build-configurations.md)__ page), the archived subprojects are displayed with the corresponding note.

### Dearchiving Projects

If you need to dearchive a project, you can do it using the __Actions__ menu in the top right of the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page or the _More_ menu next to the project on the Root project's __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page.

<seealso>
        <category ref="concepts">
            <a href="project.md">Project</a>
            <a href="managing-builds.md">Build Configuration</a>
        </category>
</seealso>