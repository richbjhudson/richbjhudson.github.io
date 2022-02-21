---
layout: techNote
category: Terraform
title: Configuration
---
# Input Variables

- They serve as parameters.
- `variables.tf` file includes variables that typically have a name, description, type and default value.

```
variable "cost_centre" {
  description = "Cost Centre Number"
  type = string 
  default = "0001"
}
```

- You can reference a variable within a `azurerm_resource_group` resource block using the follow syntax:

```
name = "${var.business_unit}-${var.environment}-${var.resource_group_name}"
location = var.resource_group_location
```

- If you do not set a default value you will be prompted for the value when you run `terraform plan` or `terraform apply`. The variable description appears in the prompt.
- When running a `terraform destroy` you will also be prompted for the input variable.
- You can combine variables and local names:

```
name = "${azurerm_virtual_network.myvnet.name}-${var.subnet_name}"
```

- You can override input variable default values from the terraform cli:

```
terraform plan -var="resource_group_name=hex-rg-01"
```

- You can save the variable values from the plan command into a file and use this to run the apply command against.

```
terraform plan -var="resource_group_name=hex-rg-01" -out v1.plan
terraform show v1.plan
terraform apply v1.plan
```

## Environment Variables

- These values override default variable values.
- You can create them on Windows using `$env:TF_VAR_variable_name=value
`:

```
$env:TF_VAR_resoure_group_location='uksouth'
```

- You can check the values that have been set on Windows using `dir env:`.

## terraform.tfvars

- This file may be used to autoload the variable values replacing any default values.
- You do not have to reference the file during terraform cli commands.
- The contents of `terraform.tfvars` may look like:

```
cost_centre = "1234"
environment = "prod"
```

- -var-file may be used when executing terraform cli when **filename.tfvars**. You may use different files if you wish to run the code against multiple environments.

```
terraform plan -var-file="dev.tfvars"
```

*Note: If you run the same code against multiple .tfvars in the same working directory, the state file uses the same local names for a resource so would attempt to replace the resources. This is what terraform workspaces is for to create separate state files.*

- `filename.auto.tfvars` autoloads the variable values from a file replacing any default values. You do not need to use the cli argument `-var-file`. Common use case is to separate variable input based on resource type.

# Complex Variable Types

## List

- Square `[]` brackets are used to define a list type variable.

```
variable "virtual_network_address_space" {
  description = "Virtual Network Address Space"
  type        = list(string)
  default     = ["10.0.0.0/16", "10.1.0.0/16", "10.2.0.0/16"]
}
```

- You can set a value for the variable type in a .tfvars files as follows:

```
virtual_network_address_space = ["10.3.0.0/16", "10.4.0.0/16", "10.5.0.0/16"]
```

- You can reference a list variable type in a resource block simply using `var.virtual_network_address_space`.

- If you wish to reference a single value you can use:

```
address_space = [var.virtual_network_address_space[0]]
```

## Maps

- Flower `{}` brackets are used to define a map type variable.

```
variable "default_tags" {
  description = "Default Tags for Azure Resources"
  type = map(string)
  default = {
    "deploymentType" = "Terraform",
    "costCentre" = "0001"
  }
}
```

- You can set a value for the variable type in a .tfvars files as follows:

```
default_tags = {
    "deploymentType" = "Terraform",
    "costCentre" = "1234"
}
```
- You can reference a map variable type in a resource block simply using `var.default_tags`.

## Lookup Function

- A lookup function gets a value from a map.
- `Lookup(map, key, default)`
- Variable set as:

```
variable "public_ip_sku" {
  description = "Azure Public IP Address SKU"
  type = map(string)
  default = {
    "ukwest" = "Basic",
    "uksouth" = "Standard"
  }
}
```

- `azurerm_public_ip` resource block set as:  

```
sku = lookup(var.public_ip_sku, var.resoure_group_location, "Basic")
```

- In this example the SKU is set based on the Resource Group Location matching the key value, otherwise a default value of **Basic** is set.

# Validation Rules in Variables


# Structural

# Output

# Locals

# Data Sources

# CLI Workspaces

# Provisioners

# Null Resource

# Dynamic Blocks

# Override Files

# External Providers & Data Sources
