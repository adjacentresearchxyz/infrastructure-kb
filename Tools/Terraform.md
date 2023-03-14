---
title: Terraform
layout: home
parent: Tools and Resources
---

# Terraform

### Overview

Terraform is a an infrastructure as code solution that allows you to provision, change, and version and resource in any cloud environment.

Some common use cases for terraform are 

- Writing infrastructure as code to automate the provisioning of your services.
- Usage for a multi cloud deployment
- Managing a kubernetes setup
- Easily manage network infrastructure

Terraform allows small teams to work fast and scale quickly. Having all of your infrastructure managed by a set of reproducible code, you can quickly manage your deployments without the need for dedicated devops or infrastructure engineers. 

<aside>
💡 If you are looking for “infrastructure as code” for bare metal machines you might want to look into [flatcar](https://flatcar-linux.org), [coreos](https://getfedora.org/en/coreos?stream=stable), [pxeboot](https://en.wikipedia.org/wiki/Preboot_Execution_Environment), or [MasS](https://maas.io)

</aside>

### Terraform CLI

The terraform cli is pretty straightforward, and documentation can be found here 

[Basic CLI Features | Terraform by HashiCorp](https://www.terraform.io/cli/commands)

One of the most useful functions of the cli is that if you are connected to a terraform cloud backend you can switch `workspaces`. This allows you to use the same provisioning code for many (even slightly different) projects. Without workspaces you generally would be overwriting current resources. 

A workspace can be configured like so 

```go
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }

  backend "remote" {
    organization = "adajcentresearch"
    hostname = "app.terraform.io"

    workspaces {
      prefix = "adjacent-"
    }
  }
}
```

### Terraform Cloud

Cloud manages all remote builds along with maintaining a compete overview of al infrastructure deployed under the organization. 

[Terraform | HashiCorp Cloud Platform](https://cloud.hashicorp.com/products/terraform)

### Examples

Examples of a terraform deployment with NixOS can be found below

- [https://github.com/adjacentresearchxyz/deployment](https://github.com/adjacentresearchxyz/deployment)
- [https://github.com/adjacentresearchxyz/solana-deployment](https://github.com/adjacentresearchxyz/solana-deployment)