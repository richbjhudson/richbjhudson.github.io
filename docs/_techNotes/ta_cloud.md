---
layout: techNote
category: Terraform
title: Cloud
---
## Overview

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

## Version Control Workflow

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

- You can use a [.auto.tfvars](https://www.terraform.io/cloud-docs/workspaces/variables/managing-variables#loading-variables-from-files) file in your the repository so that there is no need to define terraform variables in the workspace.

## Workspace - General Settings

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

## Private Module Registry

- They allow you to share modules across your organisation.
	- Module versioning
	- Configuration designer
	- Works similar to public registry
- Module naming convention - **terraform-PROVIDER-MODULE_NAME**

### Steps

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
	- The API token is stored in `credentials.tfrc.json` as plain text.

## CLI Driven Workflow

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
	organization = "richbjhudson" 
	workspaces 
		{ name = "cli-driven-azure-demo" }
	} 
```

- Make sure you add the environment variables to your workspace so that TFC can connect to your Azure subscription.

- `terraform plan` - this gets added to TFC runs tab you have to click on link from your local machine.
- `terraform apply` - this gets recorded in TFC.
- `terraform destroy`  - this gets recorded in TFC.

## Migrate State

- You may create your terraform configuration and state locally and then decide you wish to migrate the state to terraform cloud.
- This can be achieved by configuring a `backend`/ `cloud` block within a `terraform` block.

```
cloud { 
        organization = "richbjhudson" 
        workspaces { 
          name = "terraformCloudWorkspaceName"
          }
        } 
```
 - You must login to terraform cloud using `terraform login`.
	- This places a credentials file locally on your machine so that you do not need to perform this step moving forward - it can be found here: `C:\Users\username\AppData\Roaming\terraform.d\credentials.tfrc.json`.
- `terraform init` will result in a prompt to ask if you would like to migrate the local state to a terraform cloud workspace:

```
Do you wish to proceed?
  As part of migrating to Terraform Cloud, Terraform can optionally copy your current workspace state to the configured Terraform Cloud workspace.

  Answer "yes" to copy the latest state snapshot to the configured Terraform Cloud workspace.

  Answer "no" to ignore the existing state and just activate the configured Terraform Cloud workspace with its existing state, if any.

  Should Terraform migrate your existing state?

  Enter a value: yes

```
*Note: After this step it creates the workspace if it did not already exist. The workspace is built to use the terraform cli version that you migrated from.*

- At this point you need to create the environment variables in the workspace.

## Sentinel Policies

- **Policy as-code** - the policy is evaluated after `terraform plan` but before `terraform apply`.
- Enforcement modes: 
	- advisory 
	- soft mandatory - option to *override & continue*.
	- hard mandatory 

*Note: Mandatory policy checks will not pass onto the next stage if they are not compliant. The stage of the check is based on the functions e.g. tfconfig, tfrun, tfplan etc…* 
- Center for internet security policies are provided as foundation policies.
- TFC needs to be setup with the team and governance plan.
- [Terraform Governance Guides](https://github.com/hashicorp/terraform-guides/tree/master/governance) are a standard set of policies created by Hashicorp that may be used for common use cases.

- Sentinel policy language allows you to define policy rules for tfplan, tfconfig, tfstate or tfrun.
	- It includes common functions.
	- There are some Cloud agnostic policies e.g. based around spend.
	- `.sentinel` files include policy configuration that reference the functions that stipulate when the policy should be applied e.g. cost limit policy at tfrun.
	- `Sentinel.hcl` - import modules (functions) and reference policies by source and enforcement_level.

*Note: You may install the HashiCorp sentinel extension in vscode to make it easier to understand the syntax.*

### Steps
- Create new terraform cli workspace
- Update code backend
- Setup environment variables in workspace
- Create new github repo for sentinel policies
- Create a policy set in TFC> ORG Settings> Policy Sets
	- Use VCS connection and select repository in GitHub
	- Select workspaces that should use the policy set
	- Then connect policy set
	
- [Sentinel foundational policies] (https://github.com/hashicorp/terraform-foundational-policies-library) are based on CIS control. You may reference policies directly within the `sentinel.hcl` file.
	- Example:
	
```
policy "azure-cis-6.4-networking-enforce-network-watcher-flow-log-retention-period" {
  source = "https://raw.githubusercontent.com/hashicorp/terraform-foundational-policies-library/master/cis/azure/networking/azure-cis-6.4-networking-enforce-network-watcher-flow-log-retention-period/azure-cis-6.4-networking-enforce-network-watcher-flow-log-retention-period.sentinel"
  enforcement_level = "advisory"
}

```