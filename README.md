# ubuntu_2004_vfio_guide

# Hardware
| Component Type  | Model |
| --------------  | ----- |
| CPU             | i9-9900KF |
| MOBO            | MSI MPG Z390 Gaming Edge AC (MS-7B17) |
| GPU     (GUEST) | RTX 2080 |
| GPU     (HOST)  | random MSI GPU |
| Storage (GUEST) | NVME M.2 Drive |
| Storage (HOST)  | SATA SSD Drive |

# Setup

First, install the required software:
```
$ sudo apt install qemu-kvm \
                   qemu-utils \
                   libvirt-daemon-system \
                   libvirt-clients \
                   bridge-utils \
                   virt-manager \
                   ovmf \
                   -y
```



1. Put Host GPU is in PCIe slot 1
2. Put Guest GPU in PCIe slot 2
3. Plug one monitor into first GPU, second monitor into second GPU
4. Enable Intel VT-x and VT-d
5. Install Ubuntu 20.04 into SSD
6. build and compile linux kernel with ACS patch (only if your mobo does not properly separate IOMMU Groups)

to do this, simply run this script from mdPlusPlus:
https://gist.github.com/mdPlusPlus/031ec2dac2295c9aaf1fc0b0e808e21a

Choose Keep local version currently installed. Then I selected this for the following:

```
Do you want to get a [s]table, the newest [m]ainline release candidate or the newest kernel from your [r]epositories? [S/m/r] s
Do you want to apply the acs override patch? Kernels below 4.10 are not supported. [Y/n] Y
Do you want to apply the experimental AMD AGESA patch to fix VFIO setups on AGESA 0.0.7.2 and newer? [y/N] n
Do you want to apply the experimental AMD Vega PCI reset patch? [y/N] N
Do you want to install the kernel and its headers after compilation? [Y/n] Y
Do you want to make this kernel the new default? [Y/n] 
Do you want to make this kernel the new default? [Y/n] Y
The newest available stable kernel version is 5.8.12. Kernels below 4.10 are not supported.
Which version do you want to download? [5.8.12]
```
I just pressed enter at the end to install the 5.8.12 kernel in my case

After rebooting, you should see your kernel has updated:
```
mitch@lightning: ~ $ uname -a
Linux lightning 5.8.12-acso #1 SMP Mon Sep 28 17:28:53 MST 2020 x86_64 x86_64 x86_64 GNU/Linux
```


7. enable IOMMU separation and pass GPU PCI ID to vfio

## a) enable IOMMU in grub

```
$ sudo vim /etc/default/grub
```
add `intel_iommu=on iommu=pt` to GRUB_CMDLINE_LINUX. Example below:

```
---
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction"
---
```

## b) update grub
```
$ sudo update-grub
```

## c) reboot the system

```
$ reboot
```

## d) determine GPU PCI ID and add to grub

Determine IOMMU groupings with following command:
```
shopt -s nullglob
for d in /sys/kernel/iommu_groups/{0..999}/devices/*; do
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done;
```

