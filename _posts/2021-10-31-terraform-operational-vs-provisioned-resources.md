---
layout: post
title: "TF: Operational vs Provisioned Resources"
description: getting started with Terraform correctly
summary: getting started with Terraform correctly 
comments: true
tags: [aws, google-cloud, terraform, cloudformation, devops]
---

"Hello, World!" documentations are notorious for their deceptive marketing. Terraform is no exception[^1] as its _getting started_ guide hides the complexity of setting up, managing, and sharing [terraform state](https://www.terraform.io/language/state) in a secure and collaborative managing manner with team members. 


It took a while but I have finally figured out that it is best to separate the creation of the backend infrastructure needed to run terraform at scale successfully - henceforth referred to as "operational resources" - from the infrastructure provisioned and managed by Terraform. 


When working with AWS[^2], I use [Cloudformation](https://aws.amazon.com/cloudformation/) to create Terraform's operational resources. Additionally, whenever you can, I advocate holding all your Terraform operational resourcs in a separate AWS.



[Here](https://github.com/msuzoagu/cloudformation_template_for_terraform_operational_resources) is a template, heavily influenced by [Chris Kent](https://thirstydeveloper.io/series/tf-skeleton.html) work. This template assumes the existence of a separate AWS account dedicated solely to holding Terraform operational resources but it can be edited to work with a single AWS account




##### Footnotes
[^1]: To be fair, consider [Terraform Cloud](https://www.hashicorp.com/resources/what-is-terraform-cloud) if you don't mind paying for yet another Iac solution 
[^2]: For GoogleCloud, I use [Deployment Manager](https://cloud.google.com/deployment-manager/docs). Template coming soon