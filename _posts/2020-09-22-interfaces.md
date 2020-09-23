---
layout: post
title: Type Interfaces in Terraform
date: 2020-09-22 04:06 +0000
---

Interfaces are common in many programming languages. They enforce a *contractual obligation* between two entities. Terraform does not have interfaces, however, version 0.13 just released [custom variable validation](https://www.hashicorp.com/blog/custom-variable-validation-in-terraform-0-13). Using the combination of modules and custom variable validation, it is possible to replicate the behavior of an interface. To see what this looks like we are going to create an interface for a label module.


The label module is used to standardize the names and tags of resources. It has the following requirements:
* outputs must have an `id` and `tags` key
* tags must be an object and include keys: `Name`, `Environment`, and `ManagedBy`

We can implement this interface as follows:

```hcl
# label interface module

variable "check" {
  validation {
    condition = can(var.check.id)
    error_message = "Label must have an `id` key."
  }

  validation {
    condition = can(var.check.tags)
    error_message = "Label must have a `tag` key."
  }

  validation {
    condition = can(var.check.tags.Name)
    error_message = "Label must have a `tag.Name` key."
  }

  validation {
    condition = can(var.check.tags.Environment)
    error_message = "Label must have a `tag.Environment` key."
  }

  validation {
    condition = can(var.check.tags.ManagedBy)
    error_message = "Label must have a `tag.ManagedBy` key."
  }
}
```

Notice that the label interface module creates no resources, the only purpose is to ensure the input variable implements the interface we define.

Here is a practical example of using this interface:

```hcl
# vpc module

variable "label" {
  description = "Label interface."
}

module "label_iface" {
    source = "..."
    check  = var.label
}
```

If the vpc module input variable `label` does not implement the interface we defined, Terraform will generate an error:

```hcl
Error: Invalid value for variable

  on ../../../generic/labels/standard/main.tf line 49, in module "iface_label":
  49:   check = { tags = local.tags }

Label must have an `id` key.

This was checked by the validation rule at
../../../generic/labels/iface/main.tf:3,3-13.
```

You may be wondering, why not just use variable validation?

There are two main reasons to extract variable validation into a module: sharing validations and module integration.

#### Sharing Validation
What happens if you need to use the same variable validation in more than one module? You could copy and paste the validation into each module that uses it, however, this is clearly a violation of the DRY principle. Any changes to the validation logic would require remembering which modules implement the validation and updating it in multiple places.

Moving the validation into a separate module solves this problem and makes it easy to update the validation in the future.

#### Module Integration
Lets take a look at a common example of two modules in Terraform where one depends on another:

```hcl
module "vpc" {
  source     = "git::https://github.com/cloudposse/terraform-aws-vpc.git?ref=master"
  namespace  = "eg"
  stage      = "test"
  name       = "app"
  cidr_block = "10.0.0.0/16"
}

module "dynamic_subnets" {
  source             = "git::https://github.com/cloudposse/terraform-aws-dynamic-subnets.git?ref=master"
  namespace          = "eg"
  stage              = "test"
  name               = "app"
  availability_zones = ["us-west-2a","us-west-2b","us-west-2c"]
  vpc_id             = module.vpc.vpc_id
  igw_id             = module.vpc.igw_id
  cidr_block         = "10.0.0.0/16"
}
```

Notice how the `dynamic_subnets` module uses three values which are common to `vpc`, this is the perfect use case to create an interface to simplify the integration between the two modules.

To make the integration between these modules easier, create a vpc interface:

```hcl
# vpc_iface

variable "check" {

  validation {
    condition = can(var.check.vpc_id)
    error_message = "VPC must have a `vpc_id`."
  }

  validation {
    condition = can(var.check.igw_id)
    error_message = "VPC must have a `igw_id`."
  }
   
  validation {
    condition = can(var.check.cidr_block)
    error_message = "VPC must have a `cidr_block`."
  }
}
```

After creating the interface, implement it in the `vpc` module:

```hcl
### vpc module

locals {
  outputs = {
    vpc_id     = resource.vpc.self.vpc_id
    igw_id     = resource.aws_internet_gateway.self.id
    cidr_block = var.cidr_block 
  }
}

module "vpc_iface" {
  source = "..."
  check = local.outputs
}

output "outputs" {
  value = local.outputs
  description = "VPC interface."
}
```

Now incorporate the vpc interface in the `dynamic_subnets` module:

```hcl
# dynamic subnets module

variable "vpc" {
  description = "VPC interface."
}

module "vpc_iface" {
  source = "..."
  check = var.vpc
}

resource "aws_subnet" "self" {
  vpc_id            = var.vpc.vpc_id
  ...

```

Finally, lets update the integration between the two modules:

```hcl
module "vpc" {
  source     = "git::https://github.com/cloudposse/terraform-aws-vpc.git?ref=master"
  namespace  = "eg"
  stage      = "test"
  name       = "app"
  cidr_block = "10.0.0.0/16"
}

module "dynamic_subnets" {
  source             = "git::https://github.com/cloudposse/terraform-aws-dynamic-subnets.git?ref=master"
  namespace          = "eg"
  stage              = "test"
  name               = "app"
  availability_zones = ["us-west-2a","us-west-2b","us-west-2c"]
  vpc                = module.vpc
}
```

Notice how the integration between the modules becomes cleaner and requires less knowledge about the outputs of the `vpc` module and inputs of the `dynamic_subnets` module.

One point I want to highlight is the ability to ensure a module's output values conform to an interface (see the vpc module implementation above). This functionality is not available using Terraform variable validation since the validation only works on inputs.

Using this concept of an interface, we can ensure our modules fit together perfectly like puzzle pieces.
