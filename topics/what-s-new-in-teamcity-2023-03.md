[//]: # (title: What's New in TeamCity 2023.03)
[//]: # (auxiliary-id: What's New in TeamCity 2023.03)

## New Build Cache Feature

We're expanding the list of [build features](adding-build-features.md) with **Build Cache**, the feature that allows you to cache files used by a build configuration (for example, Maven dependencies or NodeJS packages). Cached files can be reused by the same build configuration during subsequent runs or shared with other configurations that belong to the same project. When a build starts, a TeamCity agent downloads the latest cache version to the project's working directory, optimizing and accelerating the build process.

<img src="dk-buildCache-split.png" width="706" alt="Reuse caches published by other configurations"/> 

> Build Cache is currently available as an experimental feature and may be subjected to changes in future releases. Navigate to **Administration | Experimental Features** and tick the corresponding checkbox to enable this build feature.
>
{type="warning"}

## The Sakura UI Improvements

We continue actively developing the Sakura UI to provide feature parity with the classic UI
and to support the new features.
In this version, we improved the visibility of changes.

<img src="dk-sakura-graph.png" width="706" alt="Change Log tab with changes graph"/>

- The **Change Log** tab is now available for projects and build configurations.
- The **Show graph** option has been implemented on all pages and tabs related to changes.
  With this option enabled, the changes are displayed as a graph of commits to the related VCS roots.


## Accelerated Artifact Upload

Starting with version 2023.03, TeamCity breaks down large artifact files into chunks and uploads these chunks in parallel. As a result, compared to previous TeamCity versions, the artifacts are faster uploaded to the server and become available sooner.


## Run Steps Only for Failed Builds

You can now choose the "Only if build status is failed" [execution policy](configuring-build-steps.md#Execution+Policy) for individual steps. This policy allows you to create steps that will be ignored when your build finishes successfully and executed only when it fails.


<img src="dk-whatsnew2303-onlywhenfails.png" width="706" alt="Run the build step only when the build fails"/>

## Kotlin DSL: Build Failure Conditions on Custom Metrics

You can now use Kotlin DSL to configure a build failure condition [on a custom statistic value](build-failure-conditions.md#Adding+Custom+Build+Metric)
reported by the build.
You do not need to edit the main-config.xml file for this,
which means that configuring a build failure condition on a custom metric no longer requires system administrator privileges.

The sample Kotlin DSL code is as follows:

```Kotlin
  failureConditions {
  failOnMetricChange {
    param("metricKey", "myReportedCustomStatisticValue")
    ....
  }
}
```

We are working on improving the web UI representation of the build failure condition on custom metrics added via DSL.


## Integration with Bitbucket Server and Data Center

In addition to Bitbucket Cloud, TeamCity now supports Bitbucket Server and Data Center. The corresponding option is available in the connection types list, and on the **Create Project** page.

<img src="dk-whatsnew202303-bbserver.png" width="708" alt="TeamCity integration with Bitbucket Server and Data Center"/>

See the following articles for more information: [Configuring Connections](configuring-connections.md#Bitbucket+Server+and+Data+Center) | [Creating and Editing Projects](creating-and-editing-projects.md#Creating+project+pointing+to+Bitbucket)

# Commit Status Publisher: Access Token Authentication to Bitbucket Server

Commit Status Publisher now supports access tokens for authentication to Bitbucket Server.
When you select the **Access Token** authentication type, TeamCity will display
a list of [configured OAuth connections](configuring-connections.md#Bitbucket+Server+and+Data+Center)
to Bitbucket Server / Data Center configured in the project and available to all project users.
Click the **Acquire** button to obtain the token for the current user.
If you are not signed in to your Bitbucket Server / Data Center account, TeamCity will ask for access to it.
After you sign in, you will be able to acquire a token for the required connection.
TeamCity will display the information about the user that obtained the token and the connection that provided the token.

<img src="dk-CSP-BBServerToken.png" width="708" alt="Acquire access token for Bitbucket Server"/>

# Reissuing Refresh Tokens for VCS Roots

For Bitbucket Server, Bitbucket Cloud, and GitLab VCS Roots, you can easily reissue the refresh token used by the VCS root with a token issued for the current user
by clicking the *Acquire new* button. For details, see [this section](git.md#refresh-token).


## Server Health Reports for Archived Projects

You can now generate [Server Health reports](server-health.md) for archived projects. To do this, select the *&lt;Archived Projects&gt;* scope.

<img src="dk-serverHealthArhived.png" width="706" alt="Server Health Reports for Archived Projects"/>



## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.

## We'd Love to Hear From You


We place a high value on your feedback and encourage you to share your thoughts and suggestions. See this link for more information: [Feedback](troubleshooting.md).