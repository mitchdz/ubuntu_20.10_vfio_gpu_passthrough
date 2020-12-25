# Ubuntu 20.10 VFIO Windows 10 VM with PCI passthrough Guide

These notes document the setup for my personal computer and situation. Your situation may be *and probably is* quite a bit different. Use this guide as a reference.


![3DMark Score](pics/2020_10_1.PNG)

# Goal
The goal of this project is to create a windows 10 VM for gaming with the following:
* low latency
* high stability (no crashes/stutters)
* near-native performance

There are many guides out there that show you how to do this exact setup, but sometimes
those guides sacrifice system stability for extra performance. My goal is to push my system as
far as it can go while being as stable as if the VM was running natively.



# Quickstart
Below are the following links in the repository to get started
* [Project Setup](docs/SETUP.md)
* [Optimization](docs/OPTIMIZATION.md)
* [Software Compatibility](docs/SOFTWARE_COMPATIBILITY.md)


# What is this repository?
This repository documents the process of setting up a windows 10 gaming VM with near-native performance under an Ubuntu 20.04 host. The goal of this guide was to be able to reference this guide later if I want to do a clean build, but also create a guide that anyone could easily follow.


# Acknowledgements
Huge thanks to the following sources:
* [The glorious Arch Wiki PCI Passthrough guide](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF)
* [Mathias Hueber's PCI Passthrough guide](https://mathiashueber.com/pci-passthrough-ubuntu-2004-virtual-machine/)
