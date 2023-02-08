---
layout: techNote
category: Terraform
title: Commands
---
## Remote State Storage & Locking

- A `backend` block is within a `terraform` block usually in a `versions.tf` file.
- Backend blocks are responsible for storing state and providing an API for state locking.
- Azure Storage Accounts support both state storage and locking (supported on all operations that may write state).
- When managing IaC in a Team:  
    - A local state file needs to be accessible to all members of a Team and stored in a shared location.
    - If two members of a Team run terraform at the same time it may create a race condition. Multiple updates to a state file may lead to conflicts, data loss or corruption. Locking prevents this scenario.
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

## Remote State Data Source

- `terraform_remote_state` data source retrieves the root module output values from some other terraform configuration.
- Example Scenario:
    - Project-1/terraform.tfstate - VNET + RG
    - Project-2/terraform.tfstate - to build a VM, it needs details regarding VNET + RG
- Within dependant Project-1 you must define outputs that may be used by Project-2 e.g. VNET + RG.
- Then within Project-2 you access the remote state of Project-1 as a data source to use its outputs.
- Here is an example of the `terraform_remote_state` block configuration used by Project-2 to access the output values defined in Project-1.

```
data "terraform_remote_state" "Project-1" {
  backend = "azurerm"
  config = {
    resource_group_name   = "rg1"
    storage_account_name  = "sta1"
    container_name        = "tfstatefiles"
    key                   = "terraform.tfstate"
  }
}
```
- At this point you may reference the outputs defined in Project-1 from a Project-2 `azurerm_linux_virtual_machine` resource block as follows:

```
resource_group_name = data.terraform_remote_state.Project-1.outputs.resource_group_name
```

## State Commands

- `terraform show` by default outputs the local terraform.tfstate file contents.
- `terraform plan -out="v1plan.out"` - write a terraform plan out to a file.
    - `terraform show "v1plan.out"` - view contents of terraform plan output file.
    - `terraform show -json "v1plan.out"` - as above but in json format.
- `terraform apply "v1plan.out"` - this applies the terraform plan without re-running a plan.

- `terraform state list` - list resources in a state file. *Note: data sources appear as resources.*
- `terraform state show data.azurerm_subscription.current` - shows attributes of a single resource in the state file.

- `terraform state mv` this allows you to change local resource names, for example:
```
terraform state list
terraform state mv -dry-run azurerm_virtual_network.myvnet azurerm_virtual_network.myvnet-new
terraform state mv azurerm_virtual_network.myvnet azurerm_virtual_network.myvnet-new
terraform state list
```
*Note: If configuration has different local names then it will see that it needs to create a new resource and destroy the incorrectly named local named resource. They need to match.*

- `terraform apply -refresh-only` - refreshes the state file to match the current settings of managed remote objects.
    - This command updates the state file only it does not make any changes to the cloud resources.
    - `terraform refresh` - This is now deprecated as default behaviour is unsafe, it is the equivalent to `terraform apply -refresh-only -auto-approve`.

- `terraform state rm` - remove resources from the terraform state e.g. `terraform state rm azurerm_virtual_network.myvnet-new`.
    - If still present in configuration , it will recreate.
    - If you do not want a resource to be managed by terraform you need to remove it from both the configuration and state file.

