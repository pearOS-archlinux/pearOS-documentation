# Hardware Limitations

With pearOS, there are numerous hardware limitations you need to be aware of before stepping foot into an installation.

The main hardware sections to verify are:

[[toc]]

And for more detailed guides on the subject, see here:

* [GPU Buyers Guide](https://dortania.github.io/GPU-Buyers-Guide/)
  * Check if your GPU is supported and which macOS version you can run.
* [Wireless Buyers Guide](https://dortania.github.io/Wireless-Buyers-Guide/)
  * Check if your WiFi card is supported.
* [Anti-Hardware Buyers Guide](https://dortania.github.io/Anti-Hackintosh-Buyers-Guide/)
  * Overall guide on what to avoid and what pitfalls your hardware may hit.

## CPU Support

For CPU support, we have the following breakdown:

* Both 32 and 64-bit CPUs are supported
  * This however requires the OS to support your architecture, see CPU Requirements section below
* Intel's Desktop CPUs are supported.
  * Yonah through Comet Lake are supported by this guide.
* Intel's High-End Desktops and Server CPUs.
  * Nehalem through Cascade Lake X are supported by this guide.
* Intel's Core "i" and Xeon series laptop CPUs
  * Arrandale through Ice Lake are supported by this guide.
  * Note that Mobile Atoms, Celeron and Pentium CPUs are not supported
* AMD's Desktop Bulldozer (15h), Jaguar (16h) and Ryzen (17h) CPUs
  * Laptop CPUs are **not** supported
  * Note not all features of macOS are supported with AMD, see below

**For more in-depth information, see here: [Anti-Hardware Buyers Guide](https://dortania.github.io/Anti-Hackintosh-Buyers-Guide/)**

::: details CPU Requirements

Architecture Requirements

* 32-bit CPUs are NOT supported
* 64-bit CPUs are supported from 10.4.1 to current


::: details macOS Ventura and AVX2 Details

macOS Ventura drops support for pre-Haswell CPUs. Much of userspace now requires AVX2 support, along with AMD Polaris GPU drivers and some instances of AVX2 instructions in some kexts. Although the kexts can be [patched](https://forums.macrumors.com/threads/monterand-probably-the-start-of-an-ongoing-saga.2320479/post-31125212) or [downgraded](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/92ff4244ae78de715977d9f8d054cdf9bdce4011/payloads/Kexts/Misc/NoAVXFSCompressionTypeZlib-AVXpel-v12.6.zip), the Polaris GPU drivers and most of userspace rely on AVX2 too much to be able to be patched.

Apple has left a dyld cache that does not use AVX2 instructions in Ventura to support Rosetta on Apple Silicon machines, but this cache is not installed by default. You can use [CryptexFixup](https://github.com/acidanthera/CryptexFixup) to force this dyld cache to be installed, but:

* Apple may remove this cache at any time in the future if they add AVX2 support to Rosetta
* Delta updates (small 1-3GB updates) will no longer be available and you must install the full update (12GB), as delta updates only contain the non-AVX2 cache on Apple Silicon machines
* Polaris GPUs remain unsupported on machines without AVX2

Because of these caveats, the Dortania guide will no longer be supporting pre-Haswell CPUs for Ventura and above. The pages for these CPUs will remain updated for Monterey.



## GPU Support

::: tip

Please see the [GPU Buyers Guide](https://dortania.github.io/GPU-Buyers-Guide/) for information about compatible GPUs.

:::

And an important note for **Laptops with discrete GPUs**:

* 90% of discrete GPUs will not work because they are wired in a configuration that macOS doesn't support (switchable graphics). With NVIDIA discrete GPUs, this is usually called Optimus. It is not possible to utilize these discrete GPUs for the internal display, so it is generally advised to disable them and power them off (will be covered later in this guide).
* However, in some cases, the discrete GPU powers any external outputs (HDMI, mini DisplayPort, etc.), which may or may not work; in the case that it will work, you will have to keep the card on and running.
* However, there are some laptops that rarely do not have switchable graphics, so the discrete card can be used (if supported by macOS), but the wiring and setup usually cause issues.

## Motherboard Support

For the most part, all motherboards are supported as long as the CPU is.

::: details MSI 500-series AMD motherboards note

~~The exception is MSI 500-series AMD motherboards (A520, B550, and X570). These motherboards have issues with macOS Monterey and above:~~

* ~~PCIe devices are not always enumerated properly~~
* ~~The BIOS update for Zen 3 support breaks boot~~

~~macOS Big Sur or earlier is recommended for these motherboards.~~

Thanks to CaseySJ, this has been fixed in the latest version of the AMD vanilla patches!

:::

## Storage Support

For the most part, all SATA based drives are supported and the majority of NVMe drives as well. There are only a few exceptions:

* **Samsung PM981, PM991 and Micron 2200S NVMe SSDs**
  * These SSDs are not compatible out of the box (causing kernel panics) and therefore require [NVMeFix.kext](https://github.com/acidanthera/NVMeFix/releases) to fix these kernel panics. Note that these drives may still cause boot issues even with NVMeFix.kext.
  * On a related note, Samsung 970 EVO Plus NVMe SSDs also had the same problem but it was fixed in a firmware update; get the update (Windows via Samsung Magician or bootable ISO) [here](https://www.samsung.com/semiconductor/minisite/ssd/download/tools/).
  * Also to note, laptops that use [Intel Optane Memory](https://www.intel.com/content/www/us/en/architecture-and-technology/optane-memory.html) or [Micron 3D XPoint](https://www.micron.com/products/advanced-solutions/3d-xpoint-technology) for HDD acceleration are unsupported in macOS. Some users have reported success in Catalina with even read and write support but we highly recommend removing the drive to prevent any potential boot issues.
    * Note that Intel Optane Memory H10/H20 models are compatible if the Optane part is disabled in macOS. More information can be found [here](https://blog.csdn.net/weixin_46341175/article/details/126626808) ([original Chinese source](https://zhuanlan.zhihu.com/p/429073173)).
  
* **Intel 600p**
  * While not unbootable, please be aware this model can cause numerous problems. [Any fix for Intel 600p NVMe Drive? #1286](https://github.com/acidanthera/bugtracker/issues/1286)
  * The 660p model is fine

## Wired Networking

Virtually all wired network adapters have some form of support in macOS, either by the built-in drivers or community made kexts. The main exceptions:

* Intel I225-V 2.5Gb NIC
  * Found on high-end Desktop Comet Lake boards
  * Requires device properties: [Source](https://www.hackintosh-forum.de/forum/thread/48568-i9-10900k-gigabyte-z490-vision-d-er-läuft/?postID=606059#post606059) and [Example](config.plist/comet-lake.md#deviceproperties)
* Intel I350 1Gb server NIC
  * Normally found on Intel and Supermicro server boards of various generations
  * [Requires device properties](config-HEDT/ivy-bridge-e.md#deviceproperties)
* Intel 10Gb server NICs
  * Workarounds are possible for [X520 and X540 chipsets](https://www.tonymacx86.com/threads/how-to-build-your-own-imac-pro-successful-build-extended-guide.229353/)
* Mellanox and Qlogic server NICs

## Wireless Networking

Most WiFi cards that come with laptops are not supported as they are usually Intel/Qualcomm. If you are lucky, you may have a supported Atheros card, but support only runs up to High Sierra.

The best option is getting a supported Broadcom card; see the [WiFi Buyer's Guide](https://dortania.github.io/Wireless-Buyers-Guide/) for recommendations.

Note: Intel WiFi is unofficially (3rd party driver) supported on macOS, check [WiFi Buyer's Guide](https://dortania.github.io/Wireless-Buyers-Guide/) for more information about the drivers and supported cards.

## Miscellaneous

* **Fingerprint sensors**
  * There is currently no way to emulate the Touch ID sensor, so fingerprint sensors will not work.
* **Windows Hello Face Recognition**
  * Some laptops come with WHFR that is I2C connected (and used through your iGPU), those will not work.
  * Some laptops come with WHFR that is USB connected, if you're lucky, you may get camera functionality, but nothing else.
* **Intel Smart Sound Technology**
  * Laptops with Intel SST will not have anything connected through them (usually internal mic) work, as it is not supported. You can check with Device Manager on Windows.
* **Headphone Jack Combo**
  * Some laptops with a combo headphone jack may not get audio input through them and will have to either use the built-in microphone or an external audio input device through USB.
* **Thunderbolt USB-C ports**
  * (Hackintosh) Thunderbolt support is currently still iffy in macOS, even more so with Alpine Ridge controllers, which most current laptops have. There have been attempts to keep the controller powered on, which allows Thunderbolt and USB-C hotplug to work, but it comes at the cost of kernel panics and/or USB-C breaking after sleep. If you want to use the USB-C side of the port and be able to sleep, you must plug it in at boot and keep it plugged in.
  * Note: This does not apply to USB-C only ports - only Thunderbolt 3 and USB-C combined ports.
  * Disabling Thunderbolt in the BIOS will also resolve this.
