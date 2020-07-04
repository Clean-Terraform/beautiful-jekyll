---
layout: post
title: Interfaces in Terraform
date: 2020-07-03 04:06 +0000
---

> interface [in-ter-feys] noun - a surface regarded as the common boundary of two bodies, spaces, or phases.

Interfaces are common in many programming languages. They enforce a *contractual obligation* between two entities. Terraform does not have interfaces, however, we can implement the concept of an interfaces using modules and the [try function](https://www.terraform.io/docs/configuration/functions/try.html). Our implementation will be slightly different than a traditional interface; the interface will be on module inputs instead of the module itself.


Let's create an interface for a label module. The module is used to standardize the names and tags of resources. It has the following requirements:
* outputs must have an `id` and `tags` key
* the tags must include a `Name`, `Env`, and `Managed By` key

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

outputs "result" {
  value = var.check
}
```

If the input variable `check` does not have the required fields, Terraform will generate an error. The names chosen for the local variables provide information to the user if a check fails. Let's look at an example of using this interface module:

```hcl
# module alb

variable "label" {}
variable "vpc" {}
... other inputs

module "i_label" {
  source = "modules/generic/label/interface"
  check  = var.label
}
```

If the variable `label` does not implement the label interface we will get an error similar to:

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


The name of the interface module is prefixed with `i_` to indicate that it is an interface.

You might be wondering, how does this help you? Can't you just variable [type constraints](https://www.terraform.io/docs/configuration/types.html)?

The benefits of using this concept of an interface include:
* The contract is defined in one place. By contrast, if you use type constraints, you have to define the contract on every module.
* You can't use type constraints on outputs.

Using this concept of an interface, we can ensure our modules fit together perfectly like puzzle pieces.
