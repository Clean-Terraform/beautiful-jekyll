---
layout: post
title: 'Renaming Terraform Resources and Modules'
date: 2020-04-11 20:10 -0700
categories: how-to
---

Assume you read [Names in Terraform](/style/2020/04/11/resource-names/) and now you need to change the names of resources or modules in your Terraform configuration, how do you do it without breaking anything?

If Terraform has not already been applied then the change is easy, do a search and replace. However, if you have already applied Terraform and you don't want to delete any infrastructure you need to change the state.

Let's assume you have the Terraform configuration below.
{% highlight hcl %}
resource "aws_s3_bucket" "albLogging" {
  ...
}

module "api" {
  ...
}
{% endhighlight %}

And you update it to this.
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
{% highlight bash %}
terraform state mv aws_s3_bucket.albLogging aws_s3_bucket.alb_logging
terraform state mv module.api module.api_rds_cluster
{% endhighlight %}

Now when you run `terraform plan` again it should show the same output as it did before renaming.
