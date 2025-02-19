[//]: # (title: Detaching Build from Agent)
[//]: # (auxiliary-id: Detaching Build from Agent)

If a final step of a build triggers some external service and the build does not require a [build agent](build-agent.md) anymore, the respective runner can detach the build from the agent. This makes this agent available to other builds. The build then continues running on the TeamCity server, and the external service reports its progress directly to the server. Such build steps are called _[agentless steps](configuring-build-steps.md#Agentless+Build+Steps)_.

## Releasing build agent

To release its current build agent, a runner needs to send the `##teamcity[buildDetachedFromAgent]` [service message](service-messages.md). After receiving this message, the agent skips all the following steps of the build, unless they have the "_[Always, even if build stop command was issued](configuring-build-steps.md#Execution+Policy)_" execution policy enabled. If necessary, you can enable it for mandatory final steps — the agent will be released only after completing them.

>Alternatively to using service messages, you can write own server-side plugin that will be polling an external service for a build status. Use the `jetbrains.buildServer.serverSide.agentless.DetachedBuildStatusProvider` extension point for this. Read more about developing custom plugins in [TeamCity Plugin Help](https://plugins.jetbrains.com/docs/teamcity/developing-teamcity-plugins.html).

If the [limit of agentless builds](#Agentless+builds%27+licensing) is not exceeded, the server releases the agent and it instantly becomes available to other builds. Otherwise, the agent stays attached to the build until some of the running agentless builds finish. 

>We highly recommend releasing the agent only during the last build step. Make sure the tasks performed outside TeamCity do not require the build agent.

This service message supports the `trackingInfo` attribute (Unicode, up to 1000 symbols) for providing an information that could help track a build on the TeamCity server (for example, a deployment ID).

## Logging build data

During agentless steps, the external tool should report all build status information and send any other types of requests directly to the TeamCity server via [REST API](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html).

To perform a request, it needs to provide:
* [build-level authentication](artifact-dependencies.md#build-level-auth) credentials specified as [build system properties](configuring-build-parameters.md):
   * username: `%\system.teamcity.auth.userId%`
   * password: `%\system.teamcity.auth.password%`
* [build ID](build-results-page.md#Internal+Build+ID): `%\teamcity.build.id%`
* TeamCity server URL: `%\teamcity.serverUrl%`

An agent should send these parameters to the external software in advance, before being released.

### Logging messages

To log messages, use the following call:

```Shell
POST /app/rest/builds/id:<build_id>/log 
(curl -v --basic --user <username>:<password> --request POST <teamcity.url>/app/rest/builds/id:<build_id>/log --data <message> --header "Content-Type: text/plain")
```

Here, you can send any [service message](service-messages.md) as `<message>`.

An example request to send a warning:

```Shell
POST /app/rest/builds/id:TestBuild/log 
(curl -v --basic --user TeamCityBuildId=87065:lqmT22NStn4ulqmT22NStn4ulqmT22NStn4u --request POST http://localhost:8111/app/rest/builds/id:TestBuild/log --data "##teamcity[message text='Deployment failed' errorDetails='stack trace' status='ERROR']" --header "Content-Type: text/plain")
```

>To structure service messages in a build log, use _[flow tracking](service-messages.md#Message+FlowId)_.

### Finishing build

It is important to ensure that a build, executed externally, delivers a finishing request to the TeamCity server. If the server is temporarily unavailable and cannot receive this request, the external tool should retry until this operation is successful. Without the finishing request, the build will be running on the TeamCity server indefinitely, until reaching its specified timeout, if any.

To finish a build, use the following call:

```Shell
PUT /app/rest/builds/id:<build_id>/finish
(curl -v --basic --user <username>:<password> --request PUT <teamcity.url>/app/rest/builds/id:<build_id>/finish)
```

Alternatively, you can finish it by sending the exact finish date as a non-empty string in the `yyyyMMdd'T'HHmmssZ` format:

```Shell
PUT /app/rest/builds/id:<build_id>/finishDate
(curl -v --basic --user <username>:<password> --request PUT <teamcity.url>/app/rest/builds/id:<build_id>/finishDate --data "20201231T235959+0000" --header "Content-Type: text/plain")
```

>If you need to end a build as "Failed", log the `buildProblem` message, as described [here](service-messages.md#Reporting+Build+Problems).

<anchor name="DetachingBuildfromAgent-agentless-licensing"/>

## Agentless builds' licensing

The number of builds that can simultaneously run without an agent is limited by the number of your active [agent licenses](licensing-policy.md#Number+of+Agents). For example, if you have 10 agent licenses, you can run in parallel up to 10 regular builds on agents plus up to 10 agentless builds. As soon as you reach the limit of running agentless builds, TeamCity will not detach steps in the following builds until some of the current agentless builds finish.
{instance="tc"}

The number of builds that can simultaneously run without an agent is limited by the number of your active agent licenses. For example, if you have 10 agent licenses, you can run in parallel up to 10 regular build on agents plus up to 10 agentless builds. As soon as you reach the limit of running agentless builds, TeamCity will not detach steps in the following builds until some of the current agentless builds finish.
{instance="tcc"}

<seealso>
        <category ref="concepts">
            <a href="configuring-build-steps.md#Agentless+Build+Steps">Agentless Build Step</a>
        </category>
</seealso>

