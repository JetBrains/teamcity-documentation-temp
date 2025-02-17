[//]: # (title: Configuring Artifacts Storage)
[//]: # (auxiliary-id: Configuring Artifacts Storage)

The __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Artifacts Storage__ tab displays artifact storages configured in this project as well as the storages inherited from parents. 

By default, the built-in TeamCity artifacts storage is displayed and marked as active. You can activate a different storage using the corresponding link.

## Built-in Artifacts Storage

TeamCity stores [artifacts](build-artifact.md) produced by builds on the file system accessible by the TeamCity server. The default artifacts directory location is [`<TeamCity Data Directory>`](teamcity-data-directory.md)`/system/artifacts` but it can be [redefined](teamcity-configuration-and-maintenance.md).

## External Artifacts Storage
{id="external-artifacts-storage"}

TeamCity provides a pluggable API to enable external storage for TeamCity build artifacts. Support for different storages can be implemented as an external plugin to TeamCity: the details are provided in the [external storage implementation guide](https://plugins.jetbrains.com/docs/teamcity/external-storage-implementation-guide.html).

Note that when an external storage for artifacts is enabled, the TeamCity [internal](build-artifact.md#Hidden+Artifacts) artifacts (including build logs) will still be published to the TeamCity server and stored in the TeamCity Data Directory in the built-in artifacts' storage.

The same applies to the metadata about artifacts mappings, which will be published to the [artifacts directory](teamcity-configuration-and-maintenance.md) of the TeamCity Data Directory. When restoring from a backup, make sure they are restored for the external artifact plugin to work properly.

### Amazon S3 Support
<anchor name="AmazonS3Support"/>

TeamCity can store build artifacts in an Amazon S3 bucket. Read more details in [this article](storing-build-artifacts-in-amazon-s3.md).


### Azure Artifact Storage

[Azure Artifact Storage](https://plugins.jetbrains.com/plugin/9617-azure-artifact-storage) is an experimental plugin by JetBrains which allows replacing the TeamCity built-in artifacts' storage by Azure Blob storage. 

### Google Cloud Artifact Storage

Google Cloud Artifact Storage is implemented as a [plugin](https://plugins.jetbrains.com/plugin/9634-google-artifact-storage) by JetBrains.





## Migrating Artifacts To a Different Storage
<anchor name="migratingArtifactsToS3"/>

<snippet id="artifactMigrationToS3">

TeamCity server ships with [a command-line tool](artifacts-migration-tool.md) that transfers build artifacts from one storage to another. You can download this tool on the __[Project Settings](project-administrator-guide.md#Edit+and+View+Modes) | Artifacts Storage__ page.

<img src="dk-downloadAMTool.png" width="706" alt="Download artifacts migration tool"/>

Currently, the tool supports the following migration routes:

* From a local directory to an Amazon S3 bucket and vice versa
* From one Amazon S3 bucket to another

We're working on supporting other cloud storage options as well.

Learn more: [](artifacts-migration-tool.md).

</snippet>



