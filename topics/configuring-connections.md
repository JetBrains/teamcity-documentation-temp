[//]: # (title: Configuring Connections)
[//]: # (auxiliary-id: Configuring Connections)

TeamCity allows storing presets of connections to external services. You can reuse these presets in various places on the server: when creating projects, configuring notifications, integrating with issue trackers, and more. This article gives instructions on how to add each type of connection.

To add a connection, do the following:

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. Select the connection type, set its _Display name_ to distinguish it from the others, and configure it as described below.


<img src="dk-bbconnection-createConnection.png" alt="Create a new TeamCity connection" width="706"/>

If your TeamCity server is [installed behind a proxy](configuring-proxy-server.md), it is important to ensure that this is reflected in the connection settings, if applicable. When configuring a callback URL for a connection, you need to specify all URLs by which the current server can be accessed.  
After configuring the proxy, remember to also set the new address as the _Server URL_ in __Global Settings__ of TeamCity.
{instance="tc"}

## Azure DevOps

<snippet id="azure-devops">

There are two types of Azure DevOps connections in TeamCity:
* __Azure DevOps OAuth 2.0__ allows signing in to TeamCity via an Azure DevOps Services account and creating TeamCity projects from Azure Git repositories.
* __Azure DevOps PAT__ allows creating TeamCity projects from Azure Git and TFVC repositories.

<anchor name="azure-devops-connection"/>

#### Azure DevOps OAuth 2.0 Connection
{id="Connecting+to+Azure+DevOps" help-id="Connecting to Azure DevOps"}

This type of connection supports only Azure DevOps Services. It uses the [OAuth 2.0 protocol](https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops) based on JWT tokens and requires creating a dedicated app in your Azure profile.

This connection can be used for [authenticating users via Azure DevOps](configuring-authentication-settings.md#Azure+DevOps+Services) as well as creating projects and build configurations.

To configure an Azure DevOps OAuth 2.0 connection:

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="Azure DevOps OAuth 2.0"/></include>
4. <include from="common-templates.md" element-id="connections-unique-callback-URL"><var name="unique-url-sample" value="/oauth/azuredevops/rid:your-unique-id/accessToken.html"/></include>
5. TeamCity will display the _Callback URL_ and _scopes_ required for registering an OAuth application in Azure DevOps.

    Go to the [Register Application](https://app.vsaex.visualstudio.com/app/register) page in Azure and create a new app using the provided parameters. When created, copy the app's ID and client secret.
6. Go back to the connection form in TeamCity and enter the Azure DevOps Services URL, the new application ID, and client secret.
7. Specify the application scope that must be the same as the scope of the created Azure DevOps OAuth App.
8. <include from="common-templates.md" element-id="test-and-save-connection"/>

To activate the Azure DevOps Services authentication on your server, proceed to enabling the respective [authentication module](configuring-authentication-settings.md#Azure+DevOps+Services).

#### Azure DevOps PAT Connection

This type of connection uses personal access tokens. It allows creating a [project from a Git or TFVC repository URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL), creating an [Azure DevOps VCS root](team-foundation-version-control.md), or integrating with the [Azure Board Work Items](azure-board-work-items.md) tracker.

To configure an Azure DevOps PAT connection:

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="Azure DevOps PAT"/></include> The page that opens provides the parameters to be used when connecting TeamCity to Azure DevOps Services.
4. Log in to your Azure DevOps Services account to create a personal access token with _All scopes_ as described in the [Microsoft documentation](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate).
5. Continue configuring the connection in TeamCity: on the __Add Connection__ page that is open, specify
   * the server URL in the `https://{account}.visualstudio.com` format or your Azure DevOps Server as `https://{server}:8080/tfs/`
   * your personal access token
6. <include from="common-templates.md" element-id="test-and-save-connection"/>
7. The connection is configured, and now a small Azure DevOps Services icon becomes active in several places where a repository URL can be specified: [create project from URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL), [create VCS root from URL](guess-settings-from-repository-url.md), create [Azure DevOps Server](team-foundation-version-control.md) VCS root, create [Azure Board Work Items](azure-board-work-items.md) tracker. Click the icon, log in to Azure DevOps Services and authorize TeamCity. TeamCity will be granted full access to all the resources that are available to you.   
   When configuring Commit Status Publisher for Git repositories hosted in TFS/VSTS, the personal access token can be filled out automatically if a VSTS project connection is configured.

>It is possible to configure several VSTS connections. In this case, the server URL will be displayed next to the VSTS icon to distinguish the server in use.

</snippet>

## Bitbucket Cloud

<snippet id="bb-cloud">

>In TeamCity Cloud, a connection to Bitbucket Cloud is already predefined in the Root project's settings, which makes it available in all the other projects.
>
{instance="tcc"}

A connection to Bitbucket Cloud can be used to:
* Create a [project from Bitbucket URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* Create a [VCS root from URL](guess-settings-from-repository-url.md).
* Create a [Mercurial VCS root](mercurial.md).
* Integrate with a [Bitbucket Cloud issue tracker](bitbucket-cloud.md).
* Enable [Bitbucket Cloud authentication](configuring-authentication-settings.md#Bitbucket+Cloud).

The Bitbucket Cloud connection form provides multiple parameters. You need to use them for [creating a new OAuth consumer in Bitbucket](https://confluence.atlassian.com/bitbucket/oauth-on-bitbucket-cloud-238027431.html#OAuthonBitbucketCloud-Createaconsumer).

After the consumer is created:
1. Copy its key and secret.
2. Go back to the connection form in TeamCity.
3. Paste the key and secret.
4. Save the connection.

A Bitbucket icon will become active in several places where a repository URL can be specified. Click it to authorize TeamCity in your Bitbucket profile. TeamCity will be granted access to your repositories. 
If you configure multiple Bitbucket connections, the server URL will be displayed next to each icon, so it is easier to distinguish the server in use.

</snippet>

## Bitbucket Server and Data Center

Integration with Bitbucket Server and Data Center currently allows you to:

* create a [project and build configuration from Bitbucket URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL)
* create a [VCS root from URL](guess-settings-from-repository-url.md)

To allow TeamCity to access Bitbucket data, you need to create an incoming application link in Bitbucket to grant TeamCity required permissions.


1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="Bitbucket Server / Data Center"/></include>
4. <include from="common-templates.md" element-id="connections-unique-callback-URL"><var name="unique-url-sample" value="/oauth/bitbucketserver/rid:your_unique_id/accessToken.html"/></include>
5. In a separate browser tab, go to the Bitbucket **"Administration | Application Links"** page.
6. Create a new [application link](https://confluence.atlassian.com/bitbucketserver/link-to-other-applications-1018764620.html) with the following parameters:

   * Application type: _External application_
   * Direction: _Incoming_
   * Redirect URL: _&lt;copy URL from the TeamCity new connection tab&gt;_
   * Application permissions: *tick "Write" under "Repositories"*

7. Once the application link is ready, Bitbucket will generate the _"Client ID"_ and _"Client Secret"_ values. Copy these values and paste them into corresponding fields on the TeamCity new connection tab.

8. <include from="common-templates.md" element-id="test-and-save-connection"/>


## GitHub

<snippet id="github">

>In TeamCity Cloud, a connection to GitHub.com is already predefined in the Root project's settings, which makes it available in all the other projects.
>
{instance="tcc"}

TeamCity allows you to create connections to both regular **GitHub.com** instances and **GitHub Enterprise**.

A connection to GitHub can be used to:
* Create a [project from GitHub URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* Create a [VCS root from URL](guess-settings-from-repository-url.md).
* Create a [Git VCS root](git.md).
* Integrate with a [GitHub issue tracker](github.md).
* Enable [GitHub.com](configuring-authentication-settings.md#GitHub) and [GitHub Enterprise](configuring-authentication-settings.md#GitHub) authentication.
* Provide access tokens for the [Commit Status Publiser](commit-status-publisher.md) and [Pull Requests](pull-requests.md) build features.
* Connections via GitHub Apps can be used to configure [webhooks](https://docs.github.com/en/rest/apps/webhooks?apiVersion=2022-11-28) that notify the TeamCity server about changes.

Depending on your needs, you can create connections to GitHub that operate via GitHub Apps or GitHub OAuth Applications.



<dl>



<dt>GitHub App</dt>
<dd>
<anchor name="github-app"/>

A <a href="https://docs.github.com/en/apps/creating-github-apps/setting-up-a-github-app/about-creating-github-apps">GitHub App</a> is an integration that allows third-party services such as TeamCity to connect to GitHub repositories without the necessity to keep a "service" user account. Compared to GitHub OAuth applications, GitHub Apps boast fine-grained permissions and grant you more control over which repositories the app can access.See this article for more information: <a href="https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/differences-between-github-apps-and-oauth-apps">Differences between GitHub Apps and OAuth Apps</a>.

In addition, configuration created via GitHub App connections can employ [GitHub Checks Webhook triggers](github-checks-trigger.md) that replace the traditional combo of VCS trigger and Commit Status Publisher build feature.


If you do not already have a suitable GitHub App, you can allow TeamCity to register it and create a connection that employs this new app in one go. TeamCity uses <a href="https://docs.github.com/en/apps/sharing-github-apps/registering-a-github-app-from-a-manifest">manifests</a> to register new GitHub Apps.

<ol>

<li>
<include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
</li>

<li>
<include from="common-templates.md" element-id="create-new-connection"/>
</li>

<li>
<include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="GitHub App"/></include> This connection type works with both regular GitHub and GitHub Enterprise accounts.
</li>


<li>
Select the <b>Automatic</b> creation mode to allow TeamCity to <a href="https://docs.github.com/en/apps/sharing-github-apps/registering-a-github-app-from-a-manifest">register a GitHub App from a manifest</a>.

<img src="dk-GhAppManifestButton.png" width="706" alt="GitHub Manifest App Button"/>

</li>

<li>Specify the URL of your GitHub server (without "/username") and choose whether you want this app to send <a href="configuring-vcs-post-commit-hooks-for-teamcity.md">post-commit hooks</a> and/or have access to your organization.</li>

<li>If the new GitHub App should provide access to organization repos, switch the <b>Owner</b> setting to <b>Organization</b> and enter the organization name in the corresponding field.</li>

<li>
Follow instructions on your screen to log into your GitHub account, authorize TeamCity to register an app, and install it to your personal and/or organization account.
</li>

</ol>


To manually create a new GitHub App and configure a TeamCity connection that uses this app:

<ol>

<li>Repeat steps 1 to 3 from the list above.</li>

<li>Switch the connection's creation mode to <b>Manual</b>.</li>

<li>

<var name="unique-url-sample" value="/oauth/githubapp/rid:your-unique-id/accessToken.html"/><include from="common-templates.md" element-id="connections-unique-callback-URL"/>

</li>


<li>In a separate browser tab, navigate to your GitHub account and follow instructions from the TeamCity connection description to create a new app. Note that GitHub will generate a private key in the process — save this <code>.private-key.pem</code> file in the secure location.</li>

<li>Open the general settings of your GitHub App. Copy required values (App ID, client ID, client secret) and paste them to the TeamCity dialog.</li>

<li>If you have a <a href="https://docs.github.com/en/rest/apps/webhooks?apiVersion=2022-11-28">GitHub App webhook</a> configured, set its secret and copy the same value to the <b>Webhook secret</b> field. GitHub can use this webhook to notify the TeamCity server it should scan a repository for changes when they occur, instead of letting the server to constantly poll GitHub for changes. See also: <a href="configuring-vcs-post-commit-hooks-for-teamcity.md#GitHub+and+GitHub+Enterprise">Configuring VCS Post-Commit Hooks</a>.</li>

<li>Enter the Owner URL — the link to a personal account or organization where this GitHub App is installed.</li>

<li>Upload the private key sent by GitHub.</li>

<li>

<include from="common-templates.md" element-id="test-and-save-connection"/>

</li>
</ol>


<note>

TeamCity asks you to sign in to your GitHub account before you can use features that employ GitHub App connections (for example, create new projects or update a Fetch URL). Permissions of refreshable access tokens depend on both permissions set in GitHub App settings, and permissions of your current GitHub user. In case of a mismatch, the lowest permission is used and certain TeamCity functionality can be unavailable.

For example, if your GitHub user has no write permissions, the access token will be issued with read-only permissions. As a result, the <a href="commit-status-publisher.md">Commit Status Publisher</a> feature you configure will not be able to successfully post build statuses.

</note>

<warning>

TeamCity currently issues single-repository access tokens for GitHub App connections. As a result, if your GitHub repository contains [submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules), TeamCity is unable to access these external repositories.

</warning>

</dd>



<dt>GitHub OAuth Application</dt>
<dd>
<anchor name="github-oauth"/>
   
OAuth Applications generate user access tokens and allow third-party services like TeamCity to perform actions on behalf of a user who authorized these services.<br/>


To create a TeamCity connection that utilizes a GitHub OAuth Application:

<ol>

<li><include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include></li>

<li><include from="common-templates.md" element-id="create-new-connection"/></li>

<li><include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="GitHub.com or GitHub Enterprise"/></include></li>

<li>If you do not already have a GitHub OAuth Application, follow TeamCity instructions to <a href="https://docs.github.com/en/developers/apps/authorizing-oauth-apps">create a new one</a>.</li>

<li>Copy the client ID and secret from OAuth Application's settings and paste them to the TeamCity dialog. For GitHub Enterprise you additionally need to paste the GitHub server URL.</li>

<li><include from="common-templates.md" element-id="test-and-save-connection"/></li>
</ol>



<note>

If you enable the <a href="configuring-authentication-settings.md#GitHub">GitHub.com authentication</a> module and want to restrict access to TeamCity to users of specific GitHub organizations, you need to ensure that your OAuth app is allowed by all these organizations. By default, GitHub does not allow OAuth apps to access the organizations. You can either disable this restriction for all apps or approve only the TeamCity app in each of the required organizations. Refer to the <a href="https://docs.github.com/en/github/setting-up-and-managing-organizations-and-teams/about-oauth-app-access-restrictions">GitHub documentation</a> for more details.

</note>

</dd>

</dl>

> When your GitHub Enterprise server is configured with a HTTPS endpoint, the connection might fail if the endpoint's certificate is not issued by a well-known commercial certification authority. In this case, you should update TeamCity server’s trusted certificates, following [these instructions](uploading-ssl-certificates.md).
> 
{style="note"}

Once a connection is successfully configured, the GitHub icon will become active in several places where a repository URL can be specified. Click it to authorize TeamCity in your GitHub profile. TeamCity will be granted full control of your private repositories and get the _Write repository hooks_ permission. If you configure multiple GitHub integrations, the server URL will be displayed next to each icon, so it is easier to distinguish the server in use.

</snippet>

## GitLab

<snippet id="gitlab">

>In TeamCity Cloud, a connection to GitLab.com is already predefined in the Root project's settings, which makes it available in all the other projects.
>
{instance="tcc"}

There are two types of GitLab connections: *GitLab.com* for accounts hosted on the [](https://gitlab.com) site, and *GitLab CE/EE* for accounts on a self-hosted GitLab Community Edition (CE) or Enterprise Edition (EE) server.

A connection to GitLab can be used to:
* Create a [project from GitLab URL](creating-and-editing-projects.md#Creating+project+pointing+to+repository+URL).
* Create a [VCS root from URL](guess-settings-from-repository-url.md).
* Integrate with a [GitLab issue tracker](gitlab.md).
* Enable [GitLab.com authentication](configuring-authentication-settings.md#GitLab.com).

OAuth Applications generate user access tokens and allow third-party services like TeamCity to perform actions on behalf of a user who authorized these services.

To create a TeamCity connection that uses a GitLab OAuth Application:

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="GitLab.com or GitLab CE/EE"/></include>
4. <b>GitLab CE/EE</b> connections only: <include from="common-templates.md" element-id="connections-unique-callback-URL"><var name="unique-url-sample" value="/oauth/gitlab/rid:your_unique_id/accessToken.html"/></include>
5. If you do not already have a GitLab OAuth Application, follow the GitLab instructions to create an OAuth Application in one of the following scopes:
   - [User owned applications](https://docs.gitlab.com/ee/integration/oauth_provider.html#create-a-user-owned-application)
   - [Group owned applications](https://docs.gitlab.com/ee/integration/oauth_provider.html#create-a-group-owned-application)
   - [Instance-wide applications](https://docs.gitlab.com/ee/integration/oauth_provider.html#create-an-instance-wide-application)
6. When filling out the *Add new application* form in GitLab:
   1. Choose a Name for the application
   2. Copy the *Redirect URL* from the TeamCity dialog into the GitLab form
   3. Under *Scopes*, check *api*
   4. Click *Save application*
    > For a GitLab instance-wide application, there is an additional *Trusted* checkbox, which skips the user authorization step when it is enabled.
    >
    {style="note"}

7. Copy the *Application ID* and *Secret* from the GitLab Application settings and paste them into the TeamCity dialog.
8. For a *GitLab CE/EE* connection, you must also enter the base URL of the GitLab CE/EE server (for example, `https://gitlab.mydomain.com`) into the *Server URL* field. Note that this field is not needed in the case of a GitLab.com connection, because the base URL is always `https://gitlab.com`.
9. <include from="common-templates.md" element-id="test-and-save-connection"/>

> When your GitLab CE/EE server is configured with a HTTPS endpoint, the connection might fail if the endpoint's certificate is not issued by a well-known commercial certification authority. In this case, you should update TeamCity server’s trusted certificates, following [these instructions](uploading-ssl-certificates.md).
>
{style="note"}

A GitLab icon will become active in several places where a repository URL can be specified. Click it to authorize TeamCity in your GitLab profile. TeamCity will be granted access to your repositories. If you configure multiple GitLab connections, the server URL will be displayed next to each icon, so it is easier to distinguish the server in use.

</snippet>

## Google

This type of connection supports Google Services. It uses the [OAuth 2.0 protocol](https://developers.google.com/identity/protocols/oauth2). 

This connection is used for authentication in TeamCity with a Google account.

Before configuring a Google connection, you need to [create a new Google project](https://cloud.google.com/resource-manager/docs/creating-managing-projects) and [register your app](https://developers.google.com/workspace/guides/configure-oauth-consent) if you have not done it before.

To configure a Google connection in TeamCity:


1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="Google"/></include>
4. Copy redirect URLs shown by TeamCity; these URLs are required to register an OAuth client.
5. Go to the [Credentials](https://console.cloud.google.com/apis/credentials) page in the Google project and create the [OAuth client ID](https://developers.google.com/workspace/guides/create-credentials#oauth-client-id) with the Web application type.
6. Paste your Callback URLs to the **Authorized redirect URIs** section in Google OAuth client ID. When the OAuth client is created, copy the Client ID and Client Secret.
7. Go back to the connection form in TeamCity, and enter the Client ID and Client secret.
8. <include from="common-templates.md" element-id="test-and-save-connection"/>

Now you can enable the [Google authentication module](configuring-authentication-settings.md#Google).

## Docker Registry

A connection to Docker Registry can be used to:
* Sign in to an authenticated Docker registry before running a build / sign out after the build.
* Clean up published images after the build.

See more information in the [dedicated article](configuring-connections-to-docker.md).

## Amazon Web Services (AWS)
{id="AmazonWebServices"}

The Amazon Web Services (AWS) connection allows defining AWS credentials once and using them in builds via the [AWS Credentials](aws-credentials.md) build feature. You can use different AWS credential types: access keys, IAM Role, and the Default credential provider chain.

To configure an AWS connection in TeamCity:

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="Amazon Web Services (AWS)"/></include>
4. Provide a name to distinguish this connection from others.
5. The _Connection ID_ field is filled out automatically. You can modify it and provide your own unique ID.
6. Select the _AWS region_ where the target resources are located.
7. From the _Type_ drop-down, select one of the credential types:

   {collapsible="true" style="full"}
   Access keys
   : 
   If you selected access keys as the credentials type, get the keys from the AWS console's [Identity and Access Management section](https://console.aws.amazon.com/iam) and provide them to TeamCity. See how to get these keys [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey).
   
     In the Access keys section in the TeamCity UI, do the following:
     1. Specify permanent **Access keys**:
          * **Access key ID**. Enter the access key ID.
          * **Secret access Key**. Enter the secret access key.

        It is [recommended](https://aws.amazon.com/blogs/security/how-to-rotate-access-keys-for-iam-users/) to change access keys regularly for security reasons. You will be able to do this after the connection is created via the **Rotate key** button.

        TeamCity will not revoke old keys immediately. After a new key is generated, TeamCity will preserve the old inactive key for 24 hours and then remove it. The lifetime of old keys can be changed via the following properties: `teamcity.internal.cloud.aws.keyRotation.old.key.preserve.time.min` or `teamcity.internal.cloud.aws.keyRotation.old.key.preserve.time.days`.
        
        To be able to successfully rotate access keys, TeamCity requires the `iam:GetUser`, `iam:CreateAccessKey`, and `iam:DeleteAccessKey` permissions. 
     
     2. Configure temporary **Session Settings**:
          * **Use session credentials**. Check the box to use an endpoint that provides [temporary access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) keys via AWS STS. Such credentials are short-term (the default session duration is 60 minutes). You can override the default session duration in the [AWS Credentials build feature](https://docs.google.com/document/d/1Y1eJCpErG8NJ9RHPvPOnfZUZZrVRkP8I85BXXSuNYEg/edit#). These credentials do not belong to a specific user and can be provided on demand to grant temporary access to specific resources. We recommend using temporary credentials since they provide better security.
          * **STS endpoint**.
            * TeamCity generates this field automatically when changing the AWS region. The regional endpoint is [recommended](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) because it is faster and has lower latency. In addition, all calls to the regional endpoint are logged in AWS Cloud Trail as any regional service call.
            * Use the [global endpoint](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) if the selected regional endpoint is [disabled](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html#rande-manage-enable) on the Amazon account and you do not want to enable it.
            * [Contact the TeamCity support team ](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?) if you need to specify a custom endpoint for Amazon alternatives like [MinIO](https://min.io/).

   IAM Role
   :
   Using [IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html), you can delegate access to your AWS resources to users, applications, or services that usually don't have these permissions. These entities will [assume this role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html) to get such access.
   
     You can use the IAM Role as the credentials type only if you already have at least one AWS connection with access keys or default credential provider chain configured in this TeamCity project.
     1. Specify **IAM Role**:
          * **AWS Connection**. Select the AWS connection that will [grant the specified IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html). Note that if the target AWS connection belongs to a parent TeamCity project, this connection's **Available for sub-projects** setting must be enabled.
          * **Role ARN**. Specify the [ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#identifiers-arns) of the role to assume by the connection you are creating.
     2. Configure **Session settings**:
          * **Session tag**. The session [tag](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_session-tags.html#id_session-tags_know) is required by Amazon. It is useful to locate sessions created by the TeamCity connection in AWS logs. TeamCity generates the tag automatically, but you can specify your own value.
          * **STS endpoint**.
            * TeamCity generates this field automatically when changing the AWS region. The regional endpoint is [recommended](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) because it is faster and has lower latency. In addition, all calls to the regional endpoint are logged in AWS Cloud Trail as any regional service call.
            * Use the [global endpoint](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_enable-regions.html) if the selected regional endpoint is [disabled](https://docs.aws.amazon.com/general/latest/gr/rande-manage.html#rande-manage-enable) on the Amazon account and you do not want to enable it.
            * [Contact the TeamCity support team ](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?) if you need to specify a custom endpoint for Amazon alternatives like [MinIO](https://min.io/).

     After the connection is created, you can view and copy the automatically generated external connection ID. We strongly recommend that you always add it to the [trust policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html) in AWS to prevent the [confused deputy problem](https://docs.aws.amazon.com/IAM/latest/UserGuide/confused-deputy.html). This ensures that only authorized TeamCity AWS connections will be able to use the specified IAM Role.

   Default credential provider chain {instance="tc"}
   :
   Select this type to provide access credentials according to the [default chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html#credentials-default). This approach provides an alternatives to storing credentials in plain text.<br/>

      When this credentials type is used, TeamCity searches for credentials in the following order:

      1. Java System Properties: `aws.accessKeyId` and `aws.secretAccessKey`.

      2. Environment Variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

      3. [Web Identity Token credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html#cli-configure-role-oidc) from system properties or environment variables.
         ```
         # In ~/.aws/config
         
         [profile web-identity]
         role_arn=arn:aws:iam:123456789012:role/RoleNameToAssume
         web_identity_token_file=/path/to/a/token
         ```

      4. Credential profiles file in the default location (`~/.aws/credentials`) shared by all AWS SDKs and the AWS CLI.

         ```
         # In ~/.aws/credentials
         [default]
         aws_access_key_id = your_key
         aws_secret_access_key = your_secret
         ```

         The default location can be overridden via the [`AWS_SHARED_CREDENTIALS_FILE`](aws-credentials.md) environment variable.

      5. Credentials delivered via the Amazon EC2 container service if the `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` environment variable is set and the security manager has access to it.

      6. Instance profile credentials delivered through the Amazon EC2 metadata service. 
      {instance="tc"}

  8. Tick the **Available for sub-projects** option if you want this connection to be available for all subprojects of the current project.

  9. Tick the **Available for build steps** option to allow choosing this connection in the [AWS Credentials build feature](aws-credentials.md) feature settings.

  10. <include from="common-templates.md" element-id="test-and-save-connection"/>

   <note instance="tc">
   Note that if your TeamCity server is hosted on an AWS instance that has an associated IAM role granting access to sensitive resources, using the <b>Default Provider Chain</b> credentials may present a security risk: in this case the TeamCity project administrators who configured this type of connection can access all AWS resources permitted by the role.    
    
   To mitigate this risk, the **Default credential provider chain** credentials type is disabled by default. To enable it, set [the internal property](server-startup-properties.md#TeamCity+Internal+Properties) `teamcity.internal.aws.connection.defaultCredentialsProviderEnabled=true` (the default value is `false`.)
    
   No server restart is required after the property is set.
   </note>

A configured AWS connection can supply credentials to the [AWS Credentials build feature](aws-credentials.md), [artifact S3 storages](storing-build-artifacts-in-amazon-s3.md), [EC2 cloud profiles](setting-up-teamcity-for-amazon-ec2.md) and other AWS connections that use IAM Roles.
{instance="tc"}

A configured AWS connection can supply credentials to the [AWS Credentials build feature](aws-credentials.md) and other AWS connections that use IAM Roles.
{instance="tcc"}

### Recommended Setup
{instance="tc"}

Amazon [key management guidelines](https://docs.aws.amazon.com/accounts/latest/reference/credentials-access-keys-best-practices.html) recommend using IAM Roles instead of access keys and static IAM user credentials. Since IAM Roles issue short-lived credentials, this approach minimizes potential damage should your credentials get exposed (accidentally or as a result of a security breach).

TeamCity allows your project to access required AWS resources using connections that assume IAM Roles and do not rely on locally stored credentials.

1. In AWS Management Console, go to the [IAM dashboard](https://console.aws.amazon.com/iam/) and navigate to the **Roles** tab.
2. Create a new IAM Role <i>without any permissions</i>. We will reference this role as "Role A".
3. Configure your TeamCity server machine to access AWS using this role instead of locally stored credentials. Required steps may vary depending on the exact type of your machine.
   * [Server running on an EC2 instance](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)
   * [Server running in an ECS container](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
   * Other configurations: [Web Identity](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html), [SAML](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithSAML.html)
    
4. In TeamCity, create a new AWS connection of the **Default Credentials Provider Chain** type. Press **Test Connection** to ensure TeamCity uses your empty "Role A".


5. If you want subprojects to have access to this new connection, check the **Available for sub-projects** option. Otherwise, only the same project that owns this connection will be able to use it.
        
    <img src="dk-shareAwsConnections.png" width="706" alt="Share AWS connections"/>


6. Create a second IAM Role ("Role B") with permissions required to access AWS resources (for example, EC2 instances or S3 buckets).
7. Modify the **Trust relationships** of this new Role B to allow Role A to assume it.
    ```JSON
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "your-Role-A-ARN"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```
8. In a TeamCity project that needs access to AWS resources, create another AWS connection.

   * **Type** — "IAM Role".
   * **AWS Connection** — the connection created in step 4.
   * **Role ARN** — the ARN of "Role B".

    Note that if you configure this new connection in a subproject of a project that owns the primary "Default Credentials Provider Chain" connection, this primary connection must have its **Available for sub-projects** setting enabled (see step 5).

9. Click **Test connection** to ensure TeamCity can assume **Role B**.
    
    ```
    Running STS get-caller-identity...
    Caller Identity:
     Account ID: <your account ID>
     User ID: <user ID:session>
     ARN: <Role B ARN>
    ```
    
10. Save your new connection and reopen it. You should now see the **External ID** value of the connection.
    
    <img src="dk-AWS-externalID.png" width="706" alt="External ID of an AWS Connection"/>
    
11. Go back to the **Trust relationships** of Role B and add the extra condition:
    ```JSON
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "your-Role-A-ARN"
          },
          "Condition": {
            "StringEquals": {
            "sts:ExternalId": "External-ID-copied-from-TeamCity"
            }
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

    
As a result, you now have a following setup:

* The primary **Default credentials provider chain** connection does not require locally stored credentials.
* This shared primary connection has no permissions to access anything. To access AWS resources, it needs to assume other IAM roles.
* Because of a condition configured in step 11, roles with actual access permissions can be assumed only by configured "IAM Role" TeamCity connections. TeamCity administrators cannot create more connections that utilize these roles.
    
    <img src="dk-aws-cannotassumeRole.png" width="706" alt="Incorrect External ID"/>
    
    To create more connections that can access Amazon resources, your AWS administrator must add additional conditions to whitelist external IDs of these new connections.


## Amazon ECR

An Amazon ECR (Elastic Container Registry) connection allows accessing private and public AWS registries. With its help, the [](docker-support.md) build feature can store Docker/LXC images produced by a build to AWS.

Connection settings:

<table>
<tr><td>Setting</td><td>Description</td></tr>

<tr>
<td>

Repository type

</td>
<td>

Choose to connect to a [private](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) or [public](https://docs.aws.amazon.com/AmazonECR/latest/public/what-is-ecr.html) registry.

</td>
</tr>

<tr>
<td>

AWS region

</td>
<td>

(Only for private registries) Select an AWS region where the target resources are located.

</td>
</tr>

<tr>
<td>

Credentials type

</td>
<td>

* __Access key__: select to use preconfigured AWS account access keys. You can find them in the [Identity and Access Management](https://console.aws.amazon.com/iam) section of your AWS console.
* __Temporary credentials__: get [temporary access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) via AWS STS. Such credentials are short-term and can be revoked anytime. They do not belong to a specific user and can be provided on demand — to grant temporary access to specific resources.

</td>
</tr>

<tr>
<td>

IAM role ARN

(_only for Temporary credentials_)

</td>
<td>

Specify a role to be used for generating temporary credentials. You need to [create this role in advance](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html) in your AWS console and assign it to all the permissions you need.

</td>
</tr>

<tr>
<td>

External ID

(_only for Temporary credentials_)

</td>
<td>

Specify an [external ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html). We strongly recommend that you always define it when using temporary credentials. This ensures that only TeamCity will be able to use the specified IAM role.

</td>
</tr>

<tr>
<td>

Default credential provider chain

</td>
<td>

Enable this option to automatically find access credentials according to the [default chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html#credentials-default).

This approach is recommended if you do not want to store the credentials anywhere in the TeamCity environment. By default, it will use the values of `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables.

</td>
</tr>

<tr>
<td>

Access key ID

</td>
<td>

Specify the access key ID.

See how to get it [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html).

</td>
</tr>

<tr>
<td>

Secret access key

</td>
<td>

Specify the secret access key.

See how to get it [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html).

</td>
</tr>

<tr>
<td>

Registry ID

</td>
<td>

Enter your [account ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html#FindingYourAWSId) number.

</td>
</tr>

</table>

## Slack

This type of connection is used to send notifications via [Slack](https://slack.com/).

Before configuring a Slack connection, you need to create a [Slack app](https://api.slack.com/apps) with the following [bot token scopes](https://api.slack.com/scopes): `channels:read`, `chat:write`, `im:read`, `im:write`, `users:read`, `team:read`, `groups:read`. You can add these in __Features | OAuth & Permissions | Scopes__ of your Slack app.

To ensure your TeamCity server can connect to Slack, specify all the possible endpoint addresses of the server as __Redirect URLs__ in __Features | OAuth &amp; Permissions__. In most cases, it would be enough to specify the _Server URL_ set in __[Global Settings](configuring-server-url.md)__ in TeamCity. However, if you use a proxy for your TeamCity server but access this server directly, the authentication in Slack might not work unless the server's IP address is also specified in __Redirect URLs__.
{instance="tc"}

To ensure your TeamCity server can connect to Slack, specify its address as __Redirect URL__ in __Features | OAuth \& Permissions__.
{instance="tcc"}

>See this [Basic app setup](https://api.slack.com/authentication/basics) guide for more details.

Now you can return to TeamCity, add a new Slack connection, and enter the following connection parameters:
* Client ID and secret from the app's __Basic Information__ page
* A [bot user token](https://api.slack.com/docs/token-types#bot) of your app

Configured Slack connections are used by [Notifier](notifications.md#Slack+Notifier) build features and [Service Messages](service-messages.md#Sending+Custom+Slack+Messages).

<anchor name="jetbrains-space-connection"/>

## JetBrains Space
{id="connect-to-jetbrains-space" help-id="Connect to JetBrains Space"}

>If you are looking for how to integrate your JetBrains Space instance with TeamCity, check out this **[full integration guide](how-to-configure-cicd-for-jetbrains-space.md)**!

This type of connection can be used for:
* Publishing build statuses in [JetBrains Space](https://www.jetbrains.com/space/) with the help of [Commit Status Publisher](commit-status-publisher.md).
* [Authenticating in TeamCity](configuring-authentication-settings.md#JetBrains+Space) with a JetBrains Space account.
* Creating [projects](creating-and-editing-projects.md), [build configurations](creating-and-editing-build-configurations.md), and [VCS roots](configuring-vcs-roots.md) from a JetBrains Space repository.
* Starting builds on [merge requests](pull-requests.md#JetBrains+Space+Merge+Requests) created in a JetBrains Space repository.

There are two ways to configure Space connections:

* Automatically — TeamCity configures read-only Space [applications](https://www.jetbrains.com/space/extensibility/external/#applications) with all required permissions and installs them to your Space instance.
* Manually — requires you to manually create and install Space applications, then set up connection settings in TeamCity.

### Automatic Connections

To configure a TeamCity project that builds and deploys a repository stored in JetBrains Space, you need two separate connections:

* Organization connection — an entry point that stores common connection settings, which allow TeamCity to access your Space instance.
* Project connection — allows TeamCity to access one specific project and its repositories.

You only need a single organization connection configured in a parent TeamCity project. However, to access separate Space projects, you will require separate project connections.

To configure an organization connection:

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="JetBrains Space"/></include>
4. Choose **Automatic: Organization Connection** under **Creation mode**.
5. Enter the name for your new connection and click **Create Space Application**.
6. TeamCity will open a separate browser window that allows you to choose the required Space instance:
    
    <img src="dk-space-chooseInstance.png" width="706" alt="Choose Space Instance"/>

   * Space Cloud — click **Install** next to the required Space Cloud instance, or click **Try another email** if you are currently logged in using a different user account.
   * Space On-Premises — enter Space organization URL and click **Install**.
   
7. Enter the Space application's name and optional description, and click **Install**.
    
    <img src="dk-installSpaceApp.png" width="706" alt="Install Space App"/>
    
    > Note that TeamCity retrieves the server URL from **Administration | General settings** and passes this URL as the endpoint for the newly created Space application. You will not be able to update this endpoint after the application is installed, so verify TeamCity uses the correct URL.
    >
    {style="note"}

8. Click **Approve all and go back to TeamCity** to grant the newly installed Space application required permissions.

With the organization connection configured and installed, TeamCity has permissions to install other applications and scan for the list of projects. Add a new build configuration or create a new VCS root choosing organization connection as a source to view this list (you will need to authorize TeamCity at the first attempt):

<img src="dk-space-projectlist.png" width="706" alt="Space Projects"/>

Note that all projects initially have grayed-out Space icons. This means TeamCity cannot access repositories in these projects yet. To grant TeamCity required permissions:

1. Click a required project.
2. Click **Proceed**. TeamCity will create and install a new pre-configured application with all permissions required to access this specific project.
3. Follow instructions on the screen to navigate to Space administration dashboard and approve these permission requests.

You should now see all repositories added to this project. Space projects with installed applications that grant TeamCity access to their repositories show colored Space icons.

<img src="dk-space-projectConnection.png" width="706" alt="Space Project Connection"/>

You can add more project connections to allow TeamCity to access additional Space projects within the same organization. To do this, navigate to **[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Connections** and create new Space connections with the **Automatic: Project Connection** creation type. This option becomes available once you add an organization connection to this project (or its parent).

<img src="dk-space-newProjectConnection.png" alt="New Space Project Connection" width="706"/>

Individual Space project connections can also be employed by [](commit-status-publisher.md) and [](pull-requests.md) build features to interact with project repositories. Space organization connections in turn allow your users to log in TeamCity [using their Space credentials](authentication-modules.md).

Space applications configured by project-level automatic connections issue access tokens that VCS roots and build features utilize to access project repositories. These access tokens are non-personal, which means if a user who initially set up TeamCity projects and issued tokens leaves your organization, these projects remain functional and do not require an update. See this article for more information on refreshable tokens: [](manage-access-tokens.md).





### Manual Connections

Configuring manual connections to JetBrains Space includes two steps: create a Space application with required permissions, and set up connection settings in TeamCity UI.

#### Create Space Application

1. Go to __Administration | Applications__ and click __New application__.
2. Enter a convenient name and save the application.
3. Go to the app's __Authorization__ tab and click __Configure requirements__ under the __In-context Authorization__ section. Enter the name of the Space project you are about to access from TeamCity.
4. Now, you need to set permissions that will be granted to the app in this project. Click __Configure__ and enable the following permissions:
   * Required for authentication and Pull Requests:
      * _Members | View member profile_
   * Required for Commit Status Publisher:
      * _Git Repositories | Report external check status_
      * _Code Review Comments | Post comments to code reviews_
   * Required for Pull Requests:
       * _Code Review | View code reviews_
   
   You can approve project-level permissions right in this **Authorization** tab if you are the project's administrator. Global permissions like viewing a member profile require a server administrator's approval.
5. Go back to the app's __Overview__ and open the __Authentication__ tab.
6. Enable _Client Credentials Flow_.
7. To be able to use authentication via Space in TeamCity or/and to create projects/configurations from Space repositories, enable _Authorization Code Flow_ as well. Enter your TeamCity server's URL as the redirect URI. If you wish TeamCity to add a unique ID to your callback URL, start with creating a TeamCity connection and copy the URL from the **Add Connection** dialog. This URL will look like the following: `.../oauth/space/rid:your-unique-id/accessToken.html`. 
   To ensure that your TeamCity server can always connect to JetBrains Space, specify all the other possible endpoint addresses of the server. In most cases, it would be enough to specify the _Server URL_ set in __Global Settings__ in TeamCity. However, if you use a proxy for your TeamCity server but access this server directly, the authentication might not work unless the server's IP address is also specified here.
8. Copy the app's _Client ID_ and _Client secret_.

__Note__: When you create a project in JetBrains Space, it does not automatically add you to this project as a member — this needs to be done manually. TeamCity will be able to see only those projects where you are listed as a member.


#### Configure Connection in TeamCity UI

When your Space connection is configured and installed, return to TeamCity and add a new JetBrains Space connection. In connection settings, enter the following connection parameters:
* URL of the Space server
* Client ID and secret of your Space application

Save the connection and proceed with adding a [Commit Status Publisher](commit-status-publisher.md) or [Pull Requests](pull-requests.md#JetBrains+Space+Merge+Requests) feature, [enabling Space authentication](configuring-authentication-settings.md#JetBrains+Space), or creating a [project](creating-and-editing-projects.md#Creating+project+pointing+to+JetBrains+Space) / [build configuration](creating-and-editing-build-configurations.md#Creating+Build+Configuration+Pointing+to+Specific+VCS) / [VCS root](configuring-vcs-roots.md).


## Kubernetes

Kubernetes connections allow TeamCity to access your Kubernetes resources. These connections can be used to [offload building tasks to K8s clusters](kubernetes-executor.md).

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
   
2. <include from="common-templates.md" element-id="create-new-connection"/>

3. <include from="common-templates.md" element-id="choose-connection-type"><var name="connection-type" value="Kubernetes Connection"/></include>

4. <include from="common-templates.md" element-id="kubernetes-settings-api-server-url"/>

5. <include from="common-templates.md" element-id="kubernetes-settings-certificate-authority"/>

6. <include from="common-templates.md" element-id="kubernetes-settings-namespace"/>

7. <include from="common-templates.md" element-id="kubernetes-settings-auth-strategy"/>

8. <include from="common-templates.md" element-id="test-and-save-connection"></include>


## NPM Registry
{id="npm-registry-settings" help-id="npm-registry-settings"}

This type of connection allows the [](nodejs.md) runner to access a private [npm registry](https://docs.npmjs.com/cli/v7/using-npm/registry) during a build. Since a project can have multiple NPM registry connections, you also need to [configure the related build feature](nodejs.md#Accessing+Private+NPM+Registries) to choose a connection that Node.js build steps should utilize.

Connection settings:

<table>

<tr><td>Setting</td><td>Description</td></tr>

<tr>

<td>Scope</td>
<td>

Specify an npm user/organization's [scope](https://docs.npmjs.com/cli/v7/using-npm/scope) (with or without the `@` character) to [associate](https://docs.npmjs.com/cli/v7/using-npm/scope#associating-a-scope-with-a-registry) with the connected registry. If you want to use multiple registries per project, you need to specify a scope for each of them.

Leave empty if you want to use only one registry in this project. It will be used by `npm`/`yarn` commands by default.

>If you don't specify a scope in the connection settings, you can still access a specific registry within a Node.js step by appending `--registry registry_url` to the `npm ...` command.

</td>

</tr>

<tr>

<td>Registry URL</td>
<td>

Specify the npm registry URL in the following format: `http(s)://hostname[:port]`. For example, `https://npm.pkg.jetbrains.space/mycompany/p/projectkey/mynpm`. The HTTPS schema is used by default.

</td>

</tr>

<tr>

<td>Access token</td>
<td>

Specify a [token](https://docs.npmjs.com/about-access-tokens) if it's needed for accessing the registry. Leave empty for anonymous access. Note that token-based authentication could differ depending on the registry type. See instructions for [npm Enterprise](https://docs.npmjs.com/creating-and-viewing-access-tokens), [Space Packages](https://www.jetbrains.com/help/space/authenticate-in-packages.html#space_account), or [GitHub Packages](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries).

</td>

</tr>

</table>

Save the connection and proceed with adding an [NPM Registry Connection](nodejs.md#Accessing+Private+NPM+Registries) build feature.

## Perforce Administrator Access

This type of connection allows [processing task streams on your Perforce server](perforce-workspace-handling-in-teamcity.md#Cleaning+Workspaces+on+Perforce+Server). In the connection settings, enter the host and user credentials for accessing the Perforce server (the user must have the [admin](https://www.perforce.com/manuals/p4sag/Content/P4SAG/protections.set.html#protections.set.access_levels) permission).

 <seealso>
        <category ref="admin-guide">
            <a href="configuring-vcs-roots.md">Configuring VCS Roots</a>
            <a href="creating-and-editing-projects.md">Creating and Editing Projects</a>
            <a href="creating-and-editing-build-configurations.md">Creating and Editing Build Configurations</a>
        </category>
        <category ref="examples">
            <a href="how-to-configure-cicd-for-jetbrains-space.md">How to Configure CI/CD for JetBrains Space</a>
        </category>
</seealso>
