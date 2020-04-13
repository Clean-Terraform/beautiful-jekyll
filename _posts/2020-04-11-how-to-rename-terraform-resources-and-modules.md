---
layout: post
title: 'Renaming Terraform Resources and Modules'
date: 2020-04-11 20:10 -0700
categories: guides
---
How do you rename a resource or module in Terrfaorm?

If Terraform has not already been applied then the change is easy, do a search and replace. However, if you have already applied Terraform and you can't delete any infrastructure then you need to update the Terraform state.

{% highlight hcl %}
resource "aws_s3_bucket" "albLogging" {
  ...
}

module "api" {
  ...
}
{% endhighlight %}

{% highlight hcl %}
# After
resource "aws_s3_bucket" "alb_logging" {
  ...
}

module "api_rds_cluster" {
  ...
}
{% endhighlight %}

When you run `terraform plan` Terraform will show that it will delete all of your existing infrastructure and create it with new names. If the infrastructure is not being used then this is acceptable. However, if this is a production environment you cannot do delete and recreate the infrastructure.

To make the changes we need to modify the state file using `terraform mv`.
```bash
terraform state mv aws_s3_bucket.albLogging aws_s3_bucket.alb_logging
terraform state mv module.api module.api_rds_cluster
```

Now when you run `terraform plan` again it should show the same output as it did before renaming.
