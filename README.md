**[Version française](https://github.com/Ganymede0/ansible_peertube_core/blob/main/README.fr.md)**

# What this role does and does not do

This role is designed to automate the deployment of [PeerTube](https://joinpeertube.org) instances with [ansible](https://www.ansible.com).

However, **it does not install all the services required by PeerTube**. There are a lot of excellent ansible roles to deploy these services, and most ansible users probably use some of them already.

Here's what needs to be installed separately:

- NodeJS
- Yarn
- Nginx
- PostgreSQL
- Redis
- A valid TLS certificate

This role installs PeerTube and configures required services (including Nginx and PostgreSQL), assuming that all dependencies are already correctly installed.

Currently, it works only with **Debian 11 (bullseye)**.


# Variables

## Version of PeerTube

The version of PeerTube that this role will install is defined in `vars/main.yml`. It is currently **5.2.0**. This variable is not meant to be edited, since it is the only supported version.


## Mandatory variables

This role requires some variables specific to the node (the PeerTube instance), for which there are no default values:

- `peertube_user_password`: hash of the peertube system user's password (see https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module)
- `peertube_postgresql_user_passwd`: peertube user password in postgresql (hashed or not, see https://docs.ansible.com/ansible/latest/collections/community/postgresql/postgresql_user_module.html#parameter-password)
- `peertube_webserver_hostname`: the node's hostname (also used in nginx' configuration and in the TLS certificate)
- `peertube_secrets`: a "secret" specific to the PeerTube instance (generated with `openssl rand -hex 32`)

For security reasons, it could be useful to encrypt some of these variables with `ansible-vault`.


## Optional variables

All other variables are optional, since they have default values.

Only one variable is not related to PeerTube's configuration itself:

- `peertube_import_key` (boolean): set this variable to `false` if you don't want to reimport the OpenPGP key necessary to verify peertube's zip archive each time the role is played. You'll have to set this variable to `true` again if this key is modified in any way (because it has expired or is revoked, because the signature has changed, etc.).

The other variables set PeerTube parameters in `production.yaml`. They are named after this file's YAML structure, prefixed by `peertube_`. Possible values are (for the most part) commented in [templates/production.yaml.j2](https://github.com/Ganymede0/ansible_peertube_core/blob/main/templates/production.yaml.j2), and default values can be found in [defaults/main.yml](https://github.com/Ganymede0/ansible_peertube_core/blob/main/defaults/main.yml).

**This role does not support all PeerTube parameters.** However, available variables should already cover most deployment needs. If you need to expose more variables in this role, feel free to contribute a merge request ;-)

Please note that two sets of variables have to be consistent with how PostgreSQL and Redis are installed:

### PostgreSQL settings

- `peertube_postgresql_hostname`
- `peertube_postgresql_port`
- `peertube_postgresql_ssl`
- `peertube_postgresql_suffix`
- `peertube_postgresql_username`
- `peertube_postgresql_pool_max`

### Redis settings

- `peertube_redis_hostname`
- `peertube_redis_port`


# Example playbook

```
- name: My PeerTube instance
  hosts:
    mynode
  become: true

  vars:
    peertube_user_password: MY_LONG_PASSWORD_HASH
    peertube_postgresql_user_passwd: MY_POSTGRESQL_PASSWORD
    peertube_webserver_hostname: MY_DOMAIN.TLD
    peertube_secrets: MY_LONG_SECRET_LINE

  roles:
    # - nodejs, yarn, nginx, etc.
    - peertube-core
```


# License

This role is distributed under the terms of the [BSD-3-Clause license](https://github.com/Ganymede0/ansible_peertube_core/blob/main/LICENSE).
