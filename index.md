# Arch Linux Installation Documentation

Course: CYB 3353-02 –System Administration 

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



## 3. Downloading Arch Linux and Setting Up QEMU

### Downloading the installation ISO

I downloaded the official Arch Linux x86_64 installation image from an Arch Linux mirror.

**Installation image:** `archlinux-2026.11.01-x86_64.iso`

**Source:** https://archlinux.org/download/

I also checked the downloaded file's checksum to help verify its integrity.

### Installing QEMU

Because my Windows computer uses an ARM64 processor, I installed QEMU through the MSYS2 CLANGARM64 environment.

I used the following command to install QEMU:

```bash
pacman -S mingw-w64-clang-aarch64-qemu
```

This installed the QEMU system emulator needed to run an x86_64 virtual machine.

### Creating the virtual hard drive

I created a 40 GB virtual disk using:

```bash
qemu-img create -f qcow2 archlinux.qcow2 40G
```

**Command explanation:**

- `qemu-img` manages virtual disk images.
- `create` creates a new disk.
- `-f qcow2` specifies the QCOW2 disk format.
- `archlinux.qcow2` is the virtual disk filename.
- `40G` sets the virtual disk capacity to 40 GB.

### Starting the virtual machine

I configured QEMU to emulate a standard x86_64 PC using software emulation.

The VM configuration included:

- Two virtual CPUs
- 4096 MB of RAM
- A 40 GB virtual disk
- An emulated E1000 network adapter
- Legacy BIOS booting

The installation ISO was attached as a virtual CD-ROM so I could start the Arch Linux live environment.

### Verifying internet connectivity

After the installation environment started, I checked the network connection using:

```bash
ping -c 4 archlinux.org
```

The command sent four packets to the Arch Linux website. All four packets were received, confirming that networking was working.

### Problems and solutions

**Problem:** My initial VirtualBox setup could not successfully boot the Arch Linux installation environment.

**Solution:** I switched to QEMU, which supports emulating an x86_64 computer on my ARM64 Windows system.

**Problem:** QEMU was slower than expected.

**Explanation:** QEMU was emulating a different processor architecture rather than using native x86_64 hardware virtualization.

**Result:** Despite slower performance, the Arch Linux installation environment started successfully and provided a working terminal and internet connection.


## 4. Disk Partitioning and Formatting

### Identifying the virtual disk

After booting into the Arch Linux installation environment, I identified the available storage devices using:

```bash
lsblk
```

This command lists block devices and their partitions.

The output showed:

- `/dev/sda` — the 40 GB virtual hard drive.
- `/dev/sr0` — the Arch Linux installation ISO.
- `/dev/loop0` — the live installation environment.

I confirmed that `/dev/sda` was the correct disk before making changes.

### Creating the partition

I opened the partition editor with:

```bash
cfdisk /dev/sda
```

This launched an interactive partitioning tool.

I made the following choices:

| Setting | Selection |
|---|---|
| Partition table | DOS/MBR |
| Disk | `/dev/sda` |
| Partition | `/dev/sda1` |
| Partition size | Approximately 40 GB |
| Partition type | Linux (83) |
| Bootable flag | Enabled |

I selected **Write** to save the partition table, confirmed the changes, and exited `cfdisk`.

I used MBR because my QEMU virtual machine was configured for legacy BIOS booting rather than UEFI.

### Formatting the partition

After creating the partition, I formatted it with the ext4 filesystem:

```bash
mkfs.ext4 /dev/sda1
```

**Explanation:** `mkfs.ext4` creates an ext4 filesystem on the specified partition. This prepared the partition to store the installed operating system.

### Mounting the filesystem

Next, I mounted the partition:

```bash
mount /dev/sda1 /mnt
```

**Explanation:** This makes the new filesystem accessible at `/mnt`, which is where Arch Linux installation files will be placed.

I checked the disk configuration using:

```bash
lsblk
```

The output confirmed that `/dev/sda1` was mounted at `/mnt`.

### Configuration decisions

I used a single root partition rather than creating separate `/home` or swap partitions.

This simplified the installation because the virtual machine was intended primarily for command-line use and coursework.

### Problems and lessons learned

During installation, the root partition appeared to have been mounted more than once. This later contributed to duplicate entries in `/etc/fstab`.

