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

- If you do not set a default you will be prompted for the value when you run `terraform plan` or `terraform apply` commands. The variable description appears in the prompt.
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

# Environment Variables

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
