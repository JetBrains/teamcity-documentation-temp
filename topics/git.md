[//]: # (title: Git)
[//]: # (auxiliary-id: Git)
<!--[//]: # (Internal note. Do not delete. "Gitd153e3.txt" "Git \(JetBrains\)d152e3.txt")-->  
 
TeamCity supports Git out of the box. Git source control with Azure DevOps Services is supported (see authentication notes [below](#Authenticating+to+Azure+DevOps+Services)).

This page contains description of the Git-specific fields of the VCS root settings.  
For common VCS root properties, see [this section](configuring-vcs-roots.md#Common+VCS+Root+Properties).

>The Git command-line client needs to be installed on the agents if the [agent-side checkout](vcs-checkout-mode.md#agent-checkout) is used.
>
{style="note"}

>Git versions earlier than 2.10.0 will be deprecated in the future versions of TeamCity. If you get the related warning on running a build on some agent, we suggest that you update Git (the one specified in the `TEAMCITY_GIT_VERSION` parameter) and restart the agent.
>
{style="warning"}

__Important notes__:

* [Remote Run](remote-run.md) and [Pre-Tested Commit](pre-tested-delayed-commit.md) are supported in the [IntelliJ IDEA](intellij-platform-plugin.md) plugin; with the [Visual Studio Add-in](visual-studio-addin.md), use the [Branch Remote Run Trigger](branch-remote-run-trigger.md).
* Initial Git [checkout](build-checkout-directory.md#Checkout+Process) may take significant time (sometimes hours), depending on the size of your project history, because the whole project history is downloaded during the initial checkout.

## Native Git for VCS-related operations on the server
{instance="tc"}

TeamCity can now use native Git as the default option for Git operations on the server.
Switching to native Git improves the performance of the checking for changes operations on the server
in comparison with the previously used JGit implementation. It also fixes a number of issues related to large Git repositories.

Before switching, make sure a [native Git client](https://git-scm.com/downloads) version 2.29 or later is installed on your server machine,
and the path to its executable is specified in the `PATH` environment variable.
Alternatively, you can set the full path to the executable
via the `teamcity.server.git.executable.path` [internal property](server-startup-properties.md#TeamCity+Internal+Properties)
(no server restart is required). On Windows, remember to use double backslashes in the path.

To switch your TeamCity server to native Git, go to __Administration | Diagnostics__ and open the __Git__ tab.
Here you can test the connection via native Git in any VCS root on your server.
If you choose to test all VCS roots, TeamCity will check whether they successfully connect via JGit and then test their connection via native Git.
This measure helps ensure that none of your pipelines will break after switching to native Git.
If the connection test is successful, you can enable the native Git support on your server(s).

>See [known issues](known-issues.md#Known+issues+of+native+Git+checkout) of the current native Git implementation in TeamCity.
> 
{style="warning"} 

## Native Git for VCS-related operations on the server
{id="Native+Git" instance="tcc"}

TeamCity can now use native Git for VCS-related operations on the server.
Using native Git improves the performance of the checking for changes operations on the server
in comparison with the previously used JGit implementation. It also fixes a number of issues related to large Git repositories.

See [known issues](known-issues.md#Known+issues+of+native+Git+checkout) of the current native Git implementation in TeamCity.

## General Settings

* **Fetch URL** — the URL of the remote Git repository used for fetching data from the repository.

   You can override the fetch URL for individual agents to allow them to use a closer proxy instead of the original VCS hosting. To do so, open a required agent's [conf/buildAgent.properties file](configure-agent-installation.md) and add the redirection rule as follows: `teamcity.git.fetchUrlMapping.<name> = <source URL> => <target URL>`. For example:
   
   ```
   teamcity.git.fetchUrlMapping.firstrule = https://example.com/org/test.git => http://proxy.com/test.git
   ```
   
   You can use partial addresses and the asterisk (`*`) wildcard to set up proxies for all fetch URLs that match the pattern. For example, the following rule allows an agent to use the `http://proxy.com/test/test.git` URL instead of the original `https://example.com/org/test/test.git`:
   
   ```
   teamcity.git.fetchUrlMapping.secondrule = https://example.com/org/* => http://proxy.com/
   ```
   
   Note that along with a fetch URL a VCS root also stores authentication settings required to access a repository. As such, replacing fetch URL is in effect only for VCS roots that use **Anonymous** or **Private key** authentication modes. Other modes do not guarantee an agent will be able to access a repository (for example, a refreshable token issued for a source VCS will not be accepted by a proxy hosting).


* **Push URL** — the URL of the target remote Git repository used for pushing annotated tags created via [VCS labeling](vcs-labeling.md) build feature to the remote repository. If blank, the fetch URL is used.

<anchor name="defaultBranch"/>

* **Default branch** - the [default branch](working-with-feature-branches.md#Default+Branch). Parameter references are supported here. Default value is `refs/heads/master`.

   <note>
   
   You can configure Git-plugin to fetch all heads by adding the `teamcity.git.fetchAllHeads=true` [custom parameter](typed-parameters.md).
   
   </note>


* **Branch specification** — lists the patterns for branch names, required for [feature branches](working-with-feature-branches.md#Configuring+Branches) support. The matched branches are monitored for changes in addition to the default branch. The syntax is similar to checkout rules: `+|-:branch_name`, where `branch_name` is specific to the VCS, i.e. `refs/heads/` in Git (with the optional `*` placeholder).

   <include from="branch-filter.md" element-id="OR-syntax-tip"/>

* **Use tags as branches** — allows monitoring / checking out git [tags](vcs-labeling.md) as branches making branch specification match tag names as well as branches (for example,`+|-:refs/tags/<tag_name>`). By default, tags are ignored.

* **Username style** — defines a way TeamCity reports username for a VCS change. Changing the username style will affect only newly collected changes. Old changes will continue to be stored with the style that was active at the time of collecting changes.

* **Submodules** — specifies whether submodule repositories should be checked out. For multi-level submodule setup, you can choose between the "Checkout" (recursively fetches the entire repository tree) and "Non-recursive checkout" (fetches only those submodules that are directly referenced by the main repository) modes. In the diagram below, submodules A and B are available in both modes, whereas submodules C and D require the recursive "Checkout" mode.

  ```
  Main_Repo
  |
  |_________ Tier 1 Submodule A
  |                 |
  |                 |_________ Tier 2 Submodule C
  |
  |_________ Tier 1 Submodule B
                   |
                   |_________ Tier 2 Submodule D
  ```

  Submodule repositories should either not require authentication or use the same protocol and accept the same authentication as configured in the VCS root. Otherwise, you need to configure additional credentials to access these repositories as described in the [](#LFS+and+Submodules+Support) section.

* **Username for tags/merge**  — a custom username used for [labeling](vcs-labeling.md).



### Branch Matching Rules
{id="branchMatchingRules" help-id="Branch Matching Rules"}

* If the branch matches a line without patterns, the line is used.
* If the branch matches several lines with patterns, the best matching line is used.
* If there are several lines with equal matching, the one below takes precedence.    
Everything that is matched by the wildcard will be shown as a branch name in the TeamCity interface. For example, `+:refs/heads/*` will match `refs/heads/feature1` branch, but in the TeamCity interface you'll see only `feature1` as a branch name.    
The short name of the branch is determined as follows:
* if the line contains no brackets, then full line is used, if there are no patterns or part of line starting with the first pattern-matched character to the last pattern-matched character.
* if the line contains brackets, then part of the line within brackets is used. When branches are specified here, and if your build configuration has a VCS trigger and a change is found in some branch, TeamCity will trigger a build in this branch.

### Supported Git Protocols

The following protocols are supported for Git repository URL:
* ssh: (for example, `ssh://git.somwhere.org/repos/test.git`, `ssh://git@git.somwhereElse.org/repos/test.git`, SCP-like syntax: `git@git.somwhere.org:repos/test.git`)

<note>

The SCP-like syntax requires a colon after the hostname, while a usual SSH URL does not. This is a common source of errors.

</note>

* Git: (for example, [`git://git.kernel.org/pub/scm/git/git.git`](git://git.kernel.org/pub/scm/git/git.git))
* HTTP: (for example, [`http://git.somewhere.org/projects/test.git`](http://git.somewhere.org/projects/test.git){nullable="true"})
* file: (for example, [`file:///c:/projects/myproject/.git`](file:///c:/projects/myproject/.git))

<note>

When you run TeamCity as a Windows service, it cannot access mapped network drives and repositories located on them.

</note>

## Authentication Settings

* **Anonymous** — select this option to clone a repository with anonymous read access.

<anchor name="passwordAuth"/>

* **Password / personal access token** — specify a valid __username__ (if there is no username in the clone URL, the username specified in this field overrides the username from the URL) and a __password__ to be used to clone the repository.    
For the [agent-side checkout](vcs-checkout-mode.md), it is supported __only if Git 1.7.3\+ client__ is installed on the agent. See [TW-18711](https://youtrack.jetbrains.com/issue/TW-18711).    
For Git hosted from Team Foundation Server 2013, specify NTLM credentials here.
   
   You can use a personal access token instead of a password to authenticate in GitHub, Azure DevOps Services, GitLab, JetBrains Space, and Bitbucket. When connecting to Azure DevOps, remember to set the _Code_ access scope to _Code (read) / Code (read and write) for versioned settings_ in the repositories you are about to access from TeamCity.

   >Beginning August 13, 2021, GitHub [will no longer accept passwords](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) when authenticating Git operations on GitHub.com.   
   >We highly recommend that you use an access token or SSH key instead of password when configuring a VCS root for a GitHub.com repository.
   >
   {style="warning"}

   When using an existing Bitbucket Cloud, Bitbucket Server, GitLab or Azure DevOps Services connection to create a VCS Root, 
   TeamCity will use a refreshable token instead of the password.

<anchor name="refresh-token"/>

* **Refreshable token** — if a VCS root that fetches data from a GitHub, GitHub App, Bitbucket Server, Bitbucket Cloud, Azure DevOps, GitLab, or JetBrains Space was configured using a TeamCity [connection](configuring-connections.md), refreshable tokens are enabled by default.

    <include from="common-templates.md" element-id="rat-single"/>

    You can specify a username if the clone (fetch) URL does not have it. The username specified here overrides the username from the URL.


* **Private Key** — valid only for SSH protocol. A private key must be in the __OpenSSH format__.

   >Recent versions of OpenSSH no longer generate keys in PEM format by default. The new OpenSSH format is not yet supported by TeamCity (see [TW-53615](https://youtrack.jetbrains.com/issue/TW-53615)). Use the following command to generate TeamCity-compatible keys: `ssh-keygen -t rsa -m PEM`.
   >
   {style="note"}

   Select one of the options from the __Private Key__ list and specify a valid username (if there is no username in the clone URL; the username specified here overrides the username from the URL).    
   Available __Private Key__ options:
   
   <snippet id="ssh-key-options">

  * __Uploaded Key__ — select this option to utilize the [key(s) uploaded to the project](ssh-keys-management.md).
  * __Default Private Key__ — select this option to utilize the keys available on the file system in the default locations used by common ssh tools: the mapping specified in `<USER_HOME>/.ssh/config` if the file exists or the private key file `<USER_HOME>/.ssh/id_rsa` (the files are required to be present on the server and also on the agent if the [agent-side checkout](vcs-checkout-mode.md) is used).
  * __Custom Private Key__ — supported __only for [server-side checkout](vcs-checkout-mode.md)__. Fill the __Private Key Path__ field with an absolute path to the private key file on the server machine. If the key is encrypted, specify the passphrase in the corresponding field.
   
   </snippet>


For all available options to connect to GitHub, see the [comment](https://youtrack.jetbrains.com/issue/TW-16194#comment=27-475793).

### Authenticating to Azure DevOps Services

If you use Git source control with Azure DevOps Services, you can use both [Azure DevOps OAuth](configuring-connections.md#azure-devops-connection) and [Azure DevOps PAT](configuring-connections.md#Azure+DevOps+PAT+Connection) connections.

## Server Settings

These are the settings used in case of the server-side [checkout](vcs-checkout-mode.md).

<table><tr>

<td width="150">

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Convert line-endings to CRLF
{id="serverAutoCRLF" help-id="serverAutoCRLF"}

</td>

<td>

Convert line-endings of all text files to CRLF (works as setting `core.autocrlf=true` in a repository config). When not selected, no line-endings conversion is performed (works as setting `core.autocrlf=false`). Affects the server-side checkout only. A change to this property causes a clean checkout.

</td></tr></table>

<anchor name="Git-agentSettings"/>

## Agent Settings
{id="GitAgentSettings" help-id="Git Agent Settings"}

These are the settings used in case of the agent-side [checkout](vcs-checkout-mode.md).   
Note that the agent-side checkout has limited support for SSH. The only supported authentication methods are "Default Private Key" and "Uploaded Private Key".   
If you plan to use the [agent-side checkout](vcs-checkout-mode.md), you need to have Git 1.6.4+ installed on the agents.

<table><tr>

<td width="150">

Option

</td>

<td>

Description

</td></tr><tr>

<td>

Path to Git

</td>

<td>

Provide the path to a Git executable to be used on the agent. When set to `%\env.TEAMCITY_GIT_PATH%`, the automatically detected Git will be used, see [Git executable on the agent](#agentGitPath) for details.

</td></tr>

<tr>

<td>

Checkout policy
{id="git-checkout-policy" help-id="git-checkout-policy"}

</td>

<td>

This setting defines how TeamCity performs a checkout to a build agent.

* __Use mirrors__: recommended for long-lived agents. With this option selected, TeamCity creates a remote repository cache on the agent machine under the `system/caches/git` directory. The cache is then added as [alternates](https://git-scm.com/docs/gitrepository-layout) when updating the [build checkout directory](build-checkout-directory.md). To speed up the following checkouts of this repository, the agent will reuse the cache in all the builds with the same fetch URL. This also speeds up clean checkout (as only the [build working directory](build-working-directory.md) is cleaned) and saves disk space (as the mirror is the only clone of the given repository on the agent).
* __Do not use mirrors__: choose to check out right into the [build checkout directory](build-checkout-directory.md), without creating a mirror. Less optimal in terms of disk usage than mirrors, but preserved for backwards compatibility with existing configurations.
* __Shallow clone__: recommended for short-lived agents; for example, for disposable cloud agents being terminated after every build. Creates a [shallow clone](https://git-scm.com/docs/git-clone) by fetching only one required revision, which allows saving time on the build start. Choose this option only if you do not require a Git commit history in your builds and if your cloud image does not contain a fresh Git mirror. You can enforce this option on certain agents by specifying the `teamcity.git.shallowClone=true` [agent configuration property](configure-agent-installation.md).
* __Auto__: TeamCity will automatically apply one of the above approaches depending on the `teamcity.cloud.agent.terminate.after.build` [agent configuration property](configure-agent-installation.md) and on the mirror presence on the agent.

>Read how to add a [Git mirror on a cloud agent](#Git+mirrors+on+cloud+agents).
>
{instance="tc"}

</td></tr>

<tr>

<td>

Clean Policy/Clean Files Policy

</td>

<td>

Specify here when the `git clean` command is to run on the agent, and which files are to be removed.

If a build configuration depends on multiple VCS roots, we suggest that you configure separate agent checkout directories for each of these roots, using [VCS checkout rules](vcs-checkout-rules.md). This way, `git clean` will never delete these checkout directories during cleaning.


</td></tr>
</table>

<tip instance="tc">

To configure a connection from a TeamCity server running behind a proxy to a remote Git repository, see [this section](how-to.md#Configure+TeamCity+to+Use+Proxy+Server+for+Outgoing+Connections).

</tip>

### Git executable on the agent
{id="agentGitPath" help-id="Git executable on the agent"}

TeamCity needs Git command line client version 1.6.4\+ on the agent in order to use the agent-side checkout.

The recommended approach is to ensure that the git client is available in `PATH` of the TeamCity agent and leave the "Path to git" setting in the VCS root blank.   
If you only have the git command line on some machines, set "Path to git" setting in the VCS root to the `%\env.TEAMCITY_GIT_PATH%` value.

Instead of adding Git to the agent's PATH, you can set the `TEAMCITY_GIT_PATH` environment variable (or `env.TEAMCITY_GIT_PATH` property in the agent's [`buildAgent.properties`](configure-agent-installation.md) file) to the full path to the git executable.

If `TEAMCITY_GIT_PATH` is not defined, the Git agent plugin tries to detect the installed git on the launch of the agent. It first tries to run git from the following locations:
* for Windows — it tries to run `git.exe` at:
   * `C:\Program Files\Git\bin`
   * `C:\Program Files (x86)\Git\bin`
   * `C:\cygwin\bin`
* for \*nix — it tries to run `git` at:
   * `/usr/local/bin`
   * `/usr/bin`
   * `/opt/local/bin`
   * `/opt/bin`

If Git is not found in any of these locations, it tries to run the git accessible via the `PATH` environment variable.   
If a compatible git (1.6.4\+) is found, it is reported in the `TEAMCITY_GIT_PATH` environment variable. This variable can be used in the __Path to git__ field in the [VCS root](vcs-root.md) settings. As a result, the configuration with such a VCS root will run only on the agents where Git was detected or specified in the agent properties.

### Git mirrors on cloud agents
{help-id="Git mirrors on cloud agents" instance="tc"}

By default, TeamCity creates a [mirror](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/duplicating-a-repository), that is a copy, of your Git repository under the agent's `system/git` directory. To save time and disk space on fetching source files, TeamCity points to this mirror via the Git alternate mechanism when updating the checkout directory for a build.

Comparing to self-hosted TeamCity agents, cloud agents require extra steps to add a Git mirror:

1. When [preparing a cloud image](teamcity-integration-with-cloud-solutions.md#Preparing+Virtual+Machine), clone the repository under the agent image's `system/git` directory. If necessary, you can store multiple `*.git` directories side by side.
2. Create a `map` file under the `system/git` directory and describe the mapping between the original repository and its mirror. For example,   
   ```Text

   ssh://git@<host>/<git_folder>.git = <git_folder>.git

   ```

When starting a build, a cloud agent will check the mirrors specified in the `map` file and fetch the difference between the required origin and its mirror. The origin URL in the `map` file must match the URL set in the VCS root.   
This way, builds will run significantly faster, with no need to check out the whole remote repository every time the new cloud agent starts.

>Over time, the Git remote origin accumulates many new commits comparing to its mirror in the agent image. If the checkout becomes noticeably slower, try updating the local mirror with the recent commits. It is also worth updating when the agent image is rebuilt due to OS or other upgrades.   
> Alternatively, you can store the `system/git` directory in a persistent volume, so it keeps all the updates even when a cloud agent is destroyed, and configure its automatic mounting on each newly created agent.

## Configuring Git Garbage Collection on Server
{id="Git_gc" help-id="Configuring Git Garbage Collection on Server" instance="tc"}

TeamCity server maintains a local clone for every Git repository used in the VCS roots configured on the server. Since the server performs fetch in those clones many times a day, the clone needs regular optimization to maintain predictable performance. If the Git garbage collection for the clone was not run for a long time, the process of collecting changes may slow down or start to report memory-related errors.  
TeamCity can automatically run `git gc` periodically when the native Git client can be found on the server. Inability to run Git GC results in a related health report.

To fix the warning / meet automatic git gc requirements, perform the following:
1. Install a native Git client manually on the TeamCity server.
2. Specify the path to the Git executable:
   * Add the directory with the executable to the `PATH` environment variable and restart the server, _or_
   * Set the full path to the executable in the `teamcity.server.git.executable.path` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) without the server restart. On Windows, remember to use double backslashes in the path.
   
When TeamCity runs Git garbage collection, the details are logged into the [`teamcity-cleanup.log`](teamcity-server-logs.md). If git garbage collection fails, a corresponding warning is displayed.

TeamCity executes Git garbage collection until the total time doesn't exceed 5 hours quota; the quota can be changed using the `teamcity.server.git.gc.quota.minutes` [internal property](server-startup-properties.md#TeamCity+Internal+Properties).   
Git garbage collection is executed every night at 2 AM. This can be changed by specifying the internal property with a cron expression like this: `teamcity.git.cleanupCron=0 0 2 * * ?`. If the `git gc` process works slowly and cannot be completed in the allotted time, check the `git-repack` configuration in the default Git configuration files (for example, you can increase `--window-memory` to improve the `git gc` performance).

If the local Git clones need some kind of manual maintenance, you can find them under the `<TeamCity Data Directory>/system/caches/git` directory. The `map` file in the directory contains mapping between the repository URL and the subdirectory storing the bare clone of the repository.

## LFS and Submodules Support

### Common Information

Submodules and LFS are integral parts of many complex software products whose source code is stored in Git-based version control systems.

* [Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) allow you to keep Git repositories as subdirectories of other Git repositories. For example, you may want to include a microservice developed by another team in your organization, or a 3rd-party open source project that your code relies on. As a result, you can clone a repository into your parent project while keeping commits separate.
* [Large File Storage (LFS)](https://git-lfs.com) allows you to drastically reduce the size of your repository by moving bulky files (media files, databases, and so on) to an external storage, and replacing these files with pointers.


Although different in their scenarios and implementation mechanisms, both concepts suggest incorporating essential project components from external sources (a file hosted on a remote LFS server or another repository within a VCS). This means your TeamCity project needs to authenticate to different resources to check out source files.


### Additional Credentials

If your repository imports submodules hosted on the same VCS and these imported repositories can be accessed via the same credentials stored in your [](vcs-root.md), you do not need to perform any additional modifications. TeamCity will check out all required source files using a single set of credentials.

Otherwise, if TeamCity needs to access an external LFS server or a different VCS that hosts required submodules, you will need to add three [configuration parameters](configuring-build-parameters.md) to your **project**:

```
teamcity.git.https.credentials.<ALIAS>.url = https://example.com/...
teamcity.git.https.credentials.<ALIAS>.username = johndoe
teamcity.git.https.credentials.<ALIAS>.password = 081ef11uh
```

* **&lt;ALIAS&gt;** is a custom string that groups your `teamcity.git.https.credentials...` properties into sets of three, and is used to identify required properties. For example, if your GitHub repo should import bulky files from [Sonatype Nexus](https://help.sonatype.com/en/git-lfs-repositories.html) LFS and additional submodules from [](configuring-connections.md#Azure+DevOps), you will need six properties. Three of them can have the `nexus` alias while the remaining have the `azure` alias. When TeamCity needs to access an Azure submodule repository, it will notice the URL is stored in an `...azure.url` property and look for matching properties with the same alias: `...azure.username` and `...azure.password`.

* **URL** is an HTTP(S) fetch URL for a submodule repo or an HTTP(S) link to an LFS storage. For example, `https://github.com/username/submodule-repo-name.git` or `https://mynexus.com/repository/repo-name/info/lfs`. SSH protocols are currently not supported.

* **Username** and **Password** store credentials for the corresponding service. Note that all limitations and guidelines that are normally in effect for accessing regular repositories apply for LFS/submodule checkout as well. For example, the `...password` property should store an access token instead of a regular account password since the latter option is being continuously discontinued by the majority of Git hostings. The `...username` property should also store a value that can be used in conjunction with an access token (for example, a regular account name for GitHub and GitLab or "x-token-auth" for Bitbucket Cloud). See also:
    * [GitHub tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#using-a-personal-access-token-on-the-command-line)
    * [GitLab tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#clone-repository-using-personal-access-token)
    * [Bitbucket Cloud tokens](https://support.atlassian.com/bitbucket-cloud/docs/using-access-tokens/)
    * [Bitbucket Server and Data Center tokens](https://confluence.atlassian.com/bitbucketserver/http-access-tokens-939515499.html)


### Limitations and Tips

* Configuration properties that store additional credentials should be configured for a TeamCity project, not a build configuration.
* For security reasons, switch all `...password` parameters to the [password type](typed-parameters.md). This will ensure your tokens and passwords are hidden from TeamCity UI, build logs, Kotlin DSL and REST API payloads. In addition, we recommend enabling the **Read-only** setting for `teamcity.git.https.credentials.<ALIAS>.url` parameters to prevent potential attackers from writing their own handcrafted URL to this parameter and stealing your authorization tokens.

  <img src="dk-lfsTips.png" width="706" alt="Secret type and read-only property"/>

* We recommend using Git LFS version 2.12.1 or later as earlier versions come with a [vulnerability exploit](https://github.com/git-lfs/git-lfs/security/advisories/GHSA-4g4p-42wc-9f3m).
* TeamCity supports Git LFS only for the [agent-side checkout](vcs-checkout-mode.md).
* SSH protocol is not supported for accessing submodule repositories and LFS files.

## Internal Properties
{id="internalProperties" help-id="Internal Properties" instance="tc"}

For Git VCS, it is possible to configure the following [internal properties](server-startup-properties.md#TeamCity+Internal+Properties):

<table><tr>

<td>

Property


</td>

<td width="20%">

Default


</td>

<td>

Description


</td></tr><tr>

<td>

teamcity.git.idle.timeout.seconds

</td>

<td>

1800

</td>

<td>

The idle timeout for communication with the remote repository. If no data were sent or received during this timeout, the plugin throws a timeout error to prevent hanging of the process forever.

>The idle timeout can also be set [on the agent side](#git-agent-config).

</td></tr><tr>

<td>

teamcity.git.fetch.timeout

</td>

<td>

1800


</td>

<td>

(deprecated) Override of `teamcity.git.idle.timeout.seconds` for the `git fetch` operation

</td></tr><tr>

<td>

teamcity.git.fetch.separate.process

</td>

<td>

true

</td>

<td>

Defines whether TeamCity runs `git fetch` in a separate process

</td></tr><tr>

<td id="max-memory">

teamcity.git.fetch.process.max.memory

</td>

<td>

</td>

<td>

<note>
  
Starting from TeamCity 2019.2, it is recommended to disable this property.   
By default, TeamCity starts nested Java processes for `git fetch` and `git patch` and automatically selects `-Xmx` for these processes.

</note>
 
This property provides the explicit `-Xmx` and disables the automatic `-Xmx` setup.

Ensure the server machine has enough memory as the memory configured will be used in addition to the main server process and there can be several child processes doing `git fetch` and `git patch`, each using the configured amount of the memory. For large repositories requiring heap memory greater than `-Xmx1024m` for Git fetch, [switching to 64-bit Java](configure-server-installation.md#Configure+Memory+Settings+for+TeamCity+Server) may be needed.
{instance="tc"}

</td></tr>

<tr>

<td id="max-memory-limit">

teamcity.git.fetch.process.max.memory.limit

</td>

<td>

</td>
<td>

By default, TeamCity starts nested Java processes for `git fetch` and `git patch` and automatically selects `-Xmx` for these processes.

This property specifies the maximum possible `-Xmx` value for `git fetch` or `git patch` that TeamCity can set automatically.

</td>


</tr>

<tr>

<td>

teamcity.git.monitoring.expiration.timeout.hours

</td>

<td>

24

</td>

<td>

</td></tr><tr>

<td>

teamcity.server.git.gc.enabled

</td>

<td>

false

</td>

<td>

Whether TeamCity should run `git gc` during the server clean-up (native git is used)

</td></tr><tr>

<td>
<anchor name="git-executable-path"/>

teamcity.server.git.executable.path

</td>

<td>

git

</td>

<td>

The path to the native git executable on the server. On Windows, remember to use double backslashes in the path.

</td></tr><tr>

<td>

teamcity.server.git.gc.quota.minutes

</td>

<td>

60

</td>

<td>

Maximum amount of time to run `git gc`

</td></tr><tr>

<td>

teamcity.git.cleanupCron

</td>

<td>

0 0 2 \* \* ? \*

</td>

<td>

[Cron expression](https://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/crontrigger.html) for the time of a clean-up in git-plugin, by default — daily at 2AM.

</td></tr><tr>

<td>

teamcity.git.stream.file.threshold.mb

</td>

<td>

128

</td>

<td>

Threshold in megabytes after which JGit uses streams to inflate objects. Increase it if you have large binary files in the repository and see symptoms described in [TW-14947](https://youtrack.jetbrains.com/issue/TW-14947)

</td></tr><tr>

<td>

teamcity.git.buildPatchInSeparateProcess

</td>

<td>

true

</td>

<td>

Git-plugin builds patches in a separate process, set it to false to build patch in the server process. To build patch, git-plugin has to read repository files into memory. To not run out of memory git-plugin reads only objects of size smaller than the threshold, for larger objects streams are used and they can be slow ([TW-14947](https://youtrack.jetbrains.com/issue/TW-14947)). With patch building in a separate process all objects are read into memory. Patch process uses the memory settings of the separate fetch process.

</td></tr><tr>

<td>

teamcity.git.mirror.expiration.timeout.days

</td>

<td>

7

</td>

<td>

The number of days after which an unused clone of the repository will be removed from the server machine. The repository is considered unused if there were no TeamCity operations on this repository, like checking for changes or getting the current version. These operations are quite frequent, so 7 days is a reasonably high value.

</td></tr><tr>

<td>

teamcity.git.commit.debug.info

</td>

<td>

false

</td>

<td>

Defines whether to log additional debug info on each found commit.

</td></tr>

<tr>

<td>

teamcity.git.repackIdleTimeoutSeconds

</td>

<td>

1800

</td>

<td>

Defines the idle timeout of `git-repack` operations, in seconds. You might need to increase this timeout for large repositories as repacking objects in them takes a lot of time.

</td></tr>

<tr>

<td>

teamcity.git.sshProxyType

</td>

<td>

</td>

<td>

Type of SSH proxy, supported values: `http`, `socks4`, `socks5`. Keep in mind that socks4 proxy cannot resolve remote host names, so if you get an UnknownHostException, either switch to socks5 or add an entry for your git server into the hosts file on the TeamCity server machine.

</td></tr><tr>

<td>

teamcity.git.sshProxyHost

</td>

<td>

</td>

<td>

SSH proxy host

</td></tr><tr>

<td>

teamcity.git.sshProxyPort

</td>

<td>

</td>

<td>

SSH proxy port

</td></tr><tr>

<td>

teamcity.git.connectionRetryAttempts

</td>

<td>

3

</td>

<td>

Number of attempts to establish connection to the remote host for testing connection and getting a current repository state before admitting a failure

</td></tr><tr>

<td>

teamcity.git.connectionRetryIntervalSeconds

</td>

<td>

4

</td>

<td>

Interval in seconds between connection attempts


</td></tr>

<tr>
</tr>

</table>

<anchor name="git-agent-config"/>

[Build parameters](configuring-build-parameters.md) for Git:

<table><tr>

<td width="300">

Property

</td>

<td>

Default

</td>

<td>

Description

</td></tr><tr>

<td>

teamcity.git.use.native.ssh

</td>

<td>

false

</td>

<td>

When checkout on agent: whether TeamCity should use native SSH implementation.

</td>

<td>

<!--[//]: # (Internal note. Do not delete. "Gitd153e964.txt")    -->

</td></tr><tr>

<td>

teamcity.git.idle.timeout.seconds

</td>

<td>

3600

</td>

<td>

The idle timeout for the `git fetch` operation when the agent-side checkout is used. The fetch is terminated if there is no output from the fetch process during this time.

</td></tr></table>

## Agent-side checkout rules limitations
{id="Limitations"}

<snippet id="git-checkout-rules-limitations">

The Git plugin uses [`git sparse-checkout`](https://git-scm.com/docs/git-sparse-checkout#_sparse_checkout) to check out Git files on an agent. 
The plugin is able to perform only simple file mapping operations which limits the set of supported [VCS checkout rules](vcs-checkout-rules.md) for Git.

The following rules are supported:

```Text

+:dirA/dirA1
-:dirA/dirA1/dirA2

+:. => dirA/dirA1/dirA2

+:dirA => dirA

+:dirA/dirA1 => dirA/dirA1

+:dirA/dirB/dirC => dirD/dirE/dirA/dirB/dirC

```
Note that rules must not remap files. That is, the following rule **is not supported**: `+:dirA/dirA1 => dirA/dirA2`.

If you specify multiple checkout rules for one root, make sure their checkout directories (the right part of the rule) have a common parent directory (`[prefix/]`).
Only the rules `+:dirA => [prefix/]dirA` are supported for agent-side checkout, and the `[prefix/]` must be the same for all rules.

For example:

```Text

+:dirA/dirB/dirC => [prefix/]dirA/dirB/dirC
+:dirD/dirE/dirF => [prefix/]dirD/dirE/dirF

```

Note that the following rule **is not supported**: `+:dirA=>[prefix/]dirA/postfix`. If you append `[/postfix]` to the checkout directory path and the configuration's  **VCS checkout mode** is set to "Always checkout files on agent", new builds will be unable to start.

<img src="dk-postfixCheckoutError.png" width="706" alt="Checkout directory path error"/>

</snippet>

## Known Issues
{instance="tc"}

* `java.lang.OutOfMemoryError` while fetching from a repository in case [`teamcity.git.fetch.process.max.memory`](#max-memory) property is specified. Since TeamCity 2019.2, the recommended approach is to disable this property thus delegating the automatic memory management to TeamCity.
* TeamCity running as a Windows service cannot access a network mapped drives, so you cannot work with git repositories located on such drives. To make this work, run TeamCity using `teamcity-server.bat`.
* Inflation using streams in JGit prevents `OutOfMemoryError`, but might get time-consuming (see the [TW-14947](https://youtrack.jetbrains.com/issue/TW-14947) issue related to the problem). If you notice a similar behavior, try to increase `teamcity.git.stream.file.threshold.mb`. Additionally, it is recommended to increase the overall amount of memory dedicated for TeamCity to prevent `OutOfMemoryError`.

<seealso>
        <category ref="admin-guide">
            <a href="branch-remote-run-trigger.md">Branch Remote Run Trigger</a>
        </category>
</seealso>
