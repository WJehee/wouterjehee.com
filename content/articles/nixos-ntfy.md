+++
title = "Niks to Nix, part 3 | Adding a ntfy.sh server" 
template = "page.html"
date = 2024-02-01
[taxonomies]
series=["Niks to Nix"]
+++

Add this to your configuration.nix and add an nginx reverse proxy just like in [part 1](https://wouterjehee.com/articles/nixos-server-install).

```nix
services.ntfy-sh = {
    enable = true;
    settings = {
        base-url = "https://ntfy.yourdomain.com";
        listen-http = ":2555";     # or any other port
        behind-proxy = true;

        auth-file = "/var/lib/ntfy-sh/user.db";
        auth-default-access = "deny-all";
    };
};
```

# Adding users

You need to be root to add users for ntfy.  
Some simple commands you might want to use ([full list of commands](https://docs.ntfy.sh/config))

- List users: `ntfy user list`
- Add regular user: `ntfy user add username`
- Add admin user: `ntfy user add --role=admin admin_username`

