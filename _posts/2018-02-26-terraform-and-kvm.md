---
published: false
title: Terraform and KVM (x86)
category: KVM
tags: kvm terraform automation
---
[Terraform](terraform.io) is what they call "Infrastructure as a code". It has a different approach from other automation tools like Puppet, Chef or Ansible because it is focused on Cloud Infrastructure.

It supports a bunch of [providers](https://www.terraform.io/docs/providers/index.html) like AWS, Azure, Softlayer... but, as you can see, there is no official support for KVM. I don't want to create a AWS account just to try terraform, so on this article I am going to write step by step how to create a KVM virtual machine using [Terraform libvirt provider](https://github.com/dmacvicar/terraform-provider-libvirt).





