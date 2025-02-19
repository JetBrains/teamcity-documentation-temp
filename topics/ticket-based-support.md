# Ticket-Based Support

Customers who own a TeamCity Enterprise server license with active maintenance can submit a ticket directly to our support team. Each ticket is assigned to one of our support engineers, who will respond to your question or issue, and assist with diagnosing the problem.

If you are using a free Professional Server license, you can explore other ways to raise issues or engage with the community by visiting our [](troubleshooting.md) page.

## Support Tiers

JetBrains offers three tiers of support: Standard support (included with all active Enterprise server licenses), Business support, and Enterprise support. The Business and Enterprise support plans are optional paid plans that provide additional benefits, such as 24/7 availability, a private Slack channel, and varying service-level agreements (SLAs) for response times. For more details, visit the [Support plans](https://www.jetbrains.com/teamcity/support/#plans) page.

## Availability

Support is provided Monday through Friday, during Central European business hours. Our typical response time is within 24 hours, excluding weekends and public holidays in Central Europe.
{instance="tc"}

Support is provided Monday through Friday, during Central European business hours. Our typical response time is within a few hours, excluding weekends and public holidays in Central Europe.
{instance="tcc"}

If you have upgraded your support tier to one of the optional [Business or Enterprise support plans](https://www.jetbrains.com/teamcity/support/#plans), you will benefit from 24/7 availability and varying SLAs for response times.

## Prerequisites

Before submitting a ticket to our support team, we suggest that you check if the answer is already available in:

* Online documentation: [Common Problems](common-problems.md), [Known Issues](known-issues.md), [Licensing Policy](licensing-policy.md)
  {instance="tc"}

* Online documentation: [Common Problems](common-problems.md), [Known Issues](known-issues.md)
  {instance="tcc"}

* [Public issue tracker](https://youtrack.jetbrains.com/issues/TW)

* [Community forum](https://jb.gg/teamcity-forum)

When submitting a ticket, make sure to:
{instance="tc"}

* Provide all necessary information by reviewing the [guidelines on reporting issues](reporting-issues.md).
  {instance="tc"}

* Use a [recent TeamCity version](previous-releases-downloads.md). Please note that we generally do not provide regular support for major versions that are more than one year old (refer to the [Release Cycle](teamcity-release-cycle.md) page for more details).
  {instance="tc"}


When submitting a ticket, make sure to provide all necessary information by reviewing the [guidelines on reporting issues](reporting-issues.md).
{instance="tcc"}

Support is restricted to TeamCity-specific issues and does not cover issues related to third-party TeamCity plugins, inappropriate environment configuration for a server application, or similar concerns.

## Submitting a Ticket

You can contact us by:

* completing the [online support form](https://teamcity-support.jetbrains.com/hc/en-us/requests/new?ticket_form_id=66621)

* or emailing [teamcity-support@jetbrains.com](mailto:teamcity-support@jetbrains.com)

Both methods will generate a ticket in our ticketing system, Zendesk.

TeamCity Enterprise users can go to **Help | Submit support request** to quickly navigate to the online support form with some of the data prepopulated (such as the server version).

<img src="docs-support-ticket.png" alt="Submit support request button in TeamCity" width="706"/>

If you experience an urgent issue, please indicate this when submitting the support form or in the subject line of your email message, and detail the nature of the urgency.

## Production outage

If you experience a production server outage without an available workaround, you can use the `teamcity-support-urgent` alias at the `jetbrains.com` domain. These tickets are prioritized for review whenever possible. Alternatively, you can mark your ticket as a “Production outage” when completing the online support form.

Customers who have purchased an [optional Business or Enterprise support plan](https://www.jetbrains.com/teamcity/support/#plans) will have appropriate SLAs automatically applied to their production outage tickets.
