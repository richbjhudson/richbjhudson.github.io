---
layout: techNote
category: Terraform
title: Commands
---
# Remote State Storage & Locking

- A `backend` block is within a `terraform` block usually in a `version.tf` file.
- Backend blocks are responsible for storing state and providing an API for state locking.
- Azure Storage Accounts support both state storage and locking.  
    - A local state file needs to be accessible to all members of a Team and stored in a shared location.
    - If two members of a Team run terraform at the same time it may create a race condition as multiple updates to state file may lead to conflicts, data loss or corruption.
    - State locking is enabled when using an Azure storage account - supported on all operations that may write state.
- When you create an Azure storage account - enable versioning for blobs.
- When you run `terraform plan` the blob container goes into a lease state of leased, once activity is finished it goes into an available state.
- Here is an example of how to configure a `backend` block within a `terraform` block to use an Azure storage account:

```

  backend "azurerm" {
    resource_group_name   = "terraform-storage-rg"
    storage_account_name  = "terraformstate201"
    container_name        = "tfstatefiles"
    key                   = "terraform.tfstate"
  } 
```

# Remote State Data Source

# Commands

# Apply Refresh-only

# Import

# Debug

# CLI User Settings

# Manage Providers
