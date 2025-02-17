[//]: # (title: Integrating TeamCity with Issue Tracker)
[//]: # (auxiliary-id: Integrating TeamCity with Issue Tracker)

TeamCity can be integrated with your issue tracker to provide a comprehensive view of your development project. TeamCity detects issues mentioned in the comments to version control changes, turning them into links to your issue tracker in the TeamCity UI.

The integration is configured at the project level: the Project Administrator permissions are required. You can configure integration if you have multiple projects on both the TeamCity and the issue tracker server or if you are using different issue trackers for different projects.

Enabling integration for the project also enables it for all its subprojects; if the configuration settings are different in a subproject, its settings have priority over the project's settings.

## Dedicated Support for Issue Trackers

TeamCity supports [Jira](jira.md), [Bugzilla](bugzilla.md), [YouTrack](youtrack.md), [GitHub](github.md), [GitLab](gitlab.md), [Bitbucket (Cloud, Server, Data Center)](bitbucket-cloud.md), and Azure DevOps Server (formerly [TFS](azure-board-work-items.md)) trackers out of the box. The [Supported Platforms and Environments](supported-platforms-and-environments.md#Issue+Trackers) page lists supported versions.

When an integration is configured, TeamCity automatically transforms an issue ID (=issue key in Jira, work item ID in Azure DevOps Server) mentioned in the VCS commit comment into a link to the corresponding issue, and the basic issue details are displayed in the TeamCity web UI when hovering over the icon next to the issue ID (for example, on the __[Changes](build-results-page.md#Changes+Tab)__ tab of Build Results).

<img src="issue-tracker-integration.png" width="661" alt="Issue tracker integration"/>

Issues fixed in the build can also be viewed on the __[Issues](build-results-page.md#Issues+Tab)__ tab of the build results. You can filter the list to a particular range of builds and view issues mentioned in comments with their states.

<img src="issue-log.png" width="750" alt="Issue log"/>

Since TeamCity 2020.1, [integration with Jira Cloud](jira.md) also allows previewing build statuses directly in Jira.

## Recommendations on Using Issue Tracker Integration

To get maximum benefit from the issue tracker integration, do the following:
* When committing changes to your version control, __always mention the issue ID (issue key)__ related to the fix in the comment to the commit.
* Resolve issues when they are fixed (the time of resolve does not really matter).
* Use __Issue Log__ of a build configuration to get issues related to builds; turn on the "_Show only resolved issues_" option to only display the issues fixed in the builds.

## Enabling Issue Tracker Integration

<anchor name="requirements"/>

### Requirements

The information about the issues is retrieved by the TeamCity server using the credentials provided and then is displayed to TeamCity users.

This has several implications:
* The TeamCity server needs direct access to the issue tracker.
* The user configured in the connection to the issue tracker has to have sufficient permissions to view the issues that can be mentioned in TeamCity. Also, TeamCity users will be able to view the issue details in TeamCity for all the issues that the configured user has access to.

### Configuring connection

To enable integration, you need to create a connection to your issue tracker on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Issue Trackers__ page.

The settings described below are common for all issue trackers:

<table>

<tr>

<td></td>
<td></td>

</tr>

<tr>

<td>

__Connection type__

</td>

<td>

Select the type of your issue tracker from the list.

</td></tr><tr>

<td>

__Display name__

</td>

<td>

A symbolic name that will be displayed in the TeamCity UI for the issue tracker.

</td></tr><tr>

<td>

__Server URL (Repository URL)__

</td>

<td>

The issue tracker URL

</td></tr><tr>

<td>

__Username/Password (Authentication)__

</td>

<td>

Credentials to log in to the issue tracker, if it requires authorization.

</td></tr></table>

Additional authentication information or/and the details on how to specify strings to be recognized by TeamCity and converted to your tracker issue links is provided in the corresponding sections:
* [YouTrack](youtrack.md)
* [Jira](jira.md)
* [Bugzilla](bugzilla.md)
* [GitHub](github.md)
* [GitLab](gitlab.md)
* [Bitbucket](bitbucket-cloud.md)
* [TFS](azure-board-work-items.md)

### Converting Strings into Links to Issues

In addition to general settings, you need to specify which strings should be recognized as references to issues in your tracker.

For __Jira__, you need to provide a space-separated list of [project keys](https://confluence.atlassian.com/display/JIRA044/What+is+a+Project). You can also load all project keys automatically: check the corresponding box and test the connection to your Jira server. If the connection is successful, the __Project keys__ field will be automatically populated. Newly created projects in Jira will be detected by TeamCity, and the project keys list will be automatically synchronized.   
For example, if a project key is `WEB`, an issue key like `WEB-101`, mentioned in a VCS comment, will be resolved to a link to the corresponding issue.

For __YouTrack__, you need to provide a [permanent token](https://www.jetbrains.com/help/youtrack/incloud/authentication-with-permanent-token.html) for authentication and a space-separated list of __Project IDs__. You can also load all project IDs automatically: check _Use all YouTrack IDs automatically_ and test the connection to your YouTrack server. If the connection is successful, the __Project IDs__ field will be automatically populated. Newly created projects in YouTrack will be detected by TeamCity, and the project ID list will be automatically synchronized.  
For example, if a project ID is `TW`, an issue ID like `TW-18802` mentioned in a VCS comment will be resolved to a link to the corresponding issue.
 
For __Bugzilla__, __GitHub__, __GitLab__, and __Bitbucket Cloud__, you need to specify the __Issue ID Pattern__: a [Java Regular Expression](https://java.sun.com/j2se/1.5.0/docs/api/java/util/regex/Pattern.html) pattern to find the issue ID in the text. The matched text (or the first group if there are groups defined) is used as the issue number. The most common case is `#(\d+)` — this will extract `1234` as the issue ID from the text `Fix for #1234`.

TeamCity will resolve the issue number mentioned in a VCS comment and will display a link to this issue in the UI (for example, on the __[Changes](build-results-page.md#Changes+Tab)__ page, or the __[Issues](build-results-page.md#Issues+Tab)__ tab of __[Build Results](working-with-build-results.md)__).

## Integrating TeamCity with Other Issue Trackers
{instance="tc"}

To integrate TeamCity with other issue trackers, configure TeamCity to turn any issue tracker issue ID mentions in change comments into links. See [mapping external links in comments](mapping-external-links-in-comments.md) for details.

Dedicated support for an issue tracker can also be added via a custom [issue tracker integration plugin](https://plugins.jetbrains.com/docs/teamcity/issue-tracker-integration-plugin.html).


<seealso>
        <category ref="concepts">
            <a href="supported-platforms-and-environments.md#Issue+Trackers">Supported Issue Trackers</a>
        </category>
        <category ref="admin-guide">
            <a href="https://www.jetbrains.com/help/teamcity/mapping-external-links-in-comments.html">Mapping External Links in Comments</a>
        </category>
        <category ref="external">
            <a href="https://plugins.jetbrains.com/docs/teamcity/issue-tracker-integration-plugin.html">Developing TeamCity Plugins: Issue Tracker Integration Plugin</a>
        </category>
</seealso>

