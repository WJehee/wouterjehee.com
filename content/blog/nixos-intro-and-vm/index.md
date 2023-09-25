+++
title = "Niks to Nix, part 1 | Introduction and VM setup"
template = "page.html"
date = 2023-09-17
weight = 1
[taxonomies]
series=["Niks to Nix"]
+++

Niks is the dutch word for "nothing", which is also where "nix" comes from.

I messed up, changed from arch to NixOS, but forgot to copy my ssh keys / add new ones to my authorized keys on the server.
But it's fine, i wanted to try NixOS on my server anyway, so this is the perfect excuse.

I used to run Dovecot + Postfix for email but I recently came across stalwart labs mail server and wanted to try it out since it seemed very simple to use.

## Goals

- Setup a simple secure server, firewall, ssh keys, etc.
- Hosting my website
- Radicale for calendar / todos
- Mail server (stalwart-mail)
- Additional side project servers

## Future / Optional

- Git server
- Matrix server
- Firefox sync
- Rainloop

## Virtual machine setup

I first setup everything on a virtual machine, so that later I could easily port it to my actual server.
I use virt-manager for this purpose.

In order to enable UEFI in virt-manager, select "Customize configuration before install", see screenshot

![](./vm-customize.webp)

