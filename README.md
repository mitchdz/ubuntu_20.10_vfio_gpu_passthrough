# ubuntu_2004_vfio_guide

# Hardware
CPU             : i9-9900KF
MOBO            : MSI MPG Z390 Gaming Edge AC (MS-7B17)
GPU     (GUEST) : RTX 2080
GPU     (HOST)  : random MSI GPU
Storage (GUEST) : NVME M.2 Drive
Storage (HOST)  : SATA SSD Drive

# Setup
1. Put Host GPU is in PCIe slot 1
2. Put Guest GPU in PCIe slot 2.
3. Plug one monitor into first GPU, second monitor into second GPU
4. Install Ubuntu 20.04 into SSD.
5. build and compile linux kernel with ACS patch (only if your mobo does not properly separate IOMMU Groups)

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













