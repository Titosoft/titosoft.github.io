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
# apt install unzip git libvirt-dev
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

Verifying the Installation 

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

Next step, install libvirt provider!

## Installing Terraform libvirt Provider

The libvirt provide will require:
- libvirt 1.2.14 or newer
- latest golang version
- mkisofs is required to use the CloudInit feature.

### Installing golang 1.9

To get the latest version of golang we are going to use a ppa:

```bash
sudo add-apt-repository ppa:gophers/archive
```

Update the repositories:

```
apt-get update
```

Remove an old version of golang:

```bash
apt remove golang
apt autoremove
```

```bash
apt-get install golang-1.9-go
```

Add golang 1.9 to your PATH, create a file on _/etc/profile.d_: 

```
vim /etc/profile.d/golang19.sh
```

Copy this content:

```bash
#!/bin/bash

if [ -d "/usr/lib/go-1.9/bin" ] ; then
   export PATH="$PATH:/usr/lib/go-1.9/bin"
fi
```

Set your GOPATH, add the content above to your _.bashrc_:

```
export GOPATH=$HOME/go
```

Logon with your user again (I am using root) and test the installation:

```
root@ubuntu-host:~# go version
go version go1.9.4 linux/amd64
```

### Building libvirt provider

Use "go get" to download the source from github:

```
root@ubuntu-host:~# go get github.com/dmacvicar/terraform-provider-libvirt
root@ubuntu-host:~# go install github.com/dmacvicar/terraform-provider-libvirt
```

You will now find the binary at $GOPATH/bin/terraform-provider-libvirt

## Moving the libvirt provider to terraform.d

There is a directory called "_terraform.d_" in your $HOME. In my example it is located in _/root/.terraform.d_

If .terraform.d is not present execute a command and terraform will create it for you, example:

```
root@ubuntu-host:~# terraform init
Terraform initialized in an empty directory!

The directory has no Terraform configuration files. You may begin working
with Terraform immediately by creating Terraform configuration files.
root@ubuntu-host:~# cd .terraform.d/
root@ubuntu-host:~/.terraform.d# ls
checkpoint_signature
```

We are going to create a folder called "_plugins_" there:

```bash
root@ubuntu-host:~/.terraform.d# mkdir plugins
root@ubuntu-host:~/.terraform.d# cd plugins/
root@ubuntu-host:~/.terraform.d/plugins# pwd
/root/.terraform.d/plugins
```

Copy our plugin binary to this new directory:

```
root@ubuntu-host:~/.terraform.d/plugins# cp ~/go/bin/terraform-provider-libvirt .
root@ubuntu-host:~/.terraform.d/plugins# ls -alh
total 31M
drwxr-xr-x 2 root root 4.0K Feb 26 13:09 .
drwxr-xr-x 3 root root 4.0K Feb 26 13:09 ..
-rwxr-xr-x 1 root root  31M Feb 26 13:09 terraform-provider-libvirt
```

We should now be able to create a environment on KVM using Terraform, check the next session.

## Creating a KVM guest using Terraform

With Terraform installed, let's dive right into it and start creating some infrastructure. Create a new directory called "terraform", we will use this directory to store the configuration file of our project.

```bash
root@ubuntu-host:~/terraform# mkdir terraform
root@ubuntu-host:~/terraform# pwd
/root/terraform
```

We'll build infrastructure on KVM. Our configuration file will create a NAT network, a new volume and install Ubuntu 16.04 using a cloud image from Canonical servers.

The format of the configuration files is [documented here](https://www.terraform.io/docs/configuration/index.html). You can check the entire configuration file above:

```
# instance the provider
provider "libvirt" {
  uri = "qemu:///system"
}

# We fetch the latest ubuntu release image from their mirrors
resource "libvirt_volume" "ubuntu-qcow2" {
  name = "ubuntu-qcow2"
  pool = "images" #CHANGE_ME
  source = "https://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.img"
  format = "qcow2"
}

# Create a network for our VMs
resource "libvirt_network" "vm_network" {
   name = "vm_network"
   addresses = ["10.0.1.0/24"]
}

# Use CloudInit to add our ssh-key to the instance
resource "libvirt_cloudinit" "commoninit" {
          name           = "commoninit.iso"
  pool = "images"
          ssh_authorized_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQA[...]" #CHANGE_ME
        }


# Create the machine
resource "libvirt_domain" "domain-ubuntu" {
  name = "ubuntu-terraform"
  memory = "512"
  vcpu = 1

  cloudinit = "${libvirt_cloudinit.commoninit.id}"

  network_interface {
    hostname = "master"
    network_name = "vm_network"
  }

  # IMPORTANT
  # Ubuntu can hang is a isa-serial is not present at boot time.
  # If you find your CPU 100% and never is available this is why
  console {
    type        = "pty"
    target_port = "0"
    target_type = "serial"
  }

  console {
      type        = "pty"
      target_type = "virtio"
      target_port = "1"
  }

  disk {
       volume_id = "${libvirt_volume.ubuntu-qcow2.id}"
  }
  graphics {
    type = "spice"
    listen_type = "address"
    autoport = "true"
  }
}

# Print the Boxes IP
# Note: you can use `virsh domifaddr <vm_name> <interface>` to get the ip later
output "ip" {
  value = "${libvirt_domain.domain-ubuntu.network_interface.0.addresses.0}"
}
```

The _provider_ block is used to configure the named provider, in our case "libvirt".

The _resource_ block defines a resource that exists within the infrastructure. We have defined:

- _libvirt_volume_ that is a qcow2 disk that will be created inside our storage pool called "_images_" (**Note**: KVM creates a storage pool called "_default_" during the installation, this example uses "_images_" as a storage pool, change to your storage pool accordingly.)
- _libvirt_network_ will create a NAT network called "_vm_network_" using network "10.0.1.0/24" for DHCP.
- _libvirt_domain_ defines our guest "ubuntu-terraform" with 512MB of RAM, 1 vcpu, with a network interface and our qcow disk created on "_libvirt_volume_" resource.
