# Arch Linux Installation Documentation

Course: CYB 2013 – Secure Software Development II  
Student: Alex Anagnost  
Operating System: Arch Linux  
Virtualization Software: QEMU  
Host Operating System: Windows 11 ARM64

## 1. Introduction

The purpose of this assignment was to manually install and configure Arch Linux inside a virtual machine by following the Arch Linux Installation Guide. The installation was completed without using the automated `archinstall` installer.
My computer runs Windows on an ARM64 processor, specifically a Qualcomm Snapdragon X Elite. This created compatibility challenges because the standard Arch Linux installation image is designed for x86_64 computers.
I initially attempted to use Oracle VirtualBox but encountered problems booting the Arch Linux installation environment. I eventually switched to QEMU, which allowed me to emulate an x86_64 computer on my ARM-based Windows system.
Throughout the installation, I documented the commands used, configuration decisions, errors encountered, and solutions applied.

## 2. Virtual Machine Configuration

I used the following virtual machine configuration:

| Component | Configuration |
|---|---|
| Host OS | Windows ARM64 |
| Processor | Qualcomm Snapdragon X Elite |
| Virtualization software | QEMU |
| Guest OS | Arch Linux x86_64 |
| Virtual disk | 40 GB QCOW2 |
| RAM | 4 GB |
| Virtual CPUs | 2 |
| Firmware | Legacy BIOS |
| Network adapter | Intel E1000 emulation |

### Configuration choices

I selected QEMU because my ARM-based computer could not successfully run the Arch Linux installation through my earlier VirtualBox setup.

I allocated 4 GB of RAM, two virtual CPUs, and a 40 GB virtual disk. These resources were sufficient for a command-line Arch Linux installation.

I used legacy BIOS booting and an MBR partition table rather than UEFI and GPT.

### Issues encountered

**Problem:** VirtualBox repeatedly failed to complete the Arch Linux boot process.

**Solution:** I switched to QEMU using MSYS2 CLANGARM64, allowing the x86_64 Arch Linux installation environment to run through software emulation.

**Result:** The Arch Linux installer booted successfully, allowing me to continue with manual installation.
