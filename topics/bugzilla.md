[//]: # (title: Integrating TeamCity with Bugzilla)
[//]: # (auxiliary-id: Integrating TeamCity with Bugzilla;Bugzilla)

You can integrate TeamCity with [Bugzilla](https://www.bugzilla.org/) (3.0 or later) to provide links to Bugzilla issues from the TeamCity UI.

## Displaying Links to Bugzilla Issues in TeamCity UI

When integration with Bugzilla is enabled, TeamCity automatically detects issue IDs mentioned in the comments of VCS commits. It transforms these IDs into links to the corresponding issues in Bugzilla and displays them to TeamCity users in the UI:

* To see the basic details of a issue in the TeamCity UI, open the __[Changes](build-results-page.md#Changes+Tab)__ tab of the related build’s results and hover over the icon next to the issue ID.
* Issues fixed in the build can be viewed on the __[Issues](build-results-page.md#Issues+Tab)__ tab of the build results.
* To view issues related to a whole build configuration (not only to individual builds), use the __Issue Log__ tab of the __Build Configuration Home__ page. You can filter the list to a particular range of builds and/or enable the _Show only resolved issues_ option to display only issues fixed in the builds.

Follow these recommendations to get the maximum benefit from the Bugzilla integration:
* When committing changes to your version control, __always mention the issue ID__ related to the fix in the comment to the commit.
* Mark fixed issues as _Resolved_ in Bugzilla to display them with the _Fixed_ status in TeamCity logs (the time of resolve does not really matter).

### Configuring Connection to Bugzilla

>Enabling TeamCity integration with Bugzilla requires Project Administrator permissions as it is configured at a project level. Note that enabling integration for a project enables it for all its subprojects as well. If the settings are different in a subproject, they have priority over the parent project's settings.

To enable the integration, create a connection to Bugzilla on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Issue Trackers__ page and specify the following settings:

<table>

<tr>

<td>

Setting

</td>

<td>

Description

</td></tr>

<tr>

<td>

Connection Type

</td>

<td>

Select __Bugzilla__ from the list.

</td></tr><tr>

<td>

Display Name

</td>

<td>

Specify the connection name to distinguish it from the other connections.

</td></tr><tr>

<td>

Server URL

</td>

<td>

Enter the URL of your Bugzilla instance.

</td></tr><tr>

<td>

Username

</td>

<td>

Enter a username of your Bugzilla user account.

</td></tr><tr>

<td>
 
Password

</td>

<td>

Enter a password for your Bugzilla user account.

</td></tr><tr>

<td>

Issue ID Pattern

</td>

<td>

Specify a [Java Regular Expression](https://java.sun.com/j2se/1.5.0/docs/api/java/util/regex/Pattern.html) pattern to recognize an issue ID in the comment text. The matched text (or the first group if there are groups defined) is used as the issue number. The most common case is `#(\d+)` — this will extract `1234` as the issue ID from the text `Fix for #1234`.


</td></tr></table>

>If the username and password are specified, you need to have Bugzilla XML-RPC interface switched on. This is not required if you use anonymous access to Bugzilla without the username and password.
>
{style="note"}

Note that a user specified in the connection to Bugzilla should have sufficient permissions to view Bugzilla issues. This will allow TeamCity to retrieve the information about issues and display it in the UI.


## Known Issues

There are several known issues in Bugzilla regarding XMLs generated for the issues, which makes it hard to communicate with it. However, this can usually be fixed by tweaking the Bugzilla configuration.
	
* If you see the _path/to/bugzilla.dtd not found_ error, this means that the issue XML contains the relative path to the `bugzilla.dtd` file, and not the URL. To fix that, set the server URL in Bugzilla.
* Sometimes you may see a `SAXParseException` saying that _Open quote is expected for attribute type_id associated with an element type flag_. This happens because the generated XML does not correspond to the bundled `bugzilla.dtd`. To fix it, make the `type_id` attribute `#IMPLIED` (optional) in the `bugzilla.dtd` file.

<seealso>
        <category ref="concepts">
            <a href="supported-platforms-and-environments.md">Supported Platforms and Environments</a>
        </category>
        <category ref="admin-guide">
            <a href="integrating-teamcity-with-issue-tracker.md">Integrating TeamCity with Issue Tracker</a>
        </category>
</seealso>