I corrected the duplicate filesystem entry during system configuration.

**Lesson learned:** Check existing mount points before mounting a partition again, and review the generated filesystem table before completing the installation.

## 5. Installing the Arch Linux Base System

### Installing the required packages

Once I finished partitioning and mounting the virtual disk, I was ready to install Arch Linux. I used the `pacstrap` command to install the base system along with the other packages I would need.

```bash
pacstrap -K /mnt base linux linux-firmware networkmanager grub
```

This command installed several important components:

- `base` — the basic files and utilities needed for Arch Linux.
- `linux` — the Linux kernel, which allows the operating system to communicate with the hardware.
- `linux-firmware` — firmware files used by supported hardware.
- `networkmanager` — manages network connections.
- `grub` — the bootloader used to start Arch Linux.

The installation took around 15 minutes, which was longer than I expected. This was probably because I was using QEMU to emulate an x86_64 computer on my ARM-based laptop.

After the installation finished, I checked that some of the main packages were installed:

```bash
pacman --root /mnt -Q base linux grub
```

This displayed the installed package versions and confirmed that the base system, Linux kernel, and GRUB were present

## 6. Configuring Arch Linux

### Setting the timezone

After installing the base system and entering it with `arch-chroot`, I needed to configure the system settings.

I started by setting the timezone to Central Time because I live in Oklahoma.

```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

This command creates a symbolic link that tells Arch Linux which timezone to use.

Next, I ran:

```bash
hwclock --systohc
```

This updates the hardware clock settings using the system time. Neither command displayed an error, so I continued.

### Configuring the system language

Next, I needed to set the system language to English.

I opened the locale configuration file:

```bash
nano /etc/locale.gen
```

I searched for the following line:

```text
#en_US.UTF-8 UTF-8
```

I removed the `#` at the beginning to enable the locale:

```text
en_US.UTF-8 UTF-8
```

After saving the file, I generated the locale settings:

```bash
locale-gen
```

The terminal displayed `Generation complete`, confirming that the locale had been generated.

I then set English as the default system language:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

I also configured the standard US keyboard layout:

```bash
echo "KEYMAP=us" > /etc/vconsole.conf
```

These settings made sure the system would use English and the expected keyboard layout.

### Setting the hostname

I decided to name my virtual machine `archlinux` because it was simple and easy to recognize.

```bash
echo "archlinux" > /etc/hostname
```

I checked that the hostname was saved correctly:

```bash
cat /etc/hostname
```

The terminal displayed `archlinux`, confirming that the configuration file contained the correct name.

### Enabling networking

Since I wanted the virtual machine to connect to the internet after installation, I enabled NetworkManager:

```bash
systemctl enable NetworkManager
```

This configures NetworkManager to start automatically whenever Arch Linux boots.

The command displayed messages about creating symbolic links, which confirmed that the service had been enabled.

### Setting the root password

Next, I created a password for the root account:

```bash
passwd
```

The terminal asked me to enter and confirm a password. Nothing appeared while I typed, which is normal for Linux password prompts.

After entering the password twice, the terminal confirmed that it had been updated successfully.

### Creating my user account

Instead of using the root account for everyday tasks, I created a regular user named `alex`.

```bash
useradd -m -G wheel -s /bin/bash alex
```

This command creates the account, gives it a home directory, adds it to the `wheel` group, and sets Bash as its default shell.

I then created a password for the account:

```bash
passwd alex
```

### Giving my account administrative permissions

I wanted my regular account to be able to perform administrative tasks without logging in as root every time.

First, I installed `sudo`:

```bash
pacman -S sudo
```

Then I opened the sudo configuration file:

```bash
EDITOR=nano visudo
```

I found the following line:

```text
# %wheel ALL=(ALL:ALL) ALL
```

I removed the `#` and the space after it


I changed the line to:

```text
%wheel ALL=(ALL:ALL) ALL
```

This allows members of the wheel group, including my user account, to run administrative commands using sudo.

After saving the file, I checked my account and the sudo configuration:

```bash
id alex
visudo -c
```

The first command shows which groups my account belongs to. The second checks the sudo configuration for syntax errors.

## 7. Installing GRUB and Fixing Boot Problems

### Installing the bootloader

After finishing the system configuration, I needed to install a bootloader so the virtual machine could start Arch Linux from its virtual hard drive.

