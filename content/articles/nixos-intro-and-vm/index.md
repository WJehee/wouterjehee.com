+++
title = "Niks to Nix, part 0 | Introduction and VM setup"
template = "page.html"
date = 2023-09-28
[taxonomies]
series=["Niks to Nix"]
+++

Okay, I messed up... I forgot to copy my SSH keys / add new ones to my authorized keys on the server while distrohopping.
This resulted in me losing access to my server since I only allow access with SSH keys.
But it's fine, i wanted to try and run NixOS on my server anyway, so this is the perfect excuse to do that.

So in this series of articles I will set up my server again, this time running NixOS.
The name of this series comes from the dutch word "niks" which means "nothing" and since we are starting from scratch I found it appropriate.
I also happen to know that the name Nix also orginated from the same word.

This series will cover at least the following topics, but I also have a list of optional goals that I might get into in the future.

### Goals

- Setup a server with sensible defaults
- Hosting my personal website
- Radicale for calendar / todos (CALDAV)
- Additional side projects

### Optional goals

- Git server
- Matrix server and bridges
- Firefox sync

# Virtual machine setup

Since my server is still running and I use the services on a daily basis, my plan is to first test the configuration in a virtual machine and then deploy it once it is done, in order to minimize downtime.

I will be using virt-manager for running my VM's and I'll be using the [nixos minimal iso image](https://nixos.org/download).

Before we get started with setting up the VM, we will setup a bridge network so that our server gets it's own IP address on the network ([Video explanation](https://www.youtube.com/watch?v=DYpaX4BnNlg&t=537s)).

## Installing

1. Open virt-manager and create a new virtual machine.
2. Select local install media (ISO or CDROM).
3. Select the Nixos minimal ISO image.
4. Choose memory and CPU settings (default).
5. Enable storage and create a disk image (again, I leave the defaults).
6. Check the "Customize configuration before install" checkbox. ![Customize configuration before install](./vm-customize.webp)
7. Under hypervisor details change the firmware to UEFI to enable it. ![Enable UEFI](./enable-uefi.webp)
8. Under the appropriate tab, change the network source. ![Network](./network.webp)
9. Click "begin installation".

Now the VM is ready to go.  

