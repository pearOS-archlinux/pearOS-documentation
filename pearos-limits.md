# Hardware Limitations

With pearOS, there are numerous hardware limitations you need to be aware of before stepping foot into an installation.

The main hardware sections to verify are:

[[toc]]

## CPU Support

pearOS is Arch-based Linux, so CPU support is broad. The real limits come from the ISO build itself:

* **64-bit (x86_64) only** — the ISO is built exclusively for `x86_64`; **32-bit (i686) CPUs are NOT supported**, no fallback image is produced.
* **Intel** — any x86_64 Intel CPU works, from old Core 2 era through current. Microcode updates are handled by the bundled `intel-ucode` package.
* **AMD** — any x86_64 AMD CPU works, same story. Microcode updates come from `amd-ucode`.
* No distinction between desktop/laptop/HEDT CPUs — if it's x86_64, the OS boots and runs.

Features aren't gated by CPU generation — everything that's a kernel/driver-level feature (GPU accel, wireless, etc.) depends on the *specific hardware component*, not the CPU family. See the sections below for those.

::: details CPU Requirements

* 32-bit (i686) CPUs are **NOT** supported — the ISO has no i686 build target.
* Any 64-bit (x86_64) Intel or AMD CPU is supported.

:::

## GPU Support

pearOS ships the following GPU stack by default:

* **Intel & AMD** — open-source `mesa` drivers (i915/iris, amdgpu/radeonsi), no extra setup needed.
* **NVIDIA** — `nvidia-utils` + `linux-cachyos-lts-nvidia-open` (open kernel module, Turing and newer). Older (pre-Turing) cards need the legacy proprietary driver, not included by default.
* **Virtualized/VM GPUs** — `vulkan-virtio` for virtio-gpu passthrough in VMs.

## Storage Support

pearOS runs on the Linux kernel, so storage support is native and broad.

* **SATA, NVMe, USB, eMMC** — all supported out of the box via the kernel's own drivers.
* **Filesystems** — ext4 by default, plus `btrfs-progs`, `xfsprogs`, `f2fs-tools`, `dosfstools`, `exfatprogs`, and `ntfs-3g` (NTFS read/write) are bundled.
* **RAID/LVM/Encryption** — `mdadm` (software RAID), `lvm2` (logical volumes), and `cryptsetup` (LUKS full-disk encryption) are all included.
* **NVMe tooling** — `nvme-cli` is bundled for drive health/management.

## Wired Networking

Virtually all wired network adapters work out of the box in pearOS

## Wireless Networking

Since pearOS runs the standard Linux kernel, WiFi support is broad — Intel, Qualcomm Atheros, Broadcom, MediaTek and Realtek cards all work with their native `mac80211`-based kernel drivers. Bluetooth follows the same coverage as the WiFi chip on combo cards.

Firmware for most chipsets ships via `linux-firmware`; a handful of Broadcom/MediaTek chips may need vendor-specific firmware packages installed manually if not covered.

## Miscellaneous

* **Fingerprint sensors**
  * Fingerprint login works through `libfprint`/`fprintd`, provided your sensor is supported. See the list below.

::: details Supported Fingerprint Sensors (libfprint)

