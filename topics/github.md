[//]: # (title: Integrating TeamCity with GitHub Issues)
[//]: # (auxiliary-id: Integrating TeamCity with GitHub Issues;Integrating TeamCity with GitHub;GitHub)

[GitHub issues](https://github.com/features/issues) allow your development team to prioritize the current problems, break them down into actionable tasks, and track their statuses.

If a commit message includes a reference to an existing issue, TeamCity shows a link to this issue on the build history and [Build Results](build-results-page.md) pages.

<img src="dk-githubissues-overview.png" width="706" alt="Link to GitHub issue in TeamCity UI"/>

This integration also works for projects with configured [](pull-requests.md) feature: if a pull request mentions an existing issue, TeamCity displays corresponding data in its UI.

Note that with this integration in place, any string that matches the GitHub issue ID pattern (for example, `#054`) is treated as a reference to the corresponding GitHub issue and converted to a clickable link. For example, you may reference issues when writing a reason for [manually switching the build status](build-actions.md) to "Successful" or "Failed".

<img src="dk-ghissue-link.png" width="706" alt="Link to GitHub issue"/>

If your GitHub repository has no issue with this ID, the **Issue not found** hint is shown.

## Configure the GitHub Issues Integration

Before you start, make sure the GitHub issues feature [is enabled](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository) in your repository, and that there is at least one active issue.

1. <include from="common-templates.md" element-id="open-project-settings"/>
2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Issue Trackers"/></include>
3. Click **Create New Connection** and choose **GitHub** as the connection type.
4. Enter the public name for your integration. This name is used solely for TeamCity UI.
5. Fill in the repository URL and authentication fields. You can enter all data manually or leverage existing [GitHub connections](configuring-connections.md#GitHub).

   <table><tr><td><tabs>
   
   <tab title="Enter Data Manually">
   
   6. In the **Repository URL** field, enter the URL of your repository main page (not the clone URL). For example, `https://github.com/johndoe/my-repo` (not `https://github.com/johndoe/my-repo.git` or `git@github.com:johndoe/my-repo.git`).
   7. Choose the required **Authentication** method.
      
      * **Anonymous** authentication can be used for public repositories and issues that do not require users to log in.
      * **Access token** requires a static [personal token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) issued on GitHub.
      * **GitHub App access token** is the most secure option that leverages dynamic non-personal tokens. If the repository whose URL you specified is available via an existing [GitHub App connection](configuring-connections.md#GitHub), TeamCity will display this option. See the following article to learn more about refreshable tokens: [](manage-access-tokens.md).
   
   </tab>
   
   <tab title="Use a GitHub connection">
   
   6. Click a GitHub icon next to the the **Repository URL** field.
   
   <tip>
   
   If multiple [GitHub connections](configuring-connections.md#GitHub) exist, hover over icons to view connection names as hints.
   
   </tip>

   <note>
   
   If this is the first time you use the connection, TeamCity prompts you to Sign in to GitHub or Sign in to GitHub App. When you click the button, you are redirected to a pop-up window to authorize access to your GitHub account.
   
   </note>
   7. TeamCity will scan for repositories available through the related connection. Choose the required repository and all required options (including authentication settings) will be filled in automatically.
   
   </tab>
   
   </tabs></td></tr></table>

<ol start="8">

<li>In the Issue ID Pattern field, specify a regular expression pattern to filter the issues that belong to this project. You can usually leave this at the default setting, <code>#(\d+)</code>. See this article for more information: <a href="integrating-teamcity-with-issue-tracker.md#Converting+Strings+into+Links+to+Issues">Converting Strings into Links to Issues</a></li>

<li>Click <b>Test Connection</b> and follow the dialog instructions to test the issue tracker connection. </li>

<li>Click <b>Create</b> to save your settings and exit the setup.</li>

</ol>

## Kotlin DSL

To create a TeamCity-GitHub issue integration in [Kotlin DSL](kotlin-dsl.md), add a new `githubIssues` object to the `features` block of your project.

```Kotlin
project {
    // ...
    features {
        githubIssues {
            id = "PROJECT_EXT_17"
            displayName = "Default GH Issue Tracker"
            repositoryURL = "..."
            authType = storedToken {
                tokenId = "..."
            }
        }
    }
}
```

See also: [GitHubIssueTracker | Kotlin DSL Documentation](https://www.jetbrains.com/help/teamcity/kotlin-dsl-documentation/projectFeatures/git-hub-issue-tracker/index.html)

<seealso>
        <category ref="concepts">
            <a href="supported-platforms-and-environments.md">Supported Platforms and Environments</a>
        </category>
        <category ref="admin-guide">
            <a href="integrating-teamcity-with-issue-tracker.md">Integrating TeamCity with Issue Tracker</a>
            <a href="integrating-teamcity-with-vcs-hosting-services.md#Integrating+TeamCity+with+GitHub">Integrating TeamCity with VCS Hosting Services</a>
            <a href="configuring-connections.md#GitHub">Configuring Connections</a>
            <a href="configuring-authentication-settings.md#GitHub">Configuring Authentication Settings</a>
            <a href="commit-status-publisher.md#GitHub">Commit Status Publisher</a>
            <a href="pull-requests.md#GitHub+Pull+Requests">Pull Requests</a>
            <a href="configuring-vcs-post-commit-hooks-for-teamcity.md#GitHub+and+GitHub+Enterprise">Configuring VCS Post-Commit Hooks for TeamCity</a>
        </category>
</seealso>
