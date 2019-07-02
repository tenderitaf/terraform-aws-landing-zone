# landing_zone
[AWS Landing Zone](https://aws.amazon.com/solutions/aws-landing-zone/) is
a solution that helps customers more quickly set up a secure, multi-account
AWS environment based on AWS best practices. This repository contains one
terraform module that dynamically deploys components of AWS Landing Zone
solution based on input list of `.tfvars` files.

Quick Links: [How Does This Module Work](#how-does-this-module-work) | [What Components Are Available](#what-components-are-available) | [Why to Use This Solution](#why-to-use-this-solution)


## How Does This Module Work
Based on terraform's [standard module structure](https://www.terraform.io/docs/modules/index.html#standard-module-structure)
guidelines, this repository contains the following folders:
* root folder - module's standard terraform configuration
* [components](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components) - yaml-based and terraform compatible configurations
* [examples](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/examples) - different ways to combine components as part of this module
* [modules](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/modules) - standalone, reusable and production-ready modules
* [tests](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/tests) - set of automated tests to use in CI/CD pipelines

This terraform module requires the following dependencies:
* [python](https://www.python.org) - referenced and validated [here](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/modules/landing_zone/scripts/apply.sh#L22)
* [terrahub](https://www.npmjs.com/package/terrahub) - referenced and validated [here](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/modules/landing_zone/scripts/apply.sh#L21)

To get started, simply include `main.tf` into your terraform codebase:
```hcl
module "landing_zone" {
  source     = "TerraHubCorp/landing-zone/aws"
  version    = "0.0.6"
  root_path  = "${path.module}"
  landing_zone_providers  = "${var.landing_zone_providers}"
  landing_zone_components = "${var.landing_zone_components}"
}
```
> NOTE: Make sure to include `variables.tf` and whatever makes sense from `outputs.tf`

To simplify and make it easier to understand, we included default values in `terraform.tfvars`:
```hcl
landing_zone_providers = {
  default = {
    account_id = "123456789012"
    region = "us-east-1"
  }
}
landing_zone_components = {
  landing_zone_vpc = "s3://terraform-aws-landing-zone/mycompany/landing_zone_vpc/config.tfvars"
  [...]
}

```

This means that when you use this terraform module, you will need to:
1. Change `landing_zone_providers` to values that describe your AWS Organization account
    * `default` reflects the default setup corresponding to AWS Organization account; add more providers by extending `landing_zone_providers` map with extra AWS accounts and/or AWS regions
        * `region` reflects the AWS region used to deploy AWS resources; create 2 different providers for the same AWS account, but different AWS regions
        * `account_id` reflects the AWS account used to deploy AWS resources; prevents provisioning AWS resources into wrong AWS account in case of valid AWS credentials
2. Change `landing_zone_components` to values that fit into your AWS Landing Zone use case
    * each key from `landing_zone_components` map represents the name of the component from [this list of components](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components)
    * each value from `landing_zone_components` map represents the path to `.tfvars` file on S3 and/or local disk
        * each `.tfvars` file must use HCL format; DO NOT USE other formats like JSON or YAML

> NOTE: This module can have tens, hundreds or thousands of deployable components, but not all of them should be and will be deployed. At runtime, components that are not part of `landing_zone_components` map variable will be ignored.


## What Components Are Available
AWS Landing Zone solution is defined by the following strategy:
1. [Multi-Account Structure](#multi-account-structure)
    * AWS Organization Account
    * Shared Services Account
    * Log Archive Account
    * Security Account
2. [Account Vending Machine](#account-vending-machine)
3. [User Access and Identity Management](#user-access-and-identity-management)
4. [Monitoring and Notifications](#monitoring-and-notifications)

> NOTE: This terraform module at this point in time covers only Multi-Account Structure.

### Multi-Account Structure
<img align="right" src="https://github.com/TerraHubCorp/terraform-aws-landing-zone/raw/master/docs/aws-landing-zone-architecture.png" alt="AWS Landing Zone Multi-Account Architecture" />

Based on the multi-account architecture, here below are currently available components:
1. [landing_zone_pipeline_s3_bucket](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_pipeline_s3_bucket/.terrahub.yml#L11)
2. [landing_zone_pipeline_artifact_s3_bucket](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_pipeline_artifact_s3_bucket/.terrahub.yml#L11)
3. [landing_zone_code_pipeline](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_code_pipeline/.terrahub.yml#L34)
4. [landing_zone_code_pipeline_role](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_code_pipeline_role/.terrahub.yml#L11)
5. [landing_zone_code_pipeline_role_policy](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_code_pipeline_role_policy/.terrahub.yml#L41)
6. [landing_zone_code_build](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_code_build/.terrahub.yml#L24)
7. [landing_zone_code_build_role](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_code_build_role/.terrahub.yml#L11)
8. [landing_zone_code_build_role_policy](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_code_build_role_policy/.terrahub.yml#L31)
9. [landing_zone_organization](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_organization/.terrahub.yml#L11)
10. [landing_zone_organization_policy](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_organization_policy/.terrahub.yml#L11)
11. [landing_zone_organization_policy_attachment](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_organization_policy_attachment/.terrahub.yml#L21)
12. [landing_zone_organization_accounts](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_organization_accounts/.terrahub.yml#L19)
13. [landing_zone_organization_unit](https://github.com/TerraHubCorp/terraform-aws-landing-zone/tree/master/components/landing_zone_organization_unit/.terrahub.yml#L19)

<br clear="right" />

### Account Vending Machine
<img align="right" src="https://github.com/TerraHubCorp/terraform-aws-landing-zone/raw/master/docs/aws-landing-zone-account-vending-machine.png" alt="AWS Landing Zone Account Vending Machine Architecture" />

Based on the account vending machine architecture, here below are currently available components:
1. Coming soon ...

<br clear="right" />

### User Access and Identity Management
<img align="right" src="https://github.com/TerraHubCorp/terraform-aws-landing-zone/raw/master/docs/aws-landing-zone-user-access.png" alt="AWS Landing Zone User Access and Identity Management Architecture" />

Based on the user access architecture, here below are currently available components:
1. Coming soon ...

> NOTE: This solution is relying on cross-account role called `OrganizationAccountAccessRole`. Follow [this link](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_create-cross-account-role) to learn more and/or create missing IAM role(s)...

<br clear="right" />

### Monitoring and Notifications
<img align="right" src="https://github.com/TerraHubCorp/terraform-aws-landing-zone/raw/master/docs/aws-landing-zone-notifications.png" alt="AWS Landing Zone Monitoring and Notifications Architecture" />

Based on the notifications architecture, here below are currently available components:
1. Coming soon ...

<br clear="right" />


## Why to Use This Solution

### No need for code changes
Terraform Module for AWS Landing Zone solution is up to 10 lines of code that receives a list of `.tfvars` files as input variables which describe providers (to be read: AWS accounts and AWS regions) and configs (to be read: AWS resources)

### No need for code rewrites
This implementation engages microservices architecture, allowing any component to be replaced with another component (or multiple components)

### No need for hard-coded values
Existing AWS resources created by your team can be reused programmatically as read only values by other teams' terraform configurations

### No need to rebuild from scratch
Existing AWS resources in your current AWS account(s) can be imported and reused without downtime by this terraform module via `terraform import` command

### No need to exclude pieces of account(s) baseline
Some customers were avoiding in the past AWS Landing Zone because it doesn't support some kind of 3rd party SSO solution or 3rd party Logging solution. By using terraform, we can easily bring those solutions into AWS Landing Zone as a set of components and empower customers to continue using best practices of both worlds

### Additionally, this module helps enforce best practices
- By removing the need for access to AWS root account(s)
- By using IAM cross-account roles and/or STS temporary credentials
- By enabling centralized CloudTrail logs and cross-region replication of CloudTrail logs
- By empowering complex organizations to separate roles and responsibilities (e.g. InfoSec team can place explicit deny on IAM, VPC, SG and STS for other teams and/or other environments like production or pre-production)