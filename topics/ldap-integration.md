[//]: # (title: LDAP Integration)
[//]: # (auxiliary-id: LDAP Integration)

LDAP integration in TeamCity has two levels: authentication (login) and users synchronization:
* __authentication__ allows you to login in to TeamCity using LDAP server credentials.
* once LDAP authentication is configured, you can enable LDAP __synchronization__ which allows the TeamCity user-set to be automatically populated with the user data from LDAP.
LDAP integration is generic and can be configured for Active Directory or other LDAP servers.

<note>

It is recommended to configure LDAP authentication on a test server before enabling it in the production one, because switching to an incorrectly configured authentication scheme may cause users' inability to log in to TeamCity.
</note>

LDAP integration might be not trivial to configure, so it might require some trial and error approach to get the right settings. Please review [Typical LDAP Configurations](typical-ldap-configurations.md). If a problem occurs, [LDAP logs](#Debugging+LDAP+Integration) should give you enough information to understand possible misconfigurations. If you are experiencing difficulties configuring LDAP integration after going through this document and investigating the logs, please [contact us](troubleshooting.md) and let us know your LDAP settings with a detailed description of what you want to achieve and what you currently get.

## Authentication

To allow logging into TeamCity with LDAP credentials, you need to configure LDAP connection settings in the `ldap-config.properties` file and enable LDAP authentication in the server's [Authentication section](configuring-authentication-settings.md).


When the "_Allow creating new users on the first login_" option is selected (by default) a new user account will be created on the first successful login. The TeamCity usernames for the new users will be derived from their LDAP data based on the configured setting. All newly created users belong to the [All Users](creating-and-managing-user-groups.md#%22All+Users%22+Group) group and have all roles assigned to this group. If some specific [roles](managing-roles-and-permissions.md) are needed for the newly registered users, these roles can [be granted](managing-roles-and-permissions.md#Managing+Roles) via the __All Users__ group.

TeamCity stores user accounts and details in its own database. For information on automatic user creation and automatic population of user details from LDAP, refer to the [Synchronization](#Synchronization) section.

### ldap-config.properties Configuration

LDAP integration settings are configured in the `<TeamCity Data Directory>/config/ldap-config.properties` file on the server.

Create the file by copying `<TeamCity Data Directory>/config/ldap-config.properties.dist` file and renaming it to the `<TeamCity Data Directory>/config/ldap-config.properties`; follow the comments in the file to edit the default settings as required.

The file uses the standard Java properties file syntax, so all the values in the file must be properly [escaped](https://java.sun.com/j2se/1.5.0/docs/api/java/util/Properties.html#load(java.io.InputStream)). For example, the following `java.naming.security.principal=DOMAIN\user` parameter should be escaped as `java.naming.security.principal=DOMAIN\\user`.

The file is reread on any modification: there is no need to restart the server to apply the changes.

It is strongly recommended backing up the previous version of the file: if you misconfigure LDAP integration, you may no longer be able to log in into TeamCity. The users who are already logged in are not affected by the modified LDAP integration settings because users are authenticated only on login.

The mandatory property in the `ldap-config.properties` file is `java.naming.provider.url` that configures the server and root DN. The property stores the URL to the LDAP server node that is used in following LDAP queries. For example, `ldaps://dc.example.com:636/CN=Users,DC=Example,DC=Com`. Note that the value of the property should use URL-escaping if necessary. For example, use `%20` if you need the space character.

<note>

For samples of `ldap-config.properties` file refer to the [Typical LDAP Configurations](typical-ldap-configurations.md) page.
The supported configuration properties are documented in comments in the `ldap-config.properties.dist` file.
</note>

### Configuring User Login

The general login sequence is as follows:
* based on the username entered in the login form by the user, an LDAP search is performed (defined by the `teamcity.users.login.filter` LDAP filter where the user-entered username is referenced via the `$login$` or `$capturedLogin$` substring within the users base LDAP node (defined by `teamcity.users.base`),
* if the search is successful, authentication (LDAP bind) is performed using the DN found during the search and the user-entered password,
* if the authentication is successful, TeamCity user is created if necessary and the user is logged in. The name of the TeamCity user is retrieved from an attribute of the found LDAP entry (the attribute name is defined via the `teamcity.users.username` property)

When users log in via LDAP, TeamCity does not store the user passwords. On each user login, authentication is performed by a direct login into LDAP with the credentials based on the values entered in the login form.

Note that in certain configurations (for example, with `java.naming.security.authentication=simple`) the login information will be sent to the LDAP server in the unencrypted form. For securing the connection, refer to [Sun documentation](https://java.sun.com/products/jndi/tutorial/ldap/security/sasl.html). Another option is to configure communications via the LDAPS protocol.

<!--[//]: # (Internal note. Do not delete. "LDAP Integrationd195e199.txt")-->    


#### Active Directory

The following template enables authentication against active directory:

Add the following code to the \<[TeamCity Data Directory](teamcity-data-directory.md)\>/config/ldap-config.properties file (assuming the domain name is `Example.Com` and domain controller is `dc.example.com`).

To use a more secure LDAPS connection (recommended), specify a URL in the corresponding format: `ldaps://dc.example.com:636/DC=Example,DC=Com`. URLs like `ldap://dc.example.com:389/DC=Example,DC=Com` allow you to establish a regular non-encrypted LDAP connection (note the different port). We recommend using LDAPS as a more secure alternative to the regular LDAP.


```Shell
java.naming.provider.url=ldaps://dc.example.com:636/DC=Example,DC=Com
java.naming.security.principal=<username>
java.naming.security.credentials=<password>
teamcity.users.login.filter=(sAMAccountName=$capturedLogin$)
teamcity.users.username=sAMAccountName
java.naming.security.authentication=simple
java.naming.referral=follow

```


<!--[//]: # (Internal note. Do not delete. "LDAP Integrationd195e234.txt")-->

### Advanced Configuration

If you need to fine-tune LDAP connection settings, you can add the `java.naming` options to the `ldap-config.properties` file: they will be passed to the underlying Java library. The default options are retrieved using `java.naming.factory.initial=com.sun.jndi.ldap.LdapCtxFactory`. Refer to the Java [documentation page](http://docs.oracle.com/javase/1.5.0/docs/guide/jndi/jndi-ldap.html) for more information about property names and values.

You can use an LDAP explorer to browse LDAP directory and verify the settings (for example, [http://www.jxplorer.org/](http://www.jxplorer.org/) or [http://www.ldapbrowser.com/softerra-ldap-browser.htm](http://www.ldapbrowser.com/softerra-ldap-browser.htm)).

There is an ability to specify failover servers using the following pattern:

```Shell
java.naming.provider.url=ldaps://ldap.mycompany.com:636 ldaps://ldap2.mycompany.com:636 ldaps://ldap3.mycompany.com:636

```

The servers are contacted until any of them responds. There is no particular order in which the address list is processed.

### Referral Chasing

When an LDAP client requests information from an LDAP server that does not have all required data, the server can return referral URLs to other servers to contact for this missing data. To disable this behavior, uncomment the `java.naming.referral=ignore` line in the `ldap-config.properties` file:

```
# Ignore referrals returned by LDAP server ("follow" by default). See also https://youtrack.jetbrains.com/issue/TW-35264
#java.naming.referral=ignore
```

## Synchronization

Synchronization with LDAP in TeamCity allows you to:
* Retrieve the user's profile data from LDAP
* Update the user groups membership based on LDAP groups
* Automatically create and remove users in TeamCity based on information retrieved from LDAP

TeamCity supports one-way synchronization with LDAP: the data is retrieved from LDAP and stored in the TeamCity database. Periodically, TeamCity fetches data from LDAP and updates users in TeamCity.

When synchronization is enabled, you can review the related data and run on-demand synchronization in the __Administration | LDAP Synchronization__ section of the [server settings](teamcity-configuration-and-maintenance.md).

### Common Configuration

You need to have LDAP authentication configured for the synchronization to function.

By default, the synchronization is turned off. To turn it on, add the following option to `ldap-config.properties` file:


```Shell
teamcity.options.users.synchronize=true

```

You also need to specify the following mandatory properties:
* `java.naming.security.principal` and `java.naming.security.credentials` — they specify the user credentials which are used by TeamCity to connect to LDAP and retrieve data,
* `teamcity.users.base` and `teamcity.users.filter` - these specify the settings to search for users
* `teamcity.users.username` - the name of the LDAP attribute containing the TeamCity user's username. Based on this setting LDAP entries are mapped to the TeamCity users.

No users are created or deleted on enabling user's synchronization. Check the [Creating and Deleting Users](#Creating+and+Deleting+Users) section for the related configuration.

### User Profile Data

When synchronization is properly configured, TeamCity can retrieve user-related information from LDAP (email, full name, or any custom property) and store it as TeamCity user's details. If updated in LDAP, the data will be updated in the user's profile in TeamCity. If modified in user's profile in TeamCity, the data will no longer be updated from LDAP for the modified fields. All the user fields synchronization properties store the name of LDAP field to retrieve the information from.

The user's profile synchronization is performed on user creation and also periodically for all users.

The list of supported user settings:
* `teamcity.users.username`
* `teamcity.users.property.displayName`
* `teamcity.users.property.email`
* `teamcity.users.property.plugin:vcs:<VCS type>:anyVcsRoot` — VCS username for all &lt;VCS type&gt; roots. The following VCS types are supported: `svn`, `perforce`, `jetbrains.git`, `cvs`, `tfs`, `vss`, `starteam`.

Example properties can be seen by configuring them for a user in the web UI and then listing the properties via [REST API](https://www.jetbrains.com/help/teamcity/rest/manage-users.html).

There is an __experimental__ feature which allows mapping user profile properties in TeamCity to a formatted combination of LDAP properties rather than to a specific property on user synchronization.   
To enable the mapping, add `teamcity.users.properties.resolve=true` to `ldap-config.properties`.   
Then you can use the `%`-references to LDAP attributes in the form of `%ldap.userEntry.<attribute>%` in the user property definitions.

### User Group Membership

TeamCity can automatically update users membership in groups based on the LDAP-provided data.

__To configure Group membership:__
1. Create groups in TeamCity manually.
2. Specify the mapping of LDAP groups to TeamCity groups in the `<TeamCity Data Directory>/config/ldap-mapping.xml` file. Use the `ldap-mapping.xml`.[`dist file`](teamcity-data-directory.md) as an example: TeamCity user groups are determined by the [Group Key](creating-and-managing-user-groups.md), LDAP groups are specified by the group DN.
3. Set the required properties in the `ldap-config.properties` file, the __groups settings__ section:
   * `teamcity.options.groups.synchronize` — enables user group membership synchronization
   * `teamcity.groups.base` and `teamcity.groups.filter` — specifies the LDAP base node and filter to find the groups in LDAP (those configured in the `ldap-mapping.xml` file should be a subset of the groups found by these settings)
   * `teamcity.groups.property.member` specifies the LDAP attribute holding the members of the group. LDAP groups should have all their members listed in the specified attribute.

Note that TeamCity only operates with the users matched by `teamcity.users.base` and `teamcity.users.filter `settings, so only the users found by these properties will be processed during the group membership synchronization process.

On each synchronization run, TeamCity updates the membership of users in the groups configured in the mapping.   
By default, TeamCity synchronizes membership only for users residing directly in the groups.

__To map nested LDAP groups to TeamCity__:
* either specify `teamcity.groups.retrieveUsersFromNestedGroups=true` in the `ldap-config.properties` file and make sure all the group hierarchy is matched by the `teamcity.groups.base/teamcity.groups.filter` settings
* or copy your LDAP groups structure in TeamCity groups, together with the group inclusions. Then configure the mapping between the TeamCity groups and corresponding LDAP groups.

If either an LDAP group or a TeamCity group that is configured in the mapping is not found, an error is reported. You can review the errors found during the last synchronization run in the __Administration | LDAP Synchronization__ section of the [server settings](teamcity-configuration-and-maintenance.md).

See also [example settings](typical-ldap-configurations.md#Active+Directory+With+Group+Synchronization) for Active Directory synchronization.

### Creating and Deleting Users

TeamCity can automatically create users in TeamCity, if they are found in one of the mapped LDAP groups and groups synchronization is turned on via the `teamcity.options.groups.synchronize` option.

By default, automatic user creation is turned off. To turn it on, set the `teamcity.options.createUsers` property to `true` in the `ldap-config.properties` file.

TeamCity can automatically delete users in TeamCity if they cannot be found in LDAP or do not belong to an LDAP group that is mapped to the predefined "__All Users__" group. By default, automatic user deletion is turned off as well; set the `teamcity.options.deleteUsers` property to turn it on.

### Username migration

The username for the existing users can be updated upon first successful login. For instance, suppose the user previously logged in using 'DOMAIN\user' name, thus the string `DOMAIN\user` was stored in TeamCity as the username. To synchronize the data with LDAP, the user can change the username to 'user' using the following options:

```Shell
teamcity.users.login.capture=DOMAIN\\\\(.*)
teamcity.users.login.filter=(cn=$login$)
teamcity.users.previousUsername=DOMAIN\\$login$

```

The first property allows you to capture the username from the input login and use it to authenticate the user (can be particularly useful when the domain 'DOMAIN' isn't stored anywhere in LDAP). The second property `teamcity.users.login.filter` allows you to fetch the username from LDAP by specifying the search filter to find this user (other mandatory properties to use this feature: `teamcity.users.base` and `teamcity.users.username`). The third property allows you to find the `DOMAIN\user` username when login with just 'user', and replace it with either the captured login, or with the username from LDAP.

Note that if any of these properties are not set or cannot be applied, the username isn't changed (the input login name is used).

<!--[//]: # (Internal note. Do not delete. "LDAP Integrationd195e537.txt")-->    

More configuration examples are available [here](typical-ldap-configurations.md).

## Miscellaneous

### Switching to LDAP Authentication

When you add the LDAP authentication module on a TeamCity server which already has users, the users can still log in using the previous authentication modules credentials (unless you remove the modules). On a successful LDAP login, LDAP retrieves a username as configured by the `teamcity.users.username` property and if there is already a user with such TeamCity username, user logs in as the matching user. If there is no existing user, a new one is created with the username retrieved from LDAP.

### Scrambling credentials in ldap-config.properties file

The `java.naming.security.credentials` property can store the password either in the plain-text or scrambled form. TeamCity needs the raw password value when authenticating in a LDAP server, so the password should be stored in a reversible form.   
You can get the scrambled value using the HTTP request below and then set the property to the scrambled value. When adding the scrambled value to the `java.naming.security.credentials` property, it is necessary to include the complete response of the request below. The scrambled entry should look like: `java.naming.security.credentials=scrambled:1234567890abcdef`.

>Scrambling is not encryption: it protects the password from being easily remembered when seen occasionally, but it does not protect against getting the real password value when someone gets the scrambled password value.
> 
{style="warning"}

To get the scrambled password value, execute the following HTTP request under a user who have the System Administrator role granted (i.e. you can just open the URL in the same browser that you use to access TeamCity):

```
GET {teamcity_url}/app/rest/debug/values/password/scrambled?value=<text to scramble>
```

>Note that this approach is relevant only for LDAP configurations. To get a scrambled value in other configurations, you can use the [`addSecureToken`](https://www.jetbrains.com/help/teamcity/rest/projectapi.html#addSecureToken) REST API method. When used for a project with disabled versioned settings, this method returns a string with the encrypted password. When used for a project with enabled versioned settings, it creates a new secure token and returns the token ID.

<!--[//]: # (Internal note. Do not delete. "LDAP Integrationd195e594.txt")-->

### Debugging LDAP Integration

Internal LDAP logs are stored in `logs/teamcity-ldap.log*` files in the [server logs](teamcity-server-logs.md). If you encounter an issue with LDAP configuration, it is advised that you look into the logs as the issue can often be figured out from the messages in there. To get detailed logs of LDAP login and synchronization processes, use the "debug-ldap" [logging preset](teamcity-server-logs.md).

If you want to [report an LDAP issue to us](troubleshooting.md), make sure to include LDAP settings (`<TeamCity Data Directory>/config/ldap-config.properties` and `ldap-mapping.xml` files, with the password masked in `ldap-config.properties`) and debug LDAP logs fully covering a login/synchronization sequence (include all `teamcity-ldap.log*` files). Make sure to describe the related structure of your LDAP (noting the attributes of the related LDAP entities) and detail expected/actual behavior. The archive with the data can be sent to us via [one of these methods](reporting-issues.md#Uploading+Large+Data+Archives).
