---
title: Password and SSH Key Login
author: troglobit
date: 2024-07-25 05:37:00 +0100
categories: [examples]
tags: [cli, ssh]
---

In this post we explore how to use the CLI to change a user's password
and set up SSH keys for authentication.

User management is available in the system authentication configuration
context and there is a dedicated `change` command available to simplify
the process:

```console
admin@example:/> configure
admin@example:/config/> edit system authentication user admin
admin@example:/config/system/authentication/user/admin/> change password
New password: 
Retype password: 
admin@example:/config/system/authentication/user/admin/> leave
```

The `change password` command starts an interactive dialogue that asks
for the new password, with a confirmation, and then salts and encrypts
the password with the default crypt algorithm.  Either sha512crypt or
yescrypt depending on the Infix build.

It is also possible to use the `set password ...` command. This allows
setting an already hashed password, which is what you must do when
managing users over NETCONF or RESTCONF.  To manually hash a password,
use the `do password encrypt` command.  This launches the admin-exec
command to hash, and optionally salt, your password.  This encrypted
string can then be used with the `set password ...` command.

> if you are having trouble thinking of a password, Infix comes with a
> `password generate` command in admin-exec context which generates
> random passwords using the UNIX command `pwgen`.  Use the `do` prefix
> when inside any configuration context to access admin-exec commands.
{: .prompt-tip }


### SSH Public Key Login

When accessing the system remotely using SSH it is very useful to have
SSH key login enabled.  The following shows how to add a an authorized
*public key* to the admin user, multiple keys are supported.

With SSH keys in place it is possible to disable password login, just
remember to verify SSH login and network connectivity before doing so.

```console
admin@example:/config/> edit system authentication user admin
admin@example:/config/system/authentication/user/admin/> edit authorized-key jacky@host
admin@example:/config/system/authentication/user/admin/authorized-key/jacky@host/> set algorithm ssh-rsa
admin@example:/config/system/authentication/user/admin/authorized-key/jacky@host/> set key-data AAAAB3NzaC1yc2EAAAADAQABAAABgQC8iBL42yeMBioFay7lty1C4ZDTHcHyo739gc91rTTH8SKvAE4g8Rr97KOz/8PFtOObBrE9G21K7d6UBuPqmd0RUF2CkXXN/eN2PBSHJ50YprRFt/z/304bsBYkDdflKlPDjuSmZ/+OMp4pTsq0R0eNFlX9wcwxEzooIb7VPEdvWE7AYoBRUdf41u3KBHuvjGd1M6QYJtbFLQMMTiVe5IUfyVSZ1RCxEyAB9fR9CBhtVheTVsY3iG0fZc9eCEo89ErDgtGUTJK4Hxt5yCNwI88YaVmkE85cNtw8YwubWQL3/tGZHfbbQ0fynfB4kWNloyRHFr7E1kDxuX5+pbv26EqRdcOVGucNn7hnGU6C1+ejLWdBD7vgsoilFrEaBWF41elJEPKDzpszEijQ9gTrrWeYOQ+x++lvmOdssDu4KvGmj2K/MQTL2jJYrMJ7GDzsUu3XikChRL7zNfS2jYYQLzovboUCgqfPUsVba9hqeX3U67GsJo+hy5MG9RSry4+ucHs=
admin@example:/config/system/authentication/user/admin/authorized-key/jacky@host/> show
algorithm ssh-rsa;
key-data AAAAB3NzaC1yc2EAAAADAQABAAABgQC8iBL42yeMBioFay7lty1C4ZDTHcHyo739gc91rTTH8SKvAE4g8Rr97KOz/8PFtOObBrE9G21K7d6UBuPqmd0RUF2CkXXN/eN2PBSHJ50YprRFt/z/304bsBYkDdflKlPDjuSmZ/+OMp4pTsq0R0eNFlX9wcwxEzooIb7VPEdvWE7AYoBRUdf41u3KBHuvjGd1M6QYJtbFLQMMTiVe5IUfyVSZ1RCxEyAB9fR9CBhtVheTVsY3iG0fZc9eCEo89ErDgtGUTJK4Hxt5yCNwI88YaVmkE85cNtw8YwubWQL3/tGZHfbbQ0fynfB4kWNloyRHFr7E1kDxuX5+pbv26EqRdcOVGucNn7hnGU6C1+ejLWdBD7vgsoilFrEaBWF41elJEPKDzpszEijQ9gTrrWeYOQ+x++lvmOdssDu4KvGmj2K/MQTL2jJYrMJ7GDzsUu3XikChRL7zNfS2jYYQLzovboUCgqfPUsVba9hqeX3U67GsJo+hy5MG9RSry4+ucHs=;
admin@example:/config/system/authentication/user/admin/authorized-key/jacky@host/> leave
```

> The `ssh-keygen` program used to create the public/private key-pair
> base64 encodes the public key data, so there is no need to use the
> text-editor command with `authorized-key`, set does the job.
{: .prompt-info }


### User Permissions

As a side note, user permissions are handled by the [Access Control
Model][0], in `nacm` configuration context.  Essentially it allows
defining a set of groups which a user can be member of.  At first boot a
single group exist: `admin`, which the default `admin` user is member
of.

To give another user administrator rights we add them to the `admin`
group:

```console
admin@example:/config/> edit nacm group admin
admin@example:/config/nacm/group/admin/> set user-name jacky
admin@example:/config/nacm/group/admin/> leave
```

Read more about [user management][1] in the official documentation.

[0]: https://datatracker.ietf.org/doc/html/rfc8341
[1]: https://kernelkit.org/infix/latest/system/#multiple-users
