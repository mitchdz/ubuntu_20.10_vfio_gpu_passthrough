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
---
02:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU104 [GeForce RTX 2080 Rev. A] [10de:1e87] (rev a1)
02:00.1 Audio device [0403]: NVIDIA Corporation TU104 HD Audio Controller [10de:10f8] (rev a1)
---
```

I am paying attention to the numbers before (rev a1), those are the PCI device IDs. I want to tell vfio to grab these PCI devices. So again we will modify /etc/default/grub as such:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction vfio-pci.ids=10de:1e87,10de:10f8"
```

## e) update grub

```
$ sudo update-grub
```