I decided to use GRUB because it was already included in the packages I installed with `pacstrap`.

Since my QEMU virtual machine was using legacy BIOS rather than UEFI, I installed GRUB using:

```bash
grub-install --target=i386-pc /dev/sda
```

The `--target=i386-pc` option tells GRUB to install for a BIOS-based system. `/dev/sda` refers to the entire virtual disk, not just the root partition.

The command finished with a message saying that the installation was successful.

Next, I generated the GRUB configuration:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

This command searches for the installed Linux kernel and creates the configuration GRUB uses to start the operating system.

The output showed that GRUB found the Linux kernel and initial RAM disk.

I also noticed a warning about `os-prober` not being executed. I learned that `os-prober` is used to detect other operating systems. Since I was only installing Arch Linux, this warning did not prevent me from continuing.

### Leaving the installation environment

Once I finished configuring Arch Linux, I exited the chroot environment:

```bash
exit
```

This returned me to the Arch Linux installation environment.

I then unmounted the installed filesystem:

```bash
umount -R /mnt
```

This unmounted the filesystem and its mounted subdirectories before shutting down the installation environment.

Finally, I entered:

```bash
poweroff
```

This shut down the virtual machine so I could restart it using the installed system instead of the installation ISO.

### Booting from the virtual hard drive

I returned to the MSYS2 CLANGARM64 terminal on Windows and started QEMU without attaching the installation ISO.

```bash
qemu-system-x86_64 \
  -machine pc \
  -accel tcg,thread=multi \
  -cpu max \
  -smp 2 \
  -m 4096 \
  -drive file=archlinux.qcow2,format=qcow2 \
  -boot c \
  -nic user,model=e1000 \
  -vga std
```

This command started up the virtual machine using the existing 40 GB virtual disk.

The `-boot c` option tells QEMU to boot from the virtual hard drive. I removed the installation ISO so the system would not start the installer again.

### Problem: GRUB opened


When I first tried to boot Arch Linux from the virtual hard drive, GRUB opened to a command prompt instead of showing the normal boot menu.

This was confusing because the GRUB installation had reported that it was successful. I needed to figure out why the installed operating system was not starting automatically.

I used the following commands to inspect the available partitions:

```text
ls
ls (hd0,msdos1)/boot/
```

I found the Linux kernel and initial RAM disk on the first partition.

I manually booted Arch Linux using:

```text
set root=(hd0,msdos1)
linux /boot/vmlinuz-linux root=/dev/sda1 rw
initrd /boot/initramfs-linux.img
boot
```

These commands told GRUB where to find the installed operating system and allowed me to start Linux.

After logging in, I regenerated the GRUB configuration:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

I verified that the configuration file existed:

```bash
ls -l /boot/grub/grub.cfg
```

Finally, I restarted the virtual machine:

```bash
sudo reboot
```

After making this correction, Arch Linux started normally without requiring manual GRUB commands.

This was one of the biggest problems I encountered during the assignment, but it helped me understand how the Linux boot process works.


## 8. Final Verification and Additional Troubleshooting

### Checking that Arch Linux started correctly

After fixing GRUB, I restarted my virtual machine to make sure Arch Linux could boot without any manual commands.

This time, the system started automatically and brought me to the login screen. I signed in using the `alex` account I created during installation.

I wanted to make sure everything was working before considering the installation finished, so I ran several commands to check the system.

### Testing the internet connection

The first thing I checked was whether the installed system could access the internet.

```bash
ping -c 4 archlinux.org
```

This command sends four packets to the Arch Linux website to test the network connection.

All four packets were received, with 0% packet loss. This confirmed that networking was working after installation and that NetworkManager was functioning.

### Verifying my user account

Next, I checked which user was currently logged in:

```bash
whoami
```

The output was:

```text
alex
```

This confirmed that I was using my regular account instead of logging in as root.

### Checking the Linux kernel

I used the following command to check the installed Linux kernel version:

```bash
uname -r
```

The output was:

```text
7.2.15-arch1-1
```

This confirmed that I was running the installed Arch Linux kernel.

### Checking the disk configuration

I also checked the virtual disk and its partitions:

```bash
lsblk
```

This displayed the 40 GB virtual disk and its Linux partition.

