---
title: Password and SSH Key Login
author: troglobit
date: 2024-07-25 05:37:00 +0100
categories: [examples]
tags: [cli, ssh]
---

User management, including passwords, SSH keys, remote authentication is
available in the system authentication configuration context.

```
admin@host:/config/> edit system authentication user admin
admin@host:/config/system/authentication/user/admin/> change password
New password: 
Retype password: 
admin@host:/config/system/authentication/user/admin/> leave
```

The change password command starts an interactive dialogue that asks for
the new password, with a confirmation, and then salts and encrypts the
password with the default crypt algorithm.  This is either sha512crypt
or yescrypt depending on the build.

It is also possible to use the `set password ...` command. This allows
setting an already hashed password.  To manually hash a password, use
the `do password encrypt` command.  This launches the admin-exec command
to hash, and optionally salt, your password.  This encrypted string can
then be used with the `set password ...` command.

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

```
admin@host:/config/> edit system authentication user admin 
admin@host:/config/system/authentication/user/admin/> edit authorized-key example@host
admin@host:/config/system/authentication/user/admin/authorized-key/example@host/> set algorithm ssh-rsa
admin@host:/config/system/authentication/user/admin/authorized-key/example@host/> set key-data AAAAB3NzaC1yc2EAAAADAQABAAABgQC8iBL42yeMBioFay7lty1C4ZDTHcHyo739gc91rTTH8SKvAE4g8Rr97KOz/8PFtOObBrE9G21K7d6UBuPqmd0RUF2CkXXN/eN2PBSHJ50YprRFt/z/304bsBYkDdflKlPDjuSmZ/+OMp4pTsq0R0eNFlX9wcwxEzooIb7VPEdvWE7AYoBRUdf41u3KBHuvjGd1M6QYJtbFLQMMTiVe5IUfyVSZ1RCxEyAB9fR9CBhtVheTVsY3iG0fZc9eCEo89ErDgtGUTJK4Hxt5yCNwI88YaVmkE85cNtw8YwubWQL3/tGZHfbbQ0fynfB4kWNloyRHFr7E1kDxuX5+pbv26EqRdcOVGucNn7hnGU6C1+ejLWdBD7vgsoilFrEaBWF41elJEPKDzpszEijQ9gTrrWeYOQ+x++lvmOdssDu4KvGmj2K/MQTL2jJYrMJ7GDzsUu3XikChRL7zNfS2jYYQLzovboUCgqfPUsVba9hqeX3U67GsJo+hy5MG9RSry4+ucHs=
admin@host:/config/system/authentication/user/admin/authorized-key/example@host/> show
algorithm ssh-rsa;
key-data AAAAB3NzaC1yc2EAAAADAQABAAABgQC8iBL42yeMBioFay7lty1C4ZDTHcHyo739gc91rTTH8SKvAE4g8Rr97KOz/8PFtOObBrE9G21K7d6UBuPqmd0RUF2CkXXN/eN2PBSHJ50YprRFt/z/304bsBYkDdflKlPDjuSmZ/+OMp4pTsq0R0eNFlX9wcwxEzooIb7VPEdvWE7AYoBRUdf41u3KBHuvjGd1M6QYJtbFLQMMTiVe5IUfyVSZ1RCxEyAB9fR9CBhtVheTVsY3iG0fZc9eCEo89ErDgtGUTJK4Hxt5yCNwI88YaVmkE85cNtw8YwubWQL3/tGZHfbbQ0fynfB4kWNloyRHFr7E1kDxuX5+pbv26EqRdcOVGucNn7hnGU6C1+ejLWdBD7vgsoilFrEaBWF41elJEPKDzpszEijQ9gTrrWeYOQ+x++lvmOdssDu4KvGmj2K/MQTL2jJYrMJ7GDzsUu3XikChRL7zNfS2jYYQLzovboUCgqfPUsVba9hqeX3U67GsJo+hy5MG9RSry4+ucHs=;
admin@host:/config/system/authentication/user/admin/authorized-key/example@host/> leave
```

> The `ssh-keygen` program used to create the public/private key-pair
> base64 encodes the public key data, so there is no need to use the
> text-editor command with `authorized-key`, set does the job.
{: .prompt-info }
