---
layout: post
title: "Terraform for multi-account, multi-region, multi-environment infrastructure in AWS"
author: "Miko"
---

Recently, I was working on tidying up Terraform code for AWS-based
infrastructure. That included refactoring existing stuff and importing
existing resource created manually on the console. 
Whole thing was running on few AWS accounts, two regions (eu-west-1 and
eu-west-2) and served different environments (equivalent to separate VPC
networks).

In this post, I'm going to share a sample code of simplified infrastructure
like that.

## Situation

See below diagram.

![Situation](https://raw.githubusercontent.com/gasiordev/terraform-aws/master/diagram.jpg)

There are two AWS accounts: Production and Test. Test account has Staging and
Dev environments, both running on separate VPC networks. Production account
runs Live environment in addition with special Manage environment, both in
separate VPC's. These are created in Current Region and they are replicated in
Failover Region. Basically, when Current Region gets unavailable, production
can be failed over to Failover Region.

Resources in Live, Live2 and Staging environment are the same (grouped as Apps
on the diagram). In Manage and Manage2 you have same resources as well (grouped
on the diagram as well).

In Apps groups we have EC2 instances, load balancers, an RDS instance and an
S3 bucket. Resources in Failover Region are replicas and hence, RDS and S3
bucket are both replicated.

There are various VPC peerings between networks, as shown on the diagram.

## Code

I'd like our code to be split into Networking, Base and Apps. Base consists
of other than networking necessary resources like some default security groups,
roles, logging configuration etc. Apps contains all the resources necessary for
running the applications, like the mentioned earlier EC2 instances, load
balancer, RDS instance, S3 bucket etc.

Each of the above should then be split into Production, Failover, Staging and
Dev.

Common resources (or groups of resources) should be extracted into modules.
Environments should have the same input variables so they are configured the
same way. Things like AWS account numbers should be global configuration.

RDS and S3 bucket are replicated to a different region and there are VPC
peerings between different accounts. It might be tricky to put that together.

**A prototype of such code can be found on my GitHub at:<br>
[https://github.com/gasiordev/terraform-aws](https://github.com/gasiordev/terraform-aws).**

It's a just a tiny bit and it's one way of how the terraform code could be
structured.
