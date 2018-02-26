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

### APT pre-requisites

```
# apt install unzip
```

## Installing Terraform

First find the appropriate package for Linux on [Terraform Download page](https://www.terraform.io/downloads.html).

I am using [https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip](https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip) this version that is the latest available for me.

```bash
wget https://releases.hashicorp.com/terraform/0.11.3/terraform_0.11.3_linux_amd64.zip

```
Unzip it, it is just a binary that we will move to _/usr/local/bin_ :
```bash
root@ubuntu-host:~# unzip terraform_0.11.3_linux_amd64.zip
Archive:  terraform_0.11.3_linux_amd64.zip
  inflating: terraform
```
```bash
root@ubuntu-host:~# chmod +x terraform
root@ubuntu-host:~# mv terraform /usr/local/bin/
```
```bash
root@ubuntu-host:~# terraform
Usage: terraform [--version] [--help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a Terraform working directory
    output             Read an output from a state file
    plan               Generate and show an execution plan
    providers          Prints a tree of the providers used in the configuration
    push               Upload this Terraform module to Atlas to run
    refresh            Update local state file against real resources
    show               Inspect Terraform state or plan
    taint              Manually mark a resource for recreation
    untaint            Manually unmark a resource as tainted
    validate           Validates the Terraform files
    version            Prints the Terraform version
    workspace          Workspace management

All other commands:
    debug              Debug output management (experimental)
    force-unlock       Manually unlock the terraform state
    state              Advanced state management
```

## Installing Terraform libvirt Provider

## Creating a KVM guest using Terraform
