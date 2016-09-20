---
layout: post
title:  "How I structure Terraform configurations"
date:   2016-09-20 22:22:38
categories: terraform,aws,devops
---

In this blog post I want to describe one of the ways I like to structure
Terraform configurations which can be used to manage medium and large
infrastructures with resources span multiple AWS accounts (for eg,
development, staging, production) and multiple regions.

Here I follow so called "layered infrastructure principle".

Infrastructure review
---------------------

Resources are grouped into layers, where each layer consist only of:

  * global resources (IAM, Cloudfront, Cloudtrail, Config, Route53 zones)
  * or regional resources (EC2, VPC, Codedeploy, Cloudformation)

Developer can interact with a single layer, which is identifiable by combination of arguments:

  * [AWS account alias] (`anton-private`, for example)
  * Region name (`eu-west-1`, for example) 
  * Layer name (`global`, `shared`, `application`, for example)

Directory structure
-------------------

```
.
├── accounts
│   ├── company-dev
│   │   ├── application.eu-west-1.tfvars
│   │   ├── global.tfvars
│   │   └── shared.eu-west-1.tfvars
│   ├── company-cicd
│   │   ├── cicd.eu-west-1.tfvars
│   │   └── global.tfvars
│   ├── company-legacy
│   │   └── legacy.tfvars
│   ├── company-prod
│   │   ├── application.eu-west-1.tfvars
│   │   ├── global.tfvars
│   │   └── shared.eu-west-1.tfvars
│   └── company-staging
│       ├── application.eu-west-1.tfvars
│       ├── global.tfvars
│       └── shared.eu-west-1.tfvars
├── layers
│   ├── company-dev
│   │   ├── application
│   │   │   ├── main.tf
│   │   │   ├── output.tf
│   │   │   └── variables.tf
│   │   ├── global
│   │   │   ├── main.tf
│   │   │   ├── output.tf
│   │   │   └── variables.tf
│   │   └── shared
│   │       ├── main.tf
│   │       ├── output.tf
│   │       └── variables.tf
│   ├── company-infra
│   │   ├── cicd
│   │   │   ├── iam.tf
│   │   │   ├── main.tf
│   │   │   ├── output.tf
│   │   │   └── variables.tf
│   │   └── global -> ../../company-dev/global
│   ├── company-static
│   │   └── static
│   │       ├── main.tf
│   │       ├── output.tf
│   │       └── variables.tf
│   ├── company-prod
│   │   ├── application -> ../../company-staging/application
│   │   ├── global -> ../../company-staging/global
│   │   └── shared -> ../../company-staging/shared
│   └── company-staging
│       ├── application
│       │   ├── main.tf
│       │   ├── output.tf
│       │   └── variables.tf
│       ├── global
│       │   ├── main.tf
│       │   ├── output.tf
│       │   └── variables.tf
│       └── shared
│           ├── main.tf
│           ├── output.tf
│           └── variables.tf
├── modules
│   ├── application
│   │   ├── codedeploy.tf
│   │   ├── output.tf
│   │   └── variables.tf
│   ├── global
│   │   ├── cloudtrail.tf
│   │   ├── iam.tf
│   │   ├── main.tf
│   │   ├── output.tf
│   │   └── variables.tf
│   ├── shared
│   │   ├── bastion.tf
│   │   ├── networking.tf
│   │   ├── output.tf
│   │   ├── route53.tf
│   │   ├── variables.tf
│   │   └── vpc_peering.tf
│   └── typical_service
│       ├── codedeploy.tf
│       ├── output.tf
│       ├── service.tf
│       └── variables.tf
└── terraform.sh
```

Notes:

  * `accounts/{aws_alias}/*.tfvars` - values
  * `layers/{aws_alias}/{layer_name}/*.tf` - Terraform configurations. Normally it contains `module` invocations and `terraform_remote_state` configurations to higher layers (global and shared).
  * `layers/company-prod/*` - is a symlink to `layers/company-staging/*` to reduce chance of configuration drift
  * `modules` - Terraform modules (typically invoked from multiple AWS accounts with various values)
  * `terraform.sh` - Terraform (or [Terragrunt]) wrapper script, which set correct working directory and load correct `tfvars` file

Pros
----

  * Easy to navigate infrastructure as code
  * Outputs of higher level layers are available as read-only values to layers below
  * Full set of Terraform features (commands and arguments) is supported
  * Versioned layers based on same `tfvars` can be implemented (A/B testing, blue-green deployment)
  * Very easy to integrate with CI pipeline

Cons
----

  * May be hard to limit permissions for Terraform users between state files inside one AWS account (all states files are readable by users in that account, by default)
  * Locking is not implemented (yet). [Terragrunt] will fix this.

Plan ahead
----------

  * Integrate [Terragrunt]
  * Simplify `terraform.sh` to not require developer to run `init` every time layer is changed
  * Release sample code


[AWS account alias]:  http://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html
[Terragrunt]:         https://github.com/gruntwork-io/terragrunt
