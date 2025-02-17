[//]: # (title: What's New in TeamCity 2022.02)
[//]: # (auxiliary-id: What's New in TeamCity 2022.02)

## Storing Docker images produced by build to public ECR registry

TeamCity can now store Docker images produced by a build to both private and — since this update — public ECR registries.

To be able to use this functionality, you need to add an [Amazon ECR connection](configuring-connections.md#Amazon+ECR) in [project settings](project-administrator-guide.md#Edit+and+View+Modes) and choose the _ECR Public_ registry type:

<img src="amazon-ecr-public.png" alt="Connecting to public ECR registry" width="750"/>

Remember to also enable [](docker-support.md) in your builds.

## Automatic import of user avatars from external systems

When a user signs in to TeamCity [via a third-party account](configuring-authentication-settings.md), like GitHub or Bitbucket, for the first time, TeamCity will automatically fetch their avatar from the external system and attach it to their TeamCity user profile. Note that TeamCity will only be able to access avatars of users with verified emails (if you are using GitLab, check that a _public email_ is set in your account).

It is possible to upload a different avatar in the TeamCity user profile settings afterwards.

## Getting project SSH keys via UI

You can now copy the public part of an uploaded non-encrypted SSH key from the [project settings](project-administrator-guide.md#Edit+and+View+Modes). To do this, go to **Project Settings | SSH keys** and click **Copy the public key** under the key name.

<img src="copy-public-ssh-keys.png" alt="Copying public SSH keys" width="750"/>

This way, project admins no longer need to ask the system administrator for a public SSH key whenever they need it (for example, to integrate their TeamCity projects with a VCS hosting service) — they can just get it via the TeamCity UI.

## Applying action to multiple builds

It is now possible to select multiple builds and apply actions to all of them at once:
* Pin/unpin
* Tag
* Compare with another build
* Add a comment
* Add to favorites
* Remove

On the **Overview** tab of **Build Configuration Home**, you can select the required builds with checkboxes that appear when hovering over builds. To apply an action to them, use the respective command of the pop-up context menu. If you need to select a range of builds, press **Shift** and click build checkboxes at the edges of the range to be selected.

<img src="select-multiple-builds.png" alt="Selecting multiple builds" width="750"/>

## Kotlin DSL update: Import statements include only current DSL version

Since this version, all `import` statements in a [Kotlin DSL](kotlin-dsl.md) code no longer include the DSL version specification. Instead, the version is determined automatically based on the current server's version.  
This change affects how a DSL code appears in the _View as Code_ mode in TeamCity and removes irrelevant suggestions when writing `import` statements manually in an IDE. It concerns only newly created projects — the syntax of existing projects is supported for compatibility.

## Perforce integration update: Automatic creation of Helix Swarm test runs and status synchronization with builds

[Commit Status Publisher](commit-status-publisher.md) has a new option for sending build status reports to Perforce Helix Swarm — **Create Swarm Test**. If you enable it, TeamCity will [create a test run](https://www.perforce.com/manuals/swarm/Content/Swarm/swarm-apidoc_endpoint_integration_tests.html#Create_a__testrun_for_a_review_version) on the Helix Swarm server and update its status according to the build status in TeamCity.

## Other updates

* [Composite build configurations](composite-build-configuration.md) no longer appear on the _Compatible Configurations_ tab of a [build agent's details](viewing-build-agent-details.md).  
  Composite builds are meta-builds designed for aggregating results of builds preceding them in a [chain](build-chain.md). Technically, they do not need a build agent to run and thus cannot be matched as compatible/incompatible with them. This usability update removes composite build configurations from _Compatible_ lists to help avoid any potential confusions.

## Fixed issues
{instance="tcc"}

See [TeamCity Build 107913 release notes](teamcity-release-notes-build-107913.md).

## Upgrade notes

* Versions 2017.1 and 2017.2 of [TeamCity REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html) have been unbundled. If you have been using any of these versions in your scripts, consider switching to the latest protocol version as described [here](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html#REST+API+Versions). If switching is not an option and this is a breaking change for your setup, please contact us via any convenient [feedback channel](feedback.md).
* Freemarker, used by TeamCity [notification templates](customizing-notification-templates.md), has been updated to version 2.3.31.  
  {instance="tc"}
* The [CVS plugin](cvs.md) has been unbundled from TeamCity. If you want to continue using it on your server, please [download it from JetBrains Marketplace](https://plugins.jetbrains.com/plugin/18552-vcs-support-cvs) and install it as described [here](installing-additional-plugins.md).
  {instance="tc"}

## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.