| Vendor | Device Name | USB ID | Driver |
|--------|------------|--------|--------|
| Microsoft | Digital Persona U.are.U 4000/4000B/4500 | 045e:00bb, 045e:00bc, 045e:00bd, 045e:00ca | Digital Persona |
| UPEK | TouchChip/Eikon Touch 300 | 0483:2015, 0483:2017 | UPEK TouchChip |
| UPEK | TouchStrip | 0483:2016 | UPEK TouchStrip |
| ElanTech | Fingerprint Sensor | 04f3:0903, 04f3:0907, 04f3:0c01-0c33, 04f3:0c3d, 04f3:0c42, 04f3:0c4b, 04f3:0c4d, 04f3:0c4f, 04f3:0c58, 04f3:0c63, 04f3:0c6e | ElanTech |
| ElanTech | MOC Sensors | 04f3:0c7d-0c8d, 04f3:0c98-0c9d, 04f3:0c9f, 04f3:0ca3, 04f3:0ca7-0cb6 | Elan MOC |
| Digital Persona | U.are.U 4000/4000B/4500 | 05ba:0007, 05ba:0008, 05ba:000a | Digital Persona |
| Veridicom | 5thSense | 061a:0110 | Veridicom |
| Synaptics | Sensors | 06cb:00bd-0109, 06cb:010d-010e, 06cb:0123-0124, 06cb:0126, 06cb:0129, 06cb:015f, 06cb:0168-0169, 06cb:016c, 06cb:0173-0174, 06cb:019d, 06cb:019f-01a0, 06cb:01a4 | Synaptics |
| AuthenTec | AES1610 | 08ff:1600 | AuthenTec |
| AuthenTec | AES1660 | 08ff:1660-1689, 08ff:168a-168f | AuthenTec |
| AuthenTec | AES2501 | 08ff:2500, 08ff:2580 | AuthenTec |
| AuthenTec | AES2550/AES2810 | 08ff:2550, 08ff:2810 | AuthenTec |
| AuthenTec | AES2660 | 08ff:2660-268f, 08ff:2691 | AuthenTec |
| AuthenTec | AES4000 | 08ff:5501 | AuthenTec |
| AuthenTec | AES3500 | 08ff:5731 | AuthenTec |
| Realtek | MOC Fingerprint Sensor | 0bda:5813, 0bda:5816, 2541:fa03, 3274:9003 | Realtek MOC |
| FPC | MOC Fingerprint Sensor | 10a5:9524, 10a5:9544, 10a5:9b24, 10a5:a305-a306, 10a5:c844, 10a5:d205, 10a5:d805, 10a5:da04, 10a5:ffe0 | FPC MOC |
| SecuGen | Hamster Pro 20 | 1162:2200 | SecuGen |
| Validity | VFS101 | 138a:0001 | Validity |
| Validity | VFS301 | 138a:0005, 138a:0008 | Validity |
| Validity | VFS5011 | 138a:0010, 138a:0011, 138a:0015-0018 | Validity |
| Validity | VFS0050 | 138a:0050 | Validity |
| Validity | VFS7552 | 138a:0091 | Validity |
| UPEK | TouchStrip Sensor-Only | 147e:1000, 147e:1001 | UPEK |
| UPEK | TouchChip Fingerprint Coprocessor | 147e:2016, 147e:2020, 147e:3001 | UPEK |
| Egis Technology (LighTuning) | 0570 | 1c7a:0570, 1c7a:0571 | Egis Technology |
| Egis Technology (LighTuning) | Match-on-Chip | 1c7a:0582-0588, 1c7a:05a1, 1c7a:05ae, 1c7a:9201 | Egis MOC |
| Egis Technology | ES603 | 1c7a:0603 | EgisTec |
| Goodix | MOC Fingerprint Sensor | 27c6:5840, 27c6:6014, 27c6:6090-6092, 27c6:6094, 27c6:609a, 27c6:609c, 27c6:60a2, 27c6:60a4, 27c6:60bc, 27c6:60c2, 27c6:6304, 27c6:631c, 27c6:633c, 27c6:634c, 27c6:6382, 27c6:6384, 27c6:639c, 27c6:63ac, 27c6:63bc, 27c6:63cc, 27c6:6496, 27c6:650a, 27c6:650c, 27c6:6512, 27c6:6582, 27c6:6584, 27c6:658c, 27c6:6592, 27c6:6594, 27c6:659a, 27c6:659c, 27c6:66a9, 27c6:6890, 27c6:689a, 27c6:6984, 27c6:6a94 | Goodix MOC |
| Focaltech | MOC Sensors | 2808:077a, 2808:079a, 2808:1579, 2808:5158, 2808:6553, 2808:9e48, 2808:a27a, 2808:a57a, 2808:a78a, 2808:a959, 2808:a97a, 2808:a99a, 2808:d979 | Focaltech MOC |
| NextBiometrics | NB-1010-U/NB-2020-U | 298d:1010, 298d:2020 | NextBiometrics |
| MAFP | MOC Fingerprint Sensor | 3274:8012 | MAFP MOC |
| ElanTech | Embedded Fingerprint Sensor (SPI) | ELAN7001/ELAN70A1 | ElanTech Embedded |

Drivers might not all be available in the stable, released version of `libfprint`. Full up-to-date list: [fprint.freedesktop.org/supported-devices.html](https://fprint.freedesktop.org/supported-devices.html)

:::

* **IR/Facial Recognition Cameras**
  * Laptops with an IR camera for facial login generally have no Linux driver for the recognition feature itself, though the camera may still work as a plain webcam.
* **Intel Smart Sound Technology**
  * Handled via `sof-firmware`; internal mics and speakers routed through Intel SST generally work through `pipewire`.
* **Headphone Jack Combo**
  * Some laptops with a combo headphone jack may not get audio input through them and will have to either use the built-in microphone or an external audio input device through USB.
* **Thunderbolt / USB-C**
  * Thunderbolt is supported via the kernel `thunderbolt` driver and the `bolt` daemon (device authorization), with `plasma-thunderbolt` providing the GUI. Hotplug and sleep/wake work as expected.
