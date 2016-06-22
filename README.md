# A fork work of [attila-lendvai](https://github.com/attila-lendvai/openwrt-auto-extroot)

# New Feature:
- vagrant support
- 15.05.1 support

# usage:
- clone the repo
- run vagrant up
- a ready made TLMR3020 will be ready
- to change router firmware just edit the vagrant file

# What

It's a script to build a customized OpenWRT firmware image
(basic familiarity with [OpenWRT](https://wiki.openwrt.org/doc/howto/user.beginner)
is assumed).

If the generated image is flashed on a device it will try to automatically
set up [extroot](http://wiki.openwrt.org/doc/howto/extroot) on **any
(!)** storage device plugged into the USB port (`/dev/sda`). Keep in
mind that **this will erase any inserted storage device while the
router is in the initial setup phase**! Unfortunately there's little
that can be done at that point to ask the user for confirmation.

# Why

So that e.g. customers can buy a router on their own, flash our custom
firmware, plug in a pendrive, and manage their SIP (telephony) node
from our webapp.

# How
### Building

e.g. `./build.sh TLWDR4300`

Results will be under `build/OpenWrt-ImageBuilder-ar71xx_generic-for-linux-x86_64`.

To see a list of available targets, run this in the ImageBuilder dir: ```make info```.

### Setup stages

Blinking leds show which phase the extroot setup scripts are in. Consult the
sources for details: [autoprovision-functions.sh](image-extras/common/root/autoprovision-functions.sh#L49).

#### Stage 1: setup extroot

At the first boot after flashing the firmware the autoprovision script will
wait for anything (!) in `/dev/sda` to show up (that is >= 512M), then erase
it and set up a `swap`, an `extroot`, and a `data`filesystem (for the remaining
space), and then reboot.

#### Stage 2: download and install some packages from the internet

Once it booted into the new extroot, it will continuously attempt to install
some OpenWRT packages until an internet connection is set up on the router
(either by using ssh or LuCI if you could fit it into the firmware).

### Login

After flashing the firmware the router will have the standard
`192.168.1.1` IP address.

By default the root passwd is not set, so the router will start telnet with
no password. If you want to set up a password, then edit the stage 2 script:
[autoprovision-stage2.sh](image-extras/common/root/autoprovision-stage2.sh#L53).

If a password is set, then telnet is disabled by OpenWRT and SSH will listen
using the keys specified in [authorized_keys](image-extras/common/etc/dropbear/authorized_keys).

Once connected, you can read the log with `logread -f`.

# Status

This is more of a template than something standalone. You most
probably want to customize this script here and there; search for
`CUSTOMIZE` for places of interest.

Most importantly, **set up a password and maybe an ssh key**.

I've extracted this from a project of mine where OpenWRT nodes auto-provision
themselves in 3 stages (stage 3 was a Python script for an app-level sync feature),
but I thought it's useful enough for making it public.

At the time of writing it only supports a few `ar71xx` routers out of the box,
but it's easy to extend it.

## Tested with

[OpenWRT Chaos Calmer 15.05 RC1](https://downloads.openwrt.org/chaos_calmer/15.05-rc1/)
on a TP-Link WDR4300.

# Troubleshooting

## Which file should I flash?

You should consult the [OpenWRT documentation](https://wiki.openwrt.org/doc/howto/user.beginner).
The produced firmware files should be somewhere around ```build/OpenWrt-ImageBuilder-15.05-ar71xx-generic.Linux-x86_64/bin/ar71xx```.

In short:

* You need a file with the name ```-factory.bin``` or ```-sysupgrade.bin```. The former is to
  be used when you first install OpenWRT, the latter is when you upgrade an already installed
  OpenWRT.

* You must carefully pick the proper firmware file for your **hardware version**! I advise you
  to look up the wiki page for your hardware on the [OpenWRT wiki](https://wiki.openwrt.org),
  because most of them have a table of the released hardawre versions with comments on their
  status (sometimes new hardware revisions are only supported by the latest OpenWRT, which is
  not released yet).

## Help! The build has finished but there's no firmware file!

If the build doesn't yield a firmware file (```*-factory.bin``` and/or ```*-sysupgrade.bin```):
when there's not enough space in the flash memory of the target device to install everything
then the OpenWRT ImageBuilder prints a hardly visible error into its flow of output and
silently continues. Look into [build.sh](build.sh#L31) and try to remove some packages
that you can live without.
