[//]: # (title: Working with Feature Branches)
[//]: # (auxiliary-id: Working with Feature Branches)

_Feature branches_ in distributed version control systems (DVCS) allow you to work on a feature independently of the main development and commit all the changes for the feature onto the branch, merging the changes into the main branch when your feature is complete. This approach brings a number of advantages to software development teams; however, in continuous integration servers that do not have dedicated support for it, it also causes a number of problems, like constant build configurations duplication, poor visibility, and, in the end, loss of control over the process.

TeamCity support for feature branches is continuously extending and, among other features, includes [Branch Remote Run Trigger](branch-remote-run-trigger.md) starting a new personal build each time TeamCity detects changes in a particular branches of the VCS roots of the build configuration and [Automatic Merge](automatic-merge.md) to merge a branch into another after a successful build.

<video src="https://youtu.be/aTE_1A0Fuh0"
title="TeamCity tutorial — How to work with feature branches"/>

## Supported Version Control Systems

[Git](git.md) and [Mercurial](mercurial.md) feature branches are supported as well as Perforce [branch streams support](perforce.md#branch-support).

<anchor name="branchSpec"/>

<anchor name="Working+with+Feature+Branches#WorkingwithFeatureBranches-branchSpec"/>

<anchor name="WorkingwithFeatureBranches-branchSpec"/>

## Configuring Branches

To start working with DVCS branches, you need to set up branch specifications. These specifications specify which branches should be watched for changes.

Once you've configured the branch specification, TeamCity will start to monitor these branches for changes. If your build configuration has [a VCS trigger and a change is found in some branch](configuring-vcs-triggers.md#branch-filter-1), TeamCity will trigger a build in this branch. From the build configuration home page you'll also be able to filter the history, change log, pending changes and issue log by the branch name. Branch names will also appear in the custom build dialog, so you'll be able to manually trigger a custom build on a branch too.

### Common Specification Syntax

<include from="project-administrator-guide.md" element-id="common-branch-spec-syntax"></include>

> For [Git](git.md) and [Mercurial](mercurial.md) VCS roots, the **Branch specification** field is available by default. For [](perforce.md) roots, tick the **Enable feature branches support** option to display this field.
> 
{style="tip"}


> The [default branch](#Default+Branch) is implicitly included in the branch specification, so you do not need to enter it manually.
> 
{style="note"}


### Wildcards

<include from="project-administrator-guide.md" element-id="branch-spec-wildcard"></include>


The part of the branch name matched by the asterisk (`*`) wildcard becomes the short branch name to be displayed in the TeamCity user-level interface (also known as the [logical branch name](#Logical+Branch+Name)). The line can also contain optional parentheses which, when present, denote the part of the pattern to be used as the logical name instead of just *-matched symbols.

> The `*` wildcard must resolve at least one symbol. For example, if the branch specification is `+:refs/heads/feature*`:
> * The `refs/heads/feature1` branch will be matched
> * The `refs/heads/feature-JohnDoe` branch will be matched
> * `refs/heads/feature` branch will NOT be matched
>
{style="warning"}


### Order and Priority

When a single VCS branch is matched by several lines of the branch specification, the most specific (the least characters matched by the pattern) last rule applies.

That is, if the specification contains an exact pattern matching the branch (i.e. a pattern without the `*` wildcard), then the last such pattern is used. So if you have a specification like this:

```
+:refs/heads/release-v1
-:refs/heads/release-v1
```

then the last pattern will win and the branch will be excluded.

If two wildcard-containing rules match the same branch but conflict, the rule generating the shortest logical name takes precedence. For example:

```
+:refs/heads/*/hotfix
-:refs/heads/v1/*
```

For the `refs/heads/v1/hotfix` branch, these rules are ambiguous:

* Include `v1/hotfix` under the logical name `v1`
* Exclude `v1/hotfix` under the logical name `hotfix`

The `v1` logical name is shorter than `hotfix`, meaning TeamCity had to resolve fewer wildcard characters. This makes the first rule more specific, and it will be prioritized. Therefore, the `refs/heads/v1/hotfix` branch will be included.

>To run builds on GitHub and GitLab pull request branches, use the [Pull Requests](pull-requests.md) build feature.

### Comments and Service Expressions

Branch specifications also support expressions starting with the `#` character. As opposed to regular expressions that define names of branches that should be included or excluded to (from) the specification, these are specially crafted "service" lines designed for specific tasks.

* Any line starting with `#` is treated as regular comment.

    ```
    +:refs/heads/main
    # Exclude legacy branch. DO NOT REMOVE!
    -:refs/heads/release-v1
    ```

* The `#! escape: <YOUR_CHARACTER>` expression defines an escape character that allows you to use special chars in branch names. For example, to write a branch specification rule for the "release-(7.1)" branch, you need to escape round brackets. The default escape symbol in TeamCity is backslash (`\`), so your branch specification should look like the following:

    ```
    +:release-\(7.1\)
    ```
    If you want to use a different escape character, define it as shown below:

    ```
    #! escape: !
    +:release-!(7.1!)
    ```
  
* The `#! fallbackToDefault: false` expression allows you to prohibit TeamCity from using a [default branch](#Default+Branch) whenever the required branch is not found. For example, when you utilize [](teamcity-rest-api.md) to start a build for a non-existent branch (by default, TeamCity will run a new build for the default branch in this case).

    ```
    #! fallbackToDefault: false
    +:included_branch
    -:excluded_branch
    ```




## Branch-Specific Build Configuration Settings

With the help of [versioned settings](storing-project-settings-in-version-control.md), you can create build configurations with variable settings for every repository branch. See the following article for an example: [Branch-Specific Settings](storing-project-settings-in-version-control.md#Example%3A+Branch-Specific+Settings).


## Default Branch

When configuring a VCS root for DVCS, you need to specify the branch name to be used as the default one. The default branch has special meaning:
* It is a fallback branch to use when the branch is not specified or the specified branch is not included by the branch specification (for example, when someone just clicks __Run__ without selecting a branch).
* It can be used when displaying sequences of builds and changes and reaching the moment of the branch creation.
* The default branch allows using different branches in different VCS roots (for example, if one of the roots is Git and another is Mercurial) and in different builds when they are linked by a snapshot dependency. When the top chain build is triggered in the default branch, all its dependencies will be built in their respective default branches as well.

Unless disabled via a [branch filter](branch-filter.md), the default branch is always implicitly included into the branch specification. In the TeamCity UI, the default branch is marked with a darker background of the branch marker.

>Note that unlike the _Branch specification_ field, _Default branch_ does not support parentheses.

## My Branches

TeamCity can identify and group branches, based on the commits of the current TeamCity user.

Select the _My Branches_ group in the branch filter to display all active branches whose last 100 changes include your commits, based on the defined VSC usernames.

 <anchor name="logicalBranchName"/>

## Logical Branch Name

A logical branch name is a branch name shown in the user interface for the builds and on build configuration level. A logical branch name is regularly a part of the full VCS-specific branch name. It is calculated by applying a [branch specification](#Configuring+Branches) to the branch name from the version control.

For example, if the branch specification is defined like this:

```
+:refs/heads/*
```

then the part matched by `*` (for example, `master`) is a logical branch name. 

If the branch specification pattern uses parentheses, the logical name is made up of the part of the name within the parentheses; to see the `v8.1/feature1` logical name displayed in the UI for the VCS branch `refs/heads/v8.1/feature1`, use this:

```
+:refs/heads/(v8.1/*)
```

You do not need to include the default branch into the branch specification as it is already included there implicitly. However, if you want to have some short logical branch name for the default branch in the UI, for example, `master`, you can include it in the branch specification and use the parentheses:

```
+:refs/heads/(master)
```

## Builds

You can manually run a build on a specific branch in one of the two ways:
* Click __Run__ opposite the required branch in the build list.
* Open the [Custom Run](running-custom-build.md) dialog, go to the __Changes__ tab, and choose the required branch in the "_Build branch_" drop-down menu.

To run builds from a specific branch or set of branches automatically, configure [build triggers](configuring-build-triggers.md).

Builds from branches are easily recognizable in the TeamCity UI, because they are marked with a special label:

<img src="branches.png" alt="Builds from branches" width="750"/>

You can also filter history by a branch name if you're interested in a particular branch. TeamCity assigns a branch label to the builds from the default branch too.

## Changes

For each build TeamCity shows changes included in it. For builds from branches the changes' calculation process takes the branch into account and presents you with the changes relevant to the build branch. The changes for a build in a branch are calculated as the changes from the build's revision to the previous build in the branch or a build in the default branch.   
The change log with its graph of commits will help you understand what is going on in the monitored branches.

<img src="branchesChange.png" alt="Build changes" width="750"/>

With the __Show graph__ option enabled by default TeamCity displays build markers on the graph.

<anchor name="ActiveBranches"/>

## Active Branches

In a build configuration with configured branches, most UI pages show active branches by default.

An _active branch_ is a branch with the recent activity: it has recent builds or exists in the repository with recent commits.    

The threshold for activity can be configured via build configuration parameters. The parameters can be changed either in a build configuration (this will affect one build configuration only), project, or in the [internal properties](server-startup-properties.md#TeamCity+Internal+Properties) (this defines defaults for the entire server). A parameter in the configuration overrides a parameter in the [internal properties](server-startup-properties.md#TeamCity+Internal+Properties).
{instance="tc"}

The threshold for activity can be configured via build configuration parameters. The parameters can be changed in a build configuration (this will affect one build configuration only) or project.
{instance="tcc"}

A branch is considered active if:
* it is present in the VCS repository and has recent commits (i.e. commits with the age less than the value of the integer parameter `teamcity.activeVcsBranch.age.days`, 7 days by default).
* or it has recent builds (i.e. builds with the age less than the value of the integer parameter `teamcity.activeBuildBranch.age.hours`, 24 hours by default).   
A closed VCS branch with builds will still be displayed as active during 24 hours after last build. To remove closed branches from display, set `teamcity.activeBuildBranch.age.hours=0`.

## Tests

TeamCity tries to detect new failing tests in a build, and for those tests which are not new, you can see in which build the test started to fail. This functionality is aware of branches too, i.e. when the first build is calculated, TeamCity traverses builds from the same branch.

Additionally, a [branch filter](branch-filter.md) is available on the test details page and you can see a history of test passes or failures in a single branch.

## Failure Conditions

If a [build failure condition](build-failure-conditions.md) is configured as follows: _build metric has changed comparing to a last successful/finished/pinned build_, TeamCity will try to compare the current build with the build from the same branch. If there is no suitable build in the same branch, it will use the build from the default branch and add the respective message to the build log. Note that currently, if the default branch is disabled by the [branch filter](branch-filter.md), TeamCity will not be able to process the build failure condition properly (see the related issue [TW-74884](https://youtrack.jetbrains.com/issue/TW-74884)).

## Triggers

The VCS trigger is fully aware of branches and will trigger a build once a check-in is detected in a branch. All VCS trigger options like per-checkin triggering, quiet period, and triggering rules are directly available for builds from branches. By default, the Schedule and Finish build trigger will watch for builds in the default branch only.

Additionally, a [branch filter](branch-filter.md) can be specified for the VCS, Schedule, and Finish build triggers.

## Dependencies

If a build configuration with branches has snapshot dependencies on other build configurations with branches, then when a build in a branch is triggered, the other builds in the chain will also get the branch associated, if the branches in the VCS roots of the builds have the same [logical name](#Logical+Branch+Name) and this branch is not excluded by the branch specification. The VCS roots of the builds can point to different repositories, but the logical branch name must be the same.

If this condition is met, the branches with this name will be checked out and all the builds down the chain (which the build triggered depends on) and all the builds up the chain (depending on the triggered build) will be marked with the same branch. Otherwise, the default branch will be checked out.

It is possible to configure artifact dependencies to retrieve artifacts from a build from a specific branch: artifact dependencies will use builds from the branch specified. The same applies to the [Schedule](configuring-schedule-triggers.md) and [Finish Build](configuring-finish-build-trigger.md) triggers.

## Notifications

All notification rules except "My changes" will only notify you on builds from the default branch. At the same time, the "My changes" rule will work for builds from all available branches.

## Build Configuration Status

The build configuration status is calculated based on the builds from the default branch only. Consequently, per-configuration investigation works for builds from the default branch. For example, a successful build from a non-default branch will not remove a per-configuration investigation, but a successful build from the default branch will.

## Multiple VCS Roots

If a build configuration has two (or more) VCS roots with specified branch filters, the triggering behavior might get more complicated.

The VCS trigger groups branches from multiple roots by their [logical branch names](#Logical+Branch+Name). When some root does not have a branch from the other root, its default branch is used.

Example: 2 VCS roots have the same default branch `refs/heads/master`. Root1 has the branch specification `refs/heads/7.1/*` and new commits in branches `refs/heads/7.1/feature1` and `refs/heads/7.1/feature2`. Root2 has the specification `refs/heads/devel/*` and new commits in the branch `refs/heads/devel/feature1`.  
Here, `feature1` is the logical name relevant to two branches with different paths: `.../7.1/feature1` and `.../devel/feature1`.

In this case, the VCS trigger will run 3 builds with revisions from the following branches' combinations:

<table><tr>

<td>Build number</td>

<td>

root1

</td>

<td>

root2

</td>

<td>Description</td>

</tr><tr>

<td>1</td>

<td>

`refs/heads/master`

</td>

<td>

`refs/heads/master`

</td>

<td>

Default branches are [implicitly added](#Default+Branch) to each specification.

</td>

</tr><tr>

<td>2</td>

<td>

`refs/heads/7.1/feature1`

</td>

<td>

`refs/heads/devel/feature1`

</td>

<td>

The `feature1` logical name is present in specifications of both roots.

</td>

</tr><tr>

<td>3</td>

<td>

`refs/heads/7.1/feature2`

</td>

<td>

`refs/heads/master`

</td>

<td>

The `feature2` logical name is present in the specification of root1 but absent in the specification of root2. Thus, root2 falls back to the default branch.

</td>

</tr></table>


## Build Parameters

If you need to get the branch name in the build script or use it in other build configuration settings as a parameter, refer to [Predefined Build Parameters](predefined-build-parameters.md).

## Clean-up

Clean-up rules are applied [independently](teamcity-data-clean-up.md#Base+Rule+Behavior+for+Build+Configurations+with+Feature+Branches) to each [active branch](#Active+Branches).

## Manual Branch Merging

You can merge branches in TeamCity manually, for example, if you want to merge branches only after a code review / approval, or if you want to perform the merge despite the test failure in a branch.

__To merge sources manually__: 

Open the [build results page](working-with-build-results.md), click the __Actions__ menu in the upper right corner and select __Merge this build sources__.   
 The dialog that appears enables you to select the destination branch and add a commit message (required).

It is also possible to merge branches [automatically](automatic-merge.md).




<seealso>
        <category ref="admin-guide">
            <a href="git.md">Git</a>
            <a href="mercurial.md">Mercurial</a>
            <a href="vcs-root.md">VCS Root</a>
        </category>
</seealso>
