---
title: Scaffold
description: Learn how to scaffold Terragrunt units.
slug: docs/features/scaffold
sidebar:
  order: 7
---

Terragrunt scaffolding can generate files for you automatically using [boilerplate](https://github.com/gruntwork-io/boilerplate) templates.

Currently, one boilerplate template is supported out-of-the-box, which you can use to generate a best-practices `terragrunt.hcl` that configures an OpenTofu/Terraform module for deployment:

```bash
terragrunt scaffold <MODULE_URL> [TEMPLATE_URL] [--var] [--var-file] [--no-include-root] [--root-file-name] [--no-dependency-prompt]
```

Description:

- `MODULE_URL` - This parameter specifies the URL to an OpenTofu/Terraform module. It can be a local file path, git URL, registry URL, or any other [module source URL](https://developer.hashicorp.com/terraform/language/modules/sources).
- `TEMPLATE_URL` - This optional parameter specifies the URL to a custom boilerplate template to generate HCL files. It can be a local file path, git URL, registry URL, or any other [module source URL](https://developer.hashicorp.com/terraform/language/modules/sources). If not specified, Terragrunt will:
  - Look for a `.boilerplate` folder in the module at `MODULE_URL`, and if found, use the boilerplate template in that folder.
  - Failing to find that, Terragrunt will use a default boilerplate template that is built-in, which creates a best-practices `terragrunt.hcl` for deploying a single OpenTofu/Terraform module.

For example, here's how you can generate a `terragrunt.hcl` file to configure an [example MySQL OpenTofu/Terraform module](https://github.com/gruntwork-io/terragrunt-infrastructure-modules-example/tree/master/mysql) for deployment:

```bash
terragrunt scaffold github.com/gruntwork-io/terragrunt-infrastructure-modules-example//modules/mysql
```

This will create a `terragrunt.hcl` in your current working directory, with roughly the following contents:

```hcl
# terragrunt.hcl

# This is a Terragrunt module generated by boilerplate.
terraform {
  source = "git::https://github.com/gruntwork-io/terragrunt-infrastructure-modules-example.git//modules/mysql?ref=v0.8.1"
}

inputs = {
  # --------------------------------------------------------------------------------------------------------------------
  # Required input variables
  # --------------------------------------------------------------------------------------------------------------------

  # Type: string
  # Description: The AWS region to deploy to (e.g. us-east-1)
  aws_region = "" # TODO: fill in value

  # Type: string
  # Description: The name of the DB
  name = "" # TODO: fill in value

  # Type: string
  # Description: The instance class of the DB (e.g. db.t2.micro)
  instance_class = "" # TODO: fill in value

  # (... full list of inputs omitted for brevity ...)
}
```

Important notes:

- The `source` URL is configured for you automatically, with the `ref` pointing to the latest "release" tag of the module (found by scanning git tags).
- The `inputs` section is generated for you automatically, and will list all required and optional variables from the module, with their types, descriptions, and defaults, so you can easily fill them in to configure the module as you like.

## Custom templates for scaffolding

Terragrunt has a basic template built-in for rendering `terragrunt.hcl` files, but you can provide your own templates to customize what code is generated! Scaffolding is done via [boilerplate](https://github.com/gruntwork-io/boilerplate), and Terragrunt allows you to specify custom boilerplate templates via three mechanisms - listed in order of priority:

1. You can specify a custom boilerplate template to use as the second argument of the `scaffold` command.
2. You can define a custom boilerplate template in a `.boilerplate` subfolder of your module.
3. You can define a default custom boilerplate template in the [catalog config](/docs/features/catalog).

If you define input variables in your boilerplate template, Terragrunt will prompt users for the values. Those values can also be passed in via `--var` and `--var-file` arguments.
There are also a set of variables that Terragrunt will automatically expose to your boilerplate templates for rendering:

- `sourceUrl` - URL to module
- `requiredVariables` - list of required variables in the module being scaffolded (see below)
- `optionalVariables` - list of optional variables in the module being scaffolded (see below)

The elements in the `requiredVariables` and `optionalVariables` lists are structs with the following fields:

- `Name` - variable name
- `Description` - variable description
- `Type` - variable type (string, number, bool, list, map, object) [Type Constants](https://developer.hashicorp.com/packer/docs/templates/hcl_templates/variables#type-constraints)
- `DefaultValue` - variable default value
- `DefaultValuePlaceholder` - default value placeholder (e.g. `""` for a string or `0` for a number)

Optional variables which can be passed to `scaffold` command:

- `Ref` - git tag or branch name for module to be used
- `EnableRootInclude` - add in default `terragrunt.hcl` inclusion for the root module, by default `true`
- `RootFileName` - name of the root configuration file, by default `terragrunt.hcl` \*
- `SourceUrlType` - if set to `git-ssh` module url will be converted to Git/SSH format
- `SourceGitSshUser` - git user for Git/SSH format, by default `git`

\* **NOTE**: `RootFileName` is set to `terragrunt.hcl` by default to ensure backwards compatibility, but the pattern of using a `terragrunt.hcl` file at the root of Terragrunt projects has since been deprecated.

   When the [root-terragrunt-hcl](/docs/reference/strict-controls#root-terragrunt-hcl) strict control is enabled, the default configuration file will change to `root.hcl`, which is considered a better practice. For more details, see [Migrating from root `terragrunt.hcl`](/docs/migrate/migrating-from-root-terragrunt-hcl).

### Convenience flags

- `--no-include-root` - Disable inclusion of the root module in the generated `terragrunt.hcl` file (equivalent to using `--var=EnableRootInclude=false`, and will be overridden if the corresponding `var` value is set).
- `--root-file-name` - Set the name of the root configuration file to include in the generated `terragrunt.hcl` file (equivalent to using `--var=RootFileName=<name>`, and will be overridden if the corresponding `var` value is set).
- `--no-dependency-prompt` - Disable dependency confirmation, but keep the interactive mode enabled (skip asking for confirmation about including dependencies defined in the boilerplate template).

\* **NOTE**: `RootFileName` is set to `terragrunt.hcl` by default to ensure backwards compatibility, but the pattern of using a `terragrunt.hcl` file at the root of Terragrunt projects has since been deprecated.

   See the note above on the [root-terragrunt-hcl](/docs/reference/strict-controls#root-terragrunt-hcl) strict control for more information.

## Examples

Scaffold new project but use specific module version:

```bash
terragrunt scaffold github.com/gruntwork-io/terragrunt.git//test/fixtures/inputs --var=Ref=v0.68.4
```

Scaffold new project but use Git/SSH URLs:

```bash
terragrunt scaffold github.com/gruntwork-io/terragrunt.git//test/fixtures/inputs --var=SourceUrlType=git-ssh

```hcl
# terragrunt.hcl
terraform {
  source = "git::ssh://git@github.com/gruntwork-io/terragrunt.git//test/fixtures/inputs?ref=v0.68.4"
}
```

Scaffold new project using a template from a Git repository:

```bash
terragrunt scaffold github.com/gruntwork-io/terragrunt.git//test/fixtures/scaffold/module-with-template
# The template from the .boilerplate directory will be used to generate terragrunt.hcl
```

**NOTE**: Scaffolding infrastructure from an external repository might introduce security or stability risks. Always review code from trusted external sources before running it.

Scaffold new project using an external template:

```bash
terragrunt scaffold github.com/gruntwork-io/terragrunt.git//test/fixtures/inputs git@github.com:gruntwork-io/terragrunt.git//test/fixtures/scaffold/external-template
# The files external-template.txt and terragrunt.hcl will be created from that external template
```
