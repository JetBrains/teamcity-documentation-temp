[//]: # (title: Integrating TeamCity with YouTrack)
[//]: # (auxiliary-id: Integrating TeamCity with YouTrack;YouTrack)

You can integrate TeamCity with [JetBrains YouTrack](https://www.jetbrains.com/youtrack/) Standalone or InCloud to provide links to YouTrack issues from the TeamCity UI.

Note that TeamCity does not support the [legacy YouTrack REST API endpoints](https://blog.jetbrains.com/youtrack/2021/02/discontinuing-the-legacy-rest-api-action-required/). See [this issue](https://youtrack.jetbrains.com/issue/TW-69857) for details.

## Displaying Links to YouTrack Issues in TeamCity UI

When integration with YouTrack is enabled, TeamCity automatically detects YouTrack issue IDs mentioned in the comments of VCS commits. It transforms these IDs into links to the corresponding issues in YouTrack and displays them to TeamCity users in the UI.

To see the basic details of an issue in the TeamCity UI, open the __[Changes](build-results-page.md#Changes+Tab)__ tab of the related build’s results and hover over the icon next to the issue ID:

<img src="issue-tracker-integration.png" width="706" alt="Issue tracker integration"/>

Issues fixed in the build can be viewed on the __[Issues](build-results-page.md#Issues+Tab)__ tab of the build results:

<img src="issue-log.png" width="706" alt="Issues tab"/>

To view issues related to a whole build configuration (not only to individual builds), use the __Issue Log__ tab of the __Build Configuration Home__ page. You can filter the list to a particular range of builds and/or enable the _Show only resolved issues_ option to display only issues fixed in the builds.

<img src="build-configuration-issue-log.png" width="706" alt="Issue log"/>

When committing changes to your version control, __always mention the issue ID__ related to the fix in the comment to the commit to get the maximum benefit from the YouTrack integration.

### Configuring Connection to YouTrack

>Enabling TeamCity integration with YouTrack requires Project Administrator permissions as it is configured at a project level. Note that enabling integration for a project enables it for all its subprojects as well. If the settings are different in a subproject, they have priority over the parent project's settings.

To enable the integration:

1. <include from="common-templates.md" element-id="open-project-settings"/>
2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Issue Trackers"/></include>
3. Create a new connection with the following settings:
    * **Connection Type** — Select __YouTrack__ from the list.
    * **Display Name** — Specify the connection name to distinguish it from the other connections.
    * **Server URL** — Enter the base URL of your YouTrack instance.
    * **Authentication** — Select the authorization type you want to use to set up the integration. You can either sign in with a username and password or use a token.
    * **Username** — Enter a username of your YouTrack user account. This option is shown when __Authentication__ is set to __Username / Password__.
    * **Password** — Enter a password for your YouTrack user account. This option is shown when __Authentication__ is set to __Username / Password__.
    * **Permanent token** — Enter your [permanent token](https://www.jetbrains.com/help/youtrack/standalone/Manage-Permanent-Token.html). This option is shown when __Authentication__ is set to __Permanent Token__.
    * **Project IDs** — Enter a space-separated list of _project IDs_ to specify which strings should be recognized as references to issues in YouTrack. For example, if a project ID is `TW`, an issue ID like `TW-18802`, mentioned in a VCS comment, will be turned into a link to the corresponding issue.
        
        You can also load all project IDs automatically: enable _Use all YouTrack IDs automatically_ and test the connection to your YouTrack server. If the connection is successful, the __Project IDs__ field will be automatically populated. TeamCity will detect newly created projects in YouTrack  and automatically synchronize the list of project IDs.



Note that a user specified in the connection to YouTrack should have sufficient permissions to view YouTrack issues. This will allow TeamCity to retrieve the information about issues and display it in the UI.

## Enhancing Integration with YouTrack

YouTrack provides native TeamCity integration, which enhances the set of available features. For example:
* YouTrack is able to fill the "_Fixed in build_" field with a specific build number.
* YouTrack allows you to apply commands to issues by specifying them in a comment to a VCS change commit.

To use these features, [configure YouTrack](https://www.jetbrains.com/help/youtrack/standalone/Integration-with-TeamCity.html).

 <seealso>
        <category ref="concepts">
            <a href="supported-platforms-and-environments.md">Supported Issue Trackers</a>
        </category>
        <category ref="admin-guide">
            <a href="integrating-teamcity-with-issue-tracker.md">Integrating TeamCity with Issue Tracker</a>
        </category>
</seealso>
