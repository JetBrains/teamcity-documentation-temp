[//]: # (title: Docker Support)
[//]: # (auxiliary-id: Docker Support)

The _Docker Support_ [build feature](adding-build-features.md) allows automatically signing in to a DockerHub or other container registry before the build start.

Add this feature to:

* Allow TeamCity monitor and detect Docker/Podman operations (for instance, `docker pull` and `podman run`);

* Automatically log in to an authenticated registry before the build and log out of it after the build.

* Clean up local (for both Docker and Podman) and pushed to a registry (only for Docker) images;

* Add the __Container Info__ tab to the _[Build Results](working-with-build-results.md)_ page. The tab provides information on operations related to the container manager used.
  
  <img src="dk-containerInfoTab.png" width="706" alt="Container Info tab"/>

Logging in and out of repositories and cleaning up images require a configured [connection to a Docker registry](configuring-connections-to-docker.md):

<img src="docker-support.png" width="750" alt="Docker Support build feature"/>

>_Docker Support_ is a part of the TeamCity-Docker/Podman integration toolset. Refer to this documentation article for information on software requirements, supported environments, and other common aspects of this integration: [](integrating-teamcity-with-container-managers.md).

## Docker Images Clean-up

### Clean-up of the Pushed Images

If you have a build configuration which publishes images, you need to remove them at some point. You can select the corresponding option and instruct TeamCity to remove the images published by a certain build when the build itself is [cleaned up](teamcity-data-clean-up.md).

It works as follows: when an image is published, TeamCity stores the information about the registry of the images published by the build. When the [server clean-up](teamcity-data-clean-up.md) is run and it deletes the build, all the configured connections are searched for the address of this registry, and the images published by the build are cleaned up using the credentials specified in the found connection.

Cleaning-up images pushed via Podman is currently enabled only if images were pushed from Linux agents, and only if the `--digestfile=<fileName>` option was set. Note that the `fileName` should be a path relative to the [checkout directory](build-checkout-directory.md) and should point to the final digest file location (this path should not be modified by further navigation commands).

### Clean-up of Images on Build Agent
                                   
As part of [Free disk space](free-disk-space.md) build feature, Docker Support cleans up images which were created by TeamCity builds on this build agent. The Docker Support assumes, that docker images are stored under

 - `/var/lib/docker` on Linux
 - `%\ProgramData%` directory on Windows
 - `$HOME` directory on other systems

The location is important, as the [Free disk space](free-disk-space.md) feature analyzes which disk volumes should be cleaned for the build. If your docker daemon uses a non-standard location for the images/containers, the location can be specified using `teamcity.docker.data.path` configuration parameter, preferably in [`buildAgent.properties`](configure-agent-installation.md) file.
<!-- We're going to avoid the need to configure manually this with https://youtrack.jetbrains.com/issue/TW-72569 -->

## Docker Registry Automatic Login/Logout

If you need to log in to a registry requiring authentication before a build, select the corresponding option and a connection to Docker configured in the __Project Settings__. Automatic logout will be performed after the build finishes.

>[See also](integrating-teamcity-with-container-managers.md#Conforming+with+Docker+Download+Rate+Limits) how to use this functionality to double the number of pulls allowed to a Free Docker Hub user profile.

## Amazon ECR

A connection to Amazon Elastic Container Registry (ECR) allows storing Docker/LXC
images in private AWS registries. For this, such a connection needs to be selected when adding a Docker Support feature to a build configuration.

Connection settings:

<table>
<tr>
<td>

Setting

</td>
<td>

Description

</td>
</tr>

<tr>
<td>

AWS region

</td>
<td>

Select an AWS region where the target resources are located.

</td>
</tr>

<tr>
<td>

Credentials type

</td>
<td>

* __Access key__: select to use preconfigured AWS account access keys. You can find them in the [Identity and Access Management](https://console.aws.amazon.com/iam) section of your AWS console.
* __Temporary credentials__: get [temporary access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) via AWS STS. Such credentials are short-term and do not belong to a specific user.

</td>
</tr>

<tr>
<td>

IAM role ARN

(_only for Temporary credentials_)

</td>
<td>

Specify a role to be used for generating temporary credentials. You need to [create this role in advance](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html) in your AWS console and assign it to all the necessary permissions.

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

Enable this option to automatically find access keys according to the [default chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html#credentials-default).

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

Enter an ID of your registry or AWS account.

</td>
</tr>

</table>


## Kotlin DSL

The following [](kotlin-dsl.md) snippet illustrates how to add a Docker Support build feature to your build configuration.


```Kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildFeatures.dockerSupport

object MyBuildConfig : BuildType({
    name = "Deploy Web (Windows)"
    
    features {
        dockerSupport {
            cleanupPushedImages = true
            loginToRegistry = on {
                dockerRegistryId = "PROJECT_EXT_5"
            }
        }
    }
})
```

> To quickly get an ID of a target [Docker Registry](configuring-connections.md#Docker+Registry):
> 1. <include from="common-templates.md" element-id="open-project-settings"/>
> 2. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
> 3. Copy a connection ID from the overview table.
> <img src="dk-copy-connection-id.png" alt="Copy connection ID" width="706"/>
>
{style="tip"}


<seealso>
        <category ref="admin-guide">
            <a href="integrating-teamcity-with-container-managers.md">Integrating TeamCity with Docker</a>
            <a href="configuring-connections-to-docker.md">Configuring Connections to Docker</a>
            <a href="docker-compose.md">Docker Compose runner</a>
            <a href="docker.md">Docker runner</a>
            <a href="container-wrapper.md">Container Wrapper extension</a>
        </category>
</seealso>