# Darwin Information Typing Architecture

## Project and Configuration Settings

<snippet id="open-project-settings">Navigate to <b>Administration | &lt;Your Project&gt;</b> to access project settings.</snippet>

<var name="tab-name" value="tab-name"/>
<snippet id="open-project-settings-tab">Open the <b>%tab-name%</b> settings tab.</snippet>


<snippet id="open-configuration-settings">Navigate to <b>Administration | &lt;Your Configuration&gt;</b> to access build configuration settings.</snippet>


<var name="configuration-tab-name" value="configuration-tab-name"/>
<snippet id="open-configuration-settings-tab">Open the <b>%configuration-tab-name%</b> settings tab.</snippet>



## Project Features

<var name="tab-name" value="tab-name"/><var name="button-name" value="button-name"/>
<snippet id="open-settings-create-new-entity">Open the <b>%tab-name%</b> settings tab and click the <b>%button-name%</b> button.</snippet>


## Connections

<snippet id="create-new-connection">Switch to the <b>Connections</b> tab and click <b>Add Connection</b>. Note that connections can be used only in their parent projects and their subprojects. If you want a connection to be available globally, add it to the <b>Root</b> project.</snippet>

<var name="connection-type" value="connection-type"/>
<snippet id="choose-connection-type">Select "%connection-type%" in the <b>Connection Type</b> drop-down menu.</snippet>

<var name="unique-url-sample" value="unique-url-sample"/>
<snippet id="connections-unique-callback-URL">
Ensure the <b>Enable unique callback URL</b> setting is enabled to generate a unique ID added to your callback URL. This setting bolsters the security of your setup by mitigating the risk of mix-up attacks: attacks utilizing malicious authorization servers that impersonate real auth servers to trick a victim client into leaking an authorization code (token). Using the <code>%unique-url-sample%</code> URL format ensures an attacker cannot hand-craft an address acknowledged by TeamCity.

<note>
<ul>
<li>Whenever you toggle this setting on or off, the callback URL changes. Update OAuth settings on the VCS side accordingly.</li>
<li>IDs are unique for every connection, including copies of existing connections. If you clone a connection with this setting enabled, remember to update your VCS OAuth settings.</li>
</ul>
</note>
</snippet>


<snippet id="test-and-save-connection">Click <b>Test connection</b> to verify TeamCity can access your resources, and save your new connection.</snippet>


## Auth Methods


<snippet id="rat-single"><b>Refreshable access tokens</b> are short-lived tokens acquired by TeamCity from a required VCS provider via existing OAuth connections (as opposed to static PAT tokens issued manually by users on a VCS hosting side). See the following article for more information on generating and using refreshable tokens: <a href="manage-access-tokens.md"></a>.</snippet>


## K8S

<snippet id="kubernetes-settings-api-server-url">Specify the URL of the <a href="https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver">Kubernetes API server</a>.</snippet>

<snippet id="kubernetes-settings-certificate-authority">Enter the content of the <a href="https://kubernetes.io/docs/concepts/cluster-administration/certificates/">CA certificate</a> for your cluster.</snippet>


<snippet id="kubernetes-settings-namespace">Specify a required <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/">Kubernetes namespace</a>. Leave empty to use the <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#viewing-namespaces">default namespace</a>.</snippet>

<snippet id="kubernetes-settings-auth-strategy">

Select the required authentication strategy. Depending on the selected strategy, the set of additional options will vary. Refer to the <a href="https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authentication-strategies">Kubernetes documentation</a> for details on available options.

<note>The <b>Token</b> strategy accepts any token types supported by Kubernetes.</note>

</snippet>