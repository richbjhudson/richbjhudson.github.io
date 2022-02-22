---
layout: techNote
category: Terraform
title: Commands
---
# Remote State Storage & Locking

- A `backend` block is within a `terraform` block usually in a `version.tf` file.
- Backend blocks are responsible for storing state and providing an API for state locking.
- Azure Storage Accounts support both state storage and locking (supported on all operations that may write state).  
    - A local state file needs to be accessible to all members of a Team and stored in a shared location.
    - If two members of a Team run terraform at the same time it may create a race condition. Multiple updates to a state file may lead to conflicts, data loss or corruption.
- When you create an Azure storage account - enable versioning for blobs. This will allow you to rollback to a previous version of the state file.
- When you run `terraform plan` the blob container goes into a lease state of **leased**, once activity is finished it goes into an **available** state.
- Here is an example of how to configure a `backend` block within a `terraform` block to use an Azure storage account:

```

  backend "azurerm" {
    resource_group_name   = "rg1"
    storage_account_name  = "sta1"
    container_name        = "tfstatefiles"
    key                   = "terraform.tfstate"
  } 
```

# Remote State Data Source

- `terraform_remote_state` data source retrieves the root module output values from some other terraform configuration.
- Example Scenario:
    - Project-1/terraform.tfstate - VNET + RG
    - Project-2/terraform.tfstate - to build a VM, it needs details regarding VNET + RG
- Within dependant Project-1 you must define outputs that may be used by Project-2 e.g. VNET + RG.
- Then within Project-2 you access the remote state of Project-1 as a data source to use its outputs.
- Here is an example of the `terraform_remote_state` block configuration used by Project-2 to access the output values defined in Project-1.

```
data "terraform_remote_state" "Project-1" {
  backend = "azurerm"
  config = {
    resource_group_name   = "rg1"
    storage_account_name  = "sta1"
    container_name        = "tfstatefiles"
    key                   = "terraform.tfstate"
  }
}
```
- At this point you may reference the outputs defined in Project-1 from a Project-2 `azurerm_linux_virtual_machine` resource block as follows:

```
resource_group_name = data.terraform_remote_state.Project-1.outputs.resource_group_name
```

# State Commands

- `terraform show` by default outputs the local terraform.tfstate file contents.
- `terraform plan -out="v1plan.out"` - write a terraform plan out to a file.
    - `terraform show "v1plan.out"` - view contents of terraform plan output file.
    - `terraform show -json "v1plan.out"` - as above but in json format.
- `terraform apply "v1plan.out"` - this applies the terraform plan without re-running a plan.

- `terraform state list` - list resources in a state file. *Note: data sources appear as resources.*
- `terraform state show data.azurerm_subscription.current` - shows attributes of a single resource in the state file.

- `terraform state mv` this allows you to change local resource names, for eaxmple:
```
terraform state list
terraform state mv -dry-run azurerm_virtual_network.myvnet azurerm_virtual_network.myvnet-new
terraform state mv azurerm_virtual_network.myvnet azurerm_virtual_network.myvnet-new
terraform state list
```
- If configuration has different local names then it will see that it needs to create a new resource and destroy the incorrectly named local named resource. They need to match.

- `terraform apply -refresh-only` - refreshes the state file to match the terraform configuration.

- `terraform state rm` - remove resources from the terraform state e.g. `terraform state rm azurerm_virtual_network.myvnet-new`.
    - If still present in configuration , it will recreate.
    - If you do not want a resource to be managed by terraform you need to remove it from both the configuration and state file.

- `terraform state replace-provider` - you may download a copy of a provider plugin and store it in an internal repository and access it from an internal source.

- `terraform state pull` - download and output state file to cli. You could copy and paste the contents into a file to make a local terraform.tfstate.
- `terraform state push terraform.tfstate` - used to migrate local state to remote state.

- `terraform force-unlock LOCK_ID` - only apply's to AWS Dynamo DB.

- `terraform taint` - force the re-creation of resources. It marks the resources as tainted so that they are destroyed and then created.
    - Real world example would be where the cloud-init file has been changed, so you need to recreate the VM to re-apply the bootstrap file.
    - Example: `terraform taint azurerm_virtual_network.myvnet-new` would result in the recreation of the VNET on the next `terraform apply`.
- `terraform untaint` - remove tain mark on resource.

- `terraform plan/appy -target` - for exceptional circumstances, recover from mistakes or work around limitations.

# Apply Refresh-only

# Import

# Debug

# CLI User Settings

# Manage Providers
