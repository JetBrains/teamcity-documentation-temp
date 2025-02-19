[//]: # (title: Projects Import)
[//]: # (auxiliary-id: Projects Import)

You can import projects with all their data and user accounts from a backup file to an existing TeamCity server, that is to add projects from one server to the target server that is normally used.

<anchor name="ProjectsImport-ProjectsImportorServerMove"/>

## Projects Import or Server Move
{instance="tc"}

Projects Import should be used only when some projects need to be added to an existing server containing some other projects, that is when you need to merge two servers into one. The import is a disruptive operation and [not all data is imported](#Data+not+included+into+import).   
If you need to move all the server data to a different machine, use [server move](how-to.md#Move+TeamCity+Installation+to+a+New+Machine).

## Importing projects
{instance="tc"}

On the source TeamCity server:
* [Create a usual backup](creating-backup-from-teamcity-web-ui.md) file containing the projects to be imported (note that the __[major version](upgrading-teamcity-server-and-agents.md#Upgrading+TeamCity+Server) of the source and target TeamCity servers has to be the same__).

On the target TeamCity server:
* Go to the __Server Administration__ area and select __Project Import__ on the left. Upload your project settings and follow the wizard. When the import finishes, TeamCity will display the results.

<warning>

To complete the import, you need to copy the artifacts from the old server to the new one by running the [provided scripts](#Moving+artifacts+and+logs): check details in the import log.

</warning>

>The Projects Import functionality does not support an external artifacts' storage. If you use an external artifacts' storage, you will need to move the externally stored artifacts manually to new locations using the build ids mapping generated during the import. Contact [TeamCity support](troubleshooting.md) for details.

### Selecting projects for import

After selecting a backup file, you need to specify which projects will be imported.

TeamCity will analyze the selected projects to see if they will be imported, merged or skipped.

* The project will be __imported__ if it is new for the target server. All its entities (build configurations, templates, builds, and so on) and their data will be created on the target server.
* The project will be __merged__ if the same project already exists on the target server (if the source and target project have the same [UUID](identifier.md#Universally+Unique+IDs) and [external ID](identifier.md#External+IDs)).   
   During merging the existing entities will remain intact and only the entities new for the target will be imported with the related data.    
   The data for the existing entities will not be imported or merged: new data will not be added to the existing entities (for example, changes will not added to an existing VCS root), the existing files will not be changed (for example, if the same template exists on both servers with different settings, the target file will be preserved).   
   It means, for example, that you cannot import missing builds to the existing build configuration. If you need to add the missing data to the existing entities, for example, import new builds into an already imported build configuration, then you should remove this build configuration using the UI and reimport its project.
* The project will be __skipped__ if a [conflict](#Conflicts) occurs: either the project's [UUID](identifier.md) is new but its [external ID](identifier.md#External+IDs) already exists on the target; or if the source and target projects have the same [UUID](identifier.md#Universally+Unique+IDs) but different [external IDs](identifier.md#External+IDs).

### Defining import scope

You can select the import scope: choose among project settings, builds and changes history, and user accounts or import all of them. Since an imported project can also use settings from its parent, TeamCity will also import all the vcs roots, templates, meta-runners and other project-related settings for parent projects. If the same project already exists on the target server, the existing objects will not be overwritten.

### Configuration files import

For each imported or merged project, the configuration files are imported to the [Data Directory](teamcity-data-directory.md) on the target server, provided they are new for the target. The existing files will not be changed.

The following files are imported:
* Configuration xml files for the Project with its Build Configurations, Templates, and VCS Roots as well as its subprojects.
* All files from the [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/plugins` directory.
* Build Numbers files for the newly added build configurations.

### Importing users and groups

When users are selected for import, TeamCity will analyze the usernames to see if users will be __imported__ or __merged__.

TeamCity users must have unique usernames. 
* A user account whose username is new for the target server will be __imported__. Such users appear on the target server in a separate group marked _Imported &lt;Import Date Time&gt;_. All the related data (personal builds, changes, test mutes and investigations) will be created on the target server. The [user account settings](configuring-your-user-profile.md) (roles, permissions, VCS names, notification settings, and so on: system-wide settings as well as the settings related to the imported projects) are preserved during import.   
* User accounts with the same username on the source and the target servers can be __merged__. During merging the existing data will remain intact and only the data new for the target will be added: all the new user-related data (personal builds, changes, test mutes and investigations) and the [user account settings](configuring-your-user-profile.md) (roles, permissions, VCS names, notification settings, and so on: system-wide settings as well as the settings related to the imported projects) will be added to the user on the target server.    
  (!) Merging may cause a problem if the same username belongs to different users on the source and the target server: during import the user information will be merged anyway.       
  __Note__ that the scope of user permissions on the target may change after import, for example:
   * if a user has the system administrator role on the source, this role will be added to the user on the target after import,   
   * if a user has several roles in several projects on the source, only the new roles for the projects within the import scope will be added on the target.   

    The __Project Import | Import scope | Users__ section will display the number of conflicts and you can view them and decide is you want to merge them. 

    TeamCity will show users with the same username and different emails on both servers, as well as the number of users with the same username and the same email. Email verification [can be enabled](enabling-email-verification.md) for the server,  and the users with the same username and email are compared based on their email verification.  You can view the conflicts information and choose whether to merge the users found. The options are active if users with verified emails are present either on the source or the target TeamCity server, or both.

Import of user groups works the same way: new groups are imported, while the existing groups can be merged.

If a [conflict](#Conflicts) occurs (the groups exists on both the source and the target, but the group roles are different), after import the group on the target server may get additional roles. As a result, a member of this group on the target will get additional roles and permissions as well. 

The __Project Import__ page | __Import scope__ section | __Groups__ will display how many conflicting groups are found. You can view all the groups that have the same group key and decide if you want to merge them. Note that ["All Users" group](creating-and-managing-user-groups.md#%22All+Users%22+Group) is always listed as a conflicting one because it is a default group on all TeamCity servers.

### Conflicts

TeamCity does not import entities from the backup file if they conflict with some entity on the target server. Before import, TeamCity analyzes the backup file and displays all detected conflicts on the __Import Scope__ configuration page.

It is __highly recommended that you resolve all conflicts__ before proceeding with the import, as unresolved conflicts may result in unpredictable behavior after the import, for example:
* Critical errors can be shown if, for example, some VCS Root was skipped, but a Build Configuration depending on it was imported.
* Imported Build Configurations may refer to the wrong Template if there was an unresolved conflict of [external IDs](identifier.md#External+IDs) between the templates from the source and target servers.

If the conflicts have not been resolved before importing, you can find the conflicting files in the `conflictingFiles` directory under the import results logs.

<anchor name="ProjectsImport-Dataexcludedfromimport"/>

## Importing projects
{instance="tcc"}

On the source TeamCity On-Premises server:
* Create a [backup file](https://www.jetbrains.com/help/teamcity/creating-backup-from-teamcity-web-ui.html) containing the projects to be imported.

On the target TeamCity server:
* Go to the __Server Administration__ area and select __Project Import__ on the left. Upload your project settings and follow the wizard. When the import finishes, TeamCity will display the results.

## Data not included into import

There is a number of limitations regarding the import:
* Agents and agent pools are not imported ([TW-39797](https://youtrack.jetbrains.com/issue/TW-39797)).
* Settings are merged on the "by file" basis. This means that new files are added but no settings files are merged. For example, if a project being imported already exists on the target server, its parameters, project features, and plugin settings will not be merged.
* If you use the "Store secure values (like passwords or API tokens) outside of VCS" option for version settings, then the credentials will not be imported for the projects already existing on the server.
* Audit records are imported only if users are selected in the scope.
* Running builds and the build queue are not included in the backup and not imported.
* Internal ids (like ids of the builds) are not preserved during import. This means that URLs to the build results pages from the old server will appear broken even if redirected to the new server as build ids change on importing.
* The backup files do not contain artifacts and logs (build logs are stored under build artifacts), so these are not imported automatically, but TeamCity provides scripts to move them [manually](#Moving+artifacts+and+logs).
{instance="tc"}
* Global server settings (authentication schemes, custom roles, and so on) are not imported.
* Import to TeamCity Cloud: build artifacts and logs.

<note>

Importing projects may take significant time. There can be only one import process per server.

</note>
    
## Moving artifacts and logs
{instance="tc"}

Although artifacts and logs are not imported right from the backup file, you can copy/move them from the source to the target server using the `.bat` and `.sh` scripts from the `projectsImport-<date>` directory under TeamCity logs. These scripts accept the source and target data directories via the command line; the scripts accept the source and target [artifact directories](build-artifact.md). The rest is done automatically. The scripts can be executed while the server is running.

It may take some time for TeamCity to display the imported build artifacts.


## Moving artifacts and logs
{instance="tcc"}

<anchor name="ProjectsImport-Movingartifactsandlogs"/>

Restoring artifacts and logs of imported projects is available only on TeamCity On-Premises instances. See this article for more information: [Migrate from TeamCity On-Premises to TeamCity Cloud](migrate-from-teamcity-on-premises-to-teamcity-cloud.md#Migration+Process).

## Viewing Import Results

Each import process creates the `projectsImport-<date>` directory under the TeamCity logs allowing you to view the import results.

The directory contains the following:
* conflicting files' directory, containing all data which has been merged
* mappings, containing mapping of the fields in the source and target databases
* scripts for copying artifacts and logs (see the section [above](#Moving+artifacts+and+logs))
{instance="tc"}
* import report, listing import results including the information on the data which has not been imported (if any)