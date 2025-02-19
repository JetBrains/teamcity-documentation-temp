[//]: # (title: Configuring Authentication Settings)
[//]: # (auxiliary-id: Configuring Authentication Settings)

TeamCity can authenticate users via an internal database, or can integrate into your system and use external authentication sources such as Windows Domain, LDAP, or Git hosting providers.

## Configuring Authentication

Authentication is configured on the __Administration | Authentication__ page. The currently used authentication modules are also displayed there.

TeamCity provides several preconfigured authentication options (presets) to cover the most common use cases. The presets are combinations of authentication modules supported by TeamCity:

* [Credentials authentication modules](#Credentials+Authentication+Modules)
  * [Built-in](#Built-in+Authentication)
  * [Windows Domain Authentication](#Windows+Domain+Authentication)
    {instance="tc"}
  * [LDAP Integration](ldap-integration.md)
    {instance="tc"}
  * [Token-Based Authentication](#Token-Based+Authentication)
* [HTTP / SSO authentication modules](#HTTP+%2F+SSO+Authentication+Modules)
  * [Basic HTTP Authentication](#Basic+HTTP+Authentication)
  * [NTLM HTTP Authentication](ntlm-http-authentication.md)
    {instance="tc"}
  * [Bitbucket Cloud](#Bitbucket+Cloud)
  * [GitHub App](#GitHub)
  * [GitHub.com](#GitHub)
  * [GitHub Enterprise](#GitHub)
  * [GitLab.com](#GitLab.com)
  * [GitLab CE/EE](#GitLab+CE%2FEE)
  * [Google](#Google)
  * [JetBrains Space](#JetBrains+Space)
  * [Azure DevOps Services](#Azure+DevOps+Services)
  * [HTTP SAML 2.0](#HTTP+SAML+2.0)

>If you are using [JetBrains Hub](https://www.jetbrains.com/hub/), you can configure the single sign-on setting (SSO) from the TeamCity login form and IDE using a [separate plugin for TeamCity](https://plugins.jetbrains.com/plugin/9156-jetbrains-hub-integration).
>
{instance="tc"}

When you first sign in to TeamCity, the default authentication, including the Built-in and Basic HTTP authentication modules, is enabled.

To modify the existing settings, click __Edit__ next to the description of the enabled authentication module.

To switch to a different preconfigured scheme, use the __Load preset__ button.
{instance="tc"}

<tip instance="tc">

Any changes made to authentication in the UI are reflected in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/auth-config.xml>` file. 
If using the UI is not possible (for example, you need to change the authentication settings before the server is started), 
you can configure authentication settings in the &lt;TeamCity data directory&gt;/config/auth-config.xml file on the server machine as follows:

 1. Add the `<auth-module>` tag inside the `<auth-modules>` tag inside the `<auth-config>` tag.
 2. Specify the type attribute for the newly created `<auth-module>` tag.

The following values are supported for the type attribute (the values are case-insensitive):

* For credentials authentication modules:
  * **Default** for Default Authentication
  * **NT-Domain** for Windows Domain Authentication
  * **LDAP** for LDAP Authentication

* For HTTP authentication modules:
  * **HTTP-Basic** for Basic HTTP Authentication
  * **HTTP-NTLM** for NTLM HTTP Authentication
  
You can also provide some properties for each authentication module depending on the module type. Each property is specified as the &lt;property&gt; tag inside the corresponding &lt;auth-module&gt; tag. Each &lt;property&gt; tag must contain the key attribute with a property key. The property value is specified as the text inside the &lt;property&gt; tag.

Also, TeamCity plugins can provide additional authentication modules. The server restart is NOT needed after editing this file. The changes will be reflected in the Web UI.

</tip>

### Using Presets
{instance="tc"}

To load a preconfigured set of modules, use the __Load preset__ button, select a required option, and __Apply__ your changes. The following presets are available:
* Default ([built-in authentication](#Built-in+Authentication) — [Token-based](#Token-Based+Authentication) and [Basic HTTP](accessing-server-by-http.md))
* [LDAP](ldap-integration.md)
* Active directory ([LDAP](ldap-integration.md) with [NTLM](ntlm-http-authentication.md) and [Token-based](https://www.jetbrains.com/help/teamcity/rest/manage-users.html#Manage+Access+Tokens))
* Microsoft Windows Domain ([NTLM](ntlm-http-authentication.md), [Token-based](https://www.jetbrains.com/help/teamcity/rest/manage-users.html#Manage+Access+Tokens) and [Basic HTTP](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html#REST+Authentication))

### Enabling Multiple Authentication Modules

TeamCity allows enabling multiple authentication modules simultaneously.

When a user attempts to sign in, modules will be tried one by one. If one of them authenticates the user, the login will be successful; if all of them fail, the user will not be able to sign in to TeamCity.

<warning>

If the System Administrator creates users without a password while several authentication modes are enabled on the server including [Built-in authentication](#Built-in+Authentication) 
and later changes authentication from mixed to built-in, the users with no password will be unable to sign in to TeamCity.

</warning>

It is possible to use a combination of internal and external authentication. The recommended approach is to configure [LDAP Integration](ldap-integration.md) for your internal employees first and then to add [Built-in](#Built-in+Authentication) authentication for external users. Since TeamCity 2020.2, you can also enable authentication via OAuth services.
{instance="tc"}

It is possible to use a combination of internal and external authentication. Since TeamCity 2020.2, you can also enable authentication via OAuth services.
{instance="tcc"}

To add a module:
1. Click __Add Module__ and select a module from the drop-down menu.
2. Use the properties available for modules by selecting checkboxes in the __Add Module__ dialog.
3. Click __Apply__ and __Save__ your changes.

>TeamCity plugins can provide [additional authentication modules](https://plugins.jetbrains.com/docs/teamcity/custom-authentication-module.html).

### General Authentication Settings

In the __General Settings__ block, you can:
* Enable the [guest login](enabling-guest-login.md) on the server and change the guest username. Please read our [security notes](security-notes.md#caution-guest-login) before enabling this option.
* Customize the view of the login form: enter an introductory text and hide the default username/password fields (convenient if you prefer [authentication through third-party services](#HTTP+%2F+SSO+Authentication+Modules)).
* Enable the [per-project authorization mode](managing-roles-and-permissions.md#Changing+Authorization+Mode).
{instance="tc"}
* Enable the [two-factor authentication](managing-two-factor-authentication.md).
{instance="tc"}
* Enforce the [email verification](enabling-email-verification.md) for all TeamCity users.
* Log out all currently signed in users and delete all [personal access tokens](configuring-your-user-profile.md#Managing+Access+Tokens).

## User Authentication Settings

The very first time TeamCity server starts with no users (and no administrator), so the first user is prompted for the administrator account. If you are not prompted for the administrator account, refer to [How To Retrieve Administrator Password](how-to.md#Retrieve+Administrator+Password) for a resolution.
{instance="tc"}

The TeamCity administrator can modify the authentication settings of every user on their profile page.

The TeamCity list of users and authentication modules just map external credentials to the users. This means that a single TeamCity user can authenticate using different modules, provided the entered credentials are mapped to the same TeamCity user. Authentication modules have a configuration on how to map external user data to a TeamCity user, and some allow editing the external user linking data on the TeamCity user profile.

Handling of the user mapping by the bundled authentication modules:
* Built-in authentication stores a TeamCity-maintained password for each user.
* Windows Domain authentication allows specifying the default domain and assumes the domain account name is equal to the TeamCity user. The domain account can be edited on the user profile page.
* LDAP Integration allows setting LDAP property to get TeamCity username from user's LDAP entry.
* The modules corresponding to Git hosting providers allow admins to map users with to their external accounts by usernames. Each user can connect their own profile to an external Git hosting account in __Your Profile | General | Authentication Settings__.

Be cautious when modifying authentication settings: there can be a case when the administrator cannot sign in after changing authentication modules.   
Let's imagine that the administrator had the "jsmith" TeamCity username and used the default authentication. 
Then, the authentication module was changed to Windows domain authentication (i.e. Windows domain authentication module was added and the default one was removed). 
If, for example, the Windows domain username of that administrator is "john.smith", they will not be able to sign in anymore: 
not via the default authentication since it is disabled, nor via Windows domain authentication since their Windows domain username is not equal to the TeamCity username. 
The solution, nevertheless, is quite simple: the administrator can sign in using the [superuser account](super-user.md) 
and change their TeamCity username or specify their Windows domain username on their own profile page.
{instance="tc"}

Be cautious when modifying authentication settings: there can be a case when the administrator cannot sign in after changing authentication modules.   
Let's imagine that the administrator had the "jsmith" TeamCity username and used the default authentication. Then, the authentication module was changed to Windows domain authentication (i.e. Windows domain authentication module was added and the default one was removed). If, for example, the Windows domain username of that administrator is "john.smith", they will not be able to sign in anymore: not via the default authentication since it is disabled nor via Windows domain authentication since their Windows domain username is not equal to the TeamCity username.
{instance="tcc"}

### Special User Accounts
{instance="tc"}

By default, TeamCity has a [Super User](super-user.md) account with maximum permissions and a [Guest User](guest-user.md) with minimal permissions. These accounts have no personal settings such as the __[Changes](viewing-user-changes-in-builds.md)__ page and Profile information as they are not related to any particular person but rather intended for special use cases.

## Credentials Authentication Modules

### Built-in Authentication

By default, TeamCity uses the built-in authentication and maintains users and their passwords by itself.

When signing in to TeamCity for the first time, the user will be prompted to create the TeamCity username and password,
that will be stored in TeamCity and used for authentication. 
If you installed TeamCity and signed in to it, it means that built-in authentication is enabled, and all user data is stored in TeamCity.

In the beginning, the user database is empty. New users are either [added by the TeamCity administrator](creating-and-managing-users.md), or users register themselves if the corresponding option is enabled in the authentication module settings. All newly created users belong to the [All Users](creating-and-managing-user-groups.md#%22All+Users%22+Group) group and have all roles assigned to this group. If some specific [roles](managing-roles-and-permissions.md) are needed for the newly registered users, these roles should [be granted](managing-roles-and-permissions.md#Managing+Roles) via the __All Users__ group.

By default, the users are allowed to change their password on their profile page.

### Token-Based Authentication

The Token-Based Authentication module allows users to authenticate via [access tokens](configuring-your-user-profile.md#Managing+Access+Tokens) that they can create and invalidate themselves. This authentication module is enabled by default.

<snippet id="access-token-expiration-note">

> For security reasons, TeamCity ignores the "Remember me" checkbox when users log in with access tokens. Additionally, users must re-login when switching [nodes](multinode-setup.md) in the TeamCity UI or when the token expires.
>
{style="note"}{instance="tc"}

> For security reasons, TeamCity ignores the "Remember me" checkbox when users log in with access tokens. Additionally, users must re-login when the token expires.
>
{style="note"}{instance="tcc"}

</snippet>

### Windows Domain Authentication
{instance="tc"}

Allows user sign in using a Windows domain name and password. TeamCity checks these credentials: the server should be aware of the domain(s) users use to sign in. The supported syntax for the username is `DOMAIN\user.name` or `<username>@<domain>`.

In addition to signing in using the login form, you can enable [NTLM HTTP Authentication](ntlm-http-authentication.md) single sign-on.    
If you select the "Microsoft Windows Domain" preset, in addition to the login via a Windows domain, the [Basic HTTP](#Basic+HTTP+Authentication) and [NTLM](ntlm-http-authentication.md) authentication modules are enabled by default.

#### Specifying Default Domain

To allow users to enter the system using the login form without specifying the domain as a part of the username, do the following:
1. Go to __Administration | Authentication__.
2. Click __Edit__ next to the __Microsoft Windows domain__ authentication description.
3. Set the name in the _Default domain_ field.
4. Click __Done__ and __Save__ your changes.

#### Registering New Users on Login

The default settings allow users to register from the login page. TeamCity usernames for the new users will be the same as their Windows domain accounts.   
All newly created users belong to the [All Users](creating-and-managing-user-groups.md#%22All+Users%22+Group) group and have all roles assigned to this group. If some specific [roles](managing-roles-and-permissions.md) are needed for the newly registered users, these roles should [be granted](managing-roles-and-permissions.md#Managing+Roles) via the __All Users__ group.

To disable new user registration on login:
1. Go to __Administration | Authentication__.
2. Click __Edit__ next to the __Microsoft Windows domain__ authentication description. Clear the _Allow user registration from the login page_ box.
3. Click __Edit__ next to the __NTLM HTTP__ authentication description. Clean the _Allow user registration from the login page_ box.

#### Linux-Specific Configuration

If your TeamCity server runs under Linux, JCIFS library is used for the Windows domain login. This only supports Windows domain servers with SMB (SMBv1) enabled. SMB2 is not supported.  
The library is configured using the properties specified in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/ntlm-config.properties` file. Changes to the file take effect immediately without the server restart.

JCIFS library settings which cannot be changed in runtime or settings to affect HTTP NTLM settings can only be set via a properties' file passed via `-Djcifs.properties` JVM option.

If the default settings do not work for your environment, refer to [JCIFS](http://jcifs.samba.org/src/docs/api/) for all available configuration properties.   
If the library does not find the domain controller to authenticate against, consider adding the `jcifs.netbios.wins` property to the `ntlm-config.properties` file with the address of your WINS server. For other domain services locating properties, see [JCIFS](http://jcifs.samba.org/src/docs/resolver.html).

### LDAP Authentication
{instance="tc"}

Please refer to the [dedicated page](ldap-integration.md).

<!--[//]: # (Internal note. Do not delete. "Configuring Authentication Settingsd70e480.txt")-->

## HTTP / SSO Authentication Modules

### Basic HTTP Authentication

Please refer to [Accessing Server by HTTP](accessing-server-by-http.md) for details about the basic HTTP authentication.

> For information on configuring basic HTTP authentication directly in the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/config/auth-config.xml`, refer to the [plugin documentation](https://plugins.jetbrains.com/docs/teamcity/custom-authentication-module.html).


### NTLM HTTP Authentication
{instance="tc"}

Please refer to the [dedicated page](ntlm-http-authentication.md).

### Bitbucket Cloud

Users can sign in to TeamCity with a Bitbucket Cloud account.

Before enabling this module, you need to configure a [Bitbucket Cloud connection](configuring-connections.md#Bitbucket+Cloud) in the Root project's settings and a dedicated application in Bitbucket.

To sign in, click the Bitbucket icon above the login form and, after the redirect, approve the TeamCity application. If a user with your Bitbucket email is registered and this email is verified both [in TeamCity](enabling-email-verification.md) and in Bitbucket, this Bitbucket account will be mapped to the respective TeamCity user and you will be signed in. Otherwise, TeamCity will create a new user profile, unless the **Allow creating new users on the first login** option is disabled. It is also possible to [map existing TeamCity users](#User+Authentication+Settings) with Bitbucket Cloud profiles.

<table>

<tr>
<td>Setting</td>
<td>Description</td>
</tr>

<tr>

<td>

Allow creating new users on the first login

</td>

<td>
Enabled by default. If this option is disabled, TeamCity will not create a new user when the provided external email is unrecognized. This is helpful if you use a publicly available TeamCity server and want to limit access to it.

</td>

</tr>

<tr>
<td>
Allow any Bitbucket user to log in
</td>
<td>
<p>Disabled by default.</p>
<p>If this option is enabled, users from any workspace can access the TeamCity server. To limit access to specific workspaces, use the <b>Restrict authentication</b> setting instead.</p>
<warning>If this option is enabled in combination with the <b>Allow creating new users on first login</b> option, any Bitbucket Cloud user could register on the TeamCity Server.</warning>
</td>
</tr>

<tr>

<td>

Restrict authentication

</td>

<td>
<p>A comma-separated list of <a href="https://support.atlassian.com/bitbucket-cloud/docs/what-is-a-workspace/">workspaces'</a> IDs.</p>

<p>This list limits the set of users who can register or authenticate in TeamCity with their Bitbucket Cloud account to the specified workspaces. When combined with the <b>Allow creating new users on the first login</b> option, this setting allows automatically registering users who have an email in one of the specified workspaces and do not have a user profile in TeamCity.</p>

<tip>Once registered on the TeamCity server, a user can create a password or token which enables them to sign in to the server directly, bypassing Bitbucket verification. If you delete a user from a workspace in Bitbucket, remember to restrict their access or delete their user profile in TeamCity.</tip>

</td>

</tr>

</table>

### GitHub

Users can sign in to TeamCity with their GitHub.com and GitHub Enterprise accounts. TeamCity provides three independent modules for this task.

* The **GitHub.com** module requires that you create an [OAuth GitHub.com connection](configuring-connections.md#GitHub) for the Root project.
* The **GitHub Enterprise** module requires that you create an [OAuth GitHub Enterprise connection](configuring-connections.md#GitHub) for the Root project.
* The **GitHub App** module requires that you create a [GitHub App](configuring-connections.md#GitHub) for the Root project. This connection type is available for both GitHub.com and GitHub Enterprise users.

To sign in, users must click the GitHub icon above the Username field. If a user with this GitHub email is already registered and the email is verified both [in TeamCity](enabling-email-verification.md) and in GitHub, the user's GitHub account will be mapped to the respective TeamCity user, and this user will be signed in.

Otherwise, TeamCity creates a new user profile. You can disable this behavior via the corresponding authentication module setting. In this case, only users who are already registered will be able to sign in. It is also possible to [map existing TeamCity users](#User+Authentication+Settings) with GitHub.com profiles.

All three modules have identical settings:

<table>

<tr>
<td>Setting</td>
<td>Description</td>
</tr>

<tr>

<td>

Allow creating new users on the first login

</td>

<td>
<p>Enabled by default.</p>
<p>If this option is disabled, TeamCity will not create a new user when the provided external email is unrecognized. This is helpful if you use a publicly available TeamCity server and want to limit access to it.</p>
</td>

</tr>

<tr>
<td>
Allow any GitHub user to log in
</td>
<td>
<p>Disabled by default.</p>
<p>If this option is enabled, users from any organization can access the TeamCity server. To limit access to specific organizations, use the <b>Restrict authentication</b> setting instead.</p>
<warning>If this option is enabled in combination with the <b>Allow creating new users on first login</b> option, any GitHub user could register on the TeamCity Server.</warning>
</td>
</tr>

<tr>

<td>

Restrict authentication to users from the specified organizations

</td>

<td>

A comma-separated list of [organizations'](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/about-organizations) IDs.

This list limits the set of users who can register or authenticate in TeamCity with their GitHub account to the specified organizations. When combined with the _Allow creating new users on the first login_ option,  this setting allows automatically registering users who have an email in one of the specified organizations and do not have a user profile in TeamCity.

>To use this restriction, make sure that the GitHub OAuth application or GitHub App used in the Root project's [GitHub connection](configuring-connections.md#GitHub) is approved/installed in each specified organization.
{style="note"}

>Once registered on the TeamCity server, a user can create a password or token which will allow them to sign in to this server directly, bypassing the GitHub verification. If you delete a user from an organization in GitHub, remember to restrict their access or delete their user profile in TeamCity.

</td>

</tr>

</table>


>If you reconnect a TeamCity server from one GitHub Enterprise server to another, TeamCity might not be able to recognize external users after this operation. This case requires reconfiguring user profiles manually. If you encounter any issues, [contact our support](troubleshooting.md).
>
{style="warning"}

### GitLab.com

Since version 2020.2, users can sign in to TeamCity with a GitLab.com account.

Before enabling this module, you need to configure a [GitLab.com connection](configuring-connections.md#GitLab) in the Root project's settings and a dedicated application in GitLab.

To sign in, click the GitLab icon above the login form and, after the redirect, approve the TeamCity application. If a user with your GitLab email is registered and this email is verified both [in TeamCity](enabling-email-verification.md) and in GitLab, this GitLab account will be mapped with the respective TeamCity user and you will be signed in. Otherwise, TeamCity will create a new user profile, unless the **Allow creating new users on the first login** option is disabled. It is also possible to [map existing TeamCity users](#User+Authentication+Settings) with GitLab.com profiles.

>If you want to be recognized in TeamCity by your email, make sure this email is set as _public_ in GitLab.

<table>

<tr>
<td>Setting</td>
<td>Description</td>
</tr>

<tr>

<td>

Allow creating new users on the first login

</td>

<td>
<p>Enabled by default.</p>
<p>If this option is disabled, TeamCity will not create a new user when the provided external email is unrecognized. This is helpful if you use a publicly available TeamCity server and want to limit access to it.</p>
</td>

</tr>

<tr>
<td>
Allow any GitLab user to log in
</td>
<td>
<p>Disabled by default.</p>
<p>If this option is enabled, users from any group can access the TeamCity server. To limit access to specific groups, use the <b>Restrict authentication</b> setting instead.</p>
<warning>If this option is enabled in combination with the <b>Allow creating new users on first login</b> option, any GitLab user could register on the TeamCity Server.</warning>
</td>
</tr>

<tr>

<td>

Restrict authentication

</td>

<td>

A comma-separated list of [groups'](https://docs.gitlab.com/ee/user/group/) IDs.

This list limits the set of users who can register or authenticate in TeamCity with their GitLab account  to the specified groups. When combined with the _Allow creating new users on the first login_ option,  this setting allows automatically registering users who have an email in one of the specified groups and do not have a user profile in TeamCity.

>Once registered on the TeamCity server, a user can create a password or token which will allow them to sign in to this server directly, bypassing the GitLab verification. If you delete a user from an organization in GitLab, remember to restrict their access or delete their user profile in TeamCity.

</td>

</tr>

</table>

### GitLab CE/EE

Users can sign in to TeamCity with a GitLab CE/EE account.

Before enabling this module, you need to configure a [GitLab CE/EE connection](integrating-teamcity-with-vcs-hosting-services.md) in the Root project's settings and a dedicated application in GitLab.

To sign in, click the GitLab icon above the login form and, after the redirect, approve the TeamCity application. If a user with your GitLab email is registered and this email is verified both [in TeamCity](enabling-email-verification.md) and in GitLab, this GitLab account will be mapped with the respective TeamCity user and you will be signed in. Otherwise, TeamCity will create a new user profile, unless the **Allow creating new users on the first login** option is disabled. It is also possible to [map existing TeamCity users](#User+Authentication+Settings) with GitLab CE/EE profiles.

>If you want to be recognized in TeamCity by your email, make sure this email is set as _public_ in GitLab.

<table>

<tr>
<td>Setting</td>
<td>Description</td>
</tr>

<tr>

<td>

Allow creating new users on the first login

</td>

<td>
<p>Enabled by default.</p>
<p>If this option is disabled, TeamCity will not create a new user when the provided external email is unrecognized. This is helpful if you use a publicly available TeamCity server and want to limit access to it.</p>
</td>

</tr>

<tr>
<td>
Allow any GitLab user to log in
</td>
<td>
<p>Disabled by default.</p>
<p>If this option is enabled, users from any group can access the TeamCity server. To limit access to specific groups, use the <b>Restrict authentication</b> setting instead.</p>
<warning>If this option is enabled in combination with the <b>Allow creating new users on first login</b> option, any GitLab user could register on the TeamCity Server.</warning>
</td>
</tr>

<tr>

<td>

Restrict authentication

</td>

<td>

A comma-separated list of [groups'](https://docs.gitlab.com/ee/user/group/) IDs.

This list limits the set of users who can register or authenticate in TeamCity with their GitLab account  to the specified groups. When combined with the _Allow creating new users on the first login_ option, this setting allows automatically registering users who have an email in one of the specified groups and do not have a user profile in TeamCity.

>Once registered on the TeamCity server, a user can create a password or token which will allow them to sign in to this server directly, bypassing the GitLab verification. If you delete a user from a group in GitLab, remember to restrict their access or delete their user profile in TeamCity.

</td>

</tr>

</table>

>If you reconnect a TeamCity server from one GitLab CE/EE server to another, TeamCity might not be able to recognize external users after this operation. This case requires reconfiguring user profiles manually. If you encounter any issues, please [contact our support](troubleshooting.md).
>
{style="warning"}

### Google

Since version 2022.10, users can sign in to TeamCity with a Google account.

Before enabling this module, you need to configure a [Google connection](configuring-connections.md#Google) in the Root project's settings.

To sign in, click the Google icon above the login form and, after the redirect, approve the TeamCity application. If a user with your Google email is registered and this email is verified [in TeamCity](enabling-email-verification.md), this Google account will be mapped to the respective TeamCity user, and you will be signed in. Otherwise, TeamCity will create a new user profile, unless the **Allow creating new users on the first login** option is disabled. It is also possible to [map existing TeamCity users](configuring-authentication-settings.md#User+Authentication+Settings) to Google profiles.

<table>

<tr>
<td>Setting</td>
<td>Description</td>
</tr>

<tr>

<td>

Allow creating new users on the first login

</td>

<td>
<p>Enabled by default.</p>
<p>If this option is disabled, TeamCity will not create a new user when the provided external email is unrecognized. This is helpful if you use a publicly available TeamCity server and want to limit access to it.</p>
</td>

</tr>

<tr>


<td>

Skip two-factor authentication

</td>

<td>

To reduce redundant verification, check this option if your organization already requires 2FA for users to log into their Google accounts.

If you want users to pass additional verification anyway, uncheck this setting.

Learn more: [](managing-two-factor-authentication.md#Reduce+Excessive+Authorization+Requests).

</td>

</tr>

<tr>

<td>

Allow users from all domains to log in, including gmail.com

</td>

<td>
<p>Disabled by default.</p>
<p>If this option is enabled, users from any domain can access the TeamCity server. To limit access to specific domains, use the <b>Restrict authentication</b> setting instead.</p>
<warning>If this option is enabled in combination with the <b>Allow creating new users on first login</b> option, any Google user could register on the TeamCity Server.</warning>
</td>

</tr>

<tr>

<td>

Restrict authentication

</td>

<td>

A comma-separated list of [organizations' domains](https://cloud.google.com/resource-manager/docs/creating-managing-organization). For example, `company.com,another.com`.

This list limits the set of users who can register or authenticate in TeamCity with their Google account to the users of the specified domains. 

When combined with the _Allow creating new users on the first login_ option, this setting allows automatically registering users who have an email in one of the specified domains and do not have a user profile in TeamCity.

>There is no synchronization of user profiles between Google and TeamCity. If you delete a user from the Google organization, you'll have to manually restrict their access in TeamCity.

</td>

</tr>

</table>

### JetBrains Space

Before enabling this module, you need to create a dedicated application in JetBrains Space and configure a connection to it in the Root project's settings, as described [here](configuring-connections.md#jetbrains-space-connection).

After the connection is configured, go to __Administration | Authentication__ and:
1. Click __Add module__ and choose the _JetBrains Space_ type.
2. Choose whether you want to allow creating new users on the first login. This is helpful if you use a publicly available TeamCity server and want to limit access to it.
3. Uncheck the **Skip two-factor authentication** setting if you want users to pass an additional 2FA verification when they log in (even if they pass it when log into Space). Learn more: [](managing-two-factor-authentication.md#Reduce+Excessive+Authorization+Requests).
4. Save the module.

To sign in, click the JetBrains Space icon above the TeamCity login form and, after the redirect, approve the TeamCity application.

### Azure DevOps Services

Before enabling this module, you need to create a [dedicated connection](configuring-connections.md#Azure+DevOps) to your Azure DevOps Services instance in the Root project's settings.

<table>

<tr>
<td>Setting</td>
<td>Description</td>
</tr>

<tr>
<td>
Skip two-factor authentication
</td>
<td>

To reduce redundant verification, check this option if your organization already requires 2FA for users to log into their Azure DevOps accounts.

If you want users to pass additional verification anyway, uncheck this setting.

Learn more: [](managing-two-factor-authentication.md#Reduce+Excessive+Authorization+Requests).

</td>
</tr>


<tr>
<td>

Allow creating new users on the first login

</td>
<td>
<p>Enabled by default.</p>
<p>If this option is disabled, TeamCity will not create a new user when the provided external email is unrecognized. This is helpful if you use a publicly available TeamCity server and want to limit access to it.</p>
</td>
</tr>

<tr>
<td>

Allow any Azure user to log in

</td>

<td>
<p>Disabled by default.</p>
<p>If this option is enabled, users from any organization can access the TeamCity server. To limit access to specific organizations, use the <b>Restrict authentication</b> setting instead.</p>
<warning>If this option is enabled in combination with the <b>Allow creating new users on first login</b> option, any Azure user could register on the TeamCity Server.</warning>
</td>

</tr>

<tr>

<td>

Restrict authentication

</td>

<td>

A comma-separated list of [Azure DevOps organizations](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/organization-management?view=azure-devops).

This list limits the set of users who can register or authenticate in TeamCity with their Azure account to the users of the specified organizations.

When combined with the _Allow creating new users on the first login_ option, this setting allows automatically registering users who have an account in one of the specified organizations and do not have a user profile in TeamCity.

</td>

</tr>

</table>

To sign in, click the Azure DevOps icon above the TeamCity login form and, after the redirect, approve the TeamCity application.


### HTTP SAML 2.0

This module enables single sign-on authentication in TeamCity via [SAML 2.0](https://en.wikipedia.org/wiki/SAML_2.0). It is confirmed to support [Okta](https://www.okta.com/), [OneLogin](https://www.onelogin.com/), [AWS SSO](https://aws.amazon.com/single-sign-on/), [AD FS](https://docs.microsoft.com/en-us/windows-server/identity/active-directory-federation-services), and other SSO providers.

The module relies on the verified third-party [SAML Authentication plugin](https://github.com/morincer/teamcity-plugin-saml).

To enable the module, in __Administration | Authentication__:
1. Click __Add module__ and choose the _HTTP-SAML.v2_ type.
2. Save the settings and go to __Administration | SAML Settings__.
3. Copy the _Single Sign-On URL (Recipient)_ value under the _Service Provider Configuration_ section. It should be entered in the settings of your SSO application, as described below.

Before configuring SAML settings in TeamCity, you need to create a dedicated application on your SSO provider's side. Here is an example quick instruction for Okta:
1. In the Okta dashboard, open __Applications__.
2. Click __Create app integration__ and choose the _SAML 2.0_ type.
3. Name the app and, on the __Configure SAML__ tab, enter the single sign-on URL of your TeamCity server which you copied in Step 3 of the above instruction.
4. Save the app.
5. On the __Assignments__ tab of the app's settings, click __Assign__ and select all the people from your Okta group who will be granted access to the TeamCity server.
6. On the __General__ tab, click __View Setup Instructions__. You will see the configuration values that need to be copied to TeamCity.

You can find the detailed how-to for Okta in the [plugin's repository](https://github.com/morincer/teamcity-plugin-saml/blob/master/docs/OktaSetup.md). Settings for other SSO providers may vary, but the key actions are the same:
* Enter the TeamCity single sign-on URL in the app's settings.
* Ensure that all the required SSO user profiles are associated with the app.
* Copy the values of the app's SSO URL, issuer URL, and certificate.

>Instead of configuring the app manually, you can download the [XML metadata file](#saml-metadata) with the TeamCity configuration in __SAML Settings__ and upload it to your SSO profile.

After the app is created, proceed with configuring __SAML Settings__ in TeamCity:

<table>

<tr><td>Setting</td><td>Description</td></tr>

<tr><td>

Identity provider configuration

</td>

<td>

Enter the values of the SSO URL, issuer URL, and certificate copied from the app's settings.

<anchor name="saml-metadata"/>

Optionally, you can upload an extra certificate whenever you are about to update one on the SSO provider's side. Having a new certificate preloaded in TeamCity will ensure uninterrupted trusted connection with the provider during and after the update.

</td></tr>

<tr><td>

Service provider configuration

</td>

<td>

Contains the autogenerated SSO URL of a recipient server, that is TeamCity. This URL has to be entered when configuring this integration on the SSO provider's side.

After you save the SAML settings for the first time, TeamCity exports them to the SP metadata XML file. The download link to this file is provided in this settings section. You can upload this file on your SSO provider's side to apply the TeamCity configuration automatically.

</td></tr>

<tr><td>

Create users automatically

</td>

<td>

If TeamCity detects that the email of a user who is signing in via SAML already belongs to some TeamCity user profile, it will authenticate them as this user.

If it does not recognize the email, the behavior depends on the value of this option:
* If enabled, it will create a new user profile with this email.
* If disabled, it will refuse to authorize this user.

</td></tr>

<tr><td>

Login Button Label

</td>

<td>

Defines the name of the SSO login button on the login form.

</td></tr>

<tr><td>

SAML Strict Mode

</td>

<td>

Checks that the request URL conforms to the expected callback URL (configured as the Teamcity root URL). We highly recommended leaving this option enabled for production environment, as an extra security check.

</td></tr>

<tr><td>

SAML Callback CORS Filer Exception

</td>

<td>

Adds an exception to the Teamcity [CORS filter](csrf-protection.md): if a `POST` request is sent to the SAML login callback URL and it contains a SAML message, it is considered CORS-safe.

</td></tr>

<tr><td>

Hide login form

</td>

<td>

Allows hiding a regular login form with a TeamCity internal username and password so users are not distracted on it.

</td></tr>

</table>

<seealso>
        <category ref="concepts">
            <a href="authentication-modules.md">Authentication Modules</a>
        </category>
</seealso>
