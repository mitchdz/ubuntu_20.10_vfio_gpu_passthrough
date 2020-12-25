# Optimizing Windows10 KVM Guest using OVMF that has a vfio-pci GPU passed through

This document will cover specifically the optimization of a windows 10 guest VM with a vfio-pci passed GPU using the OVMF firmware. The topics covered in this document may break your system or cause other issues.


# Guest OS changes
## Win10 settings
* disable all processes at Startup
	* Task Manager > Startup > [Disable all process]
* disabled mouse acceleration

## Nvidia Control Panel
* Power Magement Mode -> Prefer Maximum Performance
* Low Latency Mode -> On


# VM Configuration Optimizations
## CPU Pinning

CPU Pinning is forcing the Host to not utilize certain cores when the guest VM is running. The reason to do this is because when the Host scheduler changes between the host and guest, the L1/L2 cache is flushed out. Pinning the cores to the guest means that the L1/L2 cache can be fully utilized.

### determining your CPU topology
On your Host, run the command:
```bash
$  lscpu -e
```
```
CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE    MAXMHZ   MINMHZ
  0    0      0    0 0:0:0:0          yes 5000.0000 800.0000
  1    0      0    1 1:1:1:0          yes 5000.0000 800.0000
  2    0      0    2 2:2:2:0          yes 5100.0000 800.0000
  3    0      0    3 3:3:3:0          yes 5100.0000 800.0000
  4    0      0    4 4:4:4:0          yes 5100.0000 800.0000
  5    0      0    5 5:5:5:0          yes 5100.0000 800.0000
  6    0      0    6 6:6:6:0          yes 5100.0000 800.0000
  7    0      0    7 7:7:7:0          yes 5100.0000 800.0000
  8    0      0    0 0:0:0:0          yes 5100.0000 800.0000
  9    0      0    1 1:1:1:0          yes 5100.0000 800.0000
 10    0      0    2 2:2:2:0          yes 5100.0000 800.0000
 11    0      0    3 3:3:3:0          yes 5100.0000 800.0000
 12    0      0    4 4:4:4:0          yes 5100.0000 800.0000
 13    0      0    5 5:5:5:0          yes 5100.0000 800.0000
 14    0      0    6 6:6:6:0          yes 5100.0000 800.0000
 15    0      0    7 7:7:7:0          yes 5100.0000 800.0000
```

### Pinning CPU cores to virtual CPU
The goal is to leave core 0 for the host, and every other core for the guest. Again, to do this we need to utilize virsh edit

```bash
virsh edit vm10
```
```xml
...
  <vcpu placement='static'>14</vcpu>
  <cputune>
    <vcpupin vcpu='0' cpuset='1'/>
    <vcpupin vcpu='1' cpuset='2'/>
    <vcpupin vcpu='2' cpuset='3'/>
    <vcpupin vcpu='3' cpuset='4'/>
    <vcpupin vcpu='4' cpuset='5'/>
    <vcpupin vcpu='5' cpuset='6'/>
    <vcpupin vcpu='6' cpuset='7'/>
  </cputune>
  <os>
...
```


