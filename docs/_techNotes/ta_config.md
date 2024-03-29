---
layout: techNote
category: Terraform
title: Configuration
---
## Input Variables

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

### [Variable Definition Precedence](https://www.terraform.io/language/values/variables#variable-definition-precedence)

- From High to low:
    1. var or -var-file
    2. *.auto.tfvars
    3. Terraform.tfvars.json
    4. Terraform.tfvars
    5. Environment variable

### Environment Variables

- These values override default variable values.
- You can create them on Windows using `$env:TF_VAR_variable_name=value
`:

```
$env:TF_VAR_resoure_group_location='uksouth'
```

- You can check the values that have been set on Windows using `dir env:`.

### tfvars Files

- A `terraform.tfvars` file may be used to autoload the variable values replacing any default values.
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

*Note: If you run the same code against multiple .tfvars in the same working directory, the state file uses the same local names for a resource so would attempt to replace the resources. **This is what terraform workspaces is for to create separate state files.***

- `filename.auto.tfvars` autoloads the variable values from a file replacing any default values. You do not need to use the cli argument `-var-file`. Common use case is to separate variable input based on resource type.

## Complex Variable Types

### List

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

### Maps

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

### Lookup Function

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

## Validation Rules in Variables

- A validation rule consists of a `condition` and an `error_message`.
- The `condition` makes use of functions such as `contains`, `substr`, `length` and `lower`.
- `error_message` must end string with `.` Or `?`.

### Examples of Function Use

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

### Validation Rule Examples

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

## Sensitive Input Variables

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

- It is good practice to set values for sensitive variables using file `secrets.tfvars`.
- Never check-in `secrets.tfvars` to your code repository.
- How to execute terraform cli against the variable file:

```
terraform plan -var-file="secrets.tfvars"
```

*Note: Environment variable values will appear in command line history.*

## Structural Variable Types

### Object

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

### Tuple

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

### Sets

- Almost the same as lists and tuples, except any duplicate values are removed and any ordering is lost.

## Output Variables

- Output values are produced primarily to share output between a parent and child module.
- You can use the `terraform_remote_state` data source to access outputs that are stored in another project's remote state.
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

### Count & Splat Expression

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

### for_each

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

- Example 1 - Loop through resource block and return a list of vnet names:

