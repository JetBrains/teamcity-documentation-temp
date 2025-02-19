[//]: # (title: CSRF Protection)
[//]: # (auxiliary-id: CSRF Protection)

Сross-Site Request Forgery (CSRF) protection in TeamCity implies a number of requirements on HTTP requests.

Since version 2020.1, TeamCity uses only CSRF tokens as a protection measure. In previous versions of TeamCity, `Origin/Referer` headers were also used.

To obtain a security token, send the `GET https://your-server/authenticationTest.html?csrf` request.   
To pass the token, use the `X-TC-CSRF-Token` HTTP request header or the `tc-csrf-token` HTTP parameter.

## CSRF checks for HTTP request

When considering HTTP request safety from the TeamCity perspective, the following checks are sequentially made:
1. If an HTTP request is a non-modifying one (such as `GET`), it is considered safe.
2. If an HTTP request has a secure CSRF token either in the parameter or in the HTTP header and this token matches the one stored in user session, it is considered safe.

## Implications for non-browser HTTP clients

For non-browser API access, we recommend using [token-based authentication](configuring-your-user-profile.md#Managing+Access+Tokens).

## Implications for CORS clients

To use CORS request, configure the CORS support as described [here](https://www.jetbrains.com/help/teamcity/rest/teamcity-rest-api-documentation.html#CORS-support). This configuration will be enough for `GET` requests.   
If you need to send `POST/PUT/DELETE` requests via CORS, you should obtain a CSRF token using the `authenticationTest.html?csrf` call, and then provide this token with your modifying HTTP requests.

## Troubleshooting
{instance="tc"}

If you face problems regarding CSRF protection in TeamCity (for example, you get the "_Responding with 403 status code due to failed CSRF check_" response from the server), you can try these steps:
* Enforce verification of `Origin/Referer` headers for CORS operations by setting the `teamcity.csrf.paranoid=false` internal property, similarly to how it worked in TeamCity versions prior to 2020.1 (read our [Upgrade Notes](upgrade-notes.md#Limitation+of+CORS+support+for+writing+operations)) for more details).
* Temporary disable CSRF protection at all by setting the `teamcity.csrf.origin.check.enabled=logOnly` internal property.
* Information about failed CSRF attempts is logged into `TeamCity/logs/teamcity-auth.log` files. For more detailed diagnostics of the requests, enable the [debug-auth logging preset](reporting-issues.md#Logging+events).
* If your "unsafe" (`PUT`, `POST`, `DELETE`) requests utilize [access tokens](configuring-your-user-profile.md#Managing+Access+Tokens) to employ the Bearer authentication scheme, consider clearing your session cookies. Doing so allows TeamCity to skip validating CSRF tokens entirely.
* If your environment or scenario mandates that session cookies are present, ensure your requests are not using CSRF tokens that correspond to expired sessions. As a workaround, you may want to reuse session cookies of the first HTTP request for the following ones. To do so in cURL, use the `--cookie-jar` and `--cookie` [command-line options](https://curl.se/docs/http-cookies.html) to save and read cookies to/from a file. In addition, add the `teamcity.tokenAuth.setSessionCookie=true` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) to force TeamCity to create a session (and expect session cookies for the following requests).

In case none of the listed steps help to resolve your problem, please contact our [support](troubleshooting.md) and provide your `teamcity-auth.log` logs with the enabled teamcity-auth [logging preset](reporting-issues.md#Logging+events).

## Troubleshooting
{instance="tcc"}

If you face problems regarding CSRF protection in TeamCity, please contact our [support](troubleshooting.md).
