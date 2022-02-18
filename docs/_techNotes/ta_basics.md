---
layout: techNote
category: Terraform
title: Basics
---
# Workflow

- `terraform init` - initialises the working directory by downloading provider plugins and source modules.
- `terraform validate` - checks configuration file syntax.
- `terraform fmt` -- formats configuration files.
- `terraform plan` - creates an execution plan that compares the desired state to the known state.
- `terraform apply` - core command used to create resources according to plan.
- `terraform destroy` - removes resources according to known state.

# Blocks

## 3 Fundamental Block Types:
- **Terraform** includes:
    -  terraform cli version, terraform provider and version, and also the backend block that holds the terraform state in a remote location.
    - Only constant values are allowed in this block no variable references.
- **Provider** includes provider configuration in root module.
- **Resource** is used to create infrastructure objects including:
    - resource syntax and behaviour.
    - provisioners - post creation actions.

## Additional Block Types:
- Variable blocks - input, output and local
- Calling/ referencing - data sources and module

# Providers

# Resource Block Syntax

# State

# Meta-arguments

# Provisioners
