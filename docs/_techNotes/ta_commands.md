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

# Commands

# Apply Refresh-only

# Import

# Debug

# CLI User Settings

# Manage Providers
