---
layout: post
title:  "Names in Terraform"
date:   2020-04-11 19:21:43 -0700
categories: style
---
Choosing a name is difficult. Whether the name is for a function, your first born or a Terraform resource there should be a lot of thought that goes into choosing the right name. While it is easier to rename a Terraform resource than a child it is still better to get it rigth the first time.

## Casing
Use [snake case](https://en.wikipedia.org/wiki/Snake_case). Terraform resource types use snake case (i.e. aws_s3_bucket) and that is the covention which should be followed.
{% highlight hcl %}
# Bad
resource "aws_s3_bucket" "albLogging" {
  ...
}

resource "aws_s3_bucket" "Alb_Logging" {
  ...
}

# Good
resource "aws_s3_bucket" "alb_logging" {
  ...
}
{% endhighlight %}

## When naming resources complete the sentence: Resource X for...
When naming resources it is helpful to think of what you are creating the resource for when giving in a name. For example, Resource AWS S3 Bucket for ALB Logs.

{% highlight hcl %}
# Bad
resource "aws_s3_bucket" "alb_logging_bucket" {
  ...
}

# Good
resource "aws_s3_bucket" "alb_logs" {
  ...
}
{% endhighlight %}

## When naming modules complete the sentence: Module X creates...
Modules are a collection of Terraform resources that are logically grouped together. An example may be a module which creates and RDS Cluster with CloudWatch Alarms and IAM permissions. For more information on modules check out the [docs](https://www.terraform.io/docs/configuration/modules.html).

Since modules are a collection of resources it is helpful to indicate what they create in the name.

{% highlight hcl %}
# Bad
module "api" {
  ...
}

# Good
module "api_rds_cluster" {
  ...
}
{% endhighlight %}

## Don't use abbreviations or shortened words
{% highlight hcl %}
# Bad
resource "aws_iam_role" "ecs_exec" {
  ...
}

# Good
resource "aws_iam_role" "ecs_task_execution" {
  ...
}
{% endhighlight %}


## Use the same name for logically grouped resources
When creating multiple resources that are grouped together giving them all the same name makes it clear to others that they are all related.

{% highlight hcl %}
# Bad
data "aws_iam_policy_document" "policy" {
  ...
}

resource "aws_iam_role" "ecs_exec" {
  ...
}

resource "aws_iam_role_policy" "ecs_exec" {
  policy = data.aws_iam_policy_document.policy.json
  role   = aws_iam_role.exec_exec.id
  ...
}

# Good
data "aws_iam_policy_document" "ecs_task_execution" {
  ...
}

resource "aws_iam_role" "ecs_task_execution" {
  ...
}

resource "aws_iam_role_policy" "ecs_task_execution" {
  policy = data.aws_iam_policy_document.ecs_task_execution.json
  role   = aws_iam_role.ecs_task_execution.id
  ...
}
{% endhighlight %}


## Recap
- [ ] Use snake case
- [ ] When naming resources complete the sentence: Resource X for...
- [ ] When naming modules complete the sentence: Module X creates...
- [ ] Don't use abbreviations or shortened words
- [ ] Use the same name for logically grouped resources


## Picked a bad name?
Don't worry, [Renaming Terraform Resources and Modules](/how-to/2020/04/11/how-to-rename-terraform-resources-and-modules/) is easy!
