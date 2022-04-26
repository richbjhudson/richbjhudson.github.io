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
- Each tab within the context of a public terraform registry module maps to a `.tf` file:
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
    - External modules are downloaded into a sub folder e.g. modules\vnet.
    - Internal modules (those within a subfolder from the working directory) are referenced only in the modules.json file.

# Taint Resource

- Use `terraform state list` to identify the resource that you wish to taint.
- Run `terraform taint` command e.g. `terraform taint module.vnet.azurerm_subnet.subnet[2]` and then `terraform apply`. At this point on the next terraform apply the tainted resource will be recreated.

*Note: HashiCorp now recommend the use of `terraform apply -replace='module.vnet.azurerm_subnet.subnet[2]'` instead of the `[terraform taint](https://www.terraform.io/cli/commands/taint)` command.*

# Child Module

- Create a subdirectory `modules\module_name`.
- Add `main.tf`, `outputs.tf`, `variables.tf` and `versions.tf` files.
- Create a `README.md`.
- Here is an example of how to reference a child module from a parent module in root directory '/':

```
module "resource_group" {
  source = "./modules/resource_group"
  location                          = "uksouth"
  resource_group_name               = "rg1" 
}
```

- Variables in the child module become the arguments in the parent module.
- Configure outputs in parent module that use outputs from child module:
  - From child module:
  ```
  output "resource_group_id" {
  description = "resource group id"
  value       = azurerm_resource_group.resource_group.id
  }
  ```

  - From parent module:   
  ```
  output "resource_group_id" {
  description = "resource group id"
  value       = module.resource_group.resource_group_id
  }
  ```

# Get Command

- `terraform get` will get the modules and update the `.terraform/modules/module_name/modules.json` file, it does **not** initialize the backend.
- Terraform will automatically notice changes to modules when it is local, when remote you would need to redownload using the `terraform get` command.
- If you add a new local module then you would have to run the `terraform get` command.

# Publishing a Module

- Recommended naming convention: **terraform-provider-module_name** e.g. terraform-azurerm-staticwebsitepublic.
- Create a new Github repository and upload module code.
- Create a release on the Github repository.
- Publish the GitHub repository on [terraform registry](https://registry.terraform.io)
  - Sign in with your GitHub account.
  - Navigate to Modules> publish and then select the repository from GitHub.
- At this point if you have created a `Readme.md`, `variables.tf`, `versions.tf` and `outputs.tf` it will populate the Readme, Inputs, Outputs and Dependency tabs respectively.
- You will also notice that it will create **Provision Instructions**, here is an example:

```
module "staticwebsitepublic" {
  source  = "richbjhudson/staticwebsitepublic/azurerm"
  version = "1.0.0"
  # insert the 8 required variables here
}
```

# Module Source

- Module sources can be Local, terraform registry public/ private or from an online code repository.
- `sources` argument assumes prefix "https://registry.terraform.io/modules/"
- Example of using terraform private registry to store module: 

```
module "s3-bucket" {
  source  = "app.terraform.io/richbjhudson/s3-bucket/aws"
  version = "2.13.0"
  # insert required variables here
}
```

- Example of using GitHub to store module: `source = "github.com/richbjhudson/terraform-azurerm-staticwebsitepublic"`.

- Example of using GitHub to store module - with SSH connectivity: `source = "git@github.com:richbjhudson/terraform-azurerm-staticwebsitepublic.git"`.

- Example of using GitHub to store module and selecting a specific release: `source = "git::https://github.com/stacksimplify/terraform-azurerm-staticwebsitepublic.git?ref=1.0.0"`.
