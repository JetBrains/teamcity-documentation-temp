# Manage Refreshable Access Tokens

In TeamCity, there are multiple entities whose functionality requires accessing VCS hosting providers: [VCS roots](configuring-vcs-roots.md), [Commit Status Publishers](commit-status-publisher.md), [Pull Requests features](pull-requests.md), and so on. These entities typically offer multiple VCS authentication options:

* Regular username/login credentials — being the most vulnerable, this authentication method is gradually deprecated by major VCS hosting providers.
* Personal access tokens (PATs) — issued on the VCS provider's side and then copied into a TeamCity object. These tokens grant the same or fewer permissions as those of the issuing user.
* Refreshable tokens — personal or non-personal tokens. To acquire this token, TeamCity communicates with a VCS provider using an existing OAuth (or OAuth-like) [connection](configuring-connections.md).

This topic explains how to issue, assign, and manage refreshable tokens.

## Supported Connections

To issue a token, TeamCity communicates with a VCS hosting provider using a connection configured in project [Connections](configuring-connections.md) settings. Currently, refreshable access tokens are supported for the following connections:

* [GitHub App](configuring-connections.md#github-app) (both regular and manifest-based connections)
* [GitLab.com and GitLab CE/EE](configuring-connections.md#GitLab)
* [](configuring-connections.md#Bitbucket+Cloud)
* [](configuring-connections.md#Bitbucket+Server+and+Data+Center)
* [Azure DevOps OAuth 2.0](configuring-connections.md#Connecting+to+Azure+DevOps)
* [JetBrains Space](configuring-connections.md#connect-to-jetbrains-space)


## Token Management

The **VCS Auth Tokens** page provides centralized access to all refreshable tokens issued for this project and its subprojects. To open this page:

1. Navigate to an individual or **Root** project.
2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="VCS Auth Tokens"/></include>

<img src="dk-vcs-auth-tokens-overview.png" width="706" alt="VCS Auth Tokens Overview"/>

> For security reasons, this page is available only for users with the Project Administrator [role](managing-roles-and-permissions.md).
> 
{style="note"}

This page allows you to find tokens by their names, related connections, and users who issued them. Without any filters applied, the page shows 10 most recently used tokens.

The **Show inherited tokens** checkbox allows you to view tokens issued in parent projects but available in the currently viewed project.

To issue a new token, click the **Generate new token**. In a dialog that pops up, specify the token name and a TeamCity connection. You can additionally limit the token scope to specific TeamCity [projects](#Project+Scope) and (for GitHub App tokens) [VCS repositories](#Repository+Scope).

<img src="dk-generate-token-dialog.png" width="464" alt="Generate new token"/>

### Project Scope

Generated tokens are by default available for the currently edited project and all of its subprojects. That is, if you create a token from the Root project's **VCS Auth Tokens** page, this token can be used by any project on the server by default. You can view projects for which a token was initially configured under the **Project Scope**.

<img src="dk-view-token-scope.png" width="706" alt="View token project scope"/>


Note that **VCS Auth Tokens** pages display only the tokens available for each project. If you create a new token for project `A` and restrict its scope to subprojects `A.2` and `A.3`, TeamCity will notify you that the current project will not have access to this token and you will not be able to see this token after the dialog closes.


### Repository Scope

[GitHub App connections](configuring-connections.md#github-app) support granular per-repository token permissions. When creating a refreshable token using this connection, you need to specify which repositories this token can access. Enter repository names without user/organization names (`myRepo` instead of `myUser/myRepo`). Note that repositories must be hosted under the same account to which a corresponding GitHub App was installed.

For security reasons, you cannot issue a token with the global repository access permission.

> The repository scope of a token can automatically expand when performing actions that involve cloning or creating new VCS roots. These actions include:
> 
> * Using the "Create subproject" wizard
> * Using the "Create build configuration" wizard
> * Copying a project
> * Moving a project
> * Copying a build configuration
> * Moving a build configuration
> * Setting up a token on a Git VCS root manually by copying a token ID
> 
> This behavior ensures a smooth user experience. For example, cloned build configurations are expected to function correctly without encountering VCS access issues.
> 
{style="note"}


## How to Create and Assign Refreshable Tokens

You do not need to manually set up refreshable tokens for each object that requires access to VCS providers. For example, when you create a new project or configuration from an existing connection (by clicking a **From &lt;Connection_name&gt;** tile on the New Project/Configuration page), TeamCity will automatically generate a token and pass it to the related VCS root.

<img src="dk-githubchecks-from-connection.png" width="706" alt="Create a project from connection"/>

Similarly, when setting up build features like [](commit-status-publisher.md), you may opt for employing default VCS root authorization method instead of issuing new tokens.

<img src="dk-publisher-share-root-creds.png" width="706" alt="Share root credentials"/>

To assign a token manually, navigate to authentication settings of a build feature or a root. Here you can generate a new or use an existing token.

<procedure title="Generate new token" type="steps">
<step>
Click the <b>Generate new</b> button. Note that similarly to the <a href="#Token+Management">VCS Auth Tokens</a> page, this button is available only to project administrators.
<img src="dk-generate-new-token.png" width="706" alt="Generate new token"/>
</step>
<step>
Enter the token name and choose a TeamCity connection that should be used to communicate with the VCS provider. The <a href="#Project+Scope">project scope</a> will be automatically limited to this project and its subprojects. 
</step>
<step>Click <b>Save</b> to exit the setup.</step>
</procedure>

<procedure title="Choose an existing token" type="steps">
<step>
Click a pencil icon to enable the <b>Token</b> text box. Note that similarly to the <a href="#Token+Management">VCS Auth Tokens</a> page, this button is available only to project administrators.
<img src="dk-edit-ref-token.png" width="706" alt="Edit button"/>
</step>
<step>
Click the <b>git token list</b> link to open the <a href="#Token+Management">VCS Auth Tokens</a> page in a new tab.
</step>
<step>
On the <b>VCS Auth Tokens</b> page, locate a token you wish to use and click the <b>Copy ID</b> button.
</step>
<step>Return back to the edited object and paste the token ID.</step>
<step>Click <b>Save</b> to exit the setup.</step>
</procedure>
