# Update problems on older installs

pearOS NiceC0re moved from a Manjaro package base to a pure Arch base. Systems
installed before that change carry a mix of the two, and the seams show up as
update failures.

If you installed recently, none of this applies to you.

::: tip The one-command fix
Almost everything on this page is repaired automatically:

```sh
sudo pacman -Sy pear-temp-fix
```

It enables the CachyOS repository, refreshes your mirrors, repairs the kernel
file naming and realigns your firmware packages. The work happens once, on the
next boot, and the package then leaves your system alone.

The rest of this page explains what it is fixing and how to do it by hand.
:::

## Downloads stall and the upgrade aborts

```text
error: failed retrieving file 'qt6-tools-...pkg.tar.zst.sig' from mirror.23m.com :
Operation too slow. Less than 1 bytes/sec transferred the last 10 seconds
warning: too many errors from mirror.23m.com, skipping
error: failed to commit transaction (download library error)
```

Older images shipped a static mirror list and never refreshed it, because
`reflector` was not actually installed. Mirrors that were healthy at release
have since become slow or unreachable.

```sh
sudo pacman -S reflector
sudo reflector --save /etc/pacman.d/mirrorlist \
               --protocol https --age 12 --completion-percent 100 \
               --score 50 --sort rate --number 25
sudo pacman -Syu
```

## Dependency conflicts on gstreamer and friends

```text
:: installing gst-plugins-base-libs (1.28.5-4) breaks dependency
   'gst-plugins-base-libs=1.28.5-2' required by gst-plugins-base
```

Manjaro rebuilds some Arch packages keeping the exact same version string.
Your system has the Manjaro build, the repositories now offer the Arch one, and
any **partial** upgrade lifts half the set and leaves the other half behind.

The fix is to stop doing partial upgrades:

```sh
sudo pacman -Syu
```

::: danger Never use pacman -Sy on its own
`pacman -Sy <package>` refreshes the databases and then installs one package
against them, leaving the rest of the system behind. On a system that is
already mixed, that is exactly what produces these conflicts. Always `-Syu`.
:::

## Firmware that will not update

```text
warning: downgrading package linux-firmware-intel (1:20260309-1 => 20260622-1)
```

The pearOS repository used to ship `linux-firmware` with an epoch of `1`. Epoch
outranks version, so `1:20260309-1` is considered newer than Arch's
`20260622-1`, and pacman will never upgrade it on its own. The practical effect
is that your firmware is frozen at March 2026, which shows up as warnings like:

```text
==> WARNING: Possibly missing firmware for module: 'qla2xxx' 'aic94xx' 'bfa'
```

To realign with the repository baseline:

```sh
sudo pacman -S --noconfirm $(pacman -Qq | grep '^linux-firmware')
```

Despite the word *downgrading*, this moves you to **newer** firmware. Only the
epoch number goes down.

## The kernel time bomb

This one is worth checking even if nothing is currently broken.

```sh
pacman -Qo /boot/vmlinuz-linux
```

If that says `No package owns /boot/vmlinuz-linux`, read on. If it names a
package, or the file does not exist, you are fine.

Older images installed the kernel under a generic name, `vmlinuz-linux`, owned
by no package. Your machine boots today because that file **is** the CachyOS
kernel under a different name, and it finds its modules.

The next kernel update ends that. pacman writes the kernel to its real name,
`vmlinuz-linux-cachyos-lts`, and removes the old modules directory, while GRUB
keeps booting the stale `vmlinuz-linux`. A kernel without its modules does not
get you to a desktop.

::: danger Do not reboot in the middle of this
Step 3 deletes the kernel file GRUB is currently using. Run all four steps in
one sitting. If the machine restarts between step 3 and step 4, it will not
boot.
:::

```sh
# 1. reinstall the kernel; the hook writes the correctly named files
sudo pacman -S --noconfirm linux-cachyos-lts

# 2. confirm the new kernel exists before deleting anything
ls -la /boot/vmlinuz-linux-cachyos-lts

# 3. only if step 2 succeeded, remove the orphans
sudo rm -f /boot/vmlinuz-linux /boot/initramfs-linux.img \
           /boot/initramfs-linux-fallback.img /etc/mkinitcpio.d/linux.preset

# 4. rebuild the initramfs and the boot menu
sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

If step 2 produces nothing, stop there and ask for help on
[Discord](https://discord.gg/ZX7GnqKWNr). Deleting the old kernel at that point
would leave the machine with none.

## A hook fails during every upgrade

```text
==> Initcpio image generation successful
error: command failed to execute correctly
```

A leftover `linux.preset` describes a kernel that no longer exists, so
`mkinitcpio -P` succeeds for the real kernel and fails for the phantom one. The
transaction itself completes; only the hook fails. Removing the orphaned preset,
as in step 3 above, clears it.

```sh
ls /etc/mkinitcpio.d/
```

You should see exactly one preset, matching your installed kernel.
