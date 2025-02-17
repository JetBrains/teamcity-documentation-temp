[//]: # (title: Ordering Projects and Build Configurations)
[//]: # (auxiliary-id: Ordering Projects and Build Configurations)

By default, TeamCity displays projects, their subprojects, build configurations, and templates in the alphabetical order.

Project administrators can apply custom ordering to subprojects and build configurations on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes)__ page for the parent project and use it as the default order. To enable reordering, click __Reorder__ above the list of subprojects or build configurations.

Individual users can reorder the projects and change their visibility in the __Configure Visible Projects__ pop-up menu on the __Projects__ page.

To add an individual project to the __Projects__ page or remove it from this page, toggle the "eye" button on the __Project Home__ page:

<img src="eye-button.png" width="750" alt="Disk usage in details"/>

Note that if a project has [archived subprojects](archiving-projects.md) / [paused build configurations](changing-build-configuration-status.md#Pausing+Several+Build+Configurations+in+Project), they will also be displayed on the __Overview__ page and will be marked correspondingly.

<seealso>
        <category ref="admin-guide">
            <a href="kotlin-dsl.md#How+to+reorder+projects+and+build+configurations">How to reorder projects and build configurations via Kotlin DSL</a>
        </category>
</seealso>
