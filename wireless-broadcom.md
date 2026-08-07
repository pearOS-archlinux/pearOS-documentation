# Broadcom Wi-Fi has no driver

Most MacBooks, and a good number of older laptops, ship a Broadcom wireless
card. Linux has two drivers for these: an open one that is built into the
kernel and works badly or not at all on many models, and Broadcom's own `wl`
driver, which has to be compiled against your kernel.

pearOS installs `wl` for you during installation. This page is for when that did
not happen.

## What it looks like

```text
$ sudo modprobe wl
FATAL: Module wl not found

$ sudo modprobe -r bcma wl
FATAL: Module bcma is in use
```

Those are two separate problems in one screen, and the second one is a red
herring.

`Module wl not found` is the real fault. The `broadcom-wl-dkms` package was
installed but its module was never compiled, so there is nothing to load.

`Module bcma is in use` follows from it. The open driver is still bound to your
card, and it cannot be unloaded while another module sits on top of it. That is
normal, and it resolves itself once `wl` exists.

## Why the build was skipped

`broadcom-wl-dkms` is a DKMS package: it ships source code, not a finished
module, and DKMS compiles it against your running kernel at install time. That
needs the kernel headers.

The driver profile asks for the driver and the firmware, and nothing else:

```toml
[broadcom-wl]
packages = 'broadcom-wl-dkms linux-firmware-broadcom'
```

If the headers are not on the system, DKMS has nothing to build against. The
package installs, the build is skipped, and no error reaches the screen. You
only find out later, when the module is missing.

::: tip Fixed for new installs
Installations made with `pear-calamares-config` 26.8.0-9 or newer install the
kernel headers before hardware detection runs, so the build cannot be skipped.
This page applies to machines installed before that.
:::

## The fix

```sh
sudo pacman -S --needed linux-cachyos-lts-headers dkms broadcom-wl-dkms linux-firmware-broadcom
sudo dkms autoinstall
sudo mkinitcpio -P
sudo reboot
```

Reboot rather than juggling modules by hand. Hardware detection has already
written a blacklist that keeps the open driver away from the card:

```text
# /etc/modprobe.d/01-chwd-net-blacklist.conf
blacklist b43
blacklist b43legacy
blacklist ssb
blacklist bcm43xx
blacklist brcm80211
blacklist brcmfmac
blacklist brcmsmac
blacklist bcma
```

On the next boot none of those load, `wl` takes the card, and Wi-Fi appears.

### Without rebooting

If you would rather not restart, unload from the top of the stack downwards.
`bcma` refuses to go while `brcmsmac` sits on it, which is what the
`is in use` message was telling you:

```sh
sudo modprobe -r brcmsmac brcmfmac b43 bcma ssb
sudo modprobe wl
```

If something still refuses to unload, the third column of `lsmod` names whoever
is holding it:

```sh
lsmod | grep -E 'bcma|brcm|b43|ssb'
```

## Checking your card is supported

```sh
lspci -nn -d 14e4:
```

The `wl` driver covers these device IDs:

```text
4311 4312 4315 4727 4328 4329 432A 432B 432C 432D 0576
4353 4357 4358 4359 4365 4331 43B1 43A0 4360
```

If the four hex digits after `14e4:` are not in that list, automatic detection
skipped your card on purpose, and installing `broadcom-wl-dkms` will not help.
Newer Broadcom cards are handled by `brcmfmac` from the kernel instead, and need
only firmware:

```sh
sudo pacman -S --needed linux-firmware-broadcom
```

## Verifying it worked

```sh
dkms status
lsmod | grep wl
ip link
```

`dkms status` should report the `broadcom-wl` module as `installed` for your
kernel. If it says `added` but not `installed`, the build still has not run:

```sh
sudo dkms autoinstall
```

An empty `lsmod | grep wl` after a reboot, with `dkms status` reporting
`installed`, usually means the blacklist file is missing. Run hardware detection
again and rebuild the initramfs:

```sh
sudo chwd -a -f
sudo mkinitcpio -P
```

::: warning Wired connection first
Every command here downloads packages. On a machine whose only network card is
the broken one, you need ethernet, a USB tether from a phone, or a USB Wi-Fi
adapter to get through this.
:::
