[//]: # (title: HashiCorp Vault Integration)

[HashiCorp Vault](https://www.vaultproject.io) is a secure storage for your tokens, passwords, certificates, and encryption keys. Instead of storing sensitive information inside TeamCity parameters and tokens, you can keep it in Vault and set up TeamCity to securely access this data from Vault engines (KV/KV2, AWS, Google Cloud, and others).

<!--
To set up a TeamCity-Vault integration, download the official [Vault plugin](https://plugins.jetbrains.com/plugin/10011-hashicorp-vault-support) and install it as described in the [](installing-additional-plugins.md) article.
-->

<video src="https://youtu.be/cRNl2dTulTA"
title="HashiCorp Vault plugin for TeamCity"/>


## Common Information

To set up an integration with HashiCorp Vault, you need the following:

* The [HashiCorp Vault connection](configuring-connections.md) in the required project (or &lt;Root project&gt;, if you want any TeamCity project to use this connection).

* The [parameter](configuring-build-parameters.md) that will store a Vault path to a required secret.

* References to this parameter (`%\parameter_name%`) in your build steps (for example, in a [](command-line.md) runner that executes the `terraform apply -var password=%\myVaultParam%` line).

When a build that utilizes this parameter starts, the TeamCity server uses the Vault connection to request a one-time [response wrapping token](https://developer.hashicorp.com/vault/docs/concepts/response-wrapping), which it then passes to a TeamCity agent that runs this build. The build agent uses this token to request Vault secrets and never shares obtained credentials with the TeamCity server. When the build finishes, the agent's token is revoked.

Since all communication with Vault is orchestrated by the TeamCity server, this integration is highly scalable and not affected by the number of build agents accessing Vault secrets. The tokens received by agents from the TeamCity server are not lease tokens (which are solely utilized for testing Vault connections), thus the number of actual build agents does not impact your quota. Consequently, only one Vault license for the TeamCity server is required.

## Set Up a Vault Connection

<img src="dk-vaultConnection.png" width="460" alt="Vault connection settings"/>

1. <include from="common-templates.md" element-id="open-project-settings-tab"><var name="tab-name" value="Connections"/></include>
2. <include from="common-templates.md" element-id="create-new-connection"/>
3. Choose **HashiCorp Vault** as the connection type.
4. Specify the basic connection settings: the connection name, Vault URL and namespace, and an optional ID.

   * [Vault namespaces](https://developer.hashicorp.com/vault/docs/enterprise/namespaces) allow you to create "vaults within a vault" — isolated tenants inside a single Vault Enterprise instance. If you store Vault secrets in this isolated environment, specify its namespace in the **Vault namespace** connection field (for example, `TeamABC/secrets/`). Otherwise, leave this setting empty.
   * **ID** is a custom string that identifies this Vault connection. You can specify this field if you set up multiple Vault connections and specify which connection a specific parameter should use. Otherwise, leave this field blank.
   * **Vault URL** is the address of your Vault instance. Local Vault installations (the default URL is `http://localhost:8200`) are also supported.
   
5. Choose the desired authentication method. TeamCity can authenticate to HCP Vault using a Vault's AppRole, an AWS IAM role, or a directory access protocol (LDAP).

    <tabs>

    <!--

    <tab title="AWS IAM Auth">

    <table><tr><td>
    
    If both your build agents and Vault are hosted on AWS EC2 instances, you can let build agents to automatically communicate with Vault. Agents send signed [GetCallerIdentity](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetCallerIdentity.html) queries to the Vault server, which forwards them to the AWS STS service and authenticates the agent based on the service response. See this Vault documentation article to learn more: [IAM auth method](https://developer.hashicorp.com/vault/docs/auth/aws#iam-auth-method).<br/><br/>
    
    > The AWS IAM auth method requires only build agents and Vault server being hosted on EC2 instances. The TeamCity server can reside in any custom location. However, if the TeamCity server is deployed outside AWS, the connection's **Test connection** button will report issues that agents will not experience during building.
    > 
    {style="note"}

    </td></tr></table>

    </tab>-->
   
   
    
    <tab title="Vault AppRole">

    <table><tr><td>
    
    This authentication method requires that you enable the AppRole auth method in your Vault instance and create an **AppRole**. AppRoles are sets of Vault policies that specify which secrets a user (in this case, TeamCity) can access.<br/><br/>
    
    The Vault AppRole auth method requires you to set up the following settings:<br/><br/>
    
    * **AppRole auth endpoint path** — the path in your Vault instance where the AppRole auth endpoint is mounted. The default path is `approle` (without the preceding backslash character).
          
    * **AppRole Role ID** and **AppRole Secret ID** — these values can be obtained in CLI using the following commands:<br/><br/>
          
        ```
        vault read -field=role_id auth/<AE>/role/<RN>/role-id
        vault write -f -field=secret_id auth/<AE>/role/<RN>/secret-id  
        ```

        <br/>where `<AE>` is the auth endpoint path you specified above (for example, `approle`) and `<RN>` is the name of your role (for example, `my-teamcity-role`).<br/><br/>

    Refer to these Vault documentation articles to learn how to create and set up AppRoles: [Getting Started: Policies](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-policies), [AppRole auth method](https://developer.hashicorp.com/vault/docs/auth/approle).

    </td></tr></table>

    </tab>
    
    <tab title="LDAP Auth">

   <table><tr><td>

    This method can be used for Vault instances with enabled [LDAP auth method](https://developer.hashicorp.com/vault/docs/auth/ldap). This authentication option requires username and password. The **Path** setting stores the path to the LDAP authentication endpoint (for example, `ldap` for the default `auth/ldap/users/userA` and `auth/ldap/groups/groupB` endpoints).<br/><br/>

    Refer to this article for more information about setting up this authentication method in Vault: [LDAP auth method](https://developer.hashicorp.com/vault/docs/auth/ldap). 
    
    </td></tr></table>

    </tab>
   
    </tabs>

<!--5. Tick **Fail in case of error** if you want builds to fail with the "Error while fetching data from HashiCorp Vault" message if the agent cannot obtain Vault secrets and write them to [TeamCity parameters](#Create+and+Set+Up+a+Parameter). Note that builds will fail even if these parameters are never used. If this setting is disabled, builds will continue running with empty strings as secret parameter values.-->

6. <include from="common-templates.md" element-id="test-and-save-connection"/>



**Kotlin DSL:**

```Kotlin
project {

    features {
        hashiCorpVaultConnection {
            id = "PROJECT_EXT_34"
            name = "HashiCorp Vault"
            url = "http://127.0.0.1:8200/"
            vaultNamespace = "enterprise/vault/namespace"
            authMethod = appRole {
                roleId = "..."
                secretId = "..."
            }
        }
    }
}
```



## Create and Set Up a Parameter

To start using a secret value from HCP Vault in your builds, create a [parameter](configuring-build-parameters.md) that serves two purposes:

* Stores a path to the Vault secret whose value needs to be obtained
* Stores the secret value after Vault provided it

To create such a parameter, do the following:

1. Go to **Administration | &lt;Project or Build Configuration&gt; | Parameters** and click **Add new parameter**.

   > You can set up parameters only for projects with a valid [Vault connection](#Set+Up+a+Vault+Connection) (or inherit it from a parent project).
   >
   {style="note"}

2. Choose the **Environment variable** type and enter the parameter name. For example, `env.AWS_ACCESS_KEY_ID`.

3. Click **Edit...** next to the "Spec" label and set up the following values:

   * **Type**: Remote
   * **Remote Connection Type**: HashiCorp Vault Parameter
   * **Parameter Namespace**: choose existing [Vault connection](#Set+Up+a+Vault+Connection) you wish to use to retrieve this parameter's value. This drop-down menu shows **Display names** for all connections that have IDs.

       <img src="dk-hcv-id-in-connection.png" width="560" alt="Connection ID"/>
   
      <img src="dk-hcv-id-in-param.png" width="560" alt="Parameter ID"/>

   * **Vault Query**: the path to the secret in the `path!/key` format. For example, the following string points the parameter to the "access_key" key of the "awscreds" secret stored in the KV2 engine: `secret/data/awscreds!/access_key`.

4. Click **Save** to close the dialog.

**Kotlin DSL:**

```Kotlin
project {
    params {
        hashiCorpVaultParameter {
            name = "env.AWS_ACCESS_KEY_ID"
            query = "secret/data/awscreds!/access_key"
            vaultId = "DataLore"
        }
    }
}
```

### Update Legacy Parameters

If you already used the TeamCity Vault plugin before the 2023.11 version, you might have legacy parameters that store paths to Vault secrets directly in their parameter values (in the `%vault:PATH!/KEY%` format).
{ignore-vars="true"}

Compared to these legacy parameters, new "remote parameters" showcase the following advantages:

* Vault paths are stored in the **Vault Query** field, which leaves the **Value** field free for the default/initial parameter value.
* Secret paths (queries) use a more straightforward format without the `vault:` prefix.


In addition, Vault engines that issue dynamic secrets (for example, the AWS engine) generate new credentials each time a client (TeamCity) sends a new request. For that reason, you should avoid mixing legacy and remote parameters that access the same dynamic secrets engine within the same build.

For example, if you have a remote parameter that obtains the Access Key ID from the Vault AWS engine, and a legacy parameter that retrieves the corresponding Secret Access Key, TeamCity will send separate requests for each parameter. The engine will issue two sets of values, so your ID/Secret pair will mismatch.

We recommend that you update your existing legacy parameters to their remote counterparts.

#### Kotlin DSL

You can update legacy Vault parameters to a newer type in Kotlin DSL. The legacy Vault parameters are declared like the following:

```Kotlin
project {
    params {
        param("env.AWS_ACCESS_KEY_ID", "%\vault:secret/data/awscreds!/access_key%")
    }
}
```

To update these parameters, replace them with the following blocks:

```Kotlin
project {
    params {
        hashiCorpVaultParameter {
            name = "env.AWS_ACCESS_KEY_ID"
            query = "secret/data/awscreds!/access_key"
        }
    }
}
```

Note that, unlike the original parameter value, the `query` field of a new parameter has neither the `vault:` prefix nor the percent characters.

If your legacy parameter had the `%\vault:foobar:/path!/key%` format, the "foobar" part identifies which Vault connection this parameter should use to retrieve the secret. Move this value to the `vaultId` field of a remote parameter:

```Kotlin
project {
   params {
      hashiCorpVaultParameter {
         name = "parameter name"
         query = "path!/key"
         vaultId = "foobar"
      }
   }
}
```
