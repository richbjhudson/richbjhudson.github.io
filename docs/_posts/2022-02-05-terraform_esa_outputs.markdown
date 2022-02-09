---
layout: post
title:  "Terraform - using caf-enterprise-scale outputs"
date:   2022-02-05 18:59:00 +0000
categories: Terraform
---
# Context
Our Agile squad have been working with terraform module [caf-enterprise-scale](https://registry.terraform.io/modules/Azure/caf-enterprise-scale/azurerm/latest) to deploy an Azure management group structure and a base set of Azure policies. We needed to extend the module functionality to include monitoring. A parent module was created that sourced the caf-enterprise-scale module and a child monitor module.

As part of this process the child monitor module needed to reference resources created by the caf-enterprise-scale module e.g. the caf-enterprise-scale module creates an Azure policy set resource and then the child monitor module assigns the policy set resource.   

# Key Takeway
By converting the [caf-enterprise-scale](https://registry.terraform.io/modules/Azure/caf-enterprise-scale/azurerm/latest?tab=outputs) output into a local map we were able to feed the values into a child module input variable.

# Methodology 
Reviewed the outputs for the terraform module  [caf-enterprise-scale](https://registry.terraform.io/modules/Azure/caf-enterprise-scale/azurerm/latest?tab=outputs).

From here we identified the following output:
```teraform
azurerm_policy_set_definition
```

We then tested what was returned by the module output by adding the following output to the parent module code:
```teraform
output "policy_set_definitions" {
  value = module.enterprise_scale.azurerm_policy_set_definition
}
```

Here is an example of the first object returned by the output:
```teraform
policy_set_definitions = {
  "enterprise_scale" = {
    "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deny-PublicPaaSEndpoints" = {
      "description" = "This policy initiative is a group of policies that prevents creation of Azure PaaS services with exposed public endpoints"
      "display_name" = "Public network access should be disabled for PaaS services"
      "id" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deny-PublicPaaSEndpoints"
      "management_group_id" = "hexdev"
      "management_group_name" = "hexdev"
      "metadata" = "{\"createdBy\":\"26a3202e-2ce0-40d2-b9a2-acfbc8e4a3cd\",\"createdOn\":\"2022-02-02T10:45:20.7787165Z\",\"updatedBy\":null,\"updatedOn\":null}"
      "name" = "Deny-PublicPaaSEndpoints"
      ...
```

This led to modifying the output to return a key value pair mapping for each object `{"object.name", "object.id" }`

```teraform
output "policy_set_definition1" {
  value = { for polset in module.enterprise_scale.azurerm_policy_set_definition["enterprise_scale"] : polset.name => polset.id }
}
```

Here is an example of the the output returned:
```teraform
policy_set_definition1 = {
  "Deny-PublicPaaSEndpoints" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deny-PublicPaaSEndpoints"
  "Deploy-ASCDF-Config" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deploy-ASCDF-Config"
  "Deploy-Diagnostics-LogAnalytics" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deploy-Diagnostics-LogAnalytics"
  "Deploy-Private-DNS-Zones" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deploy-Private-DNS-Zones"
  "Deploy-Sql-Security" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Deploy-Sql-Security"
  "Enforce-EncryptTransit" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Enforce-EncryptTransit"
  "Enforce-Encryption-CMK" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/Enforce-Encryption-CMK"
  "allow_locations" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/allow_locations"
  "deploy_al_compute" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/deploy_al_compute"
  "deploy_al_network" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/deploy_al_network"
  "deploy_al_security" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/deploy_al_security"
  "deploy_al_storage" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/deploy_al_storage"
  "deploy_chgtrack_ext_vm_all" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/deploy_chgtrack_ext_vm_all"
  "deploy_diag_btmon" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/deploy_diag_btmon"
  "inherit_tags_by_service" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/inherit_tags_by_service"
  "set_sub_tags" = "/providers/Microsoft.Management/managementGroups/hexdev/providers/Microsoft.Authorization/policySetDefinitions/set_sub_tags"
}
```
At this point it was understood how to obtain the output from terraform module  [caf-enterprise-scale](https://registry.terraform.io/modules/Azure/caf-enterprise-scale/azurerm/latest?tab=outputs) in a useful readable format.

To use the caf-enterprise-scale output to feed into child monitor module input variables I used a local:
```teraform
locals {
policy_set_definitions = { for polset in module.enterprise_scale.azurerm_policy_set_definition["enterprise_scale"] : polset.name => polset.id } 
 }
```

We set the input variable value for the child module within the parent module as follows:
```teraform
es_policy_set_definitions  = local.policy_set_definitions 
```

The child monitor module input variable looks like this:
```teraform
variable "es_policy_set_definitions" {
  type = map
 }
```

Here is an example of a resource block argument within the child monitor module for resource [azurerm_management_group_policy_assignment](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/management_group_policy_assignment):
```teraform
policy_definition_id = var.es_policy_set_definitions["deploy_al_compute"]
```
<!--Reference links in article-->
[1]: https://www.rawritscloud.com/using-enterprise-scale-outputs