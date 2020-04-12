---
layout: post
title:  "Names in Terraform"
date:   2020-04-11 19:21:43 -0700
categories: style
---
## tl;dr
* Use snake case
* Do NOT include the resource type in a resource name
* Do include a reference to what a module does in a module name
* Do not use abbreviations
* Use the same name for logically grouped resources (i.e. aws_iam_role, aws_iam_role_policy)

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

## Do NOT include the resource type in a resource name
Including a reference to the type of resource in the name is repetitive and does not add any clarity for the user, it just makes them type more.

{% highlight hcl %}
# Bad
resource "aws_s3_bucket" "alb_logging_bucket" {
  ...
}

# Good
resource "aws_s3_bucket" "alb_logging" {
  ...
}
{% endhighlight %}

## Do include the resource type in a module name
Modules are a collection of Terraform resources that are logically grouped together. An example may be a module which creates and RDS Cluster with CloudWatch Alarms and IAM permissions. For more information on modules check out the [docs](https://www.terraform.io/docs/configuration/modules.html).

In the case of modules adding a reference to what a module does adds clarity for other users. When naming a module in Terraform include a reference to what the module is for in the name of the resource as a postfix.

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

## Don't use abbreviations or shortening words
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

## Picked a bad name?
Don't worry, [Renaming Terraform Resources and Modules](/how-to/2020/04/11/how-to-rename-terraform-resources-and-modules/) is easy!
