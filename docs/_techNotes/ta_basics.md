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
They have their own release cycles and version numbers.

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
The source is prefixed with *registry.terraform.io* by default.

## Provider block

This block allows you to configure a provider such as:
- Add features e.g. set service principal to use a key vault.
- Set default behaviour of when you delete a VM so that you do not delete the OS disk.
- You may wish to create multiple provider feature blocks so that you have different defaults per azure region e.g. do not delete OS disk with VMs for a region.

*Note: You can call a provider alias and thus feature settings in a resource block.*

```
provider "azurerm" {
  features {}
}
```
*Note: Local names should match between the **required_providers** and **provider** block. It is recommended to use the vendor preferred name but you could use anything in theory.*

## Dependency Lock File

It prevents you from updating a provider plugin version.
It is best practice to check in the lock file into your code repository.

`terraform init upgrade` - this will upgrade the lock file if you decide to change the allowed provider versions.

*Note: It only includes provider version tracking **not** Terraform cli nor modules.*

## Provider Badges

When you navigate to the [terraform registry list of providers](https://registry.terraform.io/browse/providers) you will notice they have 1 of 3 badges:
- Official - owned and maintained by hashicorp.
- Verified - owned by vendor.
- Community

# Resource Block Syntax

It contains:
- Resource type based on infrastructure object it can create.
- Arguments - input details to create resource.
- Resource local name - reference within existing module so you can call values elsewhere in your code.
- Meta-arguments - they allow you to change the behaviour of arguments e.g. count.

```
resource "azurerm_resource_group" "resource_group" {
  location = "uksouth"
  name = "rg-01"  
}
```

## Internal Block

A block within a resource block e.g. `Ip_configuration {}` in `azurerm_network_interface` resource.

*Note: Whereas, tag **=** {} is an argument using a map value.*

## Resource Behaviour

- Create
- Destroy - when the resource exists in state but not in config.
- Update in place - where arguments for a resource have changed.
- Destroy and recreate - when an argument has changed but can not perform an in place update e.g. location changed. 

*Note: It first destroys and then recreates. You can change this behaviour using meta-arguments.*

# State

- Once `terraform apply` is executed a state file is created - **terraform.tfstate**.
  - Desired state is contained within local .tf files.
  - Current state is the real resources in the cloud that are recorded in the state file.
- `terraform destroy` will remove configuration from the Cloud environment that is contained within the state file.
- The state file is stored locally by default.
- The state file holds all of the details about a resource not just what was set using arguments.

*Note: It is not recommended to manually edit the state file nor store it locally.*

# Meta-arguments

Changes behaviour of resource blocks:
- depends_on - handle dependencies terraform cannot infer.
- count - create mutliple instance of a resource based on value.
- for_each - create multiple instances of a resource according to map or strings.
- provider - select non default provider config.
- lifecycle - resource lifecycle management e.g. in a destroy and recreate scenario, you could 1st create and then destroy a resource.
- *Provisioners & Connections (not meta-argument)* - extra actions after resource creation e.g. install app on VM.

## depends_on

- This is not required if you create a resource that references another resource's data as an implicit dependancy is created.
- It can be used in resource/ module blocks - it contains a list of resources or child modules.
- It should only ever be used as a last resort.
- You list which resources should be created before it actions the resource block. As such it is placed  at the top of the given resource block:

```
depends_on = [
    azurerm_virtual_network.vnet1,
    azurerm_subnet.subnet1
  ]
```

## Count

- The number of instances that will be created within a resource block.
- It can be used in resource/ module blocks.
- You cannot use count and for_each in the same resource block.
-  `${Count.index}` -- can be used to set a value based on the instance number:

```
resource "azurerm_public_ip" "pip" {
  count = 2
  name                = "pip-${count.index}"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  allocation_method   = "Static" 
}
```

- You can reference an instance of the resource by using `azurerm_public_ip.pip[0]`.

## for_each

- for_each may be used with either a map of values or a set of strings.
  - A set of strings is just a list of strings - each.key and each.value return the same string.
  - Map is a key value pair, for example:

``
For_each = {
Dev = "myapp1"
}

Each.key = dev

Each.value = myapp1
``
*Note: You cannot use both count and for_each in the same resource block/ module.*

- You can reference a resource that is already configured to use a for_each and use these resource instances within an additional resource block referecning the instance using what is termed **for_each chaining**:

```
 for_each = azurerm_network_interface.nic
 ```

## lifecycle

- It features as a nested block within a resource block.
- There are 3 types of lifecyle blocks:  
  - Create_before_destroy - create new resource before destroying existing resource

```
lifecycle {
    create_before_destroy = true
  }
```

  - Prevent_destroy - database/ disk that you do not want to be removed.

```
lifecycle {
    prevent_destroy = true
  }
```

  - Ignore_changes - ignore change made within Azure portal e.g. tags.

```
lifecycle {
    ignore_changes = [ tags, ]
  }
```

# Functions
File function can be used to obtain ssh public key
Custom_data - use filebase64 function to reference cloud init file that can configure a VM

Splat expression
Terraform console - interactive console to test functions
[For o in var.list : o.id] --> Var.list[*].id
public_ip_address_id = element(azurerm_public_ip.mypublicip[*].id, count.index)  

Element function - element(list, index) -- element(["blue","red","green"], 1) --> red
Length function - length(["blue","red","green"]) -- > 3
element(["blue","red","green"], length(["blue","red","green"])-1) --> green

Toset function converts values to strings and there cannot be duplicates - they are removed, sorts alphabetically, number string 1st
resource "azurerm_resource_group" "myrg" {
  for_each = toset([ "eastus", "eastus2", "westus" ])
  name = "myrg-${each.value}"
  location = each.key 


# Provisioners


