---
title: From Embedded App to Container
author: troglobit
date: 2025-11-20 10:00:00 +0100
last_modified_at: 2026-01-23 11:00:00 +0100
categories: [howto]
tags: [containers, docker, podman, embedded, migration]
---

![Docker whale](/assets/img/docker.webp){: width="200" .right}

So you have an application running on a Raspberry Pi, BeagleBone, or
similar embedded Linux platform. Maybe it's a temperature monitor, a
data logger, or a custom control system. You've heard about Infix and
want to try running your application as a container. This guide shows
you how.

Previous posts in this series covered [running existing containers][5],
[advanced networking][6], and [basic container setups][7]. This time
we focus on something different: **how to containerize your own application**.

We'll walk through the complete process: preparing your application,
creating a container image, and running it on Infix. By the end, you'll
have a containerized application that's easier to deploy, update, and
maintain.

> This guide assumes basic familiarity with Linux and command-line tools.
> You should have [Infix installed][0] and be able to log in via SSH or
> console. For detailed container documentation, see the [container guide][1].
{: .prompt-info }

## What You'll Need

Before starting, gather:

1. **Your application** - source code or compiled binaries
2. **Dependencies** - libraries, configuration files, data files
3. **Infix system** - running [v24.11.0][2] or later
4. **Container tools** - Docker (or Podman) on your development machine (optional)
5. **Network access** - to download base images and push your container

> Infix uses [Podman][4] internally for container management, but provides
> a `docker` command alias for familiarity. All examples use `docker`, which
> works identically whether you're using Docker or Podman.
{: .prompt-tip }

## Example Application

Let's use a simple but realistic example: a Python script that monitors
system temperature and logs data to a file. This represents a typical
embedded IoT application.

**temp-monitor.py:**

```python
#!/usr/bin/env python3
import time
import os
import configparser
from datetime import datetime

def read_temperature():
    """Read temperature from system (mock for demo)"""
    # On real hardware: read from /sys/class/thermal/thermal_zone0/temp
    # For demo, return a simulated value
    return 42.5

def load_config(config_file='/etc/temp-monitor.conf'):
    """Load configuration from file, fall back to environment/defaults"""
    config = configparser.ConfigParser()

    # Set defaults
    defaults = {
        'log_file': os.getenv('LOG_FILE', '/data/temperature.log'),
        'interval': os.getenv('INTERVAL', '60'),
        'format': 'csv'
    }

    # Try to read config file if it exists
    if os.path.exists(config_file):
        config.read(config_file)
        if config.has_section('monitor'):
            return {
                'log_file': config.get('monitor', 'log_file', fallback=defaults['log_file']),
                'interval': config.getint('monitor', 'interval', fallback=int(defaults['interval'])),
                'format': config.get('monitor', 'format', fallback=defaults['format'])
            }

    # Use defaults if no config file
    return {
        'log_file': defaults['log_file'],
        'interval': int(defaults['interval']),
        'format': defaults['format']
    }

def main():
    cfg = load_config()

    print(f"Temperature monitor starting...")
    print(f"Logging to: {cfg['log_file']}")
    print(f"Interval: {cfg['interval']}s")
    print(f"Format: {cfg['format']}")

    while True:
        temp = read_temperature()
        timestamp = datetime.now().isoformat()

        if cfg['format'] == 'csv':
            log_entry = f"{timestamp},{temp}\n"
        else:
            log_entry = f"[{timestamp}] Temperature: {temp}°C\n"

        with open(cfg['log_file'], 'a') as f:
            f.write(log_entry)

        print(f"{timestamp}: {temp}°C")
        time.sleep(cfg['interval'])

if __name__ == '__main__':
    main()
```

On your current embedded system, you probably:

- Install Python: `apt install python3`
- Copy the script somewhere: `/usr/local/bin/temp-monitor.py`
- Add a systemd service or cron job
- Hope everything still works after the next system update

Let's containerize this instead.

