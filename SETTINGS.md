# Settings
There are two types of settings: those used by Ansible for the deployment, and those used by SourceHut applications. These can be found in `group_vars/all/vars.yml` and `files/sourcehut.j2`, respectively. In the managed node, `sourcehut.j2` can be found as `config.ini` in `/etc/sr.ht/`.

## `vars.yml`
### `srht`
Settings related to SourceHut deployment. Only `real_ips` is optional.

- `domain` [`str`]: Base domain to use for services. **Unused unless `services.certbot.wildcard` is enabled. Domains should be set in [`sourcehut.j2`](#sourcehut.j2)'s `origin`s.**
- `modules` [`list`]: SourceHut modules to install.
- `init_cfg` [`bool`]: Whether to copy and initialize `sourcehut.j2`, as well as create required databases.
- `init_users` [`bool`]: Whether to create the users defined. **Required when deploying.**
- `real_ips` [`list`, `optional`]: IPs to add to nginx's `set_real_ip_from`. For details, visit [nginx's website](https://nginx.org/en/docs/http/ngx_http_realip_module.html).
- `users` [`list`]: List of users to create. **At least one is required.**
    - `id` [`int`]: ID for the user. Must be unique.
    - `name` [`str`]: Username for the user.
    - `email` [`str`]: Email for the user.
    - `pass` [`str`]: Password for the user. It is recommended to use `ansible-vault`.
    - `role` [`str`]: Role for the user. Can be `PENDING`, `USER`, `ADMIN`.

### `services`
Settings related to required services by SourceHut and other optional ones. All of these are optional; **if not defined, they won't be installed**.

- `nfs` [`dict`, `optional`]: Whether to set up NFS.
    - `users` [`list`]: List of users to include in the NFS group.
    - `gid` [`int`]: ID to set for the NFS group.
    - `list` [`list`]: List of shares to mount.
        - `dir` [`str`]: Path where share will be mounted.
        - `src` [`str`]: Address of the share to mount.
        - `opts` [`str`]: NFS options for mounting the share.
- `postgres` [`dict`/`bool`, `optional`[^1]]: Whether to set up PostgreSQL. Used by SourceHut to host databases for its services.
    - `dir` [`str`, `optional`]: Path to store data and config.
- `minio` [`dict`, `optional`[^1]]: Whether to set up MinIO. S3 storage used by some of SourceHut's services.
    - `dir` [`str`, `optional`]: Path to store data.
    - `user` [`str`]: User used for login.
    - `pass` [`str`]: Password used for login. It is recommended to use `ansible-vault`.
- `redict` [`bool`, `optional`[^1]]: Whether to set up Redict. Only `true` or `false`. Used by SourceHut for caching and temporary storage.
- `certbot` [`dict`, `optional`]: Whether to set up Certbot. Only supports Cloudflare DNS Challenge for now.
    - `email` [`str`]: Email to use when requesting the certificate.
    - `cf_token` [`str`]: Cloudflare token to create DNS required by the challenge.
    - `wildcard` [`bool`]: Whether to request a wildcard certificate or one for each service.

[^1]: These can only be omitted if run externally and configured accordingly in `sourcehut.j2`.

## `sourcehut.j2`
Settings related to SourceHut applications. Most of them are explained in great detail by comments, so this section will only mention those that are read or modified during the deployment.

- Each service has a `origin` entry to change where the app will be served at. These are read by Ansible to determine which domain to use on nginx's config.
- All services except `meta` include two entries called `oauth-client-id` and `oauth-client-secret`. These are managed by Ansible regardless of their value.
- SourceHut needs three keys that can be generated with binaries included in their packages. The entries for these are `service-key`, `network-key` (both found in `[sr.ht]`) and `private-key` (found in `[webhooks]`). If set to `%AUTO%`, Ansible generates them automatically, which is recommended.