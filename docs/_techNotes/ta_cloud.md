---
layout: techNote
category: Terraform
title: Cloud
---
# Overview

- [Terraform Cloud](https://app.terraform.io ) is a cloud version of Terraform Enterprise (on-premise).
- It manages: 
    - terraform runs including cost estimations
	- Shared state
	- Access control
	- Secret data
	- Private registry - publish private modules
	- Policy controls
- Workflow type:
    - Version Control Workflow - check code into GitHub that triggers terraform runs.
    - CLI driven - runs and state are executed and stored in Terraform Cloud.

# Version Control Workflow

- Equivalent to Azure pipeline.
- Runs and state version created for every run.
- You may create a service principal to use to connect to your Azure subscription:

```
Az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/12345686-82de-4a8d-b643-12345605372a"
{
  "appId": "123456c7-dd48-41c0-aa76-52a770123456",
  "displayName": "azure-cli-2022-01-24-20-13-16",
  "name": "a88530c7-dd48-41c0-aa76-52a770fa69fb",
  "password": "123456.Y1j3j_1234562sxONHwM123456.",
  "tenant": "65123456-faea-4b9d-a488-123456123456"
}

```

- You can configure a workspace to use the service principal using environment variables:

```
ARM_CLIENT_ID=123456c7-dd48-41c0-aa76-52a770123456
ARM_CLIENT_SECRET=123456.Y1j3j_1234562sxONHwM123456.
ARM_TENANT_ID=65123456-faea-4b9d-a488-123456123456
ARM_SUBSCRIPTION_ID=12345686-82de-4a8d-b643-12345605372a
```

- You can use a .tfvars file in your the repository so that there is no need to define terraform variables in the workspace.

# Workspace - General Settings

- Execution mode
	- local - use TFC for state storage only
	- remote
- Apply method (context of terraform plan and code repository update)
	- auto apply
	- manual apply
- Terraform version - set cli version
- Remote state sharing - share state between 2 workspaces either globally or specifically.
- Settings
	- Locking - this prevents terraform runs.
	- Notifications - email/ slack or webhook and set either all or specific events.
	- Run triggers - connect workspace to one or more workspaces. You can set the workspace so that you must always manually apply regardless of auto-apply setting.
	- SSH key - use to access modules in git repository.
	- Team access - add permissions to workspace.
	- Destruction and deletion - allow destroy. 
	- Manual destroy - queue destroy and provide workspace name.

# Private Module Registry

- They allow you to share modules across your organisation.
	- Module versioning
	- Configuration designer
	- Works similar to public registry
- Module naming convention - **terraform-PROVIDER-MODULE_NAME**

## Steps

1. Create private repository in GitHub.
2. Upload code including `main.tf`, `versions.tf`, `outputs.tf`, `variables.tf` and `readme.md` files.
3. Create a release for the repository.
4. TFC > registry - publish a module:
	- Connect to a version control provider - github.com
	- You must register a new OAUTH application in Github under settings> developer settings using the settings provided by TFC.
	- Enter client id, client secret from GitHub into the TFC version control provider configuration.
	- Connect and continue to authorise GitHub access to TFC.
5. At this point the VCP is configured and you can use the configuration to select a repository that you wish to publish a module from. 
	
*Note: In a private module it did not pull in the `versions.tf` configuration into the dependency tab.*

- The terraform private registry generates a snippet of how to use the module:

```
module "staticwebsiteprivate" {
  source  = "app.terraform.io/richbjhudson/staticwebsiteprivate/azurerm"
  version = "1.0.0"
  # insert required variables here
}
```

- You need to authenticate to the private registry using `terraform login`.
	- This command prompts you to create an API token with access to the private registry.
	- `credentials.tfrc.json` is stored in plain text.

# CLI Driven Workflow

The CLI driven workflow allows you to:
- Use terraform remote backend 
- Runs execute remotely in TFC
- Use variables in workspace
- Use sentinel policies 
- Private registry

- Here is an example of how to configure the `backend` block that is included in the `terraform` block:

```
backend "remote" {
    organization = "richbjhudson"
    workspaces {
      name = "cli-driven-azure-demo"
    }
  }
```

- Alternative method:

```
cloud { 
organization = "richbjhudson" workspaces 
{ name = "cli-driven-azure-demo" }
 } 
```

- Make sure you add the environment variables to your workspace so that TFC can connect to your azure subscription.

- `terraform plan` - this gets added to TFC runs tab you have to click on link from your local machine.
- `terraform apply` - this gets recorded in TFC.
- `terraform destroy`  - this gets recorded in TFC.

# Migrate State

# Sentinel Policies