The output showed that `/dev/sda1` was mounted as the root filesystem (`/`), confirming that the system was running from the virtual hard drive.

### Problem: The hostname command was missing

One smaller issue I encountered during verification happened when I tried to check the computer's hostname:

```bash
hostname
```

Instead of displaying the hostname, the terminal returned:

```text
hostname: command not found
```

At first, I thought this meant I had configured the hostname incorrectly.

However, I checked the hostname configuration file:

```bash
cat /etc/hostname
```

The output was:

```text
archlinux
```

This showed that the hostname was configured correctly. The problem was simply that the minimal Arch Linux installation did not include the `hostname` utility.

### Fixing the missing command

To install the missing utility, I ran:

```bash
sudo pacman -S inetutils
```


The `pacman` command is Arch Linux's package manager. The uppercase `-S` option installs a package, while `inetutils` provides the hostname utility.

I initially made a mistake by using a lowercase `-s` instead of uppercase `-S`. After correcting the command, the package installed successfully.

I checked the hostname again:

```bash
hostname
```

This time, the output was:

```text
archlinux
```

Problem was solved.

## 9. Conclusion

Overall, installing Arch Linux manually was more difficult than I originally expected. The biggest challenge was that my computer uses an ARM64 processor, while the standard Arch Linux installation image is designed for x86_64 systems.

I initially attempted to use VirtualBox, but I ran into problems getting the installer to boot. So i was forced to switch to QEMU which allowed me to continue the assignment, although the installation was stalled because of emulation.

I also encountered problems with GRUB, filesystem configuration, and missing commands. Each problem required me to research what was happening and find a way to fix it.

By the end of the assignment, I successfully installed Arch Linux, configured my user account, enabled networking, and verified that the virtual machine could boot from its own virtual hard drive.

This experiment helped me obtain understand Linux installation, partitioning, system configuration, bootloaders, and troubleshooting.

## 10. References

