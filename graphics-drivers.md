# Graphics drivers

pearOS NiceC0re picks your GPU driver for you. There is no driver choice in the
boot menu, and there is nothing to configure after installing. This page
explains what actually happens, so you know where to look when it goes wrong.

## The short version

| Your GPU | Live session | After installing |
| --- | --- | --- |
| GeForce 8000 – GTX 400 (Fermi) | nouveau | nouveau |
| GTX 600 / 700 (Kepler) | nouveau | NVIDIA 470xx |
| GTX 750 – 1080 Ti (Maxwell, Pascal) | nouveau | NVIDIA 580xx |
| GTX 16xx, RTX 20xx – 40xx (Turing, Ampere, Ada) | NVIDIA open | NVIDIA open |
| RTX 50xx (Blackwell) | NVIDIA open | NVIDIA open |
| AMD | amdgpu | amdgpu |
| Intel | i915 | i915 |

If you have a Pascal card or older, the live session runs on nouveau and feels
slow. That is expected. The proprietary driver for your card is installed
during installation, and the installed system is fast.

## Why the live session differs

The ISO carries only the **open** NVIDIA kernel modules. Those support Turing
and newer. The 470xx and 580xx branches that older cards need are several
hundred megabytes each, and shipping all three would roughly double the size of
the image for a session most people spend ten minutes in.

So the ISO ships one driver and downloads the right one during installation.

## How the live session decides

Nothing is decided at boot time, and nothing is passed on the kernel command
line. Instead, every request to load an NVIDIA module is routed through a small
script:

```sh
# /etc/modprobe.d/nvidia-loader.conf
install nvidia /usr/local/bin/nvidia-module-loader
install nvidia_drm /usr/local/bin/nvidia-module-loader
install nvidia_uvm /usr/local/bin/nvidia-module-loader
install nvidia_modeset /usr/local/bin/nvidia-module-loader
```

The script asks `chwd` what profile your hardware actually needs. If the answer
is one of the older branches, or nouveau, it loads nouveau. If the answer is the
open profile, it loads the NVIDIA modules. If anything goes wrong, it falls back
to nouveau.

::: tip Why not just blacklist nouveau
Because on a pre-Turing card the open NVIDIA module refuses to load. With
nouveau blacklisted the machine would end up with no driver at all and a black
screen. The loader decides after it knows what the card is, which is the only
order that works for every card.
:::

For this to work, no GPU driver may be loaded from the initramfs, so `nouveau`
is deliberately absent from `MODULES` and the `kms` hook is not used. Intel and
AMD are unaffected: they are listed explicitly and still load early, so Plymouth
keeps its native resolution on those machines.

## How the installed system decides

During installation, `chwd --autoconfigure` detects the card and installs the
matching profile, pulling it from the CachyOS repository. Profile priorities
decide the winner when several match:

```text
nouveau            18   old cards, by device ID
nvidia-470xx       14   Kepler
nvidia-580xx       12   Maxwell, Pascal, Volta
nvidia-open        10   everything else NVIDIA
```

Turing and newer get a **prebuilt** module, so nothing is compiled. Kepler,
Maxwell and Pascal get a DKMS module, which is built against your kernel during
installation. That step needs kernel headers, which are installed automatically.

::: warning This step needs an internet connection
The driver for an older card is downloaded during installation. Without a
network connection the installer still finishes, but the machine boots on
nouveau and you will need to run `sudo chwd -a -f` once you are online.
:::

## Boot menu entries

| Entry | Use it when |
| --- | --- |
| pearOS NiceC0re arch (x86_64, UEFI) | Always, unless something is broken |
| No Plymouth (Debug) | The boot splash freezes or crashes |
| Legacy Hardware (GPU nomodeset) | Black screen, or a very old GPU |

The BIOS menu offers the same three entries.

## When something goes wrong

**Black screen right after selecting the first entry.** Reboot and pick
**Legacy Hardware (GPU nomodeset)**. That entry disables kernel modesetting and
blacklists the NVIDIA modules outright, which gets almost anything to a desktop.

**The desktop is slow on an NVIDIA card after installing.** Check what was
actually installed:

```sh
chwd --list-installed
lsmod | grep -E 'nvidia|nouveau'
```

If nouveau is loaded on a card that should be using NVIDIA, run the detection
again:

```sh
sudo chwd -a -f
sudo mkinitcpio -P
```

Then reboot.

**Nothing detected at all.** Confirm the card is visible to the system:

```sh
lspci -d '10de:*:030x' -nn
```

No output means the GPU is not being reported by the firmware, which is a
hardware or BIOS problem rather than a driver one.
