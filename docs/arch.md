# Arch Linux

## Arch Fun: From Installing the System to Useful Programs

*Arch Linux* is an independently developed `x86-64` GNU/Linux distribution designed for general purposes. It aims to provide the latest stable versions of most software, adhering to a rolling release model. Arch is installed as a minimal base system and is configured by the user to meet their specific needs, allowing the creation of a unique environment by installing only the necessary components.

Essentially, it’s a blank slate, running on the Linux kernel and equipped with a basic set of commands. It ships without a GUI (Graphical User Interface).

To put it simply: Arch is for *gentlemen*, while Ubuntu is for the *mainstream crowd* who want everything handed to them on a silver platter.

This guide will be updated and expanded over time — stay tuned! 

### Preparation of Installation Image
!!! note
    We want to emphasize that Linux systems can be installed almost anywhere: on a desktop, a laptop, a USB drive, or even a phone (if the distribution supports it). The installation process is roughly the same, but there are differences that need to be understood. For example, when installing Arch Linux as a second system, you'll need to manually configure efibootmgr. Additionally, there are differences in setting up dual boot on a machine with BIOS versus UEFI.
That's why guides will repeat certain steps to avoid creating confusion with too many "If you have BIOS, do this…" and "If you have UEFI, do that…" instructions scattered throughout.

What You’ll Need to Begin Installation:

- A USB flash drive
- The system’s ISO file
- A program to write the ISO image
- A machine where Arch will be installed

The installation will be done as the primary operating system.

Let's create the installer on a USB drive.

First, download the system image from the official Arch Linux website using the method that works best for you. We recommend using a Torrent client.

There are several programs that can help with this process, such as:

- Rufus
- Balena Etcher
- UltraISO (Yes, really…)

In our case, we'll use Rufus.
After downloading the image, open Rufus.

- Device: Select your **USB flash drive*.  
- Boot selection: Choose **"Disk or ISO image"** and specify the path to the Arch Linux ISO file.  
- Partition scheme: Select **MBR**.  
- File system: Choose **FAT32**.  
- Cluster size: Leave it as **default**.  

![rufus_arch](images/arch/rufus_arch.png)

!!! IMPORTANT
    All files on the USB drive will be erased! 

- Click Start. In the pop-up window, select the option that mentions the **ISO**.

After the process is complete, connect the USB flash drive to the machine where you want to install the system.  
Enter the BIOS of the machine and disable Secure Boot (you can enable it again after the installation is complete).  
Now, we can boot from the USB. When the machine boots from the USB, you'll see a menu. Select the first option **Arch Linux install medium**, and wait for the system to load.

![start_arch](images/arch/Arch_install1.png)

## Installation with dual-boot

### 1. Network settings

Setting Up a Wireless Network

If you are connected via a wired connection, simply check if it works using the following command:

    ping archlinux.org

If you get a response, everything is fine. If not, it means the required service didn't start. Try restarting the machine or re-creating the installation image. We will use the iwctl utility.

Execute the command:

    iwctl

![iwctl](images/arch/iwctl_settings.png)


Then enter the following commands:

1. List available wireless devices:

    device list

*Identify the name of your wireless device (e.g., wlan0).*

2. Scan for available networks:

    station <device_name> scan

3. View the list of available networks:

    station <device_name> get-networks

4. Connect to a Wi-Fi network:

    station <device_name> connect <SSID>

*After this command, you will need to enter the password if the network is secured.*

5. Check the connection status:

    station <device_name> show

Ensure the network is connected.

    ping archlinux.org

### 2. Disk Partitioning

There are two utilities for disk partitioning:

- **fdisk**
- **cfdisk**

In my opinion, `cfdisk` is simpler to use and more user-friendly for inexperienced users. We will use it.

First, let's determine the name of our disk.

!!! IMPORTANT
    It is recommended to disconnect any unnecessary disks to avoid confusion and the accidental loss of data due to inexperience.

Use the following command to determine the name of your disk:

    lsblk

This will output a list of connected disks.

![lsblk](images/arch/lsblk1.png)

!!! note
    That the disk must be in GPT format. If the disk is new, when launching cfdisk, we will be prompted to choose a format for the disk. Select GPT.  

Run the following command:  

    cfdisk /dev/sda

After launching *cfdisk*, you will see a menu for creating partitions. Navigation is done using the arrow keys (Up and Down to select a partition, Left and Right to choose a button for execution). The Enter key is used for confirmation. At the bottom, you will see buttons such as New, Quit, etc. Free space will be highlighted in green.  

1. Press `New`.
2. Enter the amount of memory you need (make sure to specify the unit: **G** for gigabytes, **M** for megabytes).
3. Press **Enter** .
4. Next, select `Type` for the partition.

We need to create three partitions:

| Partition |   Size          | Type              |
| :---      |     :---:       |              ---: |
| boot      | 700M            |  Efi-system       |
| swap      | 8G              |  linux swap       |
| root      | Remaining space |  linux filesystem | 