```
output "virtual_network_name_list_one_input" {
  description = "Virtual Network - Loop through resource block and return a list of vnet names"
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

- Example 2 - Loop through the iterator used by the resource block (var.environment) and return a list of the iterator:

```
output "virtual_network_name_list_two_inputs" {
  description = "Virtual Network - Loop through the iterator used by the resource block (var.environment) and return a list of the iterator"  
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

- Example 3 - Loop through resource block and return a map of vnet id = vnet name:

```
output "virtual_network_name_map_one_input" {
  description = "Virtual Network - Loop through resource block and return a map of vnet id = vnet name"
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

- Example 4 - Loop through the iterator used by the resource block (var.environment) and use to loop through the resource block and map iterator = vnet name:

```
output "virtual_network_name_map_two_inputs" {
  description = "Virtual Network - Loop through the iterator used by the resource block (var.environment) and use to loop through the resource block and map iterator = vnet name"
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

## Locals

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

### Conditional Expression

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

## Data Sources

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
*Note: display name is an [attribute reference](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/subscription).*

## CLI Workspaces

- Named workspaces allow you to conveniently switch between multiple instances of a **single configuration** within a **single backend**.
- Common use case: create a parallel distinct copy of a set of infrastructure in order to test a set of changes before applying to the main production infrastructure.
- HashiCorp do not recommend using workspaces for large infrastructures. They recommend using separate configuration directories for each environment - dev, qa, staging and production.

*Note: They are **not** related to terraform cloud workspaces.*

- You may reference the workspace name using `${terraform.workspace}`. It can be used in your naming and tagging e.g. `rg_name = "${var.business_unit}-${terraform.workspace}-${var.resoure_group_name}"`.

- `terraform workspace list` - shows all workspaces and the current workspace has a `*` next to it.
- `terraform workspace show` - shows current workspace.
- `terraform workspace new ws_name` - create a new workspace.
- None default states are placed in `.\terraform.tfstate.d\ws_name`.
- `terraform workspace select ws_name` - this is how you switch workspaces.
- `terraform workspace delete ws_name` - you cannot delete a workspace that has existing resources.
  - You can force the deletion but then only the state will be removed and all the resources will remain in the cloud. 
  - Also you cannot delete an active workspace nor the default.

- When a remote backend is configured the same behaviour applies as above except how the directory and file name is handled e.g. a new workspace state file is placed in the same Azure storage account container with a suffix of `env:ws_name` e.g. `terraform.tfstateenv:dev`.

## Provisioners

- Provisioners can be used to execute specific actions on the local machine or on a remote machine in order to prepare servers.
  - Passing data into VMs. 
  - Running configuration management software e.g. packer, chef or ansible.

- Provisioners are at **creation time** type by default, usually used for bootstrapping. You can use the `when` attribute to change this.

- Provisioners should be perceived as a last resort.

- Most provisioners require access to remote resources via `ssh` or `winrm` using a nested `connection` block.
- `connection` blocks cannot refer to a parent resource by local name and use a special self object e.g `user = self.admin_username`.

- If you wish to use a `provisioner` block that is not directly associated with a resource use a `null_resource`.

- Types - Creation-time or destroy-time provisioners
	- **File** - copy files from terraform executing machine into newly created resource using ssh or winrm.
	- **Remote-exec** - invokes a script on the remote resource after it is created. Used to run a configuration management tool or bootstrap into a cluster.
	- **Local-exec** - invokes a local executable after a resource is created on the machine running terraform.

### Failure behaviour

- **Continue** - on error continue with creation or destruction.
- **Fail (default)** - raise error and stop applying, if creation time it will taint the resource.
- If provisioner marks a resource as tainted the resource is recreated on the next `terraform apply` and therefore the provisioner re-run.
- You can change the tainted behaviour using `on_failure` attribute.
- Example file provisoner error: `Upload failed: scp: /var/www/html/file-copy.html: Permission denied`.
- Extract from `terraform.tfstate` showing tainted resource: 

```
"resources": [
    {
      "mode": "managed",
      "type": "azurerm_linux_virtual_machine",
      "name": "mylinuxvm",
      "provider": "provider[\"registry.terraform.io/hashicorp/azurerm\"]",
      "instances": [
        {
          "status": "tainted"
```
- Example of setting provisioner to continue on error:

```
provisioner "file" {
    source      = "files/index.html"
    destination = "/var/www/html/index.html"
    on_failure  = continue
   } 
```

### File Provisioner

- Copy files from the machine executing terraform to a newly created virtual machine resource.
- A `connection` block is used to connect to the Windows or Linux VM using winrm and ssh respectively.
  - You can place it in a resource block or provisioning block.
  - `azurerm_linux_virtual_machine` resource block example:

```
connection {
    type = "ssh"
    host = self.public_ip_address
    user = self.admin_username
    private_key = file("${path.module}/ssh-keys/key.pem")
  }  
  provisioner "file" {
    source = "files/index.html"
    destination = "/tmp/index.html"
  }
```
*Note: You have to use a self object otherwise you would create an implicit dependency with itself. e.g. the `azurerm_linux_virtual_machine` resource would have to be created to use the `user` argument within the `connection` block.*

- Copies text into a file on a VM resource:

```
  provisioner "file" {
    content = "VM Host name: ${self.computer_name}"
    destination = "/vm_folder/file.log"
  }
```

- Copies subfolder into a folder on a VM resource:

```
  provisioner "file" {
    source = "folder/subfolder"
    destination = "/vm_folder"
  }
```

- Copies all files and folders from the subfolder into a folder on a VM resource:

```
  provisioner "file" {
    source = "folder/subfolder/"
    destination = "/vm_folder"
  }
```

### Remote-exec

- Invokes a script on a remote resource after it is created.
- You can invoke inline or a script.
- Example that copies a file on the VM resource to another folder.
```
  provisioner "remote-exec" {
    inline = [
      "sudo cp /tmp/index.html /var/www/html"
    ]
  }
```

### Local-exec

- Invokes a process on the machine running terraform.
- `Command` is required argument and `working_dir` and `interpreter` are key optional arguments.

```
  provisioner "local-exec" {
    when    = destroy
    command = "echo Destroy-time provisioner Instance Destroyed at `date` >> destroy-time.txt"
    working_dir = "terraformhostfolder/"
  }
```

## Null Resource

- A `null` resource does nothing.
- This `provisioner` example is used to upload the latest application code to a VM without causing the VM to be re-provisioned.

- It uses a provider:

```
null = {
      source = "hashicorp/null"
      version = ">= 3.1.0"
    }  
```

- In this example it will be used in conjunction with the `time` provider.

```
time = {
      source = "hashicorp/time"
      version = ">= 0.7.2"
    }  
```

- Create `time_sleep` resource to wait 90 seconds after VM creation before triggering `null_resource`:

```
resource "time_sleep" "wait_90_seconds" {
  depends_on = [azurerm_linux_virtual_machine.mylinuxvm]
  create_duration = "90s"
}
```

- As the `connection` and `provisioner` blocks are in `null_resource` you must reference the VM resource rather than use `self`:

```
resource "null_resource" "sync_app1_static" {
  depends_on = [time_sleep.wait_90_seconds]
  triggers = {
    always-update = timestamp() 
  }

  connection {
    type = "ssh"
    host = azurerm_linux_virtual_machine.mylinuxvm.public_ip_address
    user = azurerm_linux_virtual_machine.mylinuxvm.admin_username
    private_key = file("${path.module}/ssh-keys/key.pem")
  }

  provisioner "file" {
    source = "folder/subfolder"
    destination = "/tmp"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo cp -r /tmp/subfolder /var/www/html"
    ]    
  }
}
```

- This part of the resource block ensures that everytime a `terraform apply` is completed the resource block is re-applied, in this scenario therefore will copy the files:

```
triggers = {
    always-update = timestamp() 
  }
```

## Dynamic Blocks

- Some resources include repeatable nested blocks in their arguments e.g. [azurerm_network_security_group](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/network_security_group) includes `security_rule {}`.
- Dynamic blocks in this example allow you use a `for_each` loop to simplify your code by prefixing the block `security_rule` with `dynamic`:

```

locals {
  ports = [22, 80, 8080, 8081, 7080, 7081]
}

resource "azurerm_network_security_group" "nsg" {
  name                = "nsg"
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name
  dynamic "security_rule" {
    for_each = local.ports 
    content {
      name                       = "inbound-rule-${security_rule.key}"
      description                = "Inbound Rule ${security_rule.key}"    
      priority                   = sum([100, security_rule.key])
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = security_rule.value
      source_address_prefix      = "*"
      destination_address_prefix = "*"      
    }
  }
}

```
*Note: security_rule.key is the same as security_rule.value because local.ports is a list.*

## Override Files

- If two resources attempt to use the same local name terraform will return an error - the local name of a resource should be unique.
- `override.tf` - you can reference a local name a 2nd time and this configuration will override the initial configuration of the object.
- `fileName_override.tf` works as above.

*Note: If you need to use override.tf files make sure you modify `.gitignore` as it will ignore this type of file my default.* 

## External Providers & Data Sources

- A **Provider** helps provide an interface between terraform and external programs.


- Here is an example where the input is captured by the script using the `query` block.

```
data "external" "ssh_key_generator" {
  program = ["bash", "${path.module}/shell-scripts/ssh_key_generator.sh"]
  
  query = {
    key_name = "terraformdemo"
    key_environment = "dev"
  }
}
```

- You can then capture the output to use in the terraform code. It is important to note that the output of the script must generate a json object.
- Then the output may be referenced as follows:

```
output "public_key" {
  description = "public_key"
  value = data.external.ssh_key_generator.result.public_key
}
```

*Note: After destroy, the files remain but the reference to the files and contents are removed from the state.*