> Infix includes [mg][8], a lightweight Emacs-compatible editor, a great
> alternative for editing files directly on the device.  Just use `mg
> Containerfile` instead of `cat >` (below) for a better editing
> experience.  {: .prompt-tip }

## Creating a Container Image

You have two options for creating container images:

1. **Build elsewhere, run on Infix** - recommended for most users
2. **Build directly on Infix** - useful for prototyping

### Choosing a Base Image

Before creating your container, consider which base image to use:

**Alpine Linux** (used in our examples):
- Minimal size (~5 MB base image)
- Uses musl libc instead of glibc
- Fast package installation with `apk`
- **Limitation:** Not 100% compatible with glibc - some applications built for
  standard Linux distributions may not work
- **Best for:** New applications, Python/Go/Rust apps, size-critical deployments

**Debian/Ubuntu:**
- Uses standard glibc (better compatibility)
- Larger base image (~50-100 MB)
- Familiar `apt` package manager
- **Best for:** Existing applications, binary compatibility requirements, complex
  dependencies

**Example alternatives:**

```dockerfile
FROM debian:bookworm-slim     # Debian 12 (minimal)
FROM ubuntu:22.04             # Ubuntu LTS
FROM python:3.11              # Python on Debian (larger but more compatible)
```

> If you're migrating an existing application from Raspberry Pi OS, Debian, or
> Ubuntu, starting with a Debian-based image often saves troubleshooting time.
{: .prompt-tip }

### Cross-Architecture Considerations

Many developers build on x86_64 (amd64) workstations but deploy to ARM devices.
Here's what you need to know:

**Architecture matching:** Your Infix device might run:
- `aarch64` (ARM 64-bit) - Raspberry Pi 4/5, most modern ARM devices
- `armv7l` (ARM 32-bit) - Older Raspberry Pi models
- `x86_64` (amd64) - PC-based systems

**Build strategies:**

1. **Multi-architecture builds** (recommended for distribution):
   ```console
   $ docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 \
       -t myapp/temp-monitor:v1.0 --push .
   ```

2. **Target-specific builds** (simplest for single deployment):
   ```console
   $ docker build --platform linux/arm64 -t myapp/temp-monitor:v1.0 .
   ```

3. **Native builds** (build directly on target architecture)

**Important notes:**
- Pure interpreted languages (Python, Ruby) work across architectures
- Compiled code must match the target architecture
- Pre-built binaries in your container must be for the target architecture
- Cross-compilation may require QEMU emulation (slower builds)

### Option 1: Build on Your Development Machine

Create a `Containerfile` (or `Dockerfile`) in your project directory:

```dockerfile
FROM python:3.11-alpine

# Install any system dependencies
# RUN apk add --no-cache some-package

# Copy your application
COPY temp-monitor.py /usr/local/bin/temp-monitor.py
RUN chmod +x /usr/local/bin/temp-monitor.py

# Set default environment variables
ENV LOG_FILE=/data/temperature.log
ENV INTERVAL=60

# Create directory for data (will be a volume)
RUN mkdir -p /data

# Run the application
CMD ["/usr/local/bin/temp-monitor.py"]
```

Build the image:

```console
# If your dev machine matches target (e.g., both x86_64, or both ARM64)
$ docker build -t myapp/temp-monitor:v1.0 .

# If your dev machine differs from target (e.g., x86_64 PC → Raspberry Pi 4B)
$ docker build --platform linux/arm64 -t myapp/temp-monitor:v1.0 .
```

> **Important:** Raspberry Pi 4/5 and most modern ARM boards use `linux/arm64`.
> Older 32-bit ARM boards use `linux/arm/v7`. PC-based Infix systems use
> `linux/amd64`. When in doubt, run `uname -m` on your Infix device.
{: .prompt-warning }

Export as OCI archive:

```console
$ docker save -o temp-monitor-v1.0.tar myapp/temp-monitor:v1.0
$ gzip temp-monitor-v1.0.tar
```

Transfer to your Infix device:

```console
$ scp temp-monitor-v1.0.tar.gz admin@infix-device:/var/tmp/
```

