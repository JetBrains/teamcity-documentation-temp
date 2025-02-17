[//]: # (title: Guess Settings from Repository URL)
[//]: # (auxiliary-id: Guess Settings from Repository URL)

TeamCity can automatically discover the VCS type and settings from the repository URL.

When configuring a [VCS root](vcs-root.md), select the __Guess from Repository URL__ option from the drop-down menu and specify the URL. TeamCity will recognize the URL for a supported version control and will create a VCS root automatically. After the VCS root is created, you can [modify its settings](configuring-vcs-roots.md) using the [project or build configuration settings](project-administrator-guide.md#Edit+and+View+Modes) page.

## VCS URL Formats

<table><tr>

<td>

VCS

</td>

<td>

URL Formats

</td></tr><tr>

<td>

__Git__

</td>

<td>

* `http(s)` URLs
* `git://`
* Maven-like URLs: [`https://maven.apache.org/scm/git.html`](https://maven.apache.org/scm/git.html)

For SSH authentication, create a VCS Root from a URL first and then open its settings to [specify the SSH key](ssh-keys-management.md) to be used. Alternatively, create a Git VCS Root manually.

</td></tr><tr>

<td>

__Mercurial__

</td>

<td>

* `http(s)` URLs
* Maven-like URLs: [`https://maven.apache.org/scm/mercurial.html`](https://maven.apache.org/scm/mercurial.html)

</td></tr><tr>

<td>

__Subversion__

</td>

<td>

* `http(s)` URLs
* \+Maven-like URLs: [`https://maven.apache.org/scm/subversion.html`](https://maven.apache.org/scm/subversion.html)
* `svn://`

</td></tr><tr>

<td>

__Azure__

</td>

<td>

Recommended URL formats:

* `http[s]://<tfs_server>:<port>/<collection name>$/<project_path>` or `http[s]://<tfs_server>:<port>/tfs/<collection name>/<project_name>`    
For example: [`http://tfshost:8080/tfs/DefaultCollection$/Project/root`](http://tfshost:8080/tfs/DefaultCollection$/Project/root){nullable="true"}

* for Azure DevOps Services (or, formerly, Visual Studio Team Services): `https://<url_to_visualstudio.com>/<project_name>` or `https://<url_to_visualstudio.com>/$/<project_path>`    
For example: [`https://username.visualstudio.com/Project`](https://username.visualstudio.com/Project)

See the [related blog post](https://blog.jetbrains.com/teamcity/2014/09/teamcity-and-visual-studio-online-source-control).

</td></tr><tr>

<td>

__Perforce__

</td>

<td>

* Maven-like URLs: [`https://maven.apache.org/scm/perforce.html`](https://maven.apache.org/scm/perforce.html)
* same as Maven but without the `scm` prefix, for example: `perforce://servername:1666/depot/main`

</td></tr><tr>

<td>

__CVS__

</td>

<td>

no support yet

</td></tr></table>
