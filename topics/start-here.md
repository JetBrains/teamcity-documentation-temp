# Start Here

JetBrains TeamCity is a user-friendly continuous integration (CI) server for developers and build engineers [free of charge with the Professional Server License](https://www.jetbrains.com/teamcity/buy/index.jsp) and easy to set up!
{instance="tc"}

JetBrains TeamCity is a user-friendly and easy to set up continuous integration (CI) server for developers and build engineers.
{instance="tcc"}

This video gives a general overview of the main TeamCity features and explains the [licensing policy](licensing-policy.md):
{instance="tc"}

This video gives a general overview of the main TeamCity features:
{instance="tcc"}

<video src="https://youtu.be/s68u2shSo6o"
       title="General TeamCity overview"/>

## What can you do with TeamCity?

* Run parallel builds simultaneously on different platforms and environments
* Optimize the code integration cycle and be sure you never get broken code in the repository
* Review on-the-fly test results reporting with intelligent tests reordering
* Run code coverage and duplicates finder for Java and .NET
* Customize statistics on the build's duration, success rate, code quality, and custom metrics
* and much more

To learn more about major TeamCity features, refer to the [official JetBrains website](https://www.jetbrains.com/teamcity/features/index.html).

TeamCity works well not only for administrators and build engineers, but also for developers.
Watch this video to learn about some of its signature features:

<video src="https://youtu.be/icuhBgEFtVM"
       title="TeamCity for developers"/>

The complete list of supported platforms and environments is available [here](supported-platforms-and-environments.md).


## CI/CD Guide

Check out our [CI/CD Guide](https://www.jetbrains.com/teamcity/ci-cd-guide/) for a curated collection of concise articles designed to help both new and experienced users quickly grasp key CI/CD concepts. The guide explores universal topics beyond just TeamCity, covering foundational CI/CD principles, the basics of DevOps, the importance of continuous integration in Agile teams, industry best practices, and much more. Whether you're new to CI/CD or looking to expand your understanding, this guide provides a solid starting point.

## Tutorials

This documentation includes a number of tutorials that walk you through the basic TeamCity operations. For more video guides on configuring TeamCity, visit the official tutorials page: [TeamCity Tutorials](https://www.jetbrains.com/teamcity/tutorials/). 

## Docs Navigation

TeamCity is a powerful and scalable CI/CD solution, adaptable for businesses of any size — from small teams with simple setups to large enterprises managing thousands of build agents running diverse configurations. Whether your team consists of just a few people sharing equal responsibilities or a larger group with specialized roles, tasks typically fall into three main categories:

Server administration
: Involves configuring and maintaining the core TeamCity infrastructure, including the TeamCity server, nodes, build agents (both cloud and local), proxies, databases, and more. Server admins also manage user access and handle maintenance routines like backups and cleanups.
: See the [](server-administrator-guide.md) section for more information.

Project administration
: Focuses on creating and managing projects and build configurations, integrating TeamCity with external services (such as version control systems, artifact storage, and image registries), and setting up user permissions for individual projects.
: Refer to the [](project-administrator-guide.md) section to learn more.

Regular operations
: Daily tasks of running builds (whether through the TeamCity UI or an IDE), tracking and investigating build results, and more.
: The [](user-guide.md) section covers these basic operations.


## Continue Reading

* TeamCity seamlessly integrates with a wide range of VCS hosting platforms, cloud providers, issue trackers, and other essential development tools. For a comprehensive overview of available integrations, visit the [](integrating-teamcity-with-other-tools.md) section.
* Every development team operates with its own unique combination of tools, languages, and workflows. To tailor TeamCity to your specific business needs, explore the [](extending-teamcity.md) section. This section explains how to leverage custom plugins and the robust REST API, which allows you to manage everything from server-wide settings to individual test investigations and mutes.
* CI/CD systems often involve complex setups that combine bare-metal and cloud machines, various tools, custom scripts, and unique environments. When things go wrong, troubleshooting can be challenging. To help you quickly resolve issues, we’ve compiled a list of common problems and solutions in the [](troubleshooting.md) section.


## Useful Resources

* [TeamCity tutorials](https://www.jetbrains.com/teamcity/tutorials/)
* [TeamCity Kotlin DSL API reference](https://teamcity.jetbrains.com/app/dsl-documentation/index.html)
* [TeamCity REST API documentation](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html)
* [SDK documentation for plugin developers](https://plugins.jetbrains.com/docs/teamcity/getting-started-with-plugin-development.html)
