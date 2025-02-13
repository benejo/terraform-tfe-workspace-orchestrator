<!-- BEGIN_TF_DOCS -->
# Terraform Cloud Multi-Region Deployment with Workspaces

If you want to deploy a terraform module to multiple locations (different VPCs, Regions, Accounts) this module can help. In most examples we use regions as the location separator but that doesnt have to be the case. The premise is simple, create your root module in a public VCS repo then using your Terraform Cloud (TFC) Organization, create workspaces for each deployment location.

## Usage

To use you must have:

1. Terraform Cloud Organization with Admin Access
1. VCS repo with your HCL root module
1. Connect the repo to TFC (To be automated)

Once the above is complete, simply execute this module with references for each location in a way that TFC can reference.

## Workspaces

Workspaces are defined in a nested map as each deployment location. A workspace key within the `var.workspaces` can utilize _any_ [workspace argument](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/workspace#argument-reference).

It can also accept `var.workspaces.<>.vars` which can accept variable declarations as described [below](#variables).

## Variables

This module allows you to specify variables in 3 different ways:

1. Attach a pre-created [variable set id](https://www.terraform.io/cloud-docs/api-docs/variable-sets) to each workspace
1. Declare variables within `shared_variable_set` as `{key = value}` to be shared to each workspace.
1. Specify on a per-workspace using the nested map structure below

```terraform
module "multi_region_deployment" {
  source = "../.."
  ...
  workspaces = {
    eastcoast = {
      vars = {
        AWS_REGION = {
          value = "us-east-1"
        }
        my_tf_var = {
          value     = "test"
          category  = "terraform"
        }
      }
    }
    westcoast = {...}
  }
```

## Examples

For examples see [here](https://github.com/aws-ia/terraform-tfe-workspace-orchestrator/tree/main/examples)

### Example terraform.tfvars

```terraform
organization            = "<>"

# variable set contains my AWS_ACCESS_KEY_ID & AWS_SECRET_ACCESS_KEY, attach to all workspaces
creds_variable_set_name = "dev_aws_creds"

vcs_repo = {
  identifier     = "drewmullen/aws-infra" # https://github.com/drewmullen/aws-infra
  oauth_token_id = "<oauth token from TFC>"
  branch         = "master"
}

shared_variable_set = {
  "test"  = { value = 123 }
  "test2" = { value = 123 }
  workspace_name = {
    value    = "test"
    category = "terraform"
  }
}
```

## Known Issues

Currently there is no way to wait for any workspace variable sets prior to the initial workspace creation. If the inital `apply` fails you can rekick them off. This will hopefully be resolved in a [future release](https://github.com/hashicorp/terraform-provider-tfe/issues/534)

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.2.2 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 4.0.0, < 5.0.0 |
| <a name="requirement_tfe"></a> [tfe](#requirement\_tfe) | >= 0.33.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_tfe"></a> [tfe](#provider\_tfe) | 0.0.1 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [tfe_variable.per_workspace](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/variable) | resource |
| [tfe_variable.shared_to_all_workspaces](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/variable) | resource |
| [tfe_variable_set.per_workspace](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/variable_set) | resource |
| [tfe_variable_set.shared_to_all_workspaces](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/variable_set) | resource |
| [tfe_workspace.main](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/workspace) | resource |
| [tfe_workspace_variable_set.shared_preexisting_variable_set_ids](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/workspace_variable_set) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_organization"></a> [organization](#input\_organization) | n/a | `string` | n/a | yes |
| <a name="input_workspaces"></a> [workspaces](#input\_workspaces) | Nested map of workspaces to create and the associated arguments they can accept:<br><br>Example:<pre>workspaces = {<br>    eastcoast = {<br>      vars = {<br>        AWS_REGION = {<br>          value = "us-east-1"<br>        }<br>      }<br>    }<br>    westcoast = {...}<br>  }</pre>Arguments accepted within workspace definition:<br><br>- All arguments from [tfe\_workspace](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/resources/workspace#argument-reference). Defaults set as documented in July 2022 (v0.33.0).<br>- `vars` = A nested map of variables, their value and category<pre>vars = {<br>    myvar_name = {<br>      value    = "my var value"<br>      category = "env" # valid values: "env" or "terraform", default = "env"<br>    }<br>  }</pre>Workspace `tag_names` will attempt to combine specific tag\_names and from `var.shared_workspace_tag_names`. | `any` | n/a | yes |
| <a name="input_shared_variable_set"></a> [shared\_variable\_set](#input\_shared\_variable\_set) | A variable set ID to create and set to all workspaces. Use if you want to share variables across all workspaces. To set per-workspace, see `var.workspaces`. | `map(map(string))` | `{}` | no |
| <a name="input_shared_variable_set_id"></a> [shared\_variable\_set\_id](#input\_shared\_variable\_set\_id) | A variable set ID to set to all workspaces. Use if you have a pre-existing variable set. | `string` | `null` | no |
| <a name="input_shared_workspace_tag_names"></a> [shared\_workspace\_tag\_names](#input\_shared\_workspace\_tag\_names) | Tag names to set for all workspaces. To set per-workspace, see `var.workspaces`. | `list(any)` | `[]` | no |
| <a name="input_vcs_repo"></a> [vcs\_repo](#input\_vcs\_repo) | Definition of the VCS repo to attach to every workspace. | <pre>object({<br>    identifier     = string<br>    oauth_token_id = string<br>    branch         = optional(string)<br>  })</pre> | `null` | no |

## Outputs

No outputs.
<!-- END_TF_DOCS -->