---
layout: techNote
category: Terraform
title: Basics
---
# Workflow

- `terraform init` - initialises working directory by downloading provider plugins and source modules.
- `terraform validate` - checks configuration file syntax.
- `terraform fmt` -- formats configuration files.
- `terraform plan` - creates an execution plan that compares the desired state to the known state.
- `terraform apply` - core command used to create resources according to plan.
- `terraform destroy` - removes resources according to known state.

# Blocks

Fundamental blocks - terraform, provider and resource
Variable blocks - input, output and local
Calling/ referencing - data sources and module

Ctrl+ space for intellisense to work within TF extension

Terraform Block - cli version + provider and version + backend block - hold state in remote location
Only constant value allowed no variables

Provider block - provider configuration in root module
Resource block - create infra objects, resource syntax and behaviour, provisioners - post creation actions

# Providers

# Resource Block Syntax

# State

# Meta-arguments

# Provisioners
