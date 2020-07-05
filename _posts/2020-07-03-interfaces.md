---
layout: post
title: Type Interfaces in Terraform
date: 2020-07-03 04:06 +0000
---

> interface [in-ter-feys] noun - a surface regarded as the common boundary of two bodies, spaces, or phases.

Interfaces are common in many programming languages. They enforce a *contractual obligation* between two entities. Terraform does not have interfaces, however, we can implement type interfaces using modules and the [try function](https://www.terraform.io/docs/configuration/functions/try.html). Let's see what this looks like, we are going to create an interface for a label module.


The label module is used to standardize the names and tags of resources. It has the following requirements:
* outputs must have an `id` and `tags` key
* tags must be an object and include keys: `Name`, `Env`, and `Managed By`

We can implement this interface as follows:

```hcl
# label interface module
# modules/generic/label/interface/main.tf

variable "check" {}

locals {
  has_id   = try(var.check.id)
  has_tags = try(var.check.tags)

  tags_have_name = try(var.check.tags.Name)
  tags_have_environment = try(var.check.tags.Environment)
  tags_have_managed_by = try(var.check.tags["Managed By"])
}

output "outputs" {
  value = var.check
}
```

Terraform will generate an error if the input variable `check` does not have the required fields. The local names are chosen to help a user understand why a check failed.

Here is a practical example of using this interface:

```hcl
# module alb

variable "label" {}
variable "vpc" {}

module "i_label" {
  source = "modules/generic/label/interface"
  check  = var.label
}
```

If the module input variable `label` does not implement our interface we will get an error similar to:

```hcl
Error: Error in function call

  on ../interface/main.tf line 8, in locals:
   8:   tags_have_name = try(var.check.tags.Name)
    |----------------
    | var.check is object with 1 attribute "foo"

Call to function "try" failed: no expression succeeded:
- Unsupported attribute (at ../interface/main.tf:8,33-38)
  This object does not have an attribute named "tags".

At least one expression must produce a successful result.
```

The check failed because the input only had one key `foo` and no `tags` key as required.

We can also use the interface to check the outputs of a module or remote state:

```hcl
# module s3 label

locals {
  outputs = {
    id = "foo"
    tags = {
      Name = "bar"
      Environment = "baz"
      "Managed By" = "terraform"
    }
  }
}

module "i_label" {
  source = "modules/generic/label/interface"
  check  = local.outputs
}

outputs "outputs" {
  value = local.outputs
}
```


The name of the interface module is prefixed with `i_` to indicate that it is an interface. This is similar to naming conventions in other programming languages where an interface begins with `I`.

There are a few benefits to implementing interfaces in this way instead of using variable [type constraints](https://www.terraform.io/docs/configuration/types.html):
* The contract is defined in one place. By contrast, if you use type constraints, you have to define the contract on every module.
* You can't use type constraints on outputs.

Using this concept of an interface, we can ensure our modules fit together perfectly like puzzle pieces.
