---
title: Flashing SD Card Images
author: troglobit
date: 2025-10-27 08:05:00 +0100
categories: [howto]
tags: [beginner]
---

This guide covers how to flash an Infix SD card image to a microSD card or
eMMC module.

### Prerequisites

You will need:

- An SD card reader (USB or built-in)
- A microSD card or eMMC module (minimum 2 GB recommended)
- An Infix SD card image from the [latest-boot][1] builds

### Linux

#### Using dd

The traditional `dd` command works on any Linux system without additional
software:

```console
$ sudo dd if=infix-aarch64-bpi-r3.img of=/dev/sdX bs=1M status=progress oflag=direct conv=fsync
```

Replace `/dev/sdX` with your actual SD card device. You can find the correct
device using `lsblk` to list all block devices.

> Make sure to unmount any partitions on the SD card before flashing.
> The device should be `/dev/sdX` (the whole disk), not `/dev/sdX1`
> (a partition).
{: .prompt-warning }

#### Using Balena Etcher

[Balena Etcher][2] provides a graphical interface and works on Linux, 
Windows, and macOS. It includes built-in verification.

1. Download and install Etcher from the [official website][2]
2. Launch Etcher
3. Click "Flash from file" and select your image file
4. Click "Select target" and choose your SD card
5. Click "Flash!" and wait for the process to complete

Etcher will automatically unmount the SD card and verify the written data.

#### Using bmaptool

The `bmaptool` utility is the fastest option for flashing images, as it only
writes data to blocks that actually contain data, skipping empty regions.

Install bmaptool on Debian-based systems:

```console
$ sudo apt install bmap-tools
```

Since Infix does not yet publish `.bmap` files, you need to generate one first:

```console
$ bmaptool create infix-aarch64-bpi-r3.img > infix-aarch64-bpi-r3.bmap
```

Flash the image:

```console
$ sudo bmaptool copy infix-aarch64-bpi-r3.img /dev/sdX
```

The tool will automatically find and use the `.bmap` file if it exists in the
same directory as the image.

### Windows

[Balena Etcher][2] is recommended for Windows:

1. Download and install Etcher from the [official website][2]
2. Insert your SD card
3. Launch Etcher
4. Click "Flash from file" and select your image file
5. Click "Select target" and choose your SD card
6. Click "Flash!" and wait for the process to complete

An alternative is [Win32 Disk Imager][3], which provides similar
functionality.

### Verifying the Flash

After flashing, safely eject the SD card from your computer and insert it
into your target device. The system should boot and be accessible via serial
console or SSH to the hostname advertised over mDNS.

The default credentials and network configuration are described in the
[getting started guide][4].

### Troubleshooting

If the device does not boot:

- Verify you wrote to the correct device (not a partition)
- Check that the SD card is properly seated in the slot
- Try a different SD card (some cards are incompatible with certain devices)
- Verify the image is for the correct board
- Check the serial console output for error messages

For additional help, consult the [documentation][5] or ask on the
[community Discord][6].

### Shell Script Example

For regular flashing tasks on Linux, a shell script with safety checks can
be helpful:

```bash
#!/bin/sh
# flash.sh - Write image file to SD card with safety checks

yorn=
file=
dev=

fatal()
{
    printf "Error: %s\n" "$*" >&2
    exit 1
}

note()
{
    printf "%s\n" "$*"
}

yorn()
{
    [ -n "$yorn" ] && return 0
    printf "%s, are you sure (y/N)? " "$1"
    read -r answer
    [ "$answer" = "y" ] || [ "$answer" = "Y" ]
}

# Parse arguments
while getopts "y" opt; do
    case "$opt" in
        y) yorn=y ;;
        *) echo "Usage: flash [-y] DEV FILE" >&2; exit 1 ;;
    esac
done
shift $((OPTIND-1))

# Get device and file
if [ $# -eq 2 ]; then
    dev=$1
    file=$2
elif [ $# -eq 1 ]; then
    file=$1
    # Try to guess SD card device
    guess_dev=$(lsblk -ndo NAME,RM,MOUNTPOINT | awk '$2=="1" && $3=="" { print $1 }')
    count=$(printf "%s\n" "$guess_dev" | wc -l)
    if [ "$count" -eq 1 ]; then
        dev="/dev/$guess_dev"
        note "Guessed SD card device: $dev"
    else
        fatal "Unable to safely guess SD card device."
    fi
else
    echo "Usage: flash [-y] DEV FILE" >&2
    exit 1
fi

[ -b "$dev" ]  || fatal "$dev is not a block device."
[ -f "$file" ] || fatal "$file does not exist."

# Safety check for system drives
case "$dev" in
    /dev/sda* | /dev/nvme0n1* | /dev/vda*)
        printf "WARNING: %s looks like a system drive!\n" "$dev" >&2
        yorn "Flash to $dev" || exit 1
        ;;
esac

# Check if mounted
if mountpoint -q "$dev" 2>/dev/null; then
    fatal "$dev is mounted. Unmount before proceeding."
fi

yorn "Write $file to $dev" || exit 0

note "Writing $file to $dev..."
if ! dd if="$file" of="$dev" bs=1M status=progress oflag=direct conv=fsync; then
    fatal "dd failed to write image."
fi

note "Done."
```

Save this as `flash.sh`, make it executable with `chmod +x flash.sh`, and use:

```console
$ ./flash.sh /dev/sdX infix-aarch64-bpi-r3.img
```

The script includes automatic device detection if you omit the device
argument, and safety checks to prevent accidentally overwriting system
drives.

[1]: https://github.com/kernelkit/infix/releases/tag/latest-boot
[2]: https://etcher.balena.io/
[3]: https://win32diskimager.org/
[4]: /posts/getting-started/
[5]: https://github.com/kernelkit/infix/tree/main/doc
[6]: https://discord.gg/6bHJWQNVxN