- `terraform state replace-provider` - you may download a copy of a provider plugin and store it in an internal repository and access it from an internal source. This command allows you to update the configuration to use an internal source e.g. [terraform state replace-provider hashicorp/aws registry.acme.corp/acme/aws](https://www.terraform.io/cli/commands/state/replace-provider).

- `terraform state pull` - download and output state file to cli. You could copy and paste the contents into a file to make a local terraform.tfstate.
- `terraform state push terraform.tfstate` - used to migrate local state to remote state.

- `terraform force-unlock LOCK_ID` - only apply's to AWS Dynamo DB.

- `terraform taint` - force the re-creation of resources. It marks a resource as tainted so that it is destroyed and then created.
    - Real world example would be where the cloud-init file has been changed, so you need to recreate the VM to re-apply the bootstrap file.
    - Example: `terraform taint azurerm_virtual_network.myvnet-new` would result in the recreation of the VNET on the next `terraform apply`.
- `terraform untaint` - remove tain mark on resource.

- `terraform plan/appy -target` - apply configuration to a specific resource. It is only used in exceptional circumstances e.g. recover from mistakes or work around limitations.

## Import

- Import existing infrastructure into the state.
- It does not generate the configuration - in the future it will.
- Example use:
  - Create dummy resource:
  ```
  resource "azurerm_resource_group" "rg1" {
   }
  ```
  - Import existing resource into state file:
  ```
  terraform init
  terraform import azurerm_resource_group.rg1 /subscriptions/a529f686-82de-4a8d-b643-747ed505372a/resourceGroups/rg1
  ```
  - At this point the state file entry is created.

- You would need to use the state file as a reference to create the respective configuration files.
- Make sure you create an argument for **all** resource attributes shown in the state file.

## Debug

- You may enable terraform cli logging to help with troubleshooting. It is enabled using environment variables. 

*Note: The typical format of  `$env:TF_VAR_<variable_name>=value` that is the equivalent to `.tfvars` does not apply.*

- TF_LOG sets the logging level: 
	- TRACE -- verbose output
	- DEBUG -- concise version of TRACE
	- ERROR -- show errors that prevent terraform from continuing
	- WARN - warning for mistakes that do not prevent execution
	- INFO - general high level messages about execution process
- TF_LOG_PATH sets the location of where to hold the log file.
- Here is an example of how to configure for a Windows client:

```

$env:TF_LOG='TRACE'
$env:TF_LOG_PATH='terraform-trace.log'
Dir env:

```

- You can permenantly set the environment variables in Linux by adding the configuration to `.bashrc`. In windows open `$profile` (opens Microsoft.Powershell_profile.ps1) and add the lines above.
- Crash.log file will hold terraform panic errors that should be reported to hashicorp.

## CLI User Settings

- Per user settings for CLI behaviours
  - Windows location of `terraform.rc` in `$env:APPDATA` --> `C:\Users\userName\AppData\Roaming`.
  - Otherwise `.terraformrc` in `$home`.
- Settings that can be included in the file:
  - `credentials_block` - add terraform cloud token.
  - `credentials_helper` - you can retrieve credentials using external source.
  - `disable_checkpoint` - if true disables upgrade and security check to HashiCorp, if used in conjuction with `plugin_cache_dir` terraform cli will not reach out to HashiCorp network services.
  - `plugin_cache_dir` - store plugins that are downloaded into a central place as well as terraform working directory.
  - `disable_checkpoint_signature` - disables use of anonymous id used to de-duplicate warning messages.
  - `Provider_installation` - default uses registry.terraform.io, you may wish to use another path.

- Example configuration:
  - Create directory for cached plugins `mkdir -p $HOME/.terraform.d/plugin-cache`
  - Sample `.terraformrc` file contents
  
  ```
  plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
  disable_checkpoint = true
  ```
- `terraform init` output demonstrating use of plugin-cache:

```
Initializing provider plugins...
- Reusing previous version of hashicorp/external from the dependency lock file
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Reusing previous version of hashicorp/random from the dependency lock file
- Using previously-installed hashicorp/external v2.2.0
- Using previously-installed hashicorp/azurerm v2.93.1
- Using previously-installed hashicorp/random v3.1.0

Terraform has been successfully initialized!
```

## Manage Providers

- `terraform providers` - show which providers are required for the configuration.
- `terraform version` - current version of terraform cli, it will also show plugin versions after `terraform init` has run.
- `terraform providers lock` - gathers information regarding providers and inserts it into `.terraform.lock.hcl`.
- You should create `.terraform.lock.hcl` for all plaforms that you intend to execute code from: `terraform providers lock -platform=windows_amd64 -platform=darwin_amd64 -platform=linux_amd64`.
- `terraform providers mirror .\mirror1\`  - downloads the providers required for the current configuration and copies them into a directory.
  - For multiple platforms: `terraform providers mirror -platform=windows_amd64 -platform=darwin_amd64 -platform=linux_amd64 .\mirror2multiplatform\`.
- `terraform providers schema -json | jq`  - print detailed schemas, this can be used to understand what resources and arguments may be used with the provider.
