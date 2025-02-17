[//]: # (title: Builds Schedule)
[//]: # (auxiliary-id: Builds Schedule)

If you have configured any [schedule triggers](configuring-schedule-triggers.md) for the [build configurations](managing-builds.md) in the current project, you can see the builds schedule page by clicking **Builds Schedule** in the [project settings](project-administrator-guide.md#Edit+and+View+Modes) sidebar.

<img src="dk-builds-schedule.png" width="706" alt="Builds Schedule"/>

> The **Builds Schedule** page is not visible, unless there is at least one schedule trigger configured in the current project.
> 
{style="note"}

> The builds schedule page for the [Root Project](project.md#Root+Project) displays the list of triggers for the entire TeamCity server.
> 
{style="tip"}

You can conveniently view the available schedule and plan your builds, optimizing the allocation of dependent hardware and software resources.

From this page it is possible to [edit](configuring-build-triggers.md) the triggers.

The following **Advanced options** are provided:

<table>
<tr>
<td width="300"><p><b>Option</b></p></td>
<td><p><b>Description</b></p></td>
</tr>

<tr>
<td><p>Show disabled triggers</p></td>
<td><p>Includes schedule triggers that were disabled from the <b>Triggers</b> page of the corresponding build configuration.</p></td>
</tr>

<tr>
<td><p>Show paused build configurations</p></td>
<td><p>Includes schedule triggers from build configurations with <a href="changing-build-configuration-status.md#Pausing+Build+Configuration">paused status</a>.</p></td>
</tr>

<tr>
<td><p>Show only upcoming builds</p></td>
<td><p>Includes only scheduled builds that are scheduled to start in the near future.</p></td>
</tr>

<tr>
<td><p>Hide triggers with 'Trigger only if there are pending changes' option enabled</p></td>
<td><p>Excludes triggers with the 'Trigger only if there are pending changes' option enabled.</p></td>
</tr>

</table>
