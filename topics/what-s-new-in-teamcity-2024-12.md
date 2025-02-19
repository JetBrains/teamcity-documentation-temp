[//]: # (title: What's New in TeamCity 2024.07)
[//]: # (auxiliary-id: What's New in TeamCity 2024.07)


## Pipelines Merge Announcement and Major UI Changes
{instance="tc"}

The first glaring change you will notice seconds after the updated 2024.12 TeamCity server starts is the redesigned UI. We replaced the top navigation bar with a sleek side menu, revamped breadcrumbs, refreshed project and configuration icons, and introduced other visual enhancements for a cleaner, more functional experience.


<img src="dk-2024-11-ui-update.png" width="706" alt="2024.12 UI Update"/>

Although significant on their own, these changes are more than a visual refresh — they lay the foundation for another major upcoming change: the integration of TeamCity and TeamCity Pipelines.

Currently, TeamCity focuses on robust CI/CD for complex workflows, while [TeamCity Pipelines](https://www.jetbrains.com/teamcity/download/#section=saas) is a user-friendly option for smaller teams that puts a high value on usability and intuitive UI. However, this separation leads us to the obvious challenges: some TeamCity users wish for the simplicity of Pipelines, while Pipelines users occasionally miss advanced features currently available only in TeamCity.

By merging these two products, we aim to eliminate this frustration and deliver a unified experience. TeamCity users will be able to choose between creating a streamlined Pipeline for quick setups or a fully customizable build configuration for advanced scenarios and complex build chains, ensuring the right level of flexibility for every user.

We encourage you to try TeamCity Pipelines now and share your feedback. Your insights are invaluable as we work to make this transition as smooth and user-focused as possible.

> While we plan to integrate Pipelines into TeamCity, it will continue as an [actively developed](https://blog.jetbrains.com/search/?q=pipelines&p=TeamCity) standalone solution.
>
{style="note"}

Learn more: [Try TeamCity Pipelines](https://www.jetbrains.com/teamcity/download/#section=saas) | [Pipelines Pulse](https://blog.jetbrains.com/search/?q=pulse&p=TeamCity) | [Pipelines documentation](https://www.jetbrains.com/help/teamcity/pipelines/teamcity-pipelines.html)


## Offload Your Builds to Kubernetes
{instance="tc"}

In a traditional TeamCity paradigm, you should set up, maintain, and manage build agents that process builds. This applies to both cloud-hosted and local agents.

Starting with version 2024.12, you will be able to take an alternative route: offload building tasks to the K8s cluster. Set up the new integration and start building right away: our K8s executor will generate pod definitions based on build parameters and required containers and submit them to K8s cluster. As a result, your cluster will act as an orchestrator that manages builds and "agents" lifecycle.

<img src="dk-k8s-integration-overview.png" alt="K8S integration" width="706"/>

<note>This feature is still in the experimental stage and may be changed in future releases.</note>

Learn more: [](kubernetes-executor.md) | [Kubernetes connection](configuring-connections.md#Kubernetes)

<!--
## New Dependency Cache Features
{instance="tc"}

This TeamCity version introduces three new [build features](adding-build-features.md) designed to cache dependencies downloaded during a build, speeding up subsequent builds and enabling offline completion:

* **Maven Cache** — caches dependencies used by [](maven.md) build steps
* **Gradle Cache** — caches dependencies used by [](gradle.md) build steps
* **NuGet Cache** — caches NuGet packages used by [](net.md) build steps

All three features require no configuration: add/enable a required build feature to enable caching, and remove/disable it to force agents to re-download missing dependencies with each build.

Learn more: [](dependency-caches.md)
-->


## Partial Chain Execution
{instance="tc"}

Version 2024.12 introduces two configuration parameters that accept tags and IDs of upstream builds. Doing so allows you to run a portion of your build chain.

<img src="dk-subchains-gif.gif" width="706" alt="Subchains"/>

These parameters can be a permanent part of your configuration or occasionally added via the [Run Custom Build](running-custom-build.md) dialog when you need to run a specific sub-chain.

In addition, we have implemented the `skipQueuedBuilds` service message that you can send from build steps to cancel builds of downstream configurations that are already queued.

Learn more: [](build-chain.md#Partial+Chain+Execution)


## Centralized Refreshable Token Management
{instance="tc"}

In recent releases, we have introduced support for refreshable access tokens across a wide range of OAuth connections. These short-lived tokens enable VCS roots, issue trackers, and build features like Commit Status Publisher to authenticate with VCS hosting providers. While refreshable tokens are a secure alternative to static tokens or username/password credentials, tracking all entities using these tokens can be challenging.

To simplify token management, we created the **VCS Auth Tokens** page in project settings. Here, you can view, create, and revoke refreshable tokens for your projects.

<img src="dk-vcs-auth-tokens-overview.png" width="706" alt="VCS Auth Tokens Overview"/>

Additionally, we replaced the old **Acquire** / **Acquire New** buttons with redesigned controls. These updated controls allow you to copy existing token IDs from the **VCS Auth Tokens** page or generate new tokens right on the spot.

<img src="dk-edit-ref-token.png" width="706" alt="Edit button"/>

Learn more: [](manage-access-tokens.md)


## Pull Request Filters
{instance="tc"}

Starting with version 2024.12, TeamCity [branch filters](branch-filter.md#Pull+Request+Branch+Filters) support the `+|-pr:` syntax. This syntax allows you to create fine-grained filter expressions that track pull requests by a number of parameters: user role, target and source branches, and more.

The new syntax is currently available only for triggers. When setting up trigger settings, click a magic wand next to the **Branch Filter** field to invoke an expression editor.

<img src="dk-visual-pr-filters-editor.png" alt="Filter Expression Editor" width="706"/>

Learn more: [](branch-filter.md#Pull+Request+Branch+Filters)


## Perforce Integration Enhancements
{instance="tc"}

* When a [Perforce VCS Root](perforce.md) is configured to check out sources by label (the **Label/changelist to sync** setting), TeamCity now records the revision number in a new `vcsRoot.{externalId}.changelist` parameter. This quality-of-life improvement enables clear identification of the synced revision.
* The [new Perforce-specific service message](service-messages.md#Perforce+Service+Messages) was added. This message allows you to manually rollback personal changes.
* You can now [build Perforce shelved changelists](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Shelved+Files) using the regular `buildQueue` REST API endpoint. Previously, this functionality was available only via a dedicated `/app/perforce/runBuildForShelve` endpoint.

## Approve Multiple Builds at Once
{instance="tc"}

TeamCity enables you to require explicit permission from designated users before starting certain builds. Approval is needed in two cases:

* When a build configuration has the [](build-approval.md) feature to prevent accidental runs.
* When [](untrusted-builds.md) are enabled, ensuring unverified pull request changes are reviewed before builds begin.

Previously, both options required approving each build individually within a chain. Starting with version 2024.12, clicking **Approve** now grants permission for all builds in the chain at once.

<img src="dk-approve-chain.png" width="706" alt="Approve chain"/>

Learn more: [](build-approval.md) | [](untrusted-builds.md)


## Upload Custom Kotlin Libraries
{instance="tc"}

Starting with this version, you can upload custom Kotlin libraries as `.jar` to you TeamCity server.

<img src="custom-dsl-library-upload.png" width="706" alt="Upload custom Kotlin library"/>

To start using these libraries in your project's [.kts files](kotlin-dsl.md), add Maven dependencies to required `pom.xml`s.

Learn more: [](kotlin-dsl.md#Add+Custom+Kotlin+Libraries)


## Execute Meta-Runners Inside Containers
{instance="tc"}

Starting with this version, [meta-runners](working-with-meta-runner.md) provide [container-related settings](container-wrapper.md). These settings are propagated to individual steps, meaning every individual step will be able to run inside the required Docker/Linux image.

<img src="dk-docker-container-settings.png" width="706" alt="Container settings in steps and meta-runners"/>

If a step has its own container settings, they override the global meta-runner configuration.

Learn more: [](working-with-meta-runner.md#Launch+Meta-Runners+in+Containers)


## AWS Integration Enhancements
{instance="tc"}

### AWS Cloud Profiles

[Amazon EC2 cloud profiles](setting-up-teamcity-for-amazon-ec2.md) will no longer use access keys or the default credentials provider chain, shifting to authentication through [TeamCity AWS connections](configuring-connections.md#AmazonWebServices). This change consolidates all authentication settings into a single connection that can be shared across multiple features (cloud profiles, [S3 artifact storages](storing-build-artifacts-in-amazon-s3.md), [](aws-credentials.md) build feature, and so on).

Existing connections will retain legacy authentication but recommend migrating to connection-based access. New EC2 cloud profiles will support only the new authentication method.

### AWS Agents Performance

We reworked the logic for AWS EC2-hosted build agents, resulting in a significant performance boost. See this blog post for more information: [Enhancing TeamCity AWS Agents’ Performance](https://blog.jetbrains.com/teamcity/2024/10/enhancing-teamcity-aws-agents-performance/).


## Performance Enhancements
{instance="tc"}

In this release, we've overhauled the internal logic for fetching build lists and project trees. Combined with frontend optimization, this upgrade delivers significant [web vitals](https://web.dev/articles/vitals) improvements, especially for users managing large TeamCity instances with extensive project setups.

## Miscellaneous Changes
{instance="tc"}

* Both TeamCity server and agents now support Java 21.
<!--* Artifacts in custom [S3 storages](storing-build-artifacts-in-amazon-s3.md) are now downloaded in parallel, using up to 5 threads per artifact. This improvement boosts download speeds by up to 80% based on our benchmarks. The feature is enabled by default, requiring no manual configuration.-->
* EC2-hosted agents now [report](setting-up-teamcity-for-amazon-ec2.md#EC2-Specific+Agent+Parameters) the `system.ec2.instance-life-cycle` parameter that allows you to identify whether this agent uses a spot or on-demand EC2 instance.
* The [](artifacts-migration-tool.md) now supports migration to and from Microsoft Azure storages. This tool allows you to easily transfer build artifacts from one storage to another. Note that you need to install an unbundled plugin to set up Azure storages: [Azure Artifact Storage](https://plugins.jetbrains.com/plugin/9617-azure-artifact-storage).
* All messages written to `teamcity-server.log` during a server startup are now duplicated to the [teamcity-startup.log](teamcity-server-logs.md). This log ensures major boot events are logged to a separate file, which may assist in troubleshooting server startup issues.
* The [HashiCorp Vault connection](hashicorp-vault.md) no longer provides a setting that allows builds to proceed when TeamCity is unable to retrieve Vault secrets. Starting with version 2024.12, such build will fail.
* [TeamCity proxy settings](configuring-proxy-server.md) are now automatically propagated to the [native Git](git.md#Native+Git+for+VCS-related+operations+on+the+server) configuration (if it does not have corresponding properties already set up). Previously, you had to configure them manually. HTTP proxy settings are propagated on both agent and server sides, whereas SSH proxy settings are currently propagated only for TeamCity server.
* TeamCity [metrics](teamcity-monitoring-and-diagnostics.md#Metrics) set now includes new experimental metrics:
  * the `log_messages` metric allows you to obtain the total number of [Log4j](teamcity-server-logs.md) messages. This metric reports a separate number of log messages for each category (`ACTIVITIES`, `AGENT`, `STARTUP`, and others) and severity (`INFO` or `WARN`).
  * the `persistTasks_global_settings_count` and `persistTasks_project_configs_count` metrics report same values as the **Settings Persist Status** tab of the [Diagnostics](teamcity-monitoring-and-diagnostics.md) page. Both metrics are reported only for the main [node](multinode-setup.md).


## Pipelines Merge Announcement and Major UI Changes
{instance="tcc"}

The first glaring change you will notice seconds after the updated 2024.12 TeamCity server starts is the redesigned UI. We replaced the top navigation bar with a sleek side menu, revamped breadcrumbs, refreshed project and configuration icons, and introduced other visual enhancements for a cleaner, more functional experience.


<img src="dk-2024-11-ui-update.png" width="706" alt="2024.12 UI Update"/>

Although significant on their own, these changes are more than a visual refresh — they lay the foundation for another major upcoming change: the integration of TeamCity and TeamCity Pipelines.

Currently, TeamCity focuses on robust CI/CD for complex workflows, while [TeamCity Pipelines](https://www.jetbrains.com/teamcity/download/#section=saas) is a user-friendly option for smaller teams that puts a high value on usability and intuitive UI. However, this separation leads us to the obvious challenges: some TeamCity users wish for the simplicity of Pipelines, while Pipelines users occasionally miss advanced features currently available only in TeamCity.

By merging these two products, we aim to eliminate this frustration and deliver a unified experience. TeamCity users will be able to choose between creating a streamlined Pipeline for quick setups or a fully customizable build configuration for advanced scenarios and complex build chains, ensuring the right level of flexibility for every user.

We encourage you to try TeamCity Pipelines now and share your feedback. Your insights are invaluable as we work to make this transition as smooth and user-focused as possible.

> While we plan to integrate Pipelines into TeamCity, it will continue as an [actively developed](https://blog.jetbrains.com/search/?q=pipelines&p=TeamCity) standalone solution.
>
{style="note"}

Learn more: [Try TeamCity Pipelines](https://www.jetbrains.com/teamcity/download/#section=saas) | [Pipelines Pulse](https://blog.jetbrains.com/search/?q=pulse&p=TeamCity) | [Pipelines documentation](https://www.jetbrains.com/help/teamcity/pipelines/teamcity-pipelines.html)


## Offload Your Builds to Kubernetes
{instance="tcc"}

In a traditional TeamCity paradigm, you should set up, maintain, and manage build agents that process builds. This applies to both cloud-hosted and local agents.

Starting with version 2024.12, you will be able to take an alternative route: offload building tasks to the K8s cluster. Set up the new integration and start building right away: our K8s executor will generate pod definitions based on build parameters and required containers and submit them to K8s cluster. As a result, your cluster will act as an orchestrator that manages builds and "agents" lifecycle.

<img src="dk-k8s-integration-overview.png" alt="K8S integration" width="706"/>

<note>This feature is still in the experimental stage and may be changed in future releases.</note>

Learn more: [](kubernetes-executor.md) | [Kubernetes connection](configuring-connections.md#Kubernetes)

<!--
## New Dependency Cache Features
{instance="tcc"}

This TeamCity version introduces three new [build features](adding-build-features.md) designed to cache dependencies downloaded during a build, speeding up subsequent builds and enabling offline completion:

* **Maven Cache** — caches dependencies used by [](maven.md) build steps
* **Gradle Cache** — caches dependencies used by [](gradle.md) build steps
* **NuGet Cache** — caches NuGet packages used by [](net.md) build steps

All three features require no configuration: add/enable a required build feature to enable caching, and remove/disable it to force agents to re-download missing dependencies with each build.

Learn more: [](dependency-caches.md)
-->


## Partial Chain Execution
{instance="tcc"}

Version 2024.12 introduces two configuration parameters that accept tags and IDs of upstream builds. Doing so allows you to run a portion of your build chain.

<img src="dk-subchains-gif.gif" width="706" alt="Subchains"/>

These parameters can be a permanent part of your configuration or occasionally added via the [Run Custom Build](running-custom-build.md) dialog when you need to run a specific sub-chain.

In addition, we have implemented the `skipQueuedBuilds` service message that you can send from build steps to cancel builds of downstream configurations that are already queued.

Learn more: [](build-chain.md#Partial+Chain+Execution)


## Centralized Refreshable Token Management
{instance="tcc"}

In recent releases, we have introduced support for refreshable access tokens across a wide range of OAuth connections. These short-lived tokens enable VCS roots, issue trackers, and build features like Commit Status Publisher to authenticate with VCS hosting providers. While refreshable tokens are a secure alternative to static tokens or username/password credentials, tracking all entities using these tokens can be challenging.

To simplify token management, we created the **VCS Auth Tokens** page in project settings. Here, you can view, create, and revoke refreshable tokens for your projects.

<img src="dk-vcs-auth-tokens-overview.png" width="706" alt="VCS Auth Tokens Overview"/>

Additionally, we replaced the old **Acquire** / **Acquire New** buttons with redesigned controls. These updated controls allow you to copy existing token IDs from the **VCS Auth Tokens** page or generate new tokens right on the spot.

<img src="dk-edit-ref-token.png" width="706" alt="Edit button"/>

Learn more: [](manage-access-tokens.md)


## Pull Request Filters
{instance="tcc"}

Starting with version 2024.12, TeamCity [branch filters](branch-filter.md#Pull+Request+Branch+Filters) support the `+|-pr:` syntax. This syntax allows you to create fine-grained filter expressions that track pull requests by a number of parameters: user role, target and source branches, and more.

The new syntax is currently available only for triggers. When setting up trigger settings, click a magic wand next to the **Branch Filter** field to invoke an expression editor.

<img src="dk-visual-pr-filters-editor.png" alt="Filter Expression Editor" width="706"/>

Learn more: [](branch-filter.md#Pull+Request+Branch+Filters)


## Perforce Integration Enhancements
{instance="tcc"}

* When a [Perforce VCS Root](perforce.md) is configured to check out sources by label (the **Label/changelist to sync** setting), TeamCity now records the revision number in a new `vcsRoot.{externalId}.changelist` parameter. This quality-of-life improvement enables clear identification of the synced revision.
* The [new Perforce-specific service message](service-messages.md#Perforce+Service+Messages) was added. This message allows you to manually rollback personal changes.
* You can now [build Perforce shelved changelists](integrating-teamcity-with-perforce.md#Running+Builds+on+Perforce+Shelved+Files) using the regular `buildQueue` REST API endpoint. Previously, this functionality was available only via a dedicated `/app/perforce/runBuildForShelve` endpoint.


## Approve Multiple Builds at Once
{instance="tcc"}

TeamCity enables you to require explicit permission from designated users before starting certain builds. Approval is needed in two cases:

* When a build configuration has the [](build-approval.md) feature to prevent accidental runs.
* When [](untrusted-builds.md) are enabled, ensuring unverified pull request changes are reviewed before builds begin.

Previously, both options required approving each build individually within a chain. Starting with version 2024.12, clicking **Approve** now grants permission for all builds in the chain at once.

<img src="dk-approve-chain.png" width="706" alt="Approve chain"/>

Learn more: [](build-approval.md) | [](untrusted-builds.md)


## Upload Custom Kotlin Libraries
{instance="tcc"}

Starting with this version, you can upload custom Kotlin libraries as `.jar` to you TeamCity server.

<img src="custom-dsl-library-upload.png" width="706" alt="Upload custom Kotlin library"/>

To start using these libraries in your project's [.kts files](kotlin-dsl.md), add Maven dependencies to required `pom.xml`s.

Learn more: [](kotlin-dsl.md#Add+Custom+Kotlin+Libraries)


## Execute Meta-Runners Inside Containers
{instance="tcc"}

Starting with this version, [meta-runners](working-with-meta-runner.md) provide [container-related settings](container-wrapper.md). These settings are propagated to individual steps, meaning every individual step will be able to run inside the required Docker/Linux image.

<img src="dk-docker-container-settings.png" width="706" alt="Container settings in steps and meta-runners"/>

If a step has its own container settings, they override the global meta-runner configuration.

Learn more: [](working-with-meta-runner.md#Launch+Meta-Runners+in+Containers)


## AWS Integration Enhancements
{instance="tcc"}

### AWS Cloud Profiles

[Amazon EC2 cloud profiles](setting-up-teamcity-for-amazon-ec2.md) will no longer use access keys or the default credentials provider chain, shifting to authentication through [TeamCity AWS connections](configuring-connections.md#AmazonWebServices). This change consolidates all authentication settings into a single connection that can be shared across EC2 cloud profiles and [](aws-credentials.md) build features.

Existing connections will retain legacy authentication but recommend migrating to connection-based access. New EC2 cloud profiles will support only the new authentication method.

### AWS Agents Performance

We reworked the logic for AWS EC2-hosted build agents, resulting in a significant performance boost. See this blog post for more information: [Enhancing TeamCity AWS Agents’ Performance](https://blog.jetbrains.com/teamcity/2024/10/enhancing-teamcity-aws-agents-performance/).


## Performance Enhancements
{instance="tcc"}

In this release, we've overhauled the internal logic for fetching build lists and project trees. Combined with frontend optimization, this upgrade delivers significant [web vitals](https://web.dev/articles/vitals) improvements, especially for users managing large TeamCity instances with extensive project setups.


## Miscellaneous Changes
{instance="tcc"}

* Both TeamCity server and agents now support Java 21.
<!--* Artifacts in the default S3 storage are now downloaded in parallel, using up to 5 threads per artifact. This improvement boosts download speeds by up to 80% based on our benchmarks. The feature is enabled by default, requiring no manual configuration.-->
* EC2-hosted agents now [report](setting-up-teamcity-for-amazon-ec2.md#EC2-Specific+Agent+Parameters) the `system.ec2.instance-life-cycle` parameter that allows you to identify whether this agent uses a spot or on-demand EC2 instance.
* The [HashiCorp Vault connection](hashicorp-vault.md) no longer provides a setting that allows builds to proceed when TeamCity is unable to retrieve Vault secrets. Starting with version 2024.12, such build will fail.


## Upgrade Notes
{instance="tc"}

Before upgrading, we highly recommend reading about important changes in version [2024.07 compared to 2023.03](upgrade-notes.md#2024.07).


## Fixed Issues
{instance="tc"}

See the [TeamCity 2024.07 release notes](teamcity-2024-07-release-notes.md) article for the summary of implemented features and fixed issues.


## Roadmap

See the [TeamCity roadmap](https://www.jetbrains.com/teamcity/roadmap/#teamcity-roadmap) to learn about future updates.


## Your Feedback Matters

We place a high value on your feedback and encourage you to share your thoughts and suggestions. See this link for more information: [Feedback](feedback.md).


