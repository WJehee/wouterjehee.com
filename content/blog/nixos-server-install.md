+++
title = "Niks to Nix, part 2 | configuring my server using Nix"
template = "page.html"
date = 2023-09-20
weight = 2
[taxonomies]
series=["Niks to Nix"]
+++

Niks is the dutch word for "nothing", which is also where "nix" comes from.

I messed up, changed from arch to NixOS, but forgot to copy my ssh keys / add new ones to my authorized keys on the server.
But it's fine, i wanted to try NixOS on my server anyway, so this is the perfect excuse.

I used to run Dovecot + Postfix for email but I recently came across stalwart labs mail server and wanted to try it out since it seemed very simple to use.

I first setup everything on a virtual machine, so that later I could easily port it to my actual server.
I use virt-manager for this purpose.

In order to enable UEFI in virt-manager, select "Customize configuration before install", see screenshot


## Goals

- Setup a simple secure server, firewall, ssh keys, etc.
- Hosting my website
- Radicale for calendar / todos

## Future / Optional

- Mail server (stalwart-mail)
- Additional side project servers
- Git server
- Matrix server
- Firefox sync
- Rainloop

## Installing in a virtual machine

```
sudo su
```

## Partioning the disk
```
fdisk /dev/vda
```

- g (gpt disk label)
- n
- 1 (partition number [1/128])
- 2048 first sector
- +500M last sector (boot sector size)
- t
- 1 (EFI System)
- n
- 2
- default (fill up partition)
- default (fill up partition)
- w (write)

```
mkfs.fat -F 32 /dev/vda1
fatlabel /dev/vda1 BOOT
mkfs.ext4 /dev/vda2 -L ROOT
mount /dev/disk/by-label/ROOT /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-label/BOOT /mnt/boot
```

## Configuration
Generate the hardware-configuration and get the configuration files
```
nixos-generate-config --root /mnt
nix-shell -p git
git clone https://github.com/wjehee/.dotfiles-nix
cp /mnt/etc/nixos/hardware-configuration.nix .dotfiles-nix/hosts/HOSTNAME/
```

## Perform the install
```
cd .dotfiles-nix/
git add .
nixos-install --flake .#HOSTNAME
```

IMPORTANT: Set the password, and set the password for the admin user: `passwd admin`  
Otherwise we cannot ssh in (since we disable root login)

# Sources

- [NixOS installation guide](https://nixos.wiki/wiki/NixOS_Installation_Guide)
- [NixOS and nginx](https://nixos.wiki/wiki/Nginx)
- [NixOS public key authentication](https://nixos.wiki/wiki/SSH_public_key_authentication)
- [Radicale docs](https://radicale.org/v3.html)

