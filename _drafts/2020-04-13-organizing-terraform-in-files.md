---
layout: post
title: Organizing Terraform in Files
date: 2020-04-13 04:54 +0000
---

A common pattern when writing Terraform configuration is to create files based on what they contain. For example you are probably familiar with a directory structure similar to: 

``` python
├── locals.tf
├── main.tf
├── outputs.tf
├── README.md
├── variables.tf
└── versions.tf
```
For directories which create infrastructure you may even see a file structure like:
``` python
├── dynamo.tf
├── iam.tf
├── locals.tf
├── outputs.tf
├── rds.tf
├── README.md
├── variables.tf
└── versions.tf
```

Most modules within the Terraform module registry use an approach similar to this. And it makes sense. If you are looking for an output you know exactly where to find it. Likewise for variables etc. However, there is a subtle downside to this organization: it makes it harder to extract certain pieces of the Terraform configuration and use it someplace else. 

Let's use the second folder structure as an example. Chances are the files contain something similar to this (you might use `terraform.tfvars` instead of locals to specify the class and count of RDS instances but the same concept applies):

```hcl
# rds.tf
module "rds_cluster" {
  source         = "..."
  instance_class = local.rds_instance_class
  instance_count = local.rdS_instance_count
  ...
}

# locals.tf
locals {
  rds_instance_class = "db.t3.small"
  rds_instance_count = 2

  ...
  # other locals not related to RDS
}

# outputs.tf

output "rds_cluster" {
  value = module.rds_cluster
}

...
# other outputs not related to RDS
```


Now imagine you are creating a new service which needs an RDS Cluster but no DynamoDB table. To do this you would most likely copy `rds.tf` into the new directory. But this isn't all you need. You also have to find the relevent parts of `locals.tf` and `outputs.tf` to include in the new directory as well. While this isn't a big deal, it does take time. And as your Terraform configuration grows you will have to do it more often.

So how can we improve this?

Group everything that is related into one file making it self contained. Here is how `rds.tf` would look in this scenario:

 ```hcl
# rds.tf
locals {
  rds_instance_class = "db.t3.small"
  rds_instance_count = 2
}

module "rds_cluster" {
  source         = "..."
  instance_class = local.rds_instance_class
  instance_count = local.rdS_instance_count
  ...
}

output "rds_cluster" {
  value = module.rds_cluster
}
```

This file could be copied into any directory and used without any extra work.
