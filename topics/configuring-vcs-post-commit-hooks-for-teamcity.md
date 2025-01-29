[//]: # (title: Configuring VCS Post-Commit Hooks for TeamCity)
[//]: # (auxiliary-id: Configuring VCS Post-Commit Hooks for TeamCity)

TeamCity periodically polls remote repositories targeted by each active [VCS root](vcs-root.md) to detect new changes. If your installation has hundreds of VCS roots, this continuous polling can significantly load the server.

You can set up post-commit hooks to decrease this load and speed up detecting new changes. In this case, whenever a new commit is pushed to the remote repository, a VCS provider will instantly send TeamCity a request to check for recent changes.

> This article explains how to configure VCS commit hooks that notify TeamCity about new repository changes. To read about TeamCity webhooks that notify third-party services about specific TeamCity events (starting and canceling builds, registering new agents, and so on), refer to the following article instead: [](teamcity-webhooks.md).
>
{style="note"}


## Overview

### Common Commit Hook Information

Commit hooks utilize [TeamCity REST API](https://www.jetbrains.com/help/teamcity/rest/quick-start.html) to issue commands for the TeamCity server. The recommended approach for post-commit hooks is to issue a command to check for new commits pushed to a remote repository. To do this, a webhook should send the `POST` request to the [/app/rest/vcs-root-instances](https://www.jetbrains.com/help/teamcity/rest/manage-vcs-roots.html) endpoint:

```Shell
<TeamCityServerURL>/app/rest/vcs-root-instances/commitHookNotification?locator=<vcsRootInstancesLocator>
```
{prompt="POST"}

* `TeamCityServerURL` is the URL of your TeamCity server.
* `vcsRootInstancesLocator` is the [locator](https://www.jetbrains.com/help/teamcity/rest/locators.html) that points to specific VCS root instance(s) used by this build configuration.

Requests sent to the `/app/rest/vcs-root-instances/commitHookNotification?locator` endpoint do not automatically trigger builds; they only force TeamCity to check for new changes. You can configure [VCS triggers](configuring-build-triggers.md) to start builds when these changes are detected.

If you want to explicitly queue new builds instead of checking for changes and letting TeamCity triggers initialize builds, send requests directly to the `/app/rest/buildQueue` endpoint instead. See this documentation article for request examples: [Start and Cancel Builds](https://www.jetbrains.com/help/teamcity/rest/start-and-cancel-builds.html).

### Instance Locator

The `vcsRootInstancesLocator` is a filter expression that finds all VCS root instances whose build configurations need to check for repository changes. This expression should define a set of specific criteria to locate required root instances and filter out all the others. Otherwise, if your locator returns many unrelated instances, TeamCity will issue excessive checks for changes, increasing the server load (beyond the initial polling load in some cases).

You can look for a required VCS root instance by its name, unique ID, the name of a parent VCS root, project, or build configuration, and other fields. See this article for the complete list of dimensions you can specify in this locator: [VcsRootInstanceLocator](https://www.jetbrains.com/help/teamcity/rest/vcsrootinstancelocator.html).

The most reliable locator is one that uses a repository URL (since it normally never changes). A VCS root instances locator does not include URL as an available dimension. Instead, you need to utilize a nested locator to check instance properties. Note that URL's colons and slashes should be escaped.

```Shell
/app/rest/vcs-root-instances?locator=property:(name:url,value:https%3A%2F%2Fgitlab.com%2Fusername%2Fsample-java-app-maven.git,matchType:equals,ignoreCase:true)
```
{prompt="POST"}

To view all VCS root instance properties and their values, request an instance using a path locator (`endpoint/value` instead of `endpoint?locator=value`) and the default "ID" dimension.

```Shell
/app/rest/vcs-root-instances/88
```
{prompt="GET"}

This request will return a payload similar to the following:


<tabs><tab title="XML">

```XML
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<vcs-root-instance id="88">
   <!--...-->
   <properties count="11">
      <property name="agentCleanFilesPolicy" value="ALL_UNTRACKED"/>
      <property name="agentCleanPolicy" value="ON_BRANCH_CHANGE"/>
      <property name="authMethod" value="ACCESS_TOKEN"/>
      <property name="branch" value="refs/heads/main"/>
      <property name="ignoreKnownHosts" value="true"/>
      <property name="submoduleCheckout" value="CHECKOUT"/>
      <property name="tokenId" value="..."/>
      <property name="url" value="https://gitlab.com/user/sample-java-app-maven.git"/>
      <property name="useAlternates" value="AUTO"/>
      <property name="username" value="oauth2"/>
      <property name="usernameStyle" value="USERID"/>
   </properties>
</vcs-root-instance>
```
 
</tab><tab title="JSON">
 
```JSON
{
   "id": "88",
   "...": "...",
   "properties": {
      "count": 11,
      "property": [
         {
            "name": "agentCleanFilesPolicy",
            "value": "ALL_UNTRACKED"
         },
         {
            "name": "agentCleanPolicy",
            "value": "ON_BRANCH_CHANGE"
         },
         {
            "name": "authMethod",
            "value": "ACCESS_TOKEN"
         },
         {
            "name": "branch",
            "value": "refs/heads/main"
         },
         {
            "name": "ignoreKnownHosts",
            "value": "true"
         },
         {
            "name": "submoduleCheckout",
            "value": "CHECKOUT"
         },
         {
            "name": "tokenId",
            "value": "..."
         },
         {
            "name": "url",
            "value": "https://gitlab.com/user/sample-java-app-maven.git"
         },
         {
            "name": "useAlternates",
            "value": "AUTO"
         },
         {
            "name": "username",
            "value": "oauth2"
         },
         {
            "name": "usernameStyle",
            "value": "USERID"
         }
      ]
   }
}
```

</tab></tabs>


To get the internal instance ID:

1. Go to your build configuration and open the **Settings** tab.
2. Copy the **ID** property value.
    
    <img src="dk-hooks-checkRootID.png" width="706" alt="Check Configuration ID"/>

3. Send the following request using any REST API testing tool, for instance, Postman.

    ```Shell
    /app/rest/vcs-root-instances?locator=buildType:<copied_value>
    ```
    {prompt="GET"}

4. You should receive a response similar to the following:
    
    <tabs><tab title="XML">
    
    ```XML
    <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <vcs-root-instances count="1" href="/app/rest/vcs-root-instances?locator=buildType:GitLabMavenApp_Build">
        <vcs-root-instance id="87" vcs-root-id="GitLabMavenApp_HttpsGitlabComUsernameReponameGitRefsHeadsMain" name="https://gitlab.com/username/reponame.git#refs/heads/main" href="/app/rest/vcs-root-instances/id:87"/>
    </vcs-root-instances>
    ```
    
    </tab><tab title="JSON">
    
    ```JSON
    {
        "count": 1,
        "href": "/app/rest/vcs-root-instances?locator=buildType:GitLabMavenApp_Build",
        "vcs-root-instance": [
            {
                "id": "87",
                "vcs-root-id": "GitLabMavenApp_HttpsGitlabComUsernameReponameGitRefsHeadsMain",
                "name": "https://gitlab.com/username/reponame.git#refs/heads/main",
                "href": "/app/rest/vcs-root-instances/id:87"
            }
        ]
    }
    ```
    
    </tab></tabs>
    
    In the sample response above, the unique ID of the VCS root instance used by a build configuration is "87".


A locator can include multiple dimensions, which allows you to specify multiple filter conditions. For example:

```Shell
vcsRoot:(type:<TYPE>,count:99999),property:(name:<URL_PROPERTY_NAME>,value:<VCS_REPOSITORY_URL_PART>,matchType:equals,ignoreCase:true),count:9999
```

This locator finds all instances of the specific VCS root. The `count` dimension raises the maximum number of returned instances to the given value (the default limit is 100 entries).

For VCS roots without the `*-references` parameter, you can use the following option with a better performance:

```Shell
vcsRoot:(type:<TYPE>,property:(name:<URL_PROPERTY_NAME>,value:<VCS_REPOSITORY_URL_PART>,matchType:equals,ignoreCase:true),count:99999),count:99999
```

### Authentication

For security reasons, TeamCity requires authentication for all REST API calls. Some VCS providers do not offer advanced webhook settings to specify a desired authentication method (basic auth, bearer token, and others) and allow you to enter only the request destination URL. In this case, add user credentials to a URL in one of the following formats:

```Shell
http(s)://username:password@TeamCityServerURL/app/rest/vcs-root-instances/commitHookNotification?locator=id:value

http(s)://username:PAT@TeamCityServerURL/app/rest/vcs-root-instances/commitHookNotification?locator=id:value
```

* **username:password** — the combination of regular credentials you use to log into TeamCity. Requires the enabled ["Basic HTTP" module](authentication-modules.md) in **Administration | Authentication**.

* **username:PAT** — the combination of a regular username and a [personal access token](configuring-your-user-profile.md#Managing+Access+Tokens). Requires the enabled ["Token-Based Authentication" module](authentication-modules.md) in **Administration | Authentication**.

Requests should use the credentials of a user with the "View project and all parent projects" and "View build configuration settings" permissions for all the projects where the VCS root is defined.

In case of authentication issues, navigate to **Administration | Diagnostics** and choose the "debug-auth" preset in the **Troubleshooting** section. You can then inspect the `TeamCity_Folder/logs/teamcity-auth.log` file to view the detailed problem information.
{instance="tc"}


### Repository Polling with Configured Hooks

<snippet id="polling-with-hooks">
Projects that have configured commit hooks still poll their remote repositories as a backup mechanism in case hooks stop working. However, with each successful hook communication, the polling interval automatically doubles. The maximum value to which TeamCity increases this interval is 4 hours, and the minimum interval is 15 minutes. Should scheduled polling reveal a change that did not trigger a commit hook, TeamCity will reset the polling interval to its default value.
</snippet>

To check the current polling interval and the commit hook status, navigate to **Administration | &lt;Your_Project&gt; | &lt;Your_Build_Configuration&gt; | Vesrion Control Settings**.

<img src="dk-vcs-root-hooks.png" width="706" alt="Polling and hook information"/>

To set a custom value for the [minimum polling interval](configuring-vcs-roots.md#checkingInterval), navigate to VCS root settings and scroll down to the **Changes Checking** section. 

<img src="dk-vcsroot-pollingInterval.png" width="706" alt="Change polling interval"/>


## Set Up a Post-Commit Hook

This section explains how to set up hooks for VCS providers that allow seamless webhook implementation and do not require writing your own post-commit scripts.

### GitHub and GitHub Enterprise

<!--

Install a stand-alone [Commit Hooks Plugin](https://github.com/JetBrains/teamcity-commit-hooks) to add GitHub webhooks from TeamCity UI in one click.

<video src="https://youtu.be/VzDI2HoiHk4"
title="Use GitHub commit hooks for faster checkouts"/>

-->

If your connection to GitHub or GitHub Enterprise is configured via [GitHub Apps](configuring-connections.md#GitHub), you can set up a post-commit hook directly in the connection properties.

1. When configuring a GitHub App connection, follow TeamCity instructions to create a [GitHub App webhook](https://docs.github.com/en/apps/creating-github-apps/setting-up-a-github-app/using-webhooks-with-github-apps) with the given URL.
2. Set up a webhook secret on the GitHub App configuration page and paste the same value to the TeamCity connection settings dialog.
3. Push a test commit to a GitHub repository to check TeamCity receives an update notification. You can check sent update notifications and TeamCity responses on the "GitHub App | Settings | Advanced" page.

   <img src="dk-webhook-deliveries.png" width="706" alt="Webhook deliveries"/>

The alternative approach is to utilize [GitHub Checks Webhook triggers](github-checks-trigger.md) that leverage [GitHub check run API](https://docs.github.com/en/rest/checks/runs) to trigger new TeamCity builds on every commit. This trigger also acts as a replacement for the Commit Status Publisher build feature.

### GitLab

1. Navigate to the required remote repository on your GitLab server.
2. Click **Settings | Webhooks** in the GitLab sidebar.
    
    <img src="dk-webhooks-gitlab.png" width="706" alt="Create webhooks in GitLab"/>

3. Click **Add new webhook**.
4. Specify the URL to which the webhook should send `POST` requests. Include username and password/PAT as described in the [](#Authentication) section. For example:
    
    ```
    http://valravn:querty@myteamcityfarm.build.gg/app/rest/vcs-root-instances/commitHookNotification?locator=87
    ```

5. Tick **Push events** under the "Triggers" section.
6. Click **Add webhook** to save your new webhook.
7. Push a new commit to test the hook. You should be able to see your commit in the TeamCity build configuration's **Pending Changes** tab.

GitLab also allows you to set up the integration with JetBrains TeamCity on the **Repository Settings | Integrations** page. This integration sends requests to the `/app/rest/buildQueue` endpoint, so instead of issuing "check for changes" requests, it explicitly queues new TeamCity builds when a new commit is pushed, or a new merge request is created.

<img src="dk-gitlab-teamcity-integration.png" width="706" alt="GitLab TeamCity integration"/>

To set up this integration, specify the TeamCity URL (without the `/app/rest/...` postfix), build configuration ID, and TeamCity user credentials.


### Bitbucket

You can utilize this approach in Bitbucket Cloud, Bitbucket Server, and Bitbucket Data Center.

1. Go to the remote repository that should send update notifications.
2. Navigate to the **Webhooks** page.

    <tabs><tab title="Bitbucket Cloud">
    
    In the sidebar menu, click **Repository settings | Webhooks**.
    
    <img src="dk-bitbucket-webhooks.png" width="706" alt="Create webhooks in BB Cloud"/>
    
    </tab><tab title="Bitbucket Server/Data Center">
    
    In the sidebar menu, click the gear icon and choose **Webhooks**.
    
    <img src="dk-bitbucketSDC-webhooks.png" width="706" alt="Create webhooks in BB Server and Data Center"/>
    
    </tab></tabs>

3. Click **Create/Add Webhook**.
4. Specify the webhook name and tick **Push** under the "Triggers" or "Events" section.
5. Specify the URL to which Bitbucket should send `POST` requests to trigger TeamCity update checkups. Include user credentials in the URL as described in the [](#Authentication) section. For example:

   ```
   https://valravn:querty@mybuildfarm.gg/app/rest/vcs-root-instances/commitHookNotification?locator=id:34
   ```

6. Click **Save** or **Create** to save your new webhook.
7. Push a sample commit to verify that your webhook successfully delivers requests.

> Bitbucket Server and Data Center users can purchase third-party Atlassian marketplace apps from the **Repository settings | Hooks** page. Some apps (for example, the [Bitbucket Post Webhook](https://help.moveworkforward.com/BPW/bitbucket-post-webhook-main-guide) app) provide additional webhook settings for more advanced setups. For instance, you may utilize the bearer token authentication strategy instead of the basic HTTP auth.
> 
{style="tip"}

### Azure DevOps Server

The latest Azure DevOps Server (formerly Team Foundation Server) and Azure DevOps Services provide service hooks for code-commit events. To create a hook, perform the following steps:

1. Open the admin page for a team project in web access.
2. Create a subscription by running the wizard.
3. Select the "Web Hooks" service to integrate with.
4. Select the "Code checked in" event and specify a filter.
5. Fill in the TeamCity username, password, and server URL in the format:
    ```Shell
    "$SERVER/app/rest/vcs-root-instances/commitHookNotification?locator=$LOCATOR"
    
    ```
   The `$LOCATOR` value depends on the TFS repository type.

   <tabs><tab title="TFVC Repository">
   
   ```Shell
   vcsRoot:(type:tfs,count:99999),property:(name:tfs-url,value:<TFS server url>,matchType:equals,ignoreCase:true),property:(name:tfs-root,value:<TFS project>,matchType:contains,ignoreCase:true),count:99999
   ```
   
   where `<TFS server url>` must be replaced with the value specified in the Azure DevOps VCS root URL and path properties. For example:
   
   ```Shell
   http://teamcity/app/rest/vcs-root-instances/commitHookNotification?locator=vcsRoot:(type:tfs,count:99999),property:(name:tfs-url,value:http%3A%2F%2Ftfs%3Aport%2Ftfs%2Fcollection,matchType:equals,ignoreCase:true),property:(name:tfs-root,value:Project,matchType:contains,ignoreCase:true),count:99999
   ```
   {ignore-vars="true"}
   
   </tab><tab title="Git Repository">
   
   ```Shell
   vcsRoot:(type:jetbrains.git,count:99999),property:(name:url,value:<VCS root repository URL>,matchType:equals,ignoreCase:true),count:99999
   ```
   
   where `<VCS root repository URL>` must be replaced with the repository URL specified in the corresponding TeamCity VCS root and the value should be URL-escaped. Example:
   
   ```Shell
   http://teamcity/app/rest/vcs-root-instances/commitHookNotification?locator=vcsRoot:(type:jetbrains.git,count:99999),property:(name:url,value:https%3A%2F%2Faccount.visualstudio.com%2FDefaultCollection%2FProject%2F_git%2FRepository,matchType:equals,ignoreCase:true),count:99999
   ```
   {ignore-vars="true"}
   
   </tab></tabs>

6. Click __Finish__ to create the service hook.



## Manual Setup

If a repository is hosted within an environment that does not expose ready-to-use webhook setup tools, you may need to implement them manually. The process commonly involves the following steps:

1. Create a [script](#Generic+Post-Commit+Hook) that sends REST API requests to the required TeamCity endpoint.
2. Save this script on your VCS server.
3. Edit VCS server settings to execute this script whenever a new change is pushed.

### Generic Post-Commit Hook

Save the script below on a VCS server as `teamcity-trigger.sh` (you will need a personal [access token](configuring-your-user-profile.md#Managing+Access+Tokens)): 


```Shell
SERVER=https://buildserver-url
ACCESS_TOKEN="<access-token>"
 
LOCATOR=$1
 
# The following is one-line:
(sleep 10;  curl --header "Authorization: Bearer $ACCESS_TOKEN" -X POST "$SERVER/app/rest/vcs-root-instances/commitHookNotification?locator=$LOCATOR" -o /dev/null) >/dev/null 2>&1 <&1 &
 
exit 0

```

For Perforce, you can use this [dedicated script](#Perforce+Server).

Set the  variables according to your TeamCity server. The user must have the "View build configuration settings" permission for projects where the VCS root is defined. This permission is included in the Project Developer role by default.


> If your TeamCity server uses a custom SSL certificate, you need to pass the `-k` or `--cacert /path/to/correct/internal/CACertificate` parameter to the `curl` command above.
>
{style="note"}

### Git Server

1. Locate the Git repository root on the target VCS server. It should contain the `.git/hooks` directory with some templates.

2. Create the `.git/hooks/post-receive` file with a line:

   ```Shell
   /path/to/teamcity-trigger.sh 'vcsRoot:(type:jetbrains.git,count:99999),property:(name:url,value:<VCS root repository URL>,matchType:equals,ignoreCase:true),count:99999'
   ```
   
   where `<VCS root repository URL>` must be replaced with the repository URL specified in the corresponding TeamCity VCS root and the value should be URL-escaped.

3. Make sure that both `teamcity-trigger.sh` and `hooks/post-receive` scripts can be read and executed by Git user(s). You may need to execute the following command:
    
   ```Shell
   chmod 755 /path/to/teamcity-trigger.sh /path/to/git_root/.git/hooks/post-receive
   ```


### Mercurial Server

1. Locate the Mercurial repository root on the target VCS server.

2. Create or edit the `.hg/hgrc config` and add the following snippet:

   ```Shell
   [hooks]
   changegroup = /path/to/teamcity-trigger.sh 'vcsRoot:(type:mercurial,count:99999),property:(name:repositoryPath,value:<VCS root repository url>,matchType:equals,ignoreCase:true),count:99999'
   ```
   
   where `<VCS root repository URL>` must be replaced with the repository URL specified in the corresponding TeamCity VCS root and the value should be URL-escaped.

3. Make sure that `teamcity-trigger.sh` is executable. You may need to execute the following command:

   ```Shell
   chmod 755 /path/to/teamcity-trigger.sh
   ```

### Subversion Server

1. Locate the Subversion repository root containing the `db`, `hooks`, `locks`, and other directories. We need the `hooks` directory.

2. Create the `hooks/post-commit` file with a line:

   ```Shell
   /path/to/teamcity-trigger.sh 'vcsRoot:(type:svn,count:99999),property:(name:url,value:<VCS root repository url>,matchType:equals,ignoreCase:true),count:99999'
   ```
   
   where `<VCS root repository URL>` must be replaced with the repository URL specified in the corresponding TeamCity VCS root and the value should be URL-escaped.

3. Make sure that both `teamcity-trigger.sh` and `hooks/post-commit` script can be read and executed by the process of the Subversion server. You may need to execute the following command:


   ```Shell
   chmod 755 /path/to/teamcity-trigger.sh /path/to/svn_repository_root/hooks/post-commit
   ```

### Perforce Server

This section explains how to set up the dedicated Perforce script. Note that utilizing the [generic script](#Edit+Perforce+Specification+with+Generic+Script) is also possible, but this is the outdated approach that is no longer recommended.

The dedicated post-commit script on installed on your Perforce server automatically detects Perforce VCS roots in TeamCity and triggers the respective builds. To be able to use the script, you need to generate an [access token](configuring-your-user-profile.md#Managing+Access+Tokens) first. The TeamCity user assigned to this token must have the "Run build" permission for projects where Perforce VCS roots are defined. This permission is included in the Project Developer role by default.

It is also recommended configuring a _[Perforce Administrator Access](perforce-workspace-handling-in-teamcity.md#perforce-admin-access)_ connection in the project settings. TeamCity will use it to ensure that all changed files in the Perforce changelist are collected. If such a connection is not configured explicitly, TeamCity will try to connect to Perforce using settings of one of the project's VCS roots. 

1. Save this script on your Perforce server as `teamcity-hook.sh`:

    ```Shell
    #!/bin/sh
    # How to use this script:
    # 1. Save the script on the Perforce server under the name /path/teamcity-hook.sh.
    # 2. Run chmod +x /path/teamcity-hook.sh.
    # 3. Set up a change-commit trigger by adding the following line when editing the specification with the `p4 triggers` command:
    #    check-for-changes-teamcity change-commit //depot/... "/path/teamcity-hook.sh %\change%"
    # 4. Update the variables below.
    #
    # Update the following variables: 
    
    # TeamCity server root URL:
    TEAMCITY_SERVER=http://localhost:8111/bs
    
    # Access token of a user on TeamCity server which can run builds in all relevant configurations (Developer role)
    TC_ACCESS_TOKEN="<token_value>"
    
    # This P4PORT value will be used to find relevant VCS roots, it should be equal to the P4Port setting in the VCS root:
    P4PORT="localhost:1666"
    CHANGE=$1
    
    # The following is one line:
    (sleep 10;  curl -H "Authorization: Bearer $TC_ACCESS_TOKEN" -d "p4port=$P4PORT&changelistId=$CHANGE" "$TEAMCITY_SERVER/app/perforce/commitHook" -o /dev/null) >/dev/null 2>&1 <&1 &
    exit 0
    ```

2. Run `chmod +x /path/teamcity-hook.sh`.
3. [Edit the Perforce specification](https://www.perforce.com/perforce/r15.1/manuals/p4sag/chapter.scripting.html#scripting.trigger.table.fields) with the `p4 triggers` command and set up a change-commit trigger by adding the following line:
   
   ```Shell
   check-for-changes-teamcity change-commit //depot/... "/path/teamcity-hook.sh %\change%"
   ```
   
   where `//depot/...` is the depot which is used in TeamCity builds. If there are multiple depots, you can replace this path with `//...`.
   
4. Update the variables from the script with your TeamCity settings.

>If your TeamCity server uses a custom SSL certificate, you need to pass the `-k` or `--cacert /path/to/correct/internal/CACertificate` parameter to the `curl` command above.
>
{style="note"}

#### Edit Perforce Specification with Generic Script

Set up a `change-commit` trigger by adding one or several lines when [editing the specification](https://www.perforce.com/perforce/r15.1/manuals/p4sag/chapter.scripting.html#scripting.trigger.table.fields). The text below must be placed in one line, one line per trigger.

```Shell
check-for-changes-teamcity change-commit //depot/project1/... "/path/teamcity-trigger.sh '<VCS Root locator>'"
```

where `<VCS Root locator>` can be one of the following:

* for Stream-based VCS roots:
    ```Shell
    vcsRoot:(type:perforce,count:99999),property:(name:stream,value://streamdepot/streamname,matchType:equals,ignoreCase:true),count:99999
    ```

* for Client-based VCS roots:
    ```Shell
    vcsRoot:(type:perforce,count:99999),property:(name:client,value:<client name>,matchType:equals,ignoreCase:true),count:99999
    ```

* for Client-mapping VCS roots:
    ```Shell
    vcsRoot:(type:perforce,count:99999),property:(name:client-mapping,value:<unique client mapping>,matchType:equals,ignoreCase:true),count:99999
    ```
    where `<unique client mapping>` should match the Perforce depot path in the TeamCity VCS root after all parameters' resolution. For the rule `check-for-changes-teamcity change-commit //depot/project1/...` it would be `//depot/project1/`.  

  Each `check-for-changes-teamcity` rule line describes an association between the path with the commit (`//depot/project1`) and a set of VCS roots which should be checked for changes.



## Troubleshooting

It is recommended to try executing the following command from the command line before configuring the actual hook:


```Shell
curl --header "Authorization: Bearer $ACCESS_TOKEN" -X POST "$SERVER/app/rest/vcs-root-instances/commitHookNotification?locator=$LOCATOR"
```

If the commit hook matches the VCS root on the server correctly, you should see the output similar to this:


```Shell
Scheduled checking for changes for 1 VCS roots. (Server time: 20160719T192540.787+0300)
```

If the commit hook has not found any VCS roots, it will report an error:


```Shell
No VCS roots are found ...
```

Possible reasons for this output:
* the specified locator is incorrect, it does not match any VCS root on the server
* the specified user does not have enough permission for at least one of the matched VCS roots.

To check what roots are actually matched, use the request (see also [details](https://www.jetbrains.com/help/teamcity/rest/manage-vcs-roots.html)):

```Shell
curl --header "Authorization: Bearer $ACCESS_TOKEN" -X POST "$SERVER/app/rest/vcs-root-instances?locator=$LOCATOR"
```