!!! IMPORTANT
    SWAP is the swap partition; it is recommended to include it if your machine has less than 8 GB of RAM.

![cfdisk](images/arch/cfdisk.png)

After creating all partitions and selecting their `Type`, press `Write` and type `YES` to confirm. Quit.

**Now, let's format and mount the partitions.**

!!! IMPORTANT
    We are using a disk named `sda` with partitions `sda1`, `sda2`, `sda3`. These names may be different, up to `nvme0n1` with partitions `nvme0n1p1`, `nvme0n1p2`, and so on.

**EFI System:**

    mkfs.fat -F32 /dev/sda1

**Swap:**

    mkswap /dev/sda2
    swapon /dev/sda2

**Linux filesystem:**

    mkfs.ext4 /dev/sda3

Once everything is formatted, it's time to mount it.

**EFI:**

    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot

**Linux filesystem:**

    mount /dev/sda3 /mnt

Check `lsblk`

![final_lsblk](images/arch/finish_lsblk.png)

### 3. Installing

Okay, we’re done with partitioning the disk. Now, let’s proceed with installing Arch with all the components.

**Enter** and **wait**:

    archinstall   

We will be greeted with the following menu.

![archinstall](images/arch/archinstall.png)

Let's go step by step through the process of installing Arch Linux using the installer:

**Locale**

*Here, you can add the language to be used in the system.*

- Select Locale language and search for the desired language. I choose `ru_RU`.

!!! Note 
    To quickly search, you can press ? and type the language you need, such as en or ru.

 
**Disk configuration**

- Select `Partitioning`.

- Choose `Pre-mounted configuration`.

- Type `/mnt`.

- Press **Enter**.


**Bootloader**

- Select `GRUB`


**Hostname**
    
*Name is used as the computer’s network name*.

- Enter the **name** for your system (you can leave it as archlinux).

**Root password**

*Set a password for the root user*.

- Enter the **password** and **confirm it**.
    

**User account**

*Add a new user account*.

- Enter a **username**.

- Enter a **password** for the user.

*You will be prompted to confirm the addition of superuser privileges (sudo)*.

- Select `Yes` to confirm.

    

**Profile**

*Choose a GUI for your Arch installation*.

- Press `Type`.

- Select `Desktop`.

- Choose `KDE plasma`.

Choose a desktop environment that suits you. For beginners, it is recommended to choose **KDE Plasma**.

*You will be redirected to the next menu*.

*Choose the appropriate Graphics driver based on your system configuration*:

- `AMD/ATI` for AMD graphics cards.
    
- `Intel` for integrated graphics in Intel processors.
    
- `Nvidia (open-source/kernel, proprietary)` for Nvidia graphics cards.
    
- `VMware/VirtualBox` for virtual machines.

!!! Recommendation
    In the case of Nvidia, it’s recommended to research the difference between open-source/kernel and proprietary drivers and choose the one that suits you best.

*After selecting the driver, return to the main menu*.

**Network configuration**

- Choose `Use NetworkManager`

    
**Additional packages**

*Add additional programs that you might find useful*.

For now, let’s add:
    
    nano

!!! Note
    You can also add additional software like neofetch, firefox, htop, etc., but you can always add them later after the installation.


**Timezone**

*Choose your time zone.*

!!! Note
    For quick search, press ? and type your desired time zone.


**Installing Arch**

After the pre-installation setup, select `Install`.

Then select `Yes` and wait for the installation to complete.

Once the installation is complete. After the installation is complete, we are prompted to continue the setup in chroot. Select `Yes`.

### 4. GRUB configuration

*Now, let's set up dual-boot*. 

Enter the following command:

    pacman -Sy efibootmgr dosfstools mtools

You will be asked if you want to continue the installation. Press **Enter** to proceed.

Next, enter the following command:

    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

If the installation is successful, you will see:

    Installation finished. No error reported.

Now, generate the GRUB configuration file with:

    grub-mkconfig -o /boot/grub/grub.cfg

Reboot the system:

    Reboot

After booting into the system, open the terminal (in KDE, the pre-installed terminal is Konsole).

*We need to uncomment one line and modify another in GRUB’s configuration settings.*

Open the GRUB configuration file:

    nano /etc/default/grub

At the top, you will see the parameter GRUB_TIMEOUT.
This controls the time (in seconds) GRUB waits before booting the default OS.
You can leave it as is or set a custom value, for example:

    GRUB_TIMEOUT=10

Scroll down to the bottom and find the line:

    #GRUB_DISABLE_OS_PROBER=false

Uncomment it by removing the # at the beginning:

    GRUB_DISABLE_OS_PROBER=false

Save the changes:

- Press CTRL+O to save.
- Press Enter to confirm.
- Press CTRL+X to exit.

Well done, bro. You install Arch linux on your PC.