---
layout: techNote
category: Terraform
title: Modules
---
# Overview

- Modules are containers for multiple resources that are used together - a collection of .tf files kept together in a directory.
- A root module calls child modules (they can be called multiple times).
- You can load modules from the local file system, public or private registries.
- You can publish modules for others to use.

# Terraform Registry - Public

- [Terraform Public Registry](https://registry.terraform.io/browse/modules?provider=azurerm)
- A tick next to a module means that it is from a verified partner.
    - Inputs - variables.tf
    - Outputs - outputs.tf
    - Dependencies - versions.tf
- In respects to the module source it is assumed [Terraform Registry](https://registry.terraform.io/) is the suffix.
- Example use of a public module:

```
module "vnet" {
  source  = "Azure/vnet/azurerm"
  version = "2.5.0" 
  vnet_name = "vnet1"
  resource_group_name = "rg1"
  address_space       = ["10.0.0.0/16"]
  subnet_prefixes     = ["10.0.1.0/24"]
  subnet_names        = ["subnet1"]
  tags = {
    environment = "dev"
  } 
}
```

- Example resource block argument referencing a value output by a module - see [output](https://registry.terraform.io/modules/Azure/vnet/azurerm/latest?tab=outputs):

```
subnet_id = module.vnet.vnet_subnets[0]
```

- Example of using source module [outputs](https://registry.terraform.io/modules/Azure/vnet/azurerm/latest?tab=outputs) in parent module `outputs.tf`:

```
output "virtual_network_name" {
  description = "Virtual Network Name"
  value = module.vnet.vnet_name
}
```

- `terraform init` downloads the modules into .terraform directory prior to downloading the provider plugins.
    - Creates a modules folder including modules.json file that contains details of the modules. 
    - Public modules are downloaded into a sub folder e.g. modules\vnet.
    - Private modules are referenced only in the modules.json file.

# Tainting Resources

# Child Module

# Get Command

# Module Source
