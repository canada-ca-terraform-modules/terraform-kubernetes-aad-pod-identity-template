# Terraform Kubernetes AAD Pod Identity Templates

## Introduction

This module allows for the deployment and management of AzureIdentity and AzureIdentityBinding objects into a Kubenertes cluster which has Active Directory Pod Identity installed.

## Requirements

This module requires:

* AAD Pod Identity >= 1.6

## Security Controls

The following security controls can be met through configuration of this template:

* TBD

## Dependencies

* None

## Optional (depending on options configured):

* None

## Usage
Add the following code block to the desired Terraform namespace definition:
```terraform
module "aad_identity_test" {
  source = "git::https://github.com/StatCan/terraform-kubernetes-aad-pod-identity-template.git?ref=v3.0.0"

  type = 0
  identity_name = "test"
  namespace     = "default"
  resource_id   = "/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<named_identity>"
  client_id     = "<client_id>"
}
```
The **Client ID** and **Resource ID** are to be provided by the client for their managed identities.

## Variables Values

| Name          | Type   | Required             | Value                                                                                                                                                                |
| ------------- | ------ | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| client_id     | string | yes                  | The client id to be used for the Managed Identity                                                                                                                    |
| identity_name | string | yes                  | The name of the identity in the Cluster.                                                                                                                             |
| namespace     | string | yes                  | The namespace Helm will install the chart under                                                                                                                      |
| resource_id   | string | For type:0           | The resource id to be used for the Managed Identity                                                                                                                  |
| secret_name   | string | For type:1 or type:2 | The name of the secret from which to retrieve the certificate or client_secret.                                                                                      |
| tenant_id     | string | For type:1 or type:2 | The ID of the Azure Tenant where the Service Principal is located.                                                                                                   |
| type          | int    | yes                  | The type of identity to use. Set type: 0 for user-assigned MSI, type: 1 for Service Principal with client secret, or type: 2 for Service Principal with certificate. |

## Migrating to v3.0.0

To migrate to the v3.0.0 module without destroying and recreating the resources, run the following command with the required substitutions:

```bash
terraform state rm $moduleLabel.null_resource.vault_aad_pod_identity

# for var.type == 0
terraform import $moduleLabel.kubernetes_manifest.azureidentity_user_assigned[0] "apiVersion=aadpodidentity.k8s.io/v1,kind=AzureIdentity,namespace=$namespace,name=$identity_name"

# for var.type > 0
terraform import $moduleLabel.kubernetes_manifest.azureidentity_service_principal[0] "apiVersion=aadpodidentity.k8s.io/v1,kind=AzureIdentity,namespace=$namespace,name=$identity_name"

terraform import $moduleLabel.kubernetes_manifest.azureidentitybinding "apiVersion=aadpodidentity.k8s.io/v1,kind=AzureIdentityBinding,namespace=$namespace,name=$identity_name"
```

## History

| Date     | Release | Change                                          |
| -------- | ------- | ----------------------------------------------- |
| 20201022 | v1.0.0  | Initial release.                                |
| 20201022 | v1.1.0  | Add outputs for namespace and identity_name.    |
| 20210824 | v2.0.0  | Update module to align with Terraform v0.13     |
| 20220427 | v3.0.0  | Migrate to use *kubernetes_manifest* resources. |
