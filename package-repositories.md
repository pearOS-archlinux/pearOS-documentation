# Package repositories

pearOS NiceC0re is Arch Linux underneath, with three extra repositories layered
on top. Understanding the order they are listed in explains most of what pacman
does on a pearOS system.

## What is enabled

```ini
[pearos]     https://mirror.pearos.xyz/main/x86_64/
[inled]      https://apt.inled.es/arch/
[core]       Arch mirrors
[extra]      Arch mirrors
[cachyos]    https://mirror.cachyos.org/repo/$arch/$repo
[multilib]   Arch mirrors
```

| Repository | What it provides |
| --- | --- |
| `pearos` | Everything pearOS-branded: the desktop, Filer, Seafari, Calamares and its configuration, the keyrings |
| `inled` | A small set of third-party packages |
| `core`, `extra`, `multilib` | The Arch base. The overwhelming majority of the system |
| `cachyos` | The kernel, `chwd`, `zfs-utils`, and the legacy NVIDIA driver branches |

Roughly 1080 of the packages on a pearOS system come from Arch, about 30 from
`pearos`, and a handful from `cachyos`.

## Order decides the winner, not the version

This is the part that surprises people:

::: warning
pacman searches repositories **in the order they appear** in `pacman.conf` and
takes the first one that has the package. The version number is irrelevant in
that comparison.
:::

Because `[pearos]` is listed first, a pearOS package always wins, even when
another repository ships a newer version of the same name. That is the whole
mechanism protecting pearOS from being overwritten. For example, `plymouth` is
held at the pearOS build on purpose, and pacman keeps it there even though Arch
carries a much newer release.

The same rule cuts the other way, which is why `[cachyos]` sits **below**
`[extra]`.

::: danger Never move [cachyos] above [core] or [extra]
The CachyOS repository contains its own rebuilds of `mesa`, `vulkan-*`,
`pacman` and `linux-firmware`. Placed above the Arch repositories it would
replace the Arch base with CachyOS builds, package by package, on the next
upgrade. Below them, only what Arch does not have is taken from it: the kernel,
`chwd` and the legacy NVIDIA branches.
:::

## Checking where a package comes from

```sh
pacman -Si mesa | head -3
```

The `Repository` line tells you which one pacman would use. To see every
repository carrying a name:

```sh
pacman -Ss '^mesa$'
```

## Repository keys

Each repository is signed, and `SigLevel = Required DatabaseOptional` means
pacman refuses any package whose signature it cannot verify.

The CachyOS keyring is republished in the pearOS repository and signed with the
pearOS key, which your system already trusts. That is what makes enabling
`[cachyos]` safe: the key arrives through a channel you already verify, rather
than being downloaded with signature checking turned off.

```sh
sudo pacman -S cachyos-keyring
sudo pacman-key --populate cachyos
```

If signature verification ever fails across the board, the usual cause is a
stale keyring rather than a compromised mirror:

```sh
sudo pacman -S archlinux-keyring pearos-keyring cachyos-keyring
sudo pacman-key --populate
```

## Mirrors

The installed system refreshes its Arch mirror list with `reflector`, which
ranks mirrors by measured download speed **from your machine**. A mirror that is
fast in Germany can be unusable from Brazil, and only a local measurement can
tell the difference.

To refresh manually:

```sh
sudo reflector --save /etc/pacman.d/mirrorlist \
               --protocol https --age 12 --completion-percent 100 \
               --score 50 --sort rate --number 25
```

::: tip Symptoms of a bad mirror
`Operation too slow. Less than 1 bytes/sec transferred the last 10 seconds`,
followed by `failed to commit transaction (download library error)`. Nothing is
broken on your system. Refresh the mirror list and retry.
:::
