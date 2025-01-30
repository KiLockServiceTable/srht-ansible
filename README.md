# srht-ansible
This Ansible playbook intends to provide a simpler and smoother experience when deploying SourceHut's services locally.
It was made with my specific use-case in mind, but I still tried to broaden it a bit through some customizations provided via variables.

> Only Alpine 3.20 is supported. SourceHut dropped support of their packages for other distributions.

> Tested with `meta`, `git`, `paste`, `man` and `todo`. The rest may not work.

## Features
- Install and set up multiple SourceHut services.
- Create SourceHut users on deployment.
- Automatically configure nginx for the specified domains.
- Support for running dependencies locally or externally.
- Support for mounting NFS shares for services to use.
- Support for SSL through Cloudflare DNS challenge.

## Prerequisites
- Alpine system with Python installed.
- Configured `vars.yml`.
- Configured `sourcehut.j2` file with categories for all services that will be deployed. Example configs can be found on [SourceHut's repositories](https://git.sr.ht/~sircmpwn/) (except [hg](https://hg.sr.ht/~sircmpwn/hg.sr.ht/raw/config.example.ini)). All of them are redundant with `meta`, so only the service's category (a.e. `[todo.sr.ht]`) and subcategories (a.e. `[todo.sr.ht::mail]`) should be copied over.
- A GPG key's public and private keys exported with `--armor` and without passphrase in the `files` directory with the names `gpg_public.asc` and `gpg_private.asc`, as well as `files/sourcehut.j2`'s `pgp-key-id` set to the ID of the key.

## Variables
See [SETTINGS.md](SETTINGS.md).

## Issues
At the time of writing this, `man`'s nginx package has broken CSS and `todo`'s initdb does not properly create the database, resulting in constraint errors. To resolve `todo`'s database, run its [schema.sql](https://git.sr.ht/~sircmpwn/todo.sr.ht/tree/master/item/schema.sql) directly. While a fix could be included in this playbook, SourceHut will eventually fix these issues, thus I consider it not worth the effort.