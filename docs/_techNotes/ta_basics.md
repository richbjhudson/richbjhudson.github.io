---
layout: techNote
category: Terraform
title: Basics
---
# Workflow

- `terraform init` - initialises the working directory by downloading provider plugins and source modules.
- `terraform validate` - checks configuration file syntax.
- `terraform fmt` -- formats configuration files.
- `terraform plan` - creates an execution plan that compares the desired state to the known state.
- `terraform apply` - core command used to create resources according to plan.
- `terraform destroy` - removes resources according to known state.

# Blocks

## 3 Fundamental Block Types:
- **Terraform** includes:
    -  terraform cli version, terraform provider and version, and also the backend block that holds the terraform state in a remote location.
    - Only constant values are allowed in this block no variable references.
- **Provider** includes provider configuration in root module.
- **Resource** is used to create infrastructure objects including:
    - resource syntax and behaviour.
    - provisioners - post creation actions.

## Additional Block Types:
- Variable blocks including input, output and local.
- Calling/ referencing such as data sources and module.

# Providers

Provider plugins are downloaded by terraform cli and used to interact with Cloud APIs e.g. the azure provider communicates with Microsoft Azure API. 
Have their own release cycles and version numbers.

## Required Provider Block

This is contained within the **terraform** block and includes the source and version as shown below:

```
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = ">= 2.0"
    }    
  }
}
```
## Provider block

This block allows you to configure a provider such as:
- Add features e.g. set service principal to use a key vault.
- Set default behaviour of when you delete a VM so that you do not delete the OS disk.

```
provider "azurerm" {
  features {}
}
```
*Note: Local names should match between the **required_providers** and **provider block**. It is recommended to use vendor preferred name but you could use anything in theory.*

## Dependency Lock File

It prevents you from updating a provider plugin version.

Registry.terraform.io
Provider badges 
	• if official - owned and maintained by hashicorp
	• Verified - owned by vendor
	• Community
Terraform init upgrade -- this will upgrade the lock file if you decide to change the allowed provider versions

You may wish to create multiple provider feature block so that you have different defaults per azure region e.g. do not delete os disk with vm for a region.
You can call a provider alias and thus feature settings in a resource block.


# Resource Block Syntax

# State

# Meta-arguments

# Provisioners