### Option 2: Build on Infix (Prototyping)

For quick experiments, you can build directly on Infix. Exit the CLI to
your login shell and prepare your application files in `/var/tmp/myapp/`:

```console
admin@infix:~$ mkdir -p /var/tmp/myapp
admin@infix:~$ cd /var/tmp/myapp
admin@infix:/var/tmp/myapp$ cat > temp-monitor.py
#!/usr/bin/env python3
... paste your script here ...
^D
admin@infix:/var/tmp/myapp$ cat > Containerfile
FROM python:3.11-alpine
... paste containerfile here ...
^D
```

Build the image:

```console
admin@infix:/var/tmp/myapp$ sudo docker build -t temp-monitor:latest .
```

> Building on the device consumes storage in `/var` and requires CPU
> resources. For production, build on a development machine instead.
{: .prompt-warning }

## Running Your Container on Infix

Now that you have a container image, let's configure Infix to run it.

### Basic Configuration

First, load the image (if you transferred an archive):

```console
admin@infix:/> container load /var/tmp/temp-monitor-v1.0.tar.gz name temp-monitor:v1.0
```

Or reference the OCI archive directly in your configuration:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set image oci-archive:/var/tmp/temp-monitor-v1.0.tar.gz
admin@infix:/config/container/temp-monitor/> set hostname temp-mon
admin@infix:/config/container/temp-monitor/> leave
```

Check the status:

```console
admin@infix:/> show container
CONTAINER ID  IMAGE                               COMMAND     CREATED        STATUS        PORTS  NAMES
a1b2c3d4e5f6  localhost/temp-monitor:v1.0                     5 seconds ago  Up 4 seconds         temp-monitor
```

View the logs:

```console
admin@infix:/> container log temp-monitor
Temperature monitor starting...
Logging to: /data/temperature.log
Interval: 60s
2025-11-20T10:15:00: 42.5°C
```

### Adding Persistent Storage

The container is running, but logs are lost when it restarts. Add a volume:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set volume logs target /data
admin@infix:/config/container/temp-monitor/> leave
```

Now `/data` in the container is persistent and survives container upgrades.

### Customizing Environment Variables

Override the default logging interval:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> edit env INTERVAL
admin@infix:/config/container/temp-monitor/env/INTERVAL/> set value 30
admin@infix:/config/container/temp-monitor/env/INTERVAL/> end
admin@infix:/config/container/temp-monitor/> leave
```

## Container Networking

Your application may need network access. Infix provides flexible options.

### Host Networking (Simplest)

For applications that need full network access:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set network host true
admin@infix:/config/container/temp-monitor/> leave
```

The container shares the host's network namespace - all ports, interfaces,
and routing are identical to the host.

### Bridge Network (Isolated)

For better isolation, use a container bridge:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface docker0
admin@infix:/config/interface/docker0/> set container-network
admin@infix:/config/interface/docker0/> end
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set network interface docker0
admin@infix:/config/container/temp-monitor/> leave
```

If your application exposes an HTTP API on port 8080:

```console
admin@infix:/config/container/temp-monitor/> set network publish 8080:8080
```

### Direct Interface Access

Infix can give containers direct access to physical ports or VETH pairs:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface veth0
admin@infix:/config/interface/veth0/> set veth peer veth0c
admin@infix:/config/interface/veth0/> set ipv4 address 192.168.10.1 prefix-length 24
admin@infix:/config/interface/veth0/> end
admin@infix:/config/> edit interface veth0c
admin@infix:/config/interface/veth0c/> set ipv4 address 192.168.10.2 prefix-length 24
admin@infix:/config/interface/veth0c/> set container-network
admin@infix:/config/interface/veth0c/> end
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set network interface veth0c
admin@infix:/config/container/temp-monitor/> leave
```

This gives precise control over container networking and enables advanced
setups like dedicated monitoring networks.

## Migrating from systemd

On traditional embedded Linux, you probably use systemd to manage your
application. Here's how container concepts map to systemd:

