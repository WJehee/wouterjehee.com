+++
title = "Niks to Nix, part 2 | Configuring my server using Nix"
template = "page.html"
date = 2023-09-20
weight = 2
[taxonomies]
series=["Niks to Nix"]
+++

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

