---
title: Docker Containers
author: troglobit
date: 2024-03-08 15:44:42 +0100
categories: [showcase]
tags: [container, containers, docker, podman]
---

![Docker whale](/assets/img/docker.webp){: width="200" .right}

A network operating system for switches and routers, that runs Docker?

Yes, as of Infix v24.02 support for running containers using [podman][1]
is supported.  Because networking is a first class citizen in Infix, you
can set up quite advanced *virtual topologies* with containers.  This
blog post is the first in a series of posts that aims to show this.

> This post assumes knowledge and familiarity with the [Infix Network
> Operating System](https://kernelkit.github.io/).  Ensure you have
> either a network connection or console access to your Infix system and
> can log in to it using SSH.  Recommended reading includes the
> [networking documentation][0].
{: .prompt-info }


----

## Introduction

All configuration and administration of networking and containers is
done through the CLI:

```console
admin@infix:~$ cli

See the 'help' command for an introduction to the system

admin@infix:/>
```

Notice the slight change in the prompt.  Return to the Bash shell using
the `exit` command, or Ctrl-D, from the "admin-exec" top level of the
CLI.


## Networking Basics

![Dataplane overview](/assets/img/dataplane.svg){: width="430" .right}

In Infix all network access has to be set up explicitly, so there is no
default container networking setup (it's a security thing).  There are
two types available to choose from:

 - `host`: Ethernet interface
 - `bridge`: Masquerading bridge

The first can be any physical port/interface which is handed over to the
container or, more commonly, one end of a *VETH pair*.

The latter type is usually available as `docker0`, or `podman0`, on your
host system.  These bridges are managed by the container runtime, in the
case of Infix this is podman.  When a container is set to a container
bridge network, a VETH pair is automatically created when the container
is started -- one end is attached to the bridge and the other connected
to the container as a regular interface.

Here's how you create a container bridge:

```console
admin@infix:/> configure
admin@infix:/config> edit interface docker0
admin@infix:/config/interface/docker0> set container-network
admin@infix:/config/interface/docker0> leave
```


## Web Server Container

![nginx](/assets/img/nginx.png){: width="60" .left}

Now, time for a basic web server example.  For our first container we'll
be using [docker://nginx:alpine](https://hub.docker.com/_/nginx).  It's
a relatively small container with the Nginx web server built on top of
the Alpine Linux image.

```console
admin@infix:/> configure
admin@infix:/config> edit container web
admin@infix:/config/container/web/> set image docker://nginx:alpine
admin@infix:/config/container/web/> edit network
admin@infix:/config/container/web/network/> set interface docker0
admin@infix:/config/container/web/network/> set publish 8080:80
admin@infix:/config/container/web/network/> leave
```

Issuing the command `leave` queues a job to download the image and
create a container in the background.  To see the progress:

```console
admin@infix:/> show log container
```

or just poll the status command:

```console
admin@infix:/> show container
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS         PORTS               NAMES
c60a6deeea4e  docker.io/library/nginx:alpine  nginx -g daemon o...  2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp  web
```

You should now be able to access the web server on port 8080 of the
host's IP address.

![](/assets/img/nginx-welcome.png)
_Nginx default landing page._


> See the [end of this post](#container-content-in-device-configuration)
> for how to store container file content in the Infix configuration!
> Meaning custom(er) builds of Infix can bundle a built-in container's
> initial configuration in the Infix `factory-config`, which can be very
> useful when deploying at new installations.
{: .prompt-info }

## Customizing Content

Containers in Infix are created *read-only by default*, to change the
content, or store state data across host restarts and upgrades, use
volumes.  They are a specialized type of "mount", for people familiar
with UNIX systems.

Here's how to add a volume to your container:

```console
admin@infix:/> configure
admin@infix:/config/> edit container web
admin@infix:/config/container/web/> edit volume content
admin@infix:/config/container/web/volume/content/> set target /usr/share/nginx/html
admin@infix:/config/container/web/volume/content/> leave
```

Named volumes have the downside of being opaque to the host, so the
easiest is to upload the content using `scp` or editing it directly
in the container:

```console
admin@infix:/> container shell web
d95ce9f7674d:/# vi /usr/share/nginx/html/
50x.html    index.html
d95ce9f7674d:/# vi /usr/share/nginx/html/index.html
... edit, save & exit from vi ...
d95ce9f7674d:/# 
```


## Container Content in Device Configuration

Save the best for last?  A neat feature is that container content can be
saved in the system's `startup-config` and therefore be automatically be
backed up by administrators snapshotting the system.

This feature is perfectly suited for container applications that need a
specific site setup.  For example a configuration file.  Here we use the
same container image to bundle an `index.html` file:

```console
admin@infix:/> configure
admin@infix:/config/> edit container web
admin@infix:/config/container/web/> edit mount index.html
admin@infix:/config/container/web/mount/index.html/> set target /usr/share/nginx/html/index.html
admin@infix:/config/container/web/mount/index.html/> text-editor content
```

The `content` setting is an alternative to `source` for file mounts
which allows providing the contents through the device's configuration.

> The `text-editor` command can be changed to use other editors in the
> `system` configuration context, by default it starts a Micro Emacs
> clone, Mg.  See the [documentation][3] for more information.
{: .prompt-tip }

Paste in this:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Infix!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to the Infix Operating System!</h1>
<p>If you see this page, the nginx web server container has been
installed and is working.</p>

<p>For online documentation and support please refer to the
<a href="https://kernelkit.org/">Infix Homepage</a>.<br/>

Commercial support and customer adaptations are available from
<a href="https://wires.se">Wires</a>.</p>

<p><em>Thank you for reading this blog post!</em></p>
</body>
</html>
```

Save and exit with the usual Emacs salute: C-x C-x (Ctrl-X Ctrl-c, or
hold down Ctrl while tapping the X and C keys).

Leave configuration context to activate your changes:

```console
admin@infix:/config/container/web/mount/index.html/> leave
```

Reload your browser to see the change.


## Fin

That's the end of the first post about containers in Infix.  Remember to
save your changes for next boot:

```console
admin@infix:/> copy running-config startup-config
```

Take care! 🧡

[0]: https://github.com/kernelkit/infix/blob/main/doc/networking.md
[1]: https://podman.io
[2]: https://www.docker.com/resources/what-container/
[3]: https://github.com/kernelkit/infix/blob/main/doc/cli/text-editor.md
