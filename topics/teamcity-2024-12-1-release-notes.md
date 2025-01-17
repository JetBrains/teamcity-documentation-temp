[//]: # (title: TeamCity 2024.12.1 Release Notes)
[//]: # (auxiliary-id: TeamCity 2024.12.1 Release Notes)


**Build 174458, 17 January 2025**


<!-- project: TeamCity Fix versions: {2024.12.1} #Fixed #Testing visible to: {All Users} -{Trunk issue} -->

### Bug

**[TW-91446](https://youtrack.jetbrains.com/issue/TW-91446/Build-tags-are-truncated)** — Build tags are truncated

**[TW-91072](https://youtrack.jetbrains.com/issue/TW-91072/Token-management-tokens-scope-can-be-silently-expanded-during-the-copying-project)** — Token management: token's scope can be silently expanded during the copying project

**[TW-89399](https://youtrack.jetbrains.com/issue/TW-89399/Revision-computation-in-a-build-with-checkout-rules-should-consider-all-available-DAGs-for-all-variations-of-VCS-root-parent)** — Revision computation in a build with checkout rules should consider all available DAGs for all variations of VCS root parent

**[TW-91513](https://youtrack.jetbrains.com/issue/TW-91513/Builds-history-can-be-lost-of-project-was-deleted-from-DSL-and-then-restored)** — Builds history can be lost of project was deleted from DSL and then restored

**[TW-75215](https://youtrack.jetbrains.com/issue/TW-75215/Information-about-dependencies-is-not-updated-after-the-finishing-the-build)** — Information about dependencies is not updated after the finishing the build

**[TW-91309](https://youtrack.jetbrains.com/issue/TW-91309/TeamCity.Node-plugin-fails-with-Failed-to-find-build-runner-settings-error)** — TeamCity.Node plugin fails with "Failed to find build runner settings" error

**[TW-91178](https://youtrack.jetbrains.com/issue/TW-91178/Kubernetes-Executor-swabra-usage-can-potentially-eliminate-compilation-results)** — Kubernetes Executor: swabra usage can potentially eliminate compilation results

**[TW-89052](https://youtrack.jetbrains.com/issue/TW-89052/Unable-to-specify-context-parameters-for-versioned-settings-Editing-of-the-project-settings-is-disabled)** — Unable to specify context parameters for versioned settings: Editing of the project settings is disabled

**[TW-90750](https://youtrack.jetbrains.com/issue/TW-90750/Lots-of-Requested-pool-for-non-existing-project-with-id-...-messages-in-the-server-log)** — Lots of "Requested pool for non-existing project with id ..." messages in the server log

**[TW-90374](https://youtrack.jetbrains.com/issue/TW-90374/AWS-connection-Sakura-UI-all-fields-are-responsive-in-read-only-mode)** — AWS connection Sakura UI: all fields are responsive in read-only mode

**[TW-85769](https://youtrack.jetbrains.com/issue/TW-85769/SSH-agent-build-feature-fails-with-NPE-with-Windows-native-SSH)** — SSH agent build feature fails with NPE with Windows native SSH

**[TW-90610](https://youtrack.jetbrains.com/issue/TW-90610/Indicate-in-the-build-log-why-build-cannot-be-detached-from-the-agent)** — Indicate in the build log why build cannot be detached from the agent

**[TW-91529](https://youtrack.jetbrains.com/issue/TW-91529/MySQL-8.4-add-allowPublicKeyRetrieval-to-database-connection-URL-by-default-to-avoid-Public-Key-Retrieval-is-not-allowed-error)** — MySQL 8.4: add allowPublicKeyRetrieval to database connection URL by default to avoid "Public Key Retrieval is not allowed" error

**[TW-90946](https://youtrack.jetbrains.com/issue/TW-90946/Kubernetes-Executor-Manage-projects-agent-cloud-profiles-permission-doesnt-allow-user-to-edit-Executor)** — Kubernetes Executor: "Manage project's agent cloud profiles" permission doesn't allow user to edit Executor

**[TW-90965](https://youtrack.jetbrains.com/issue/TW-90965/Favorite-builds-page-does-not-show-favorite-builds-if-there-are-many-queued-and-or-running-builds)** — Favorite builds page does not show favorite builds if there are many queued and/or running builds

**[TW-91159](https://youtrack.jetbrains.com/issue/TW-91159/Token-management-the-same-404-error-is-shown-always-during-generation-of-new-token-with-incorrect-repository-settings)** — Token management: the same 404 error is shown always during generation of new token with incorrect repository settings

**[TW-90942](https://youtrack.jetbrains.com/issue/TW-90942/Kubernetes-Executor-Choose-agent-provider-page-is-opened-after-choosing-Edit-from-the-table-with-subprojects-executors)** — Kubernetes Executor: "Choose agent provider" page is opened after choosing "Edit" from the table with subprojects' executors

**[TW-91575](https://youtrack.jetbrains.com/issue/TW-91575/Agent-may-hang-during-artifact-publishing)** — Agent may hang during artifact publishing

**[TW-91531](https://youtrack.jetbrains.com/issue/TW-91531/Cannot-request-build-details-via-REST-API-if-the-user-who-approved-the-build-does-not-exist-anymore)** — Cannot request build details via REST API, if the user who approved the build does not exist anymore

**[TW-91517](https://youtrack.jetbrains.com/issue/TW-91517/Cannot-install-IntelliJ-Inspections-and-Duplicates-Engine-2024.3)** — Cannot install IntelliJ Inspections and Duplicates Engine 2024.3

**[TW-91144](https://youtrack.jetbrains.com/issue/TW-91144/Token-Management-Provide-better-errors-in-case-of-network-problems-during-a-token-generation)** — Token Management: Provide better errors in case of network problems during a token generation

**[TW-91348](https://youtrack.jetbrains.com/issue/TW-91348/teamcity-startup.log-contains-all-messages-from-teamcity-server.log-after-changing-logging-preset)** — teamcity-startup.log contains all messages from teamcity-server.log after changing logging preset

**[TW-87316](https://youtrack.jetbrains.com/issue/TW-87316/Test-Retry-doesnt-retry-when-NUnit-test-fixture-has-type-parameters)** — Test Retry doesn't retry when NUnit test fixture has type parameters

**[TW-91551](https://youtrack.jetbrains.com/issue/TW-91551/Unexpected-error-on-Agent-Requirements-page)** — Unexpected error on Agent Requirements page

**[TW-90365](https://youtrack.jetbrains.com/issue/TW-90365/AWS-EC2-cannot-load-StartInstanceData-from-event-wont-start-a-new-instance)** — AWS EC2: cannot load StartInstanceData from event; won't start a new instance

**[TW-91135](https://youtrack.jetbrains.com/issue/TW-91135/Token-management-Error-message-during-the-token-generation-can-be-hidden-in-VCS-root-settings-under-the-Generate-new-token)** — Token management: Error message during the token generation can be hidden in VCS root settings under the Generate new token dialog

**[TW-91193](https://youtrack.jetbrains.com/issue/TW-91193/Version-banner-is-in-the-way)** — Version banner is in the way

**[TW-90887](https://youtrack.jetbrains.com/issue/TW-90887/Token-Management-Add-a-way-to-see-more-than-10-tokens)** — Token Management: Add a way to see more than 10 tokens

**[TW-91187](https://youtrack.jetbrains.com/issue/TW-91187/Token-management-Tokens-are-not-applied-to-VCS-root-feature-without-clicking-on-the-Confirm-button)** — Token management: Tokens are not applied to VCS root/feature without clicking on the Confirm button

**[TW-90166](https://youtrack.jetbrains.com/issue/TW-90166/Deadlock-prevents-builds-from-starting-Waiting-for-the-build-queue-distribution-process)** — Deadlock prevents builds from starting 'Waiting for the build queue distribution process'

**[TW-91354](https://youtrack.jetbrains.com/issue/TW-91354/Build-does-not-start-because-of-exception-in-NuGetFeedParametersProvider)** — Build does not start because of exception in NuGetFeedParametersProvider

**[TW-90927](https://youtrack.jetbrains.com/issue/TW-90927/Kubernetes-UI-design-review)** — Kubernetes UI design review

**[TW-82627](https://youtrack.jetbrains.com/issue/TW-82627/Exception-in-search-request)** — Exception in search request

**[TW-90857](https://youtrack.jetbrains.com/issue/TW-90857/Horizontal-scroll-is-shown-under-the-main-navigation-sidebar)** — Horizontal scroll is shown under the main navigation sidebar

**[TW-91493](https://youtrack.jetbrains.com/issue/TW-91493/Experimental-Overview-usage-statistics-are-reported-incorrectly-when-Sakura-is-enabled-by-default-globally)** — Experimental Overview usage statistics are reported incorrectly when Sakura is enabled by default globally

**[TW-90986](https://youtrack.jetbrains.com/issue/TW-90986/Improve-wording-for-GitHub-Webhook-trigger-tests-connection-in-case-when-VCS-root-doesnt-use-App-as-authentication-method)** — Improve wording for GitHub Webhook trigger tests connection, in case when VCS root doesn't use App as authentication method

**[TW-87491](https://youtrack.jetbrains.com/issue/TW-87491/S3-Artifacts-Storage-not-listing-CloudFront-Distributions)** — S3 Artifacts Storage not listing CloudFront Distributions

**[TW-91430](https://youtrack.jetbrains.com/issue/TW-91430/Unexpected-JSP-error-when-trying-to-open-Dependencies-tab-and-inaccessible-snapshot-dependency-exists)** — Unexpected JSP error when trying to open Dependencies tab and inaccessible snapshot dependency exists

**[TW-88888](https://youtrack.jetbrains.com/issue/TW-88888/New-failed-tests-can-be-shown-only-in-the-full-report-on-the-GitHub-Checks-page-if-there-are-a-lot-of-tests-should-be-shown-at)** — New failed tests can be shown only in the full report on the GitHub Checks page if there are a lot of tests (should be shown at the top of the list)

**[TW-90665](https://youtrack.jetbrains.com/issue/TW-90665/Usability-problem-when-activating-legacy-licenses)** — Usability problem when activating legacy licenses

**[TW-90879](https://youtrack.jetbrains.com/issue/TW-90879/Token-Management-The-search-by-projects-is-missing-in-the-Generate-new-token-dialog)** — Token Management: The search by projects is missing in the "Generate new token" dialog

**[TW-91188](https://youtrack.jetbrains.com/issue/TW-91188/Token-management-popup-may-freeze-during-the-generation-of-the-new-GitLab-token)** — Token management: popup may freeze during the generation of the new GitLab token

**[TW-89606](https://youtrack.jetbrains.com/issue/TW-89606/Artifacts-migrated-between-non-builtin-storages-are-not-cleaned-up)** — Artifacts migrated between non-builtin storages are not cleaned up

**[TW-89404](https://youtrack.jetbrains.com/issue/TW-89404/Artifacts-migrated-to-Azure-Storage-are-not-cleaned-up)** — Artifacts migrated to Azure Storage are not cleaned up

**[TW-91421](https://youtrack.jetbrains.com/issue/TW-91421/Native-git-with-propagated-ssh-proxy-does-not-work-on-Windows-bin-sh-line-1-exec-nc-not-found)** — Native git with propagated ssh proxy does not work on Windows: "/bin/sh: line 1: exec: nc: not found"

**[TW-91301](https://youtrack.jetbrains.com/issue/TW-91301/Ensure-cloud-profile-can-be-visited-with-view-permissions)** — Ensure cloud profile can be visited with view permissions

**[TW-88954](https://youtrack.jetbrains.com/issue/TW-88954/Build-in-virtual-configuration-doesnt-show-changes-of-Versioned-settings)** — Build in virtual configuration doesn't show changes of Versioned settings

**[TW-90781](https://youtrack.jetbrains.com/issue/TW-90781/Approve-the-whole-build-chain-group-approval-check-if-the-build-was-already-approved)** — Approve the whole build chain: group approval, check if the build was already approved

**[TW-55523](https://youtrack.jetbrains.com/issue/TW-55523/Composite-builds-publish-available-Artifacts-as-soon-as-they-are-published-by-used-builds)** — Composite builds: publish available Artifacts as soon as they are published by used builds

**[TW-87003](https://youtrack.jetbrains.com/issue/TW-87003/Artifact-rules-with-whitespaces-dont-work-for-composite-builds.)** — Artifact rules with whitespaces don't work for composite builds.

**[TW-91194](https://youtrack.jetbrains.com/issue/TW-91194/Data-sharing-banner-returns-after-every-server-restart)** — Data sharing banner returns after every server restart

**[TW-91108](https://youtrack.jetbrains.com/issue/TW-91108/Token-management-an-unnecessary-horizontal-scroll-is-shown-during-the-generation-of-a-new-token-with-the-GitHub-App-Connection)** — Token management: an unnecessary horizontal scroll is shown during the generation of a new token with the GitHub App Connection

**[TW-89704](https://youtrack.jetbrains.com/issue/TW-89704/Error-Unexpected-root-page-type-20-after-quitting-from-the-Artifacts-Migration-Tool-if-migration-target-was-changed)** — Error "Unexpected root page type: 20" after quitting from the Artifacts Migration Tool if migration target was changed

**[TW-89352](https://youtrack.jetbrains.com/issue/TW-89352/Two-copies-of-artifacts-are-in-TeamCity-after-migration-of-artifacts-from-external-storage-to-the-internal-one)** — Two copies of artifacts are in TeamCity after migration of artifacts from external storage to the internal one

**[TW-89347](https://youtrack.jetbrains.com/issue/TW-89347/Artifacts-migrated-from-not-buit-in-storage-are-not-available-in-TeamCity-after-migration)** — Artifacts migrated from not buit-in storage are not available in TeamCity after migration

**[TW-90985](https://youtrack.jetbrains.com/issue/TW-90985/New-Build-build-step-selector-Show-more-for-meta-runners-doesnt-open-the-whole-list-of-meta-runners)** — New Build build step selector: "Show more" for meta-runners doesn't open the whole list of meta-runners

**[TW-86927](https://youtrack.jetbrains.com/issue/TW-86927/Matrix-build.-No-enabled-compatible-agents-UI-message-using-Shared-Resources-build-feature-and-teamcity.locks.readLock-parameter)** — Matrix build. "No enabled compatible agents" UI message using "Shared Resources" build feature and teamcity.locks.readLock parameter

**[TW-91308](https://youtrack.jetbrains.com/issue/TW-91308/Error-viewing-settings-for-project-with-NUnit-runner)** — Error viewing settings for project with NUnit runner

**[TW-91186](https://youtrack.jetbrains.com/issue/TW-91186/Token-management-no-loader-is-shown-while-loading-the-VCS-Auth-Tokens-tab)** — Token management: no loader is shown while loading the VCS Auth Tokens tab

**[TW-91074](https://youtrack.jetbrains.com/issue/TW-91074/Token-Management-A-message-about-expanding-the-tokens-scope-can-be-wrongly-shown-for-GitHub-App-roots)** — Token Management: A message about expanding the token's scope can be wrongly shown for GitHub App roots

**[TW-91302](https://youtrack.jetbrains.com/issue/TW-91302/Unauthorized-AJAX-request-should-not-try-to-redirect-to-home)** — Unauthorized AJAX request should not try to redirect to home

**[TW-75412](https://youtrack.jetbrains.com/issue/TW-75412/Tests-from-usual-dependencies-are-reported-to-a-build-with-Parallel-Tests-feature)** — Tests from usual dependencies are reported to a build with Parallel Tests feature

**[TW-90675](https://youtrack.jetbrains.com/issue/TW-90675/Canceled-Snapshot-dependency-prevents-parent-build-from-starting-if-Build-on-the-same-agent-option-is-set)** — Canceled Snapshot dependency prevents parent build from starting, if "Build on the same agent" option is set

**[TW-91017](https://youtrack.jetbrains.com/issue/TW-91017/Conditional-dependencies-Tags-with-white-spaces-do-not-work-from-skipQueuedBuilds-service-message)** — [Conditional dependencies] Tags with white spaces do not work from skipQueuedBuilds service message

**[TW-90619](https://youtrack.jetbrains.com/issue/TW-90619/Build-step-conditions-help-icon-points-to-wrong-article)** — Build step conditions help icon points to wrong article

**[TW-91177](https://youtrack.jetbrains.com/issue/TW-91177/Deprecated-options-in-DSL-extension-generate-non-deprecated-classes)** — Deprecated options in DSL extension generate non-deprecated classes

**[TW-88045](https://youtrack.jetbrains.com/issue/TW-88045/Close-button-not-visible-in-build-log-dialog)** — Close button not visible in build log dialog

**[TW-90909](https://youtrack.jetbrains.com/issue/TW-90909/When-per-project-agents-filtering-is-enabled-terminal-to-agent-cannot-be-opened)** — When per-project agents filtering is enabled, terminal to agent cannot be opened

**[TW-90560](https://youtrack.jetbrains.com/issue/TW-90560/Long-build-status-is-too-close-to-the-Edit-configuration-button)** — Long build status is too close to the "Edit configuration" button

**[TW-88941](https://youtrack.jetbrains.com/issue/TW-88941/JB-license-DELETE-and-POST-requests-to-app-rest-server-licensingData-do-not-work-with-new-licenses)** — JB license: DELETE and POST requests to app/rest/server/licensingData do not work with new licenses

**[TW-90244](https://youtrack.jetbrains.com/issue/TW-90244/New-breadcrumbs-Inconsistent-breadcrumb-for-Edit-project-Cloud-profiles-tab)** — New breadcrumbs: Inconsistent breadcrumb for Edit project -> Cloud profiles tab

**[TW-90899](https://youtrack.jetbrains.com/issue/TW-90899/Static-UI-extensions-look-bad-on-Administration-pages-after-main-navigation-redesign)** — Static UI extensions look bad on Administration pages after main navigation redesign

**[TW-90635](https://youtrack.jetbrains.com/issue/TW-90635/Provide-a-way-to-see-the-VCS-Auth-Tokens-tab-in-read-only-mode-for-users-with-read-only-access-to-Project-Settings-Project)** — Provide a way to see the "VCS Auth Tokens" tab in read-only mode for users with read-only access to Project Settings (Project Developers)

**[TW-90552](https://youtrack.jetbrains.com/issue/TW-90552/Modal-dialogs-break-layout)** — Modal dialogs break layout

**[TW-83706](https://youtrack.jetbrains.com/issue/TW-83706/AWS-cloud-integration-does-not-recover-in-case-of-an-error-in-Refresh-Instances-action)** — AWS cloud integration does not recover in case of an error in "Refresh Instances" action

**[TW-70202](https://youtrack.jetbrains.com/issue/TW-70202/Do-not-disable-clean-up-of-the-deleted-entities-if-the-problematic-project-build-configuration-is-known)** — Do not disable clean-up of the deleted entities if the problematic project / build configuration is known


### Performance Problem

**[TW-90115](https://youtrack.jetbrains.com/issue/TW-90115/Reduce-delay-till-copied-project-configuration-files-are-stored-on-disk)** — Reduce delay till copied project configuration files are stored on disk

**[TW-91174](https://youtrack.jetbrains.com/issue/TW-91174/vacuum-analyze-customdatabody-doesnt-reduce-dead-tuple-count)** — 'vacuum analyze custom_data_body' doesn't reduce dead tuple count

**[TW-91377](https://youtrack.jetbrains.com/issue/TW-91377/Hanging-REST-API-call-dependencies-loop)** — Hanging REST API call (dependencies loop?)

**[TW-91351](https://youtrack.jetbrains.com/issue/TW-91351/Reducing-HTTP-requests-and-data-transfer-during-page-load)** — Reducing HTTP requests and data transfer during page load

**[TW-91155](https://youtrack.jetbrains.com/issue/TW-91155/Requests-to-win32-userMessages.html-can-be-handled-too-slowly)** — Requests to /win32/userMessages.html can be handled too slowly

**[TW-91140](https://youtrack.jetbrains.com/issue/TW-91140/NodesAwareLogMessagePersister-persistRemotely-can-occupy-lots-of-threads-of-the-normal-executor-for-significant-time)** — NodesAwareLogMessagePersister persistRemotely can occupy lots of threads of the normal executor for significant time

**[TW-90966](https://youtrack.jetbrains.com/issue/TW-90966/Slow-freezing-of-the-settings-of-a-build-chain-from-a-project-with-many-sub-projects-because-of-constant-calling-of)** — Slow freezing of the settings of a build chain from a project with many sub projects because of constant calling of ProjectImpl.clearOrganizationProjectCacheForSelfAndChildren

**[TW-90773](https://youtrack.jetbrains.com/issue/TW-90773/Slow-opening-run-custom-build-dialog-due-to-slowness-getting-running-build)** — Slow opening run custom build dialog due to slowness getting running build


### Task

**[TW-91636](https://youtrack.jetbrains.com/issue/TW-91636/Docker-images-replace-package-based-Git-installation-with-multistage-build-from-sources-to-keep-Git-2.47.1)** — Docker images: replace package-based Git installation with multistage build from sources to keep Git 2.47.1

**[TW-91392](https://youtrack.jetbrains.com/issue/TW-91392/Allow-to-approve-only-current-build-not-the-whole-buildchain)** — Allow to approve only current build, not the whole buildchain

**[TW-89605](https://youtrack.jetbrains.com/issue/TW-89605/Add-a-metric-for-a-number-of-unregistered-agents)** — Add a metric for a number of unregistered agents


<!-- project: TeamCity Fix versions: 2024.12.1 #Testing #Fixed -{Trunk issue} #{Security Problem}  -->
### Security

5 security problems have been fixed. This number includes both native TeamCity issues and vulnerabilities found in 3rd-party libraries TeamCity depends on. Upstream library issues usually make up the majority of this total number, and are promptly resolved by updating these libraries to their newest versions.

To learn more about fixed vulnerabilities directly related to TeamCity, check out our [Security Bulletin](https://www.jetbrains.com/privacy-security/issues-fixed/?product=TeamCity&version=2024.12). Security bulletins for new versions are typically published within the next few days after the release date.