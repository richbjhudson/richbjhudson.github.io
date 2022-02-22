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

## Variable Definition Precedence

- From High to low:
    1. var or -var-file
    2. *.auto.tfvars
    3. Terraform.tfvars.json
    4. Terraform.tfvars
    5. Environment variable

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

*Note: If a key value starts with a number you must use `:` instead of `=` when setting a value.*

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

- In this example the `sku` is set based on the `resoure_group_location` matching the key value `ukwest` or `uksouth`, otherwise a default value of **Basic** is set.

# Validation Rules in Variables

- A validation rule consists of a `condition` and an `error_message`.
- The `condition` makes use of functions such as `contains`, `substr`, `length` and `lower`.
- `error_message` must end string with `.` Or `?`.

## Examples of Function Use

- length function:

```
Length ("hi")
2
Length(["a","b"])
2
Length ({"key" = "value"})
1
```

- Substring function extracts a substring from a given string - `Substr(string, offset, length)`:

```
Substr("hello world", 1, 4)
ello
```

- Contains function - `Contains(list, value)`:

```
Contains (["a","b"], "a")
true
```

- Lower/ Upper function

```
Lower("ABC")
abc
```

- [Regex](https://www.terraform.io/language/functions/regex) function applies a regular expression to a string and returns the matching substring - `Regex(pattern, string)`:

```
regex("india$", "westindia")
india
```

- Can function evaluates the given expression and returns a boolean value based on result without error.

```
Can(regex("india$", "westindia"))
true
```

## Validation Rule Examples

- Two examples of setting explicit allowed values for a variable:
```
variable "resoure_group_location" {
  description = "Resource Group Location"
  type = string
  default = "uksouth"
  validation {
    condition = var.resoure_group_location == "uksouth" || var.resoure_group_location =="ukwest"
    error_message = "We only allow Resources to be created in uksouth or ukwest locations."
  }
```


```
variable "resoure_group_location" {
  description = "Resource Group Location"
  type = string
  default = "uksouth"
  validation {
    condition = contains(["uksouth", "ukwest"], var.resoure_group_location)
    error_message = "We only allow Resources to be created in uksouth or ukwest locations."
  }
```
- Example of providing more flexibility using regex expression:
```
variable "resoure_group_location" {
  description = "Resource Group Location"
  type = string
  default = "eastus"
  validation {
    condition = can(regex("india$", var.resoure_group_location))
    error_message = "We only allow Resources to be created in westindia or southindia locations."
  }
```

# Sensitive Input Variables

- Terraform will redact values in command output and log files for variables defined as `sensitive = true`.
- It is important to recognise that terraform state **shows** values of sensitive variables.
- How to set a variable as sensitive:

```
variable "localadmin_password" {
  description = "Local Administrator Password"
  type = string  
  sensitive = true
}
```

- Set values for sensitive variables using file `secrets.tfvars`.
- Never check-in `secrets.tfvars` to your code repository.
- How to execute terraform cli against the variable file:

```
terraform plan -var-file="secrets.tfvars"
```

*Note: Environment variable values will appear in command line history.*

# Structural Variable Types

## Object

- This is a map with different types of key value pairs. 

```
variable "mysql_policy" {
  description = "Azure MySQL DB Threat Detection Policy"
  type = object({
    enabled = bool,
    retention_days = number
    email_account_admins = bool
    email_addresses = list(string)
  })
}

```

- You can set a value for the variable type in a .tfvars files as follows:

```
mysql_policy = {
    enabled = true,
    retention_days = 10,
    email_account_admins = true,
    email_addresses = [ "email1@gmail.com", "email2@gmail.com" ]
  }

```
- You can reference an object variable type in a `azurerm_mysql_server` resource block as follows:

```
threat_detection_policy {
    enabled = var.mysql_policy.enabled
    retention_days = var.mysql_policy.retention_days
    email_account_admins = var.mysql_policy.email_account_admins
    email_addresses = var.mysql_policy.email_addresses
  } 
```

## Tuple

- A list with values of different types.

```
variable "mysql_policy" {
  description = "Azure MySQL DB Threat Detection Policy"
  type = tuple([ bool, number, bool, list(string) ])
}
```
- You can set a value for the variable type in a .tfvars files as follows:

```
mysql_policy = [true, 10, true, [ "email1@gmail.com", "email2@gmail.com" ]]
```
- You can reference a tuple variable type in a `azurerm_mysql_server` resource block as follows:

```
threat_detection_policy {
    enabled = var.mysql_policy[0]
    retention_days = var.mysql_policy[1]
    email_account_admins = var.mysql_policy[2]
    email_addresses = var.mysql_policy[3]
  }  
}

```

## Sets

- Almost the same as lists and tuples, except any duplicate values are removed and any ordering is lost.

# Output Variables

- Output values are produced primarily to share output between a parent and child module.
- You can use the `terraform_remote_state` data source to access outputs that are using remote state.
- Outputs can be used with arguments or attributes of a resource type.

```
output "resource_group_id" {
  description = "Resource Group ID"
  # Attribute Reference
  value = azurerm_resource_group.rg.id 
}
```

- `terraform output` - this shows the output from the local state file.
- When you set an output variable as sentitive it is not honored and will show the value when you explicitly call an output variable e.g. `terraform output virtual_network_name`.

## Count & Splat Expression

- Splat expression `var.list[*].id` is a concise way to express a common expression.
- This only works with lists, tuples and sets.
- Here is an example `azurerm_virtual_network` resource block that is using `count`:

```
resource "azurerm_virtual_network" "vnet" {
  count = 4
  name                = "vnet-${count.index}"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name
}
```

- Here is an example output variable using the splat expression:

```
output "virtual_network_name" {
  description = "Virtual Network Name"
  value = azurerm_virtual_network.vnet[*].name 
}
```
*Note: count.index cannot not be used in an output block.*

- Here is how the output looks:

```
terraform output
virtual_network_name = [
  "it-dev-vnet-0",
  "it-dev-vnet-1",
  "it-dev-vnet-2",
  "it-dev-vnet-3",
]
```

## for_each

- Resource blocks that use the`for_each` meta-argument will create a map of objects so you cannot use a splat expression as above.
- You need to use a `for` in the output value.
- Here is an example where a VNET is created for each environment.

```
variable "environment" {
  description = "Environment Name"
  type = set(string)
  default = ["dev1", "qa1", "staging1", "prod1" ]
}
```

```
resource "azurerm_virtual_network" "vnet" {
  for_each = var.environment
  name                = "vnet-${each.key}"
  address_space       = ["10.0.0.0/16"]
  location            = "UKSouth"
  resource_group_name = "rg1"
}
```

- Example 1

```
output "virtual_network_name_list_one_input" {
  description = "Virtual Network - For Loop One Input and List Output with VNET Name "
  value = [for vnet in azurerm_virtual_network.vnet: vnet.name ]  
}

terraform output virtual_network_name_list_one_input
[
  "vnet-dev1",
  "vnet-prod1",
  "vnet-qa1",
  "vnet-staging1",
]

```

- Example 2

```
# Output - For Loop Two Inputs, List Output which is Iterator i (var.environment)
output "virtual_network_name_list_two_inputs" {
  description = "Virtual Network - For Loop Two Inputs, List Output which is Iterator i (var.environment)"  
  value = [for env, vnet in azurerm_virtual_network.vnet: env ]
}

terraform output virtual_network_name_list_two_inputs
[
  "dev1",
  "prod1",
  "qa1",
  "staging1",
]

```

- Example 3

```
# Output - For Loop One Input and Map Output with VNET ID and VNET Name
output "virtual_network_name_map_one_input" {
  description = "Virtual Network - For Loop One Input and Map Output with VNET ID and VNET Name"
  value = {for vnet in azurerm_virtual_network.vnet: vnet.id => vnet.name }
}

terraform output virtual_network_name_map_one_input  
{
  "/subscriptions/a529f686-82de-4a8d-b643-747ed505372a/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnet-dev1" = "vnet-dev1" 
  "/subscriptions/a529f686-82de-4a8d-b643-747ed505372a/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnet-prod1" = "vnet-prod1"
  "/subscriptions/a529f686-82de-4a8d-b643-747ed505372a/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnet-qa1" = "vnet-qa1"   
  "/subscriptions/a529f686-82de-4a8d-b643-747ed505372a/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnet-staging1" = "vnet-staging1"
}

```

- Example 4

```
# Output - For Loop Two Inputs and Map Output with Iterator env and VNET Name
output "virtual_network_name_map_two_inputs" {
  description = "Virtual Network - For Loop Two Inputs and Map Output with Iterator env and VNET Name"
  value = {for env, vnet in azurerm_virtual_network.vnet: env => vnet.name }
}

terraform output virtual_network_name_map_two_inputs 
{
  "dev1" = "vnet-dev1"
  "prod1" = "vnet-prod1"
  "qa1" = "vnet-qa1"
  "staging1" = "vnet-staging1"
}

```

# Locals

- DRY principle - don't repeat yourself.
- Local value - you can assign a name to an expression, you can then use that name multiple times within a module.
- Reference using `local.name`.
- You can use in combination with conditional expressions to dynamically set the value.
- Example locals definition:

```
locals {
  # Default tags to be assigned to all resources
  businessService = "Reporting"
  owner = "IT"
  default_tags = {
    Service = local.businessService
    Owner = local.owner
  }
```
- Example of how to reference a local within a `azurerm_resource_group` resource block:

```
  tags = local.default_tags
```

## Conditional Expression

- You cannot write conditional expressions within a variable definition, you can only use them with locals.
- `condition ? true_val : false_val`.

- Example use of conditional expression within locals:

```
variable "environment" {
  description = "Environment Name"
  type = string
  default = "qa"
}

variable "vnet_address_space_dev" {
  description = "Virtual Network Address Space for Dev Environment"
  type = list(string)
  default = [ "10.0.0.0/16" ]
}

variable "vnet_address_space_all" {
  description = "Virtual Network Address Space for All Environments except dev"
  type = list(string)
  default = [ "10.1.0.0/16", "10.2.0.0/16", "10.3.0.0/16"  ]
}
```
```
locals {
vnet_address_space = (var.environment == "dev" ? var.vnet_address_space_dev : var.vnet_address_space_all)
}
```

- Example use of conditional expression within a `azurerm_virtual_network` resource block to manipulate count:

```
count = var.environment == "dev" ? 1 : 5
```

# Data Sources

- Data sources allow data to be fetched/ computed for use elsewhere in your terraform configuration.
- Use when you need to make use of information defined outside of Terraform or from another terraform configuration.
- A data resource is known as a data block.

*Note: Lifecycle meta-arguments are not supported.*
 
```
data "azurerm_subscription" "current" {
}
```

- You can test data sources using outputs:

```
output "current_subscription_display_name" {
  value = data.azurerm_subscription.current.display_name
}
```

# CLI Workspaces

# Provisioners

# Null Resource

# Dynamic Blocks

# Override Files

# External Providers & Data Sources
