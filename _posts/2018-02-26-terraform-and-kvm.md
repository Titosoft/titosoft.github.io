---
published: false
title: Terraform and KVM (x86)
category: KVM
tags: kvm terraform automation
toc: true
---
[Terraform](terraform.io) is what they call "Infrastructure as a code". It has a different approach from other automation tools like Puppet, Chef or Ansible because it is focused on Cloud Infrastructure.

It supports a bunch of [providers](https://www.terraform.io/docs/providers/index.html) like AWS, Azure, Softlayer... but, as you can see, there is no official support for KVM. I don't want to create a AWS account just to try terraform, so on this article I am going to write step by step how to create a KVM virtual machine using [Terraform libvirt provider](https://github.com/dmacvicar/terraform-provider-libvirt).

## Pre-requisites
- x86 server
- Ubuntu 16.04
- KVM installed and configured
- Some disk space for your guests

## Installing Terraform

First find the appropriate package for Linux on [Terraform Download page](https://www.terraform.io/downloads.html).

I am using [https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip](https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip) this version that is the latest available for me.

```
wget https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip

```

## Installing Terraform libvirt Provider

## Creating a KVM guest using Terraform
