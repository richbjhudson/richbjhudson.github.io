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

*Note: If run the same code against multiple .tfvars in the same working directory, the state file uses the same local names for a resource so would attempt to replace the resources. This is what terraform workspaces is for to create separate state files.*

- `filename.auto.tfvars` autoloads the variable values from a file replacing any default values. You do not need to use the cli argument `-var-file`. Common use case is to separate variable input based on resource type.

# Complex

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
