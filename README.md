# Azure Data Factory

[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/claranet/aci/azurerm/)

This module creates an Azure Data Factory with diagnostic settings enabled.

<!-- BEGIN_TF_DOCS -->
## Global versioning rule for Claranet Azure modules

| Module version | Terraform version | AzureRM version |
| -------------- | ----------------- | --------------- |
| >= 7.x.x       | 1.3.x             | >= 3.0          |
| >= 6.x.x       | 1.x               | >= 3.0          |
| >= 5.x.x       | 0.15.x            | >= 2.0          |
| >= 4.x.x       | 0.13.x / 0.14.x   | >= 2.0          |
| >= 3.x.x       | 0.12.x            | >= 2.0          |
| >= 2.x.x       | 0.12.x            | < 2.0           |
| <  2.x.x       | 0.11.x            | < 2.0           |

## Contributing

If you want to contribute to this repository, feel free to use our [pre-commit](https://pre-commit.com/) git hook configuration
which will help you automatically update and format some files for you by enforcing our Terraform code module best-practices.

More details are available in the [CONTRIBUTING.md](./CONTRIBUTING.md#pull-request-process) file.

## Usage

This module is optimized to work with the [Claranet terraform-wrapper](https://github.com/claranet/terraform-wrapper) tool
which set some terraform variables in the environment needed by this module.
More details about variables set by the `terraform-wrapper` available in the [documentation](https://github.com/claranet/terraform-wrapper#environment).

```hcl
module "azure_region" {
  source  = "claranet/regions/azurerm"
  version = "x.x.x"

  azure_region = var.azure_region
}

module "rg" {
  source  = "claranet/rg/azurerm"
  version = "x.x.x"

  location    = module.azure_region.location
  client_name = var.client_name
  environment = var.environment
  stack       = var.stack
}

module "logs" {
  source  = "claranet/run-common/azurerm//modules/logs"
  version = "x.x.x"

  client_name         = var.client_name
  environment         = var.environment
  stack               = var.stack
  location            = module.azure_region.location
  location_short      = module.azure_region.location_short
  resource_group_name = module.rg.resource_group_name
}

module "data_factory" {
  source  = "claranet/data-factory/azurerm"
  version = "x.x.x"

  client_name    = var.client_name
  environment    = var.environment
  location       = module.azure_region.location
  location_short = module.azure_region.location_short
  stack          = var.stack

  resource_group_name = module.rg.resource_group_name

  integration_runtime_type = "SelfHosted"

  logs_destinations_ids = [module.logs.log_analytics_workspace_id]
  logs_retention_days   = 90
}
```

## Providers

| Name | Version |
|------|---------|
| azurecaf | >= 1.2.22 |
| azurerm | ~> 3.22 |

## Modules

| Name | Source | Version |
|------|--------|---------|
| diagnostics | claranet/diagnostic-settings/azurerm | 6.2.0 |

## Resources

| Name | Type |
|------|------|
| [azurecaf_name.data_factory_name](https://registry.terraform.io/providers/aztfmod/azurecaf/latest/docs/resources/name) | resource |
| [azurerm_data_factory.main_data_factory](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory) | resource |
| [azurerm_data_factory_integration_runtime_azure.integration_runtime](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory_integration_runtime_azure) | resource |
| [azurerm_data_factory_integration_runtime_azure_ssis.integration_runtime](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory_integration_runtime_azure_ssis) | resource |
| [azurerm_data_factory_integration_runtime_self_hosted.integration_runtime](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory_integration_runtime_self_hosted) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| azure\_devops\_configuration | Azure DevOps configuration for data factory. See documentation at https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory#vsts_configuration | `map(string)` | `null` | no |
| client\_name | Client name/account used in naming | `string` | n/a | yes |
| custom\_data\_factory\_name | Custom name of the Data Factory, generated if not set. | `string` | `null` | no |
| custom\_diagnostic\_settings\_name | Custom name of the diagnostics settings, name will be 'default' if not set. | `string` | `"default"` | no |
| default\_tags\_enabled | Option to enable or disable default tags | `bool` | `true` | no |
| environment | Project environment | `string` | n/a | yes |
| extra\_tags | Extra tags to add | `map(string)` | `{}` | no |
| github\_configuration | Github configuration for data factory. See documentation at https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory#github_configuration | `map(string)` | `null` | no |
| global\_parameters | Global parameters for data factory. See documentation at https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_factory#global_parameter | `list(map(string))` | `[]` | no |
| integration\_runtime\_configuration | Parameters used to configure `AzureSSIS` integration runtime:<br>    `node_size` (optional, defaults to `Standard_D2_v3`)<br>    `number_of_nodes` (optional, defaults to `1`)<br>    `max_parallel_executions_per_nodes` (optional, defaults to `1`)<br>    `edition` (optional, defaults to `Standard`)<br>    `license_type` (optional, defaults to `LicenseIncluded`)<br>  Parameters used to configure `Azure` integration runtime<br>    `cleanup_enabled` (optional, defaults to `true`)<br>    `compute_type` (optional, defaults to `General`)<br>    `core_count` (optional, defaults to `8`)<br>    `time_to_live_min` (optional, defaults to `0`)<br>    `virtual_network_enabled` (optional, defaults to `false`) | `map(any)` | `{}` | no |
| integration\_runtime\_custom\_name | Name of the integration\_runtime resource | `string` | `null` | no |
| integration\_runtime\_description | Integration runtime description | `string` | `null` | no |
| integration\_runtime\_type | Specifies the integration runtime type. Possible values are `Azure`, `AzureSSIS` and `SelfHosted` | `string` | `null` | no |
| location | Azure region to use | `string` | n/a | yes |
| location\_short | Short string for Azure location | `string` | n/a | yes |
| logs\_categories | Log categories to send to destinations. | `list(string)` | `null` | no |
| logs\_destinations\_ids | List of destination resources IDs for logs diagnostic destination.<br>Can be `Storage Account`, `Log Analytics Workspace` and `Event Hub`. No more than one of each can be set.<br>If you want to specify an Azure EventHub to send logs and metrics to, you need to provide a formated string with both the EventHub Namespace authorization send ID and the EventHub name (name of the queue to use in the Namespace) separated by the `|` character. | `list(string)` | n/a | yes |
| logs\_metrics\_categories | Metrics categories to send to destinations. | `list(string)` | `null` | no |
| logs\_retention\_days | Number of days to keep logs on storage account. | `number` | `30` | no |
| managed\_virtual\_network\_enabled | True to enable managed virtual network | `bool` | `true` | no |
| name\_prefix | Optional prefix for the generated name | `string` | `""` | no |
| name\_suffix | Optional suffix for the generated name | `string` | `""` | no |
| public\_network\_enabled | True to make data factory visible to the public network | `bool` | `false` | no |
| resource\_group\_name | Name of the resource group | `string` | n/a | yes |
| stack | Project stack name | `string` | n/a | yes |
| use\_caf\_naming | Use the Azure CAF naming provider to generate default resource name. `custom_name` override this if set. Legacy default name is used if this is set to `false`. | `bool` | `true` | no |

## Outputs

| Name | Description |
|------|-------------|
| data\_factory\_id | Data factory id |
| data\_factory\_integration\_runtime\_id | Data factory integration runtime id |
| data\_factory\_integration\_runtime\_type | Data factory integration runtime type |
| data\_factory\_managed\_identity | Type of managed identity |
| data\_factory\_name | Data factory name |
| data\_factory\_self\_hosted\_integration\_runtime\_primary\_authorization\_key | The self hosted integration runtime primary authentication key |
| data\_factory\_self\_hosted\_integration\_runtime\_secondary\_authorization\_key | The self hosted integration runtime secondary authentication key |
<!-- END_TF_DOCS -->

## Related documentation

Microsoft Azure documentation: [docs.microsoft.com/en-us/azure/data-factory](https://docs.microsoft.com/en-us/azure/data-factory/)
