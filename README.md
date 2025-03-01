# k8s-external-secret-demo
This is a demo of using the external secret operator for secret management with azure key vault.

## Key vault
Create a key vault in azure and disable rbac authentication and instead use access policies for authentication.

## Authentication

For kubernetes to fetch the secrets from Azure key vault, it needs to authenticate with azure. We can do it in many ways but the best way in my opinion is to use a service principle(app registration). So create an app registraion in azure and then for the app registration create a client secret and note down the secret. After the creation of the service principle and the client secret, create an access policy in key vault with Get and List permissions. Read more about authentication in <a href=https://external-secrets.io/main/provider/azure-key-vault/>the documentation</a>.

Now onto kubernetes, we need to create a secret with the necessary credentials so it can authenticate with Azure. Execute the following command ```kubectl create secret generic <secret_obj_name> --from-literal=ClientID=<sp-client-id> --from-literal=ClientSecret=<sp-secret>```. Now the cluster will be able to authenticate with the provider.

## Secret store
Since we are using service princple for authentication the secret-store object needs to access the keys in the secret we created previously. 
```
provider:
    azurekv:
      tenantId: <tenant_id> 
      vaultUrl: <key-vault_url>
      authSecretRef:
        clientId:
          name: <secret_obj_name>
          key: ClientID
        clientSecret:
          name: <secret_obj_name>
          key: ClientSecret
```
This is how we create a connection with the azure vault using secret-store object in k8s.

## External Secret

External secret object uses the secret store created to authenticate with Azure and then it can be used to fetch secrets from the vault to create a new k8s secret object with the secrets from the vault. Refer to the external-secret.yml file to know how or read the <a href=https://external-secrets.io/main/guides/introduction/>docs</a>. One great feature of this object is that it refreshes by fetching the value from the vault which means we dont have to do anything in the cluster for secret rotation rather all we need to do is manage it where ever the secret is stored i.e in my case Azure key vault.

