+++
title = "Niks to Nix, part 2 | Configuring my server using Nix"
template = "page.html"
date = 2023-09-20
[taxonomies]
series=["Niks to Nix"]
+++

In this part we will configure the general settings of the server, the Nginx server for hosting a static site and Radicale.
Both the static site and Radicale will automatically have SSL certificates generated through ACME.

All the configuration is done in the hosts/HOSTNAME/configuration.nix file unless otherwise stated.
Read [my dotfiles' readme](https://github.com/WJehee/.dotfiles-nix) for more information about adding hosts, etc.

## General configuration

First we set up the bootloader, since we are using UEFI we need the following settings.
```nix
boot.loader = {
    grub.enable = true;
    grub.device = "nodev";
    grub.efiSupport = true;
    grub.useOSProber = true;
    efi.canTouchEfiVariables = true;
};
```

Here we install some basic programs, such as neovim for editing files and git for cloning the repository we put our config files in.

```nix
environment.systemPackages = with pkgs; [
    git
];
system.stateVersion = "23.05";
time.timeZone = "Europe/Amsterdam";         # Change to your timezone
i18n.defaultLocale = "en_US.UTF-8";

programs.neovim.enable = true;
```

Here we create an admin user and disable root login, but we allow the admin user to use doas to get root permissions.
We also only enable public key authentication for ssh logins, so be sure to add your public key to the authorized keys section.

```nix
services.openssh = {
    enable = true;
    settings.PasswordAuthentication = false;
    settings.KbdInteractiveAuthentication = false;
    settings.PermitRootLogin = "no";
};
users.users.admin = {
    isNormalUser = true;
    extraGroups = [
        "docker"
    ];
    openssh.authorizedKeys.keys = [
        "YOUR SSH PUBLIC KEY HERE"
    ];
};
security.sudo.enable = false;
security.doas.enable = true;
security.doas.extraRules = [{
    users = [ "admin" ];
    keepEnv = true;
    persist = true;
}];
```

I prefer using doas but you can also decide to use sudo, then you must at least change the following in the configuration:

```nix
users.users.admin.extraGroups = [
    "docker"
    "wheel"             # add the admin user to the wheel group
];
security.sudo = {
    enable = true;      # enable sudo
    groups = [
        "wheel"         # allow users in the wheel group to use sudo
    ];
};
```

Lastly, we set the firewall to allow only the following ports:
- Port 22 (ssh)
- Port 80 (http)
- Port 443 (https)
- Port 5232 (radicale web service)

```nix
networking.firewall = {
    enable = true;
    allowedTCPPorts = [ 22 80 443 5232 ];
};
```

## Radicale

```nix
services.radicale = {
    enable = true;
    settings = {
        server = {
            hosts = [ "0.0.0.0:5232" "[::]:5232" ];
        };
        auth = {
            type = "htpasswd";
            htpasswd_filename = "/etc/radicale/users";
            htpasswd_encryption = "bcrypt";
        };
        storage = {
            filesystem_folder = "/var/lib/radicale/collections";
        };
    };
};
```

## Nginx

```nix
security.acme = {
    acceptTerms = true;
    defaults.email = "YOUR EMAIL HERE";
};
services.nginx = {
    enable = true;
    recommendedProxySettings = true;
    recommendedTlsSettings = true;
    virtualHosts = {
        "domain.com" = {
            addSSL = true;
            enableACME = true;
            root = "/var/www/wouterjehee.com";
        };
       "radicale.domain.com" = {
           forceSSL = true;
           enableACME = true;
           locations."/" = {
               proxyPass = "http://localhost:5232/";
               extraConfig = ''
                   proxy_set_header X-Script-Name /radicale;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                   proxy_pass_header Authorization;
               '';
           };
       };
    };
};
```

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

