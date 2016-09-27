---
layout: post
title:  "Running terraform fmt as pre-commit hook"
date:   2016-09-27 13:22:38
categories: terraform,devops
---

This is my second blog post related to Terraform. Previous one was about [how I structure Terraform configuration], which got some interesting feedback on [reddit].

This time I want to share [pre-commit] hook which runs `terraform fmt` on your Terraform configuration files. Nothing more, nothing less.

Code is here - [antonbabenko/pre-commit-terraform]


[pre-commit]:                              http://pre-commit.com/
[antonbabenko/pre-commit-terraform]:       https://github.com/antonbabenko/pre-commit-terraform
[how I structure Terraform configuration]: http://www.antonbabenko.com/2016/09/21/how-i-structure-terraform-configurations.html
[reddit]:                                  https://www.reddit.com/r/devops/comments/53sijz/how_do_you_structure_terraform_configurations/