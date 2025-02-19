[//]: # (title: Licensing Policy)
[//]: # (auxiliary-id: Licensing Policy)

Pricing and new licenses/upgrades are available via the __[official web site](https://www.jetbrains.com/teamcity/buy/)__. If you have any questions on the licensing terms, obtaining or upgrading license keys, or other related topics, please [contact](https://www.jetbrains.com/support/sales/) JetBrains sales department.   
You can review the TeamCity [license agreement](https://www.jetbrains.com/teamcity/buy/license.html) on the official website or in the footer of the installed TeamCity server web UI.

## Licensing Overview

JetBrains offers several licensing options that allow you to scale TeamCity to your needs.  
This section illustrates the main differences between the TeamCity server [editions](#Editions) and provides general information on the TeamCity [Build Agent](build-agent.md) license.

_In general, to use TeamCity for production (Enterprise edition), you need to own one **server license**. In addition, you can acquire an indefinite number of **build agent licenses**, depending on how many builds you want to run in parallel._

For detailed information, refer to the sections below.

<table><tr>

<td>

Professional Server

</td>

<td>

Enterprise Server

</td></tr><tr>

<td>

No license key is required,  __free__

</td>

<td>

A license key is required, [price options](https://www.jetbrains.com/teamcity/buy/#on-premises?licence=enterprise)


</td></tr><tr>

<td>

100 build configurations

</td>

<td>

Unlimited number of build configurations

</td></tr><tr>

<td>

Full access to all product features

</td>

<td>

Free 1-year subscription to upgrades

</td></tr><tr>

<td>

Support via [community forum](https://jb.gg/teamcity-forum)

</td>

<td>

Priority [support](troubleshooting.md) for the [supported releases](teamcity-release-cycle.md)

</td></tr><tr>

<td>

3 build agents included, buy more as necessary

</td>

<td>

3 build agents included, buy more as necessary

</td></tr></table>

If you need more build agents than are included with your TeamCity server edition, you can purchase additional build agent licenses.

<table><tr>

<td>

Build Agent License

</td></tr><tr>

<td>

Connects 1 additional build agent

</td></tr><tr>

<td>

If using Professional edition, adds 10 additional build configurations

</td></tr><tr>

<td>

A license key is required, [price options](https://www.jetbrains.com/teamcity/buy/choose_edition.jsp?license=ADDITIONAL)

</td></tr></table>

<anchor name="LicensingPolicy-editions"/>

## Editions

There are two editions of TeamCity: __Professional__ and __Enterprise__.   
The editions are equal in all the features except for the maximum number of build configurations allowed.   
The same TeamCity distribution and installation are used for both editions. You can switch to the Enterprise edition by entering the appropriate license key. All the data is preserved when the edition is switched.

The current edition in use is noted in the footer of every TeamCity web UI page and on the __Administration__ &gt; __Licenses__ page as well as in `teamcity-server.log` on the server startup.

The __Professional edition__ does not require any license key and can be used free of charge. The only functional difference from the Enterprise edition is a limitation of the maximum number of [build configurations](managing-builds.md). The limit is 100 build configurations. It can be extended by 10 with each agent license key added. You can install several servers with the Professional license.

The __Enterprise edition__ requires a license key, has no limit on the number of build configurations and entitles you to TeamCity [support](troubleshooting.md) from JetBrains for the maintenance period of the license, provided you use a [recent TeamCity version](teamcity-release-cycle.md).

<note>

An additional server in a [high availability set-up](multinode-setup.md) uses the license from the main server and does not require a separate license at this time.
</note>

Each TeamCity edition comes bundled with 3 or more [build agents](build-agent.md). To use more agents than the bundled number, separate build agent license keys can be entered. Additional agents can be added to both editions.

<anchor name="evaluation-license"/>

Besides the Professional and Enterprise licenses, there are two more license types:
* __Trial__ — has an expiration date and provides an unlimited number of agents and build configurations. To obtain the trial license, use the link on [TeamCity download page](https://www.jetbrains.com/teamcity/download/). The trial license can be obtained only once for each major TeamCity version. A second trial license key from the site is not accepted by the same major version of TeamCity server. If you need to extend/repeat the trial, please [contact](https://www.jetbrains.com/support/sales/) our sales department.
* __Open Source__ — this is a special type of license granted for open source projects, it is time-based, and provides an unlimited number of agents. Refer to the details on [this page](https://www.jetbrains.com/teamcity/buy/choose_edition.jsp?license=OPEN_SOURCE).

The TeamCity Licensing Policy does not impose any limitations on the number of instances for any of the IDE plugins or Notifiers.

### Number of Build Configurations

The Enterprise edition has no limit on the number of build configurations.

The Professional edition allows 100 [build configurations](managing-builds.md) per server. Each build agent license key gives you 10 more build configurations in Professional edition in addition to one more agent. All build configurations are counted (i.e. including those in archived projects).

### Number of Agents

TeamCity Professional edition comes bundled with 3 [build agents](build-agent.md). More build agents can be added by purchasing additional agent license keys.

A server license key might include more agent licenses than the default 3. The number of agents stated for the server license on jetbrains.com site notes the total number of agents which will be available. Separate agent license keys can be used with either TeamCity edition (Enterprise and Professional). For more information about purchasing agent licenses, refer to the [product page](https://www.jetbrains.com/teamcity/buy/).

The number of agent licenses limits the number of agents which can be [authorized](build-agent.md#Build+Agent+Status) in TeamCity. The license keys are not bound to specific agents, they just limit the maximum number of functional agents. The licensing makes no difference between local (installed on the TeamCity server machine) and remote agents.   
When there are more authorized agents than the valid agent licenses available, the server fails to start any builds and displays a warning message to all users in the web browser.

## Managing Licenses

You can activate server license keys (and enter non-activated legacy keys) on the __Administration__ &gt; __Licenses__ page of the TeamCity web UI. By default, only users with the System Administrator role can access the page.

<include element-id="license-limitations" from="manage-teamcity-license.md"/>

See the following article for more information: [](manage-teamcity-license.md).



## Valid TeamCity Versions

TeamCity licenses are perpetual for the TeamCity versions they cover. This means that you can run a covered TeamCity version with existing licenses for an unlimited time and the licenses will stay valid for this TeamCity version.   
Each TeamCity license (including Enterprise Server and Agent) has a __maintenance period__ (generally 1 year). The license key is valid for any version of TeamCity released before the license purchase as well as for any version released within the maintenance period. Licenses valid for the major release (changes in the first two release numbers) are also considered valid for the corresponding minor (bugfix) updates (changes in the third release number).

The set of valid licenses defines if the server works in Enterprise mode and how many agents can run builds on the server. Agent licenses are not bound to specific agents and are only used to determine the maximum number of authorized agents.

Before you [upgrade](upgrading-teamcity-server-and-agents.md) to a newer TeamCity version, check the validity of the existing licenses with the new version.   
If the new TeamCity server effective [release date](previous-releases-downloads.md) is not covered by the maintenance period of some of the licenses, the corresponding licenses will not be valid with the TeamCity version and would need [renewing](https://www.jetbrains.com/teamcity/buy/index.jsp#upgradeuser). Generally, license renewal is priced at approximately 50% of the new license price, provided licenses are renewed prior to their expiration.

When a new version is available, TeamCity displays a notification in the web UI and warns you if any of your license keys are incompatible with this new version. A notification on the new TeamCity version is also displayed in the Global Configuration Items of the [Server Health](server-health.md) report, visible to system administrators. System administrators can use the link in the "Some Licenses are incompatible" message to quickly navigate to the [Licenses](#Managing+Licenses) page, where all incompatible licenses will have a warning icon. The information about the license keys installed on your server is secure as it is not sent over the Internet.

Regular upgrades are highly recommended not only because each new release includes a lot of improvements and new features, but also as this is the only way to run a supported version with the latest security patch level.

Note that TeamCity [email support](troubleshooting.md) covers only the [recent TeamCity versions](teamcity-release-cycle.md) and can only be provided to customers who are under active maintenance for their Enterprise server license.


<!--[//]: # (Internal note. Do not delete. "Licensing Policyd197e369.txt")-->    

<anchor name="LicensingPolicy-LicenseExpiration"/>

## License Expiration

If an Enterprise license key is removed from the server, or an trial license expires, or a TeamCity server is upgraded to a version released out of the maintenance window of the available Enterprise license, TeamCity automatically switches to the Professional mode.

If the number of build configurations or the number of authorized agents exceeds the limits imposed by the valid licenses, the server stops to start any builds (pauses the build queue) and displays a warning message to all users in the web browser.

Build Agent Licenses work the same way as the Server Licenses. If you upgrade the server to the version which is not covered by the agent license maintenance window, then this agent license will expire.

Once sufficient valid license keys are entered which cover the server configuration, the builds begin to be started again.

<anchor name="LicensingPolicy-WaystoObtainaLicense"/>

## Ways to Obtain a License

The following ways to switch your server into the Enterprise mode exist:
* [buy](https://www.jetbrains.com/teamcity/buy/) an Enterprise Server license;
* request a 30-day trial license on the [download page](https://www.jetbrains.com/teamcity/download/) (see details [above](#Editions));
* use TeamCity for open-source projects only and [request an open-source license](https://www.jetbrains.com/buy/opensource/?product=teamcity).

## Upgrading From Previous Versions

### Upgrading from TeamCity 5.x and later

Each license has a maintenance period (typically one year since the purchase date). The license is suitable for any TeamCity version released within the maintenance period. Please check the maintenance period of your licenses before upgrading.

### Upgrading from TeamCity 4.x to TeamCity 5.0 and later

Licenses for previous versions of TeamCity needs upgrading, see details at [Licensing and Upgrade](https://www.jetbrains.com/teamcity/buy#upgradeuser) section on the official site.

### Upgrading from TeamCity 3.x to TeamCity 4.0

Owners of TeamCity 3.x Enterprise Server Licenses upgrade to TeamCity 4.x Enterprise Edition free of charge. TeamCity 3.x Build Agent Licenses are compatible with both Professional and Enterprise editions of TeamCity 4.0.

### Upgrading from TeamCity 1.x-2.x to TeamCity 4.0

Any TeamCity 1.x\-2.x license purchased before December, 05, 2008 can be used as one TeamCity 4.0 Build Agent license for both Professional and Enterprise editions of TeamCity 4.0. Additionally, TeamCity 1.x\-2.x customers qualify for _one_ TeamCity Enterprise Server License free of charge. To request your Enterprise Server License, please contact [sales department](https://www.jetbrains.com/support/sales/) with one of your TeamCity 1.x\-2.x licenses.

### Upgrading with IntelliJ IDEA 6.0 License Key

Any IntelliJ IDEA 6.0 license purchased between July 12, 2006 and January 15, 2007 can be used as one TeamCity 4.0 Build Agent license. Additionally, IntelliJ IDEA customers with such licenses qualify for one TeamCity Enterprise Server license free of charge.   
To check TeamCity upgrade availability for your IntelliJ IDEA licenses and to request your Enterprise Server license, please contact [sales department](https://www.jetbrains.com/support/sales/) with one of your IntelliJ IDEA licenses purchased within the above period.


<!--[//]: # (Internal note. Do not delete. "Licensing Policyd197e452.txt")-->

## Copyright and Trademark Notice

The software described in this documentation is furnished under a software license agreement. _JetBrains_, _IntelliJ_, _IntelliJ IDEA_, _YouTrack_, and _TeamCity_ are trademarks or registered trademarks of _JetBrains, s.r.o._ _Windows_ is a registered trademark of _Microsoft Corporation_ in the United States and other countries. _Mac,_ _Mac OS, macOS_ are trademarks of _Apple Inc._, registered in the U.S. and other countries. _Linux_ is a registered trademark of _Linus Torvalds_. All other trademarks are the properties of their respective owners.

<seealso>
        <category ref="concepts">
            <a href="build-agent.md">Build Agent</a>
        </category>
        <category ref="licensing">
            <a href="https://www.jetbrains.com/teamcity/buy/">Licensing &amp; Upgrade</a>
            <a href="teamcity-release-cycle.md">TeamCity Release Cycle</a>
        </category>
</seealso>
