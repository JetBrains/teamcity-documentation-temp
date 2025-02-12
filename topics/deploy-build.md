[//]: # (title: Deploy Build)
[//]: # (auxiliary-id: Deploy Build)

This article gives an overview of the deployment instruments offered by TeamCity.

Most often, a deployment build is a finishing stage of a pipeline, or [build chain](build-chain.md). However, if your project is rather simple and you build it with a single configuration, you can deploy its result with just the final step of this configuration — it's totally up to you.

Depending on your environment, all the deployment processes can be executed on a [build agent](build-agent.md) or in any third-party system.

## Deployment Build Configuration

When you create or edit a build configuration, you can change its type from _Regular_ to _Composite_ or _Deployment_. We talked about composite builds in the [previous tutorial](create-build-chain.md#Complete+Chain+with+Tests). Unlike them, _deployment builds_ work almost like regular ones: you can add the same steps and features or tweak the same settings. In addition to that, a deployment config offers several enhancements for an easier and transparent workflow:
* The __Run__ button of such a build changes to __Deploy__.
* All dependency builds get an extra __Deployments__ section of __Build Results__, from where you can quickly deploy the product.
* Personal builds are disabled to prevent any accidental deployment.

See [more details](deployment-build-configuration.md) about these and other features of such builds.

We suggest that you always use deployment configurations to deliver your software to production.

## Ways to Deploy Product with TeamCity

Here's how you can deploy your build artifacts:
* __Via a command line__, using any general runner like [Command Line](command-line.md) or [PowerShell](powershell.md). This is the most straightforward approach. Just add a build step, select any such runner, and enter commands as if in a regular terminal. The benefits you get from TeamCity in this case are flexible automation, synchronization with the previous build stages, and a convenient view of build results in the TeamCity UI.  
  This way, you can also update distribution files in a third-party storage, like Amazon S3.
* __Using a specific runner for your platform__. For example, if you build .NET projects, the best way to deploy them is via our [.NET runner](net.md). It supports all the relevant .NET commands such as `pack` or `publish` and offers a variety of other features. Other runners are listed under [this section](configuring-build-steps.md).
* __Using a deployer__. TeamCity offers several build runners dedicated to deployment: [SMB Upload](smb-upload.md), [FTP Upload](ftp-upload.md), [SSH Upload](ssh-upload.md), and [SSH Exec](ssh-exec.md). They can upload build artifacts via different protocols and let you configure this upload process in the TeamCity UI.
* __Using the AWS CodeDeploy runner__ to deploy applications to AWS EC2 and on-premises instances. To use this runner, you need to download and install our [AWS CodeDeploy plugin](https://plugins.jetbrains.com/plugin/9018-aws-codedeploy) as described [here](installing-additional-plugins.md). See the related [blog posts](https://blog.jetbrains.com/teamcity/tag/codedeploy/).
{instance="tc"}

>If you deploy products by means of third-party services, TeamCity allows [detaching builds from agents](detaching-build-from-agent.md) before starting the external deployment operations. This helps utilize agents more optimally.  
>This method requires some advanced configuration, so we suggest trying it only after you feel comfortable configuring builds and agents in TeamCity.

## Takeaway

* It is convenient to create a dedicated deployment configuration at the end of the build chain.
* To deploy a product from TeamCity, you need to add a build step and choose its runner based on your preferred solution.
