# Overlay: NoOps Accelerator - Azure Data Bricks Workspace

## Overview

This overlay module deploys a premium Azure Data Bricks Workspace suitable for hosting docker containers. The registry will be deployed to the Hub/Spoke shared services resource group using default naming unless alternative values are provided at run time.

Read on to understand what this overlay does, and when you're ready, collect all of the pre-requisites, then deploy the overlay

## Deploy Azure Data Bricks Workspace

The docs on Azure Data Bricks Workspace: <https://docs.microsoft.com/en-us/azure/container-registry/>. By default, this overlay will deploy resources into standard default hub/spoke subscriptions and resource groups.  

The subscription and resource group can be changed by providing the resource group name (Param: parTargetSubscriptionId/parTargetResourceGroup) and ensuring that the Azure context is set the proper subscription.  

## Pre-requisites

* A hub/spoke LZ deployment (a deployment of [deploy.bicep](../../../../bicep/platforms/lz-platform-scca-hub-3spoke/deploy.bicep))
* Decide if the optional parameters is appropriate for your deployment. If it needs to change, override one of the optional parameters.

## Parameters

See below for information on how to use the appropriate deployment parameters for use with this overlay:

Required Parameters | Type | Allowed Values | Description
| :-- | :-- | :-- | :-- |
parRequired | object | {object} | Required values used with all resources.
parTags | object | {object} | Required tags values used with all resources.
parLocation | string | `[deployment().location]` | The region to deploy resources into. It defaults to the deployment location.
parDataBricksWorkspace | object | {object} | Defines the Data Bricks Workspace.
parTargetSubscriptionId | string | `xxxxxx-xxxx-xxxx-xxxxx-xxxxxx` | The target subscription ID for the target Network and resources. It defaults to the deployment subscription.
parTargetResourceGroup | string | `anoa-eastus-platforms-hub-rg` | The name of the resource group in which the Data Bricks Workspace will be deployed. If unchanged or not specified, the NoOps Accelerator shared services resource group is used.
parTargetVNetName | string | `anoa-eastus-platforms-hub-vnet` | The name of the VNet in which the aks will be deployed. If unchanged or not specified, the NoOps Accelerator shared services resource group is used.
parTargetSubnetName | string | `anoa-eastus-platforms-hub-snet` | The name of the Subnet in which the aks will be deployed. If unchanged or not specified, the NoOps Accelerator shared services resource group is used.

Optional Parameters | Description
------------------- | -----------
None

## Deploy the Overlay

Connect to the appropriate Azure Environment and set appropriate context, see getting started with Azure PowerShell or Azure CLI for help if needed. The commands below assume you are deploying in Azure Commercial and show the entire process from deploying Platform Hub/Spoke Design and then adding an Azure Data Bricks Workspace post-deployment.

> NOTE: Since you can deploy this overlay post-deployment, you can also build this overlay within other deployment models such as Platforms & Workloads.

Once you have the hub/spoke output values, you can pass those in as parameters to this deployment.

For example, deploying using the `az deployment sub create` command in the Azure CLI:

### Azure CLI

```bash
# For Azure Commerical regions
az login
cd src/bicep
cd platforms/lz-platform-scca-hub-3spoke
az deployment sub create \ 
--name contoso \
--subscription xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx \
--template-file platforms/lz-platform-scca-hub-3spoke/deploy.bicep \
--location eastus \
--parameters @platforms/lz-platform-scca-hub-3spoke/parameters/deploy.parameters.json
cd overlays
cd containerRegistry
az deployment sub create \
   --name deploy-AppServicePlan
   --template-file overlays/containerRegistry/deploy.bicep \
   --parameters @overlays/containerRegistry/deploy.parameters.json \
   --subscription xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx \
   --location 'eastus'
```

OR

```bash
# For Azure Government regions
az deployment group create \
  --template-file overlays/containerRegistry/deploy.bicep \
  --parameters @overlays/containerRegistry/deploy.parameters.json \
  --subscription xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx \
  --resource-group anoa-usgovvirginia-platforms-hub-rg \
  --location 'usgovvirginia'
```

### PowerShell

```powershell
# For Azure Commerical regions
New-AzGroupDeployment `
  -ManagementGroupId xxxxxxx-xxxx-xxxxxx-xxxxx-xxxx
  -TemplateFile overlays/containerRegistry/deploy.bicepp `
  -TemplateParameterFile overlays/containerRegistry/deploy.parameters.json `
  -Subscription xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx `
  -ResourceGroup anoa-eastus-platforms-hub-rg `
  -Location 'eastus'
```

OR

```powershell
# For Azure Government regions
New-AzGroupDeployment `
  -TemplateFile overlays/containerRegistry/deploy.bicepp `
  -TemplateParameterFile overlays/containerRegistry/deploy.parameters.json `
  -Subscription xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx `
  -Location  'usgovvirginia'
```

## Extending the Overlay

By default, this overlay has the minium parmeters needed to deploy the service. If you like to add addtional parmeters to the service, please refer to the module description located in AzResources here: [Container Registries `[Microsoft.DataBricksWorkspace/registries]`](../../../azresources/Modules/Microsoft.DataBricksWorkspace/registries/readmd.md)

## Air-Gapped Clouds

For air-gapped clouds it may be convenient to transfer and deploy the compiled ARM template instead of the Bicep template if the Bicep CLI tools are not available or if it is desirable to transfer only one file into the air gap.

## Validate the deployment

Use the Azure portal, Azure CLI, or Azure PowerShell to list the deployed resources in the resource group.

Configure the default group using:

```bash
az configure --defaults group=anoa-eastus-databricks-rg.
```

```bash
az resource list --location eastus --subscription xxxxxx-xxxx-xxxx-xxxx-xxxxxxxx --resource-groupanoa-eastus-databricks-rg
```

OR

```powershell
Get-AzResource -ResourceGroupName anoa-eastus-databricks-rg
```

## Cleanup

The Bicep/ARM deployment of NoOps Accelerator - Azure Data Bricks Workspace deployment can be deleted with these steps:

### Delete Resource Groups

```bash
az group delete --name anoa-eastus-databricks-rg
```

OR

```powershell
Remove-AzResourceGroup -Name anoa-eastus-databricks-rg
```

### Delete Deployments

```bash
az deployment delete --name deploy-databricks
```

OR

```powershell
Remove-AzSubscriptionDeployment -Name deploy-databricks
```

## Example Output in Azure

![Example Deployment Output](media/databricksExampleDeploymentOutput.png "Example Deployment Output in Azure commerical regions")

### References

* [What is Azure Databricks?](https://learn.microsoft.com/en-us/azure/databricks/scenarios/what-is-azure-databricks)
* [What is Databricks Data Science & Engineering?](https://learn.microsoft.com/en-us/azure/databricks/scenarios/what-is-azure-databricks-ws)