| systemd | Infix Container | Notes |
|---------|-----------------|-------|
| `systemctl start` | Container runs automatically | Set `manual false` (default) |
| `systemctl enable` | Persistent config | Save with `copy running-config startup-config` |
| `Restart=always` | `restart-policy always` | Default behavior |
| Environment file | `env` settings | Set key-value pairs in config |
| `ExecStartPre=` | Not needed | Handle in Containerfile |
| Logs in journald | Container logs | Use `container log NAME` |

To make your container start automatically at boot (default):

```console
admin@infix:/config/container/temp-monitor/> show manual
manual false;
```

To require manual start:

```console
admin@infix:/config/container/temp-monitor/> set manual true
```

## Configuration Files

Many embedded applications need configuration files. Infix provides two
approaches:

### Approach 1: Content Mounts (Small Files)

For small config files, store content directly in Infix configuration:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> edit mount settings
admin@infix:/config/container/temp-monitor/mount/settings/> set target /etc/temp-monitor.conf
admin@infix:/config/container/temp-monitor/mount/settings/> text-editor content
```

An editor opens where you can paste your configuration:

```ini
[monitor]
interval = 15
log_file = /data/temperature.log
format = text
```

On exit, it's automatically base64 encoded and stored in `startup-config`.
The content is available to your container at `/etc/temp-monitor.conf`, and
your Python application will now use these settings instead of the defaults.

### Approach 2: Volume Mounts (Large Files)

For larger files or collections, use a volume:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set volume config target /etc/app
admin@infix:/config/container/temp-monitor/> leave
```

Then populate files from your login shell:

```console
admin@infix:~$ sudo -i
root@infix:~# cd /var/lib/containers/storage/volumes/temp-monitor-config/_data
root@infix:/var/lib/containers/storage/volumes/temp-monitor-config/_data# vi config.yaml
```

## Upgrading Your Application

When you release a new version of your application:

### Using Version Tags (Recommended)

Build and export the new version:

```console
$ docker build -t myapp/temp-monitor:v1.1 .
$ docker save -o temp-monitor-v1.1.tar.gz myapp/temp-monitor:v1.1
```

