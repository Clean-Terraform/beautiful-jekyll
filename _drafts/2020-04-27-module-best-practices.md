---
layout: post
title: Modules - Limit the Number of Inputs
date: 2020-04-27 04:06 +0000
---

> The ideal number of arguments for a function is zero (niladic). Next comes one (monadic), followed closely by two (dyadic). Three arguments (triadic) should be avoided where possible. More than three (polyadic) requires very special justification -- and then shouldn't be used anyway.
>
> Arguments are hard. They take a lot of cenceptual power.
>
> Clean Code p. 40

Imagine reviewing a Pull Request and seeing the following function declaration:

```javascript
function ecs_app(
    alb_arn_suffix,
    alb_ingress_authenticated_hosts,
    alb_ingress_authenticated_listener_arns,
    alb_ingress_authenticated_listener_arns_count,
    alb_ingress_authenticated_paths,
    alb_ingress_healthcheck_path,
    alb_ingress_listener_authenticated_priority,
    alb_ingress_listener_unauthenticated_priority,
    alb_ingress_unauthenticated_hosts,
    alb_ingress_unauthenticated_listener_arns,
    alb_ingress_unauthenticated_listener_arns_count,
    alb_ingress_unauthenticated_paths,
    alb_security_group,
    alb_target_group_alarms_3xx_threshold,
    alb_target_group_alarms_4xx_threshold,
    alb_target_group_alarms_5xx_threshold,
    alb_target_group_alarms_alarm_actions,
    alb_target_group_alarms_enabled,
    ...
    100 more parameters
    ...
    webhook_filter_match_equals,
    webhook_target_action
){}
```

I think most developers would agree with the sentiment "This function has wayyyyyyy too many inputs." Yet, I also find it far too common for Terraform modules to have a large number of inputs. I have personally created numerous modules with a long list of inputs.

Terraform modules are comparable to functions - given a set of inputs it returns a result in the form of provisioned infrastructure. So why not apply the same coding principles you would for another language? 

When you are creating terraform modules chances are someone other than you is going to be using them. When creating modules be keep the user of the module in mind.

You need to create the infrastructure for a new web app on ECS. You search for a pre-existing module within your organization and you find one. Great. You click on the read me and you see the following list of inputs to the module:

Here is the list of inputs for an open source Terraform module which creates a web app running on ECS:

How easy is it for you to use this module? I would need to spend quite a bit of time trying to understand all of the different options and how to use this module to suite my needs.

Make modules easy to use by limiting the number of inputs. Strive to keep them under 10.

No one would write a function that has over 100 inputs however it seems to be common practice with Terraform.

### Group like inputs

### Be opinionated
I find that it is easy with Terraform to over generalize modules to handle more possible use cases than what you need. The result is a module with more inputs to enable/disable features or configuration options. As the number of inputs goes up the module becomes harder to use. Be opinionated. What is the point of making a module if every option is a variable? At that point you might as well give someone a list of the resources they need to use to accomplish their task rather than a module.

Look at this example from a resource inside of a module:

```
resource "aws_lb_target_group" "default" {
  count = var.enabled && var.default_target_group_enabled ? 1 : 0

  name        = module.default_label.id
  port        = var.port
  protocol    = var.protocol
  slow_start  = var.slow_start
  tags        = var.tags
  target_type = var.target_type
  vpc_id      = var.vpc_id

  deregistration_delay = var.deregistration_delay

  stickiness {
    type            = var.stickiness_type
    cookie_duration = var.stickiness_cookie_duration
    enabled         = var.stickiness_enabled
  }

  health_check {
    enabled             = var.health_check_enabled
    path                = var.health_check_path
    port                = coalesce(var.health_check_port, var.port)
    protocol            = coalesce(var.health_check_protocol, var.protocol)
    timeout             = var.health_check_timeout
    healthy_threshold   = var.health_check_healthy_threshold
    unhealthy_threshold = var.health_check_unhealthy_threshold
    interval            = var.health_check_interval
    matcher             = var.health_check_matcher
  }
}
```





### Autogenerate Documentation
Use a tool such as [Terraform Docs]() or [Terraform Config Inspect]() to autogenerate documentation for your Terraform. I like to put the documentation into a file named TF_DOCS.md rather than in the README. In this way I can completly overwrite the file each time and not worry about losing content in the readme.

### Include Examples

### Maintain backward compatibility
As much as possible, when 
