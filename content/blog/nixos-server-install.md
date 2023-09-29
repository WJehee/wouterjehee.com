+++
title = "Niks to Nix, part 2 | Configuring my server using Nix"
template = "page.html"
date = 2023-09-29
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
    apacheHttpd                             # We need this for htpasswd, which is used by radicale
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
        "ANOTHER KEY?"
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
- Port 443 (https), we will redirect all http traffic to https.

```nix
networking.firewall = {
    enable = true;
    allowedTCPPorts = [ 22 80 443 ];
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
            forceSSL = true;
            enableACME = true;
            root = "/var/www/domain.com";
            serverAliases = [ "radicale.domain.com" ];
        };
       "radicale.domain.com" = {
           forceSSL = true;
           useACMEHost = "domain.com"       # reuse the same certificate for this domain
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

## DNS

Since we are replacing an old server, we first need to change our DNS records such that they point to our newly configured server.
This is important because we are trying to setup SSL certificates for our domain name.

First we need to find our public IP, there are several ways to do it, I like to do it like this:
```sh
curl -4 ifconfig.co     # IPV4
curl ifconfig.co        # IPV6
```
Then go to your domain registrar and change your A and AAAA records.

We also need to setup port forwarding on our router in order to access it from the internet.
So setup a port forwarding rule for ports 80 and 443 using your local IP address (found by using `ip addr`).

# Installing in a virtual machine

```sh
sudo su
```

## Partioning the disk

```sh
parted /dev/vda mklabel gpt
parted /dev/vda mkpart primary fat32 2048s 500M
parted /dev/vda set 1 esp on
parted -- /dev/vda mkpart primary ext4 500M -1s
parted /dev/vda quit

mkfs.fat -F 32 /dev/vda1
fatlabel /dev/vda1 BOOT
mkfs.ext4 /dev/vda2 -L ROOT
mount /dev/disk/by-label/ROOT /mnt
mkdir -p /mnt/boot
mount /dev/disk/by-label/BOOT /mnt/boot
```

## Configuration
Generate the hardware-configuration and get the configuration files
```sh
nixos-generate-config --root /mnt
nix-shell -p git
git clone https://github.com/wjehee/.dotfiles-nix
cp /mnt/etc/nixos/hardware-configuration.nix .dotfiles-nix/hosts/HOSTNAME/
```

## Perform the install

```sh
cd .dotfiles-nix/
git add .
nixos-install --flake .#HOSTNAME
```

Copy the configuration onto the installed version
```sh
cd ..
cp -r .dotfiles-nix /mnt/home/admin
```

1. Change into the installed version by running: `nixos-enter`
2. Change ownership of .dotfiles-nix: `chown -R admin:users home/admin/.dotfiles-nix`
3. Set the password for the admin user, otherwise we cannot login through ssh: `passwd admin`
4. Create the user file for radicale so we can login: `htpasswd -B -c /etc/radicale-users USERNAME`
5. Optionally create more calendar users, by running `htpasswd -B /etc/radicale-users USERNAME`

Now we can reboot!  
It might take some time before the SSL certificates are set up, but once they do everything should work.
If it doesn't work after a while, just SSH into the server and rebuild it.

Everything seems to work! But our website is looking a bit blank...  
In the next part we will deploy the server in a VPS with a custom ISO image and setup CI to auto-deploy our website!

# Sources

- [NixOS installation guide](https://nixos.wiki/wiki/NixOS_Installation_Guide)
- [NixOS and nginx](https://nixos.wiki/wiki/Nginx)
- [NixOS public key authentication](https://nixos.wiki/wiki/SSH_public_key_authentication)
- [NixOS security ACME nginx](https://nixos.org/manual/nixos/stable/#module-security-acme-nginx)
- [Radicale docs](https://radicale.org/v3.html)