Find the GPU you want to passthrough with its audio controller. Example below:
```
IOMMU Group 0 00:00.0 Host bridge [0600]: Intel Corporation 8th Gen Core 8-core Desktop Processor Host Bridge/DRAM Registers [Coffee Lake S] [8086:3e30] (rev 0a)
IOMMU Group 1 00:01.0 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x16) [8086:1901] (rev 0a)
IOMMU Group 2 00:01.1 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x8) [8086:1905] (rev 0a)
IOMMU Group 3 00:08.0 System peripheral [0880]: Intel Corporation Xeon E3-1200 v5/v6 / E3-1500 v5 / 6th/7th/8th Gen Core Processor Gaussian Mixture Model [8086:1911]
IOMMU Group 4 00:12.0 Signal processing controller [1180]: Intel Corporation Cannon Lake PCH Thermal Controller [8086:a379] (rev 10)
IOMMU Group 5 00:14.0 USB controller [0c03]: Intel Corporation Cannon Lake PCH USB 3.1 xHCI Host Controller [8086:a36d] (rev 10)
IOMMU Group 5 00:14.2 RAM memory [0500]: Intel Corporation Cannon Lake PCH Shared SRAM [8086:a36f] (rev 10)
IOMMU Group 6 00:14.3 Network controller [0280]: Intel Corporation Wireless-AC 9560 [Jefferson Peak] [8086:a370] (rev 10)
IOMMU Group 7 00:16.0 Communication controller [0780]: Intel Corporation Cannon Lake PCH HECI Controller [8086:a360] (rev 10)
IOMMU Group 8 00:17.0 SATA controller [0106]: Intel Corporation Cannon Lake PCH SATA AHCI Controller [8086:a352] (rev 10)
IOMMU Group 9 00:1d.0 PCI bridge [0604]: Intel Corporation Cannon Lake PCH PCI Express Root Port #9 [8086:a330] (rev f0)
IOMMU Group 10 00:1f.0 ISA bridge [0601]: Intel Corporation Z390 Chipset LPC/eSPI Controller [8086:a305] (rev 10)
IOMMU Group 10 00:1f.3 Audio device [0403]: Intel Corporation Cannon Lake PCH cAVS [8086:a348] (rev 10)
IOMMU Group 10 00:1f.4 SMBus [0c05]: Intel Corporation Cannon Lake PCH SMBus Controller [8086:a323] (rev 10)
IOMMU Group 10 00:1f.5 Serial bus controller [0c80]: Intel Corporation Cannon Lake PCH SPI Controller [8086:a324] (rev 10)
IOMMU Group 10 00:1f.6 Ethernet controller [0200]: Intel Corporation Ethernet Connection (7) I219-V [8086:15bc] (rev 10)
IOMMU Group 11 01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GF108 [GeForce GT 730] [10de:0f02] (rev a1)
IOMMU Group 12 01:00.1 Audio device [0403]: NVIDIA Corporation GF108 High Definition Audio Controller [10de:0bea] (rev a1)
IOMMU Group 13 02:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2080 Rev. A] [10de:1e87] (rev a1)
IOMMU Group 14 02:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
IOMMU Group 15 02:00.2 USB controller [0c03]: NVIDIA Corporation TU104 USB 3.1 Host Controller [10de:1ad8] (rev a1)
IOMMU Group 16 02:00.3 Serial bus controller [0c80]: NVIDIA Corporation TU104 USB Type-C UCSI Controller [10de:1ad9] (rev a1)
IOMMU Group 17 03:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983 [144d:a808]
```

I am paying attention to the numbers before (rev a1), those are the PCI device IDs. I want to tell vfio to grab these PCI devices. So again we will modify /etc/default/grub by adding `vfio-pci.ids=10de:1e87,10de:10f8` to GRUB_CMDLINE_LINUX as such:

```
---
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction vfio-pci.ids=10de:1e87,10de:10f8"
---
```

## e) update grub and reboot again

```
$ sudo update-grub
$ reboot
```


## f) verify vfio-pci is controlling GPU


Run the following command to list pci devices and their info:

```
$ lspci -nnv
```

```
---
02:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2080 Rev. A] [10de:1e87] (rev a1) (prog-if 00 [VGA controller])
	Subsystem: ASUSTeK Computer Inc. TU104 [GeForce RTX 2080 Rev. A] [1043:8660]
	Flags: fast devsel, IRQ 10
	Memory at b2000000 (32-bit, non-prefetchable) [disabled] [size=16M]
	Memory at 90000000 (64-bit, prefetchable) [disabled] [size=256M]
	Memory at a0000000 (64-bit, prefetchable) [disabled] [size=32M]
	I/O ports at 3000 [disabled] [size=128]
	Expansion ROM at b3000000 [disabled] [size=512K]
	Capabilities: <access denied>
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau

02:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
	Subsystem: ASUSTeK Computer Inc. TU104 HD Audio Controller [1043:8660]
	Flags: fast devsel, IRQ 11
	Memory at b3080000 (32-bit, non-prefetchable) [disabled] [size=16K]
	Capabilities: <access denied>
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
---
```


Congrats! vfio is now controlling the GPU. Now we can begin creating the Windows 10 VM



8. Have Windows 10 ISO and virtio drivers downloaded








