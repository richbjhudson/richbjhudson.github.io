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

# Apply Refresh-only

# Import

# Debug

# CLI User Settings

# Manage Providers