Transfer and update configuration:

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set image oci-archive:/var/tmp/temp-monitor-v1.1.tar.gz
admin@infix:/config/container/temp-monitor/> leave
```

Infix automatically stops the old container and starts the new one.
Your volumes remain intact.

### Using Mutable Tags (Development)

For development with `:latest` tags:

```console
admin@infix:/> container upgrade temp-monitor
```

This pulls the newest image and recreates the container.

## Publishing to a Registry

For easier deployment across multiple devices, publish to a container
registry:

### GitHub Container Registry

```console
$ docker login ghcr.io
$ docker tag temp-monitor:v1.0 ghcr.io/username/temp-monitor:v1.0
$ docker push ghcr.io/username/temp-monitor:v1.0
```

On Infix:

```console
admin@infix:/config/container/temp-monitor/> set image ghcr.io/username/temp-monitor:v1.0
```

### Docker Hub

```console
$ docker login docker.io
$ docker tag temp-monitor:v1.0 username/temp-monitor:v1.0
$ docker push username/temp-monitor:v1.0
```

On Infix:

```console
admin@infix:/config/container/temp-monitor/> set image username/temp-monitor:v1.0
```

Now deploying to new devices is as simple as adding the container
configuration - no need to transfer archive files manually.

## Advanced Topics

### Hardware Access

If your application needs GPIO, I2C, or other hardware:

```console
admin@infix:/config/container/temp-monitor/> edit mount gpio
admin@infix:/config/container/temp-monitor/mount/gpio/> set source /sys/class/gpio
admin@infix:/config/container/temp-monitor/mount/gpio/> set target /sys/class/gpio
admin@infix:/config/container/temp-monitor/mount/gpio/> set read-only false
admin@infix:/config/container/temp-monitor/mount/gpio/> end
admin@infix:/config/container/temp-monitor/> edit capabilities
admin@infix:/config/container/temp-monitor/capabilities/> set add sys_admin
```

For device access (e.g., `/dev/i2c-1`):

```console
admin@infix:/config/container/temp-monitor/> edit mount i2c
admin@infix:/config/container/temp-monitor/mount/i2c/> set source /dev/i2c-1
admin@infix:/config/container/temp-monitor/mount/i2c/> set target /dev/i2c-1
admin@infix:/config/container/temp-monitor/mount/i2c/> set read-only false
```

### Running Multiple Instances

Need to run multiple copies with different configurations?

```console
admin@infix:/> configure
admin@infix:/config/> edit container temp-monitor-1
admin@infix:/config/container/temp-monitor-1/> set image temp-monitor:v1.0
admin@infix:/config/container/temp-monitor-1/> edit env SENSOR
admin@infix:/config/container/temp-monitor-1/env/SENSOR/> set value sensor1
admin@infix:/config/container/temp-monitor-1/env/SENSOR/> end
admin@infix:/config/container/temp-monitor-1/> end
admin@infix:/config/> edit container temp-monitor-2
admin@infix:/config/container/temp-monitor-2/> set image temp-monitor:v1.0
admin@infix:/config/container/temp-monitor-2/> edit env SENSOR
admin@infix:/config/container/temp-monitor-2/env/SENSOR/> set value sensor2
```

## Complete Example Configuration

Here's a complete working configuration for reference:

```console
admin@infix:/> configure
admin@infix:/config/> edit interface docker0
admin@infix:/config/interface/docker0/> set container-network
admin@infix:/config/interface/docker0/> end
admin@infix:/config/> edit container temp-monitor
admin@infix:/config/container/temp-monitor/> set image oci-archive:/var/tmp/temp-monitor-v1.0.tar.gz
admin@infix:/config/container/temp-monitor/> set hostname temp-mon
admin@infix:/config/container/temp-monitor/> set network interface docker0
admin@infix:/config/container/temp-monitor/> edit env INTERVAL
admin@infix:/config/container/temp-monitor/env/INTERVAL/> set value 30
admin@infix:/config/container/temp-monitor/env/INTERVAL/> end
admin@infix:/config/container/temp-monitor/> set volume logs target /data
admin@infix:/config/container/temp-monitor/> leave
admin@infix:/> copy running-config startup-config
```

## Troubleshooting

**Container won't start:**

```console
admin@infix:/> show log
admin@infix:/> container log temp-monitor
```

**Check container status:**

```console
admin@infix:/> show container
```

**Inspect running container:**

Exit the CLI and use the docker command to exec into the container:

```console
admin@infix:~$ sudo docker exec -it temp-monitor sh
/ #
```

**Remove and recreate:**

```console
admin@infix:/> configure
admin@infix:/config/> delete container temp-monitor
admin@infix:/config/> leave
```

Then recreate with the configuration above.

## Conclusion

You've successfully containerized an embedded application and deployed
it on Infix. Your application is now:

- **Isolated** - dependencies don't conflict with the system
- **Portable** - runs on any Infix device
- **Upgradeable** - new versions deploy without affecting data
- **Maintainable** - configuration is versioned and reproducible

The same approach works for any embedded application: data loggers,
control systems, protocol gateways, monitoring agents, and more.

For more advanced networking scenarios with your containers, check out
the [advanced networking][6] post. If you need to understand the basics
of running containers on Infix first, start with the [Docker containers][5]
introduction.

For complete technical details, see the [container documentation][1] or
explore the [YANG model][3] for all available configuration options.

Remember to save your configuration:

```console
admin@infix:/> copy running-config startup-config
```

Take care!

[0]: https://github.com/kernelkit/infix/
[1]: https://kernelkit.org/infix/latest/container/
[2]: https://github.com/kernelkit/infix/releases/tag/latest
[3]: https://github.com/kernelkit/infix/blob/main/src/confd/yang/confd/infix-containers.yang
[4]: https://podman.io
[5]: /posts/containers/
[6]: /posts/advanced-containers/
[7]: /posts/basic-container/
[8]: https://github.com/troglobit/mg
