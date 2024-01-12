+++
title = "Niks to Nix, part 4 | Mail server"
template = "page.html"
date = 2023-11-30
draft = true
[taxonomies]
series=["Niks to Nix"]
+++

In this part we will configure a mail server using [stalwart mail](https://stalw.art/)
and self-host a project behind a nginx reverse proxy

First we include a new module in our server configuration
```nix
imports = [
    other/path/to/module.nix
    ...
    path/to/stalwart-mail.nix
];
```

Then in this new module:
```nix
services.stalwart-mail = {
    enable = true;
    settings = {};
```

This enables stalwart-mail, but we also need to set some [configuration options](https://stalw.art/docs/category/configuration/).
The most important ones to set are:

```nix

```

Run stalwart installer:
```
doas stalwart-install
```
Follow the steps as usual, the only differences on NixOS are the TLS settings:

- TLS Certificate location: /var/lib/acme/YOURDOMAIN.com/fullchain.pem
- TLS private key location: /var/lib/acme/YOURDOMAIN.com/key.pem

Add the DNS records with your DNS provider as specified by the output of the command

# Creating an account

Change user to `root` instead of your user account so we can access the right files (by doing `doas su`).

Go into the data folder in the install directory: `/var/lib/stalwart-mail/data`

Run `sqlite3`

```sqlite
.open accounts.sqlite3
```