- [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [Arch Linux GRUB Documentation](https://wiki.archlinux.org/title/GRUB)
- [QEMU Documentation](https://www.qemu.org/docs/master/system/)
- [MSYS2 Official Website](https://www.msys2.org/)


## 11. Additional System Configuration

After completing the main Arch Linux installation, I needed to configure several additional features for the assignment. These included installing a new shell, customizing the terminal colors, creating command aliases, enabling SSH, and installing a graphical desktop environment.

### 11.1 Installing a New Shell (Zsh)

Arch Linux initially used Bash as my default shell. I decided to install Zsh because it supports additional customization and useful command-line features.

I installed Zsh using:

```bash
sudo pacman -S zsh
```

This command downloads and installs the Zsh shell using the Pacman package manager.

Next, I changed my default shell:

```bash
chsh -s /bin/zsh
```

The `chsh` command changes the login shell associated with my user account.

I logged out using:

```bash
exit
```

After logging back in, Zsh displayed an initial configuration menu. I selected option `0` to create a basic configuration file.

I verified the new shell using:

```bash
echo $SHELL
```

The output was:

```text
/bin/zsh
```

This confirmed that Zsh was now my default shell.

### 11.2 Customizing Shell Colors

Next, I customized my terminal to make it easier to read.

I opened my Zsh configuration file:

```bash
nano ~/.zshrc
```

The `.zshrc` file stores settings that Zsh loads when starting an interactive shell.

I added the following lines:

```bash
autoload -Uz colors
colors
PROMPT='%F{green}%n@%m%f:%F{blue}%~%f%# '
alias ls='ls --color=auto'
```

The first two commands enable Zsh's color functionality.

The `PROMPT` setting changes the appearance of the command prompt. I configured the username and hostname to appear in green and the current directory to appear in blue.

The `ls` alias enables colored output when listing files.

After saving the file, I applied the changes:

```bash
source ~/.zshrc
```

This reloads the configuration without requiring another login.

The terminal prompt displayed the new colors successfully.

### 11.3 Creating Two Command Aliases

I also created two aliases to make frequently used commands easier to execute.

I opened the configuration file again:

```bash
nano ~/.zshrc
```

I added:

```bash
alias ll='ls -la --color=auto'
alias update='sudo pacman -Syu'
```

The first alias, `ll`, displays a detailed list of files, including hidden files.

The second alias, `update`, provides a shortcut for updating the system using Pacman.

I saved the file and reloaded the configuration:

```bash
source ~/.zshrc
```

I tested the first alias:

```bash
ll
```

The terminal displayed a detailed directory listing.

I checked the second alias:

```bash
type update
```

The terminal confirmed that `update` was an alias for `sudo pacman -Syu`.

Both aliases worked successfully.

### 11.4 Installing and Configuring SSH

The next requirement was installing SSH, which allows encrypted remote access to a Linux computer.

I installed OpenSSH using:

```bash
sudo pacman -S openssh
```

I enabled and started the SSH service:

```bash
sudo systemctl enable --now sshd
```

The `enable` option configures SSH to start automatically when the system boots. The `--now` option starts the service immediately.

I checked the service status:

```bash
systemctl status sshd
```

The output showed:

```text
active (running)
```

This confirmed that the SSH server was operating.

To test an actual connection, I ran:

```bash
ssh alex@localhost
```

Because this was my first connection, SSH asked me to confirm the host's identity. I accepted the connection and entered my account password.

After connecting, I verified the logged-in account:

```bash
whoami
```

The output was:

```text
alex
```

I then exited the SSH session:

```bash
exit
```

The test confirmed that I could successfully establish an SSH connection to my Arch Linux system.

### 11.5 Installing the XFCE Desktop Environment

My initial Arch Linux installation only provided a command-line interface. For the desktop environment requirement, I decided to install XFCE.

I selected XFCE because it is relatively lightweight, which was important because my virtual machine was running through QEMU software emulation.

First, I updated the system:

```bash
sudo pacman -Syu
```

Next, I attempted to install the graphical desktop packages:

```bash
sudo pacman -S --needed xorg-server xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
```

These packages provide the graphical display server, XFCE desktop environment, additional utilities, and LightDM login manager.

I then attempted to enable LightDM:

```bash
sudo systemctl enable lightdm
```

Initially, I received an error indicating that the service unit did not exist.

To investigate, I checked whether LightDM was installed:

```bash
pacman -Q lightdm
```

I also used the following installation command while troubleshooting:

```bash
sudo pacman -S lightdm lightdm-gtk-greeter
```

After getting LightDM installed and enabled, I reached the graphical login screen.

However, another problem appeared.

### 11.6 Troubleshooting the XFCE Login

When I attempted to log into the graphical environment, LightDM displayed a message saying:

```text
Failed to start session
```

At first, I thought there might be a problem with my password.

I switched back to a text terminal using `Ctrl + Alt + F2` and successfully logged into my account.

This confirmed that my username and password were working.

Next, I checked whether the XFCE session package was installed:

```bash
pacman -Q xfce4-session
```

The terminal reported that the package was not found.

This explained why LightDM could display the graphical login screen but could not start the desktop.

I installed the missing XFCE components:

```bash
sudo pacman -S --needed xfce4-session xfce4-panel xfdesktop xfwm4 xfce4-settings xfce4-terminal
```

These packages provide the XFCE session manager, desktop panel, desktop background manager, window manager, settings, and terminal.

After the installation finished, I restarted LightDM:

```bash
sudo systemctl restart lightdm
```

The graphical login screen appeared again.

I selected the XFCE session and logged in using my regular account.

This time, the desktop loaded successfully.

### 11.7 Final Results

After completing these additional configuration steps, my Arch Linux virtual machine had the following features:

- Zsh installed as the default shell.
- A customized terminal with green and blue prompt colors.
- Two working aliases named `ll` and `update`.
- An active SSH server.
- A successful SSH connection using `ssh alex@localhost`.
- A working XFCE graphical desktop environment.

The most difficult part was configuring the graphical environment. LightDM initially failed to start a session because the necessary XFCE session package was missing.

After installing the missing packages, I was able to log into the desktop normally.

This part of the assignment helped me understand how Linux shells, configuration files, system services, SSH, and desktop environments work together.



### 11.8 Screenshots

**Screenshot 1: Zsh Shell Colors and Aliases**

![Zsh configuration](<Screenshot 2026-11-16 201838.png>)

**Screenshot 2: SSH Connection and Verification**

![SSH verification](<Screenshot 2026-11-16 202115.png>)

**Screenshot 3: XFCE Desktop and Terminal**

![XFCE desktop](<Screenshot 2026-11-16 202026.png>)

