---
layout: post
title: 'Renaming Terraform Resources and Modules'
date: 2020-04-11 20:10 -0700
categories: guides
---
How do you rename a resource or module in Terrfaorm?

If Terraform has not already been applied then the change is easy, do a search and replace of the old resource name with the new one.

However, if the Terraform configuration has already been applied and you can't delete any infrastructure then you need to update the Terraform state.

### Initial Configuration
{% highlight hcl %}
resource "aws_s3_bucket" "albLogging" {
  ...
}

{% endhighlight %}

### New Configuration
{% highlight hcl %}
resource "aws_s3_bucket" "alb_logs" {
  ...
}
{% endhighlight %}

The result of running `terraform plan` after making this change is:

```
------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create
  - destroy

Terraform will perform the following actions:

  # aws_s3_bucket.albLogging will be destroyed
  - resource "aws_s3_bucket" "albLogging" {
      ...
    }

  # aws_s3_bucket.alb_logs will be created
  + resource "aws_s3_bucket" "alb_logs" {
      ...
    }

Plan: 1 to add, 0 to change, 1 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.

```

Running apply is a destructive behavior which would result in losing all items in the S3 bucket. To address this issue we can update the Terraform state file with the name change as well.  To make the changes we need to modify the state file using `terraform mv`.
``` shell
terraform state mv aws_s3_bucket.albLogging aws_s3_bucket.alb_logging
terraform state mv module.api module.api_rds_cluster
```

Now when you run `terraform plan` the output should be no changes.

```
aws_s3_bucket.alb_logs: Refreshing state... [id=alb-logging20200413044401506100000001]

------------------------------------------------------------------------

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```
