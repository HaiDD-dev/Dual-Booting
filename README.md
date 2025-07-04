# Guide to Dual Booting Windows and Arch Linux (UEFI) with GRUB

Hello! In this guide, I will walk you through the process of setting up a dual-boot system with Windows and Arch Linux. We will use the GRUB bootloader, and I'll provide two paths for the installation: an easy one using the `archinstall` script and a traditional, manual approach for those who want more control. Let's begin!

---

### Step 1: Preparing in Windows

Our first task is to make some room on your hard drive for Arch Linux.

1.  Press `Windows + R`, type `diskmgmt.msc`, and press Enter. This opens the **Disk Management** tool.
2.  In Disk Management, find your main drive (usually `C:`). Right-click on it and select **Shrink Volume...**.
3.  You'll be asked how much you want to shrink. In the "Enter the amount of space to shrink in MB" field, let's enter the value for 75GB, which is approximately **76800** MB.
4.  Click **Shrink**. Once it's done, you'll see a new "Unallocated" space of about 75GB. This is the perfect spot for our Arch Linux installation.

---

### Step 2: Creating a Bootable Arch Linux USB

Next, we need to create a bootable USB drive with the Arch Linux installation files.

1.  **Download Arch Linux:** First, head over to the [official Arch Linux download page](https://archlinux.org/download/) and grab the latest ISO file.
2.  **Download Rufus:** We'll use a great tool called Rufus. You can download the latest version from the [Rufus website](https://rufus.ie/).
3.  **Create the USB:**
    * Plug a USB drive into your computer (I recommend at least 4GB).
    * Open Rufus.
    * **Device:** Make sure your USB drive is selected.
    * **Boot selection:** Click the **SELECT** button and find the Arch Linux ISO file you just downloaded.
    * **Partition scheme:** It's very important that you select **GPT**.
    * **Target system:** Ensure this is set to **UEFI (non CSM)**.
    * Click **START** and give Rufus the okay to format the USB and create the bootable drive.

---

### Step 3: Configuring Your BIOS/UEFI

Now, we need to tell your computer to boot from the USB drive we just created.

1.  Restart your computer. As it starts up, you'll need to press a special key to enter the BIOS/UEFI settings. This key is often `F2`, `F10`, `F12`, or `DEL`. It varies depending on your computer's manufacturer.
2.  **Disable Secure Boot:** Once you're in the BIOS, look for a **Security** or **Boot** tab. Find the **Secure Boot** option and set it to **Disabled**. This step is mandatory for Arch Linux to boot.
3.  **Enable USB Boot:** Make sure that booting from a USB device is enabled.
4.  **Set Boot Order:** Change the boot priority so that your USB Drive is the first device in the list.
5.  Save your changes and exit the BIOS (usually by pressing the `F10` key). Your computer will now restart and should boot from the Arch Linux USB.

---

### Step 4: Booting into the Arch Linux Live Environment

You're doing great! Your computer should now display a black screen with a menu.

1.  Select the first option: **Arch Linux install medium**.
2.  **Increase Font Size (Optional):** If the text on the screen is too small, you can make it larger with this command:
    ```bash
    setfont ter-132n
    ```
3.  **Connect to the Internet:** A network connection is essential for the installation.
    * First, check if you're already connected: `ping archlinux.org`. If you see replies, you're all set and can skip the rest of this step.
    * **For wired (Ethernet) connections:** You should be connected automatically.
    * **For WiFi (using `iwdctl`):** Let's connect you manually.
        ```bash
        # Start the iwd tool
        iwctl
        
        # [iwd]# List your wireless devices (e.g., wlan0)
        device list
        
        # [iwd]# Scan for networks (replace <device_name> with your device from the list)
        station wlan0 scan
        
        # [iwd]# Show the networks it found
        station wlan0 get-networks
        
        # [iwd]# Connect to your WiFi (enter your password when prompted)
        station wlan0 connect "Your_WiFi_Name"
        
        # [iwd]# Exit the tool
        exit
        ```
    * **Verify connection again:** Let's be sure. Run `ping archlinux.org` one more time.
4.  **Update package lists and keyring:** Let's get everything up to date before we proceed.
    ```bash
    pacman -Sy
    pacman -S archlinux-keyring
    ```

---

Now, you have a choice. **Step 5** is the easy, guided installation. If you prefer to do things by hand for a deeper understanding, you can **skip Step 5** and go directly to **Step 5 (Alternative): Manual Installation**.

---

### Step 5: Using `archinstall` (The Guided Installer)

The `archinstall` tool simplifies the installation process into a series of menus. It's a fantastic way to get started.

1.  **Check and Identify Your Drive:** This is the most critical step. We must ensure we install Arch on the correct drive to avoid data loss.
    ```bash
    lsblk
    fdisk -l
    ```
    **How to understand the output:**
    * `lsblk` shows your drives in a simple tree format.
    * `fdisk -l` gives more detail about the partitions on each disk.
    * **Your Drive:** Look for your main hard drive. It will be named `/dev/sda` (for SATA drives) or `/dev/nvme0n1` (for NVMe drives). Its size should match your computer's storage capacity (e.g., 256G, 512G, 1T). The 75GB of unallocated space we created won't be listed as a partition just yet.
    * **Your EFI Partition:** In the output of `fdisk -l`, find a small partition (100-500MB) of type **`EFI System`**. This is your Windows boot partition. We will share this with Arch Linux, and `archinstall` is smart enough to detect it automatically.
    * **Conclusion:** Take note of the exact name of your main drive (e.g., `/dev/sda`). We'll need it in a moment.

2.  **Run the installer:**
    ```bash
    archinstall
    ```
3.  **Configuration within `archinstall`:** Navigate through the menus and configure your system.
    * **`Disk configuration`**:
        * Choose **`Use a best-effort default partition layout`**.
        * Select the drive you identified earlier (e.g., `/dev/sda`).
        * **IMPORTANT (for Dual Boot):** The installer will find the existing EFI partition from Windows. **SELECT** this partition and assign it the mount point `/boot`. **DO NOT FORMAT IT**. `archinstall` usually handles this perfectly; you just need to confirm.
        * When asked "Would you like to use a default partition layout?", select **`btrfs`** as the filesystem.
    * **`Bootloader`**: Select **`grub`**.
    * **`Hostname`**: Give your computer a name, like `arch-pc`.
    * **`Root password`**: Set a strong password for the `root` (administrator) account.
    * **`User account`**: Select **`Add a user`**, enter your username and password. Crucially, when asked, "Should this user be a superuser (sudo)?", select **Yes**.
    * **`Profile`**:
        * Select `Desktop`.
        * Select **`Hyprland`**.
    * **`Graphics drivers`**:
        * **Intel:** `xf86-video-intel` (or leave blank to use the modern default).
        * **AMD:** `xf86-video-amdgpu` (or leave blank to use the modern default).
        * **NVIDIA:** **`nvidia-dkms`** (this is highly recommended).
        * If you're unsure, **`All open-source`** is a safe bet.
    * **`Audio`**: Select **`pipewire`**.
    * **`Kernels`**: The default **`linux`** is perfect.
    * **`Additional packages`**: Add some useful tools. I suggest:
        ```
        git nano vim firefox network-manager-applet btop os-prober
        ```
        *(We're adding `os-prober` here so GRUB can find your Windows installation later).*
    * **`Network configuration`**: Select **`Use NetworkManager`**.
    * **`Timezone`**: Type your timezone, for example, `Asia/Ho_Chi_Minh`.

4.  **Install**: Review your settings, then select **`Install`**. The script will now partition your drive, download all the necessary packages, and set up your system. This will take a while, so feel free to grab a coffee.

5.  **Chroot**: After the installation finishes, you'll be asked, "Would you like to chroot into the newly created installation...". Select **`Yes`**. This will place you directly inside your new system for final configuration. Now, you can jump to **Step 6**.

---

### Step 5 (Alternative): Manual Installation

For those who want to understand every part of the process, this is the manual path. It's more involved but very rewarding.

1.  **Identify Your Disks:** Just as in the `archinstall` path, you must identify your target drive and your existing EFI partition.
    ```bash
    fdisk -l
    ```
    Take note of your drive (e.g., `/dev/nvme0n1`) and the EFI partition (e.g., `/dev/nvme0n1p1`).

2.  **Partition the Drive:** We'll use a friendly tool called `cfdisk` to partition the unallocated space.
    ```bash
    cfdisk /dev/nvme0n1  # Replace with your drive name
    ```
    * You will see the `Free space` we created earlier.
    * Select the `Free space` and choose `[ New ]`.
    * Enter a size for your root partition. Using all the available space (e.g., 75G) is fine. Press Enter.
    * The partition type (`Linux filesystem`) will be correct by default.
    * Select `[ Write ]`, type `yes` to confirm, and then select `[ Quit ]`.
    * Your new Arch Linux partition will now exist (e.g., `/dev/nvme0n1p4`). Note this new partition name.

3.  **Format the Partitions:** Now we prepare the new partition.
    * **Format the root partition:** We'll use the BTRFS filesystem as planned.
        ```bash
        mkfs.btrfs /dev/nvme0n1p4  # Use your new partition name
        ```
    * **IMPORTANT:** We do **NOT** format the EFI partition. It already contains the Windows boot files.

4.  **Mount the File Systems:** We need to mount our new system to prepare it for installation.
    * Mount the root partition:
        ```bash
        mount /dev/nvme0n1p4 /mnt  # Use your new partition name
        ```
    * Create a boot directory and mount the EFI partition there:
        ```bash
        mkdir /mnt/boot
        mount /dev/nvme0n1p1 /mnt/boot  # Use your existing EFI partition name
        ```

5.  **Install the Base System:** Now we'll install the core Arch Linux system onto our new partitions using the `pacstrap` script.
    ```bash
    pacstrap -K /mnt base linux linux-firmware nano grub efibootmgr networkmanager os-prober
    ```
    *(This installs the base system, the Linux kernel, firmware, a text editor, the GRUB bootloader, tools for UEFI, the network manager, and the `os-prober` to find Windows).*

6.  **Generate fstab:** This file tells your system where its partitions are.
    ```bash
    genfstab -U /mnt >> /mnt/etc/fstab
    ```

7.  **Chroot:** Now, we'll change our root into the newly installed system to configure it.
    ```bash
    arch-chroot /mnt
    ```
    You are now operating inside your new Arch Linux system! Please proceed to **Step 6 (Alternative)**.

---

### Step 6: Post-installation Configuration (Inside Chroot for `archinstall` users)

You're inside your new system. Let's get GRUB ready to see Windows.

1.  **Configure GRUB to Detect Windows:**
    * Open the GRUB configuration file with the nano text editor:
        ```bash
        nano /etc/default/grub
        ```
    * Find the line `#GRUB_DISABLE_OS_PROBER=false`. You need to remove the `#` at the beginning to uncomment it. This is what allows GRUB to find other operating systems.
    * Save the file and exit nano (Press `Ctrl + X`, then `Y`, then `Enter`).
    * Now, let's regenerate the GRUB configuration file with this new setting:
        ```bash
        grub-mkconfig -o /boot/grub/grub.cfg
        ```
    * You should see output that says it found "Windows Boot Manager". This confirms it worked!

2.  **Update the entire system:**
    ```bash
    pacman -Syyu
    ```
3.  **Install other necessary applications:**
    ```bash
    pacman -S gcc vlc
    ```
4.  You are done in the chroot environment. Type `exit` and then reboot.
    ```bash
    exit
    reboot
    ```
    Remember to remove your USB drive as the computer restarts. Then, continue to **Step 7**.

---

### Step 6 (Alternative): Post-installation Configuration (Inside Chroot for Manual users)

Welcome to your new system! Let's get it configured.

1.  **Set Timezone:**
    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
    hwclock --systohc
    ```
2.  **Localization:**
    * Open `/etc/locale.gen` with nano: `nano /etc/locale.gen`.
    * Uncomment `en_US.UTF-8 UTF-8` and your local language (e.g., `vi_VN.UTF-8 UTF-8`) by removing the `#`.
    * Save (`Ctrl+X`, `Y`, `Enter`) and run `locale-gen`.
    * Create the `locale.conf` file:
        ```bash
        echo "LANG=en_US.UTF-8" > /etc/locale.conf
        ```
3.  **Network Configuration:**
    * Create a hostname file:
        ```bash
        echo "arch-pc" > /etc/hostname  # Choose any name you like
        ```
    * Enable NetworkManager so you have internet after rebooting:
        ```bash
        systemctl enable NetworkManager
        ```
4.  **Set Root Password:**
    ```bash
    passwd
    ```
    Enter a secure password for your `root` user.

5.  **Create a User Account:**
    ```bash
    useradd -m yourusername
    passwd yourusername
    usermod -aG wheel,audio,video,optical,storage yourusername
    ```
    * This creates a user, sets their password, and adds them to important groups.
    * Now, let's give this user `sudo` privileges. Run `EDITOR=nano visudo` and uncomment the line `%wheel ALL=(ALL:ALL) ALL`. Save and exit.

6.  **Install Graphics, Audio, and Desktop Environment:**
    ```bash
    pacman -S hyprland xorg-xwayland kitty pipewire wireplumber xf86-video-intel nvidia-dkms # Choose your graphics driver
    pacman -S git firefox network-manager-applet btop
    ```
7.  **Install and Configure GRUB:**
    * Install the GRUB bootloader to your EFI partition:
        ```bash
        grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
        ```
    * Now, enable the os-prober we installed earlier:
        ```bash
        nano /etc/default/grub
        ```
    * Uncomment the line `#GRUB_DISABLE_OS_PROBER=false`. Save and exit.
    * Finally, generate the main GRUB configuration file:
        ```bash
        grub-mkconfig -o /boot/grub/grub.cfg
        ```
    * You should see that it found both Arch Linux and the Windows Boot Manager. Success!

8.  **Exit and Reboot:** The manual setup is complete.
    ```bash
    exit
    reboot
    ```
    Don't forget to remove the installation USB!

---

### Step 7: First Boot and Environment Setup

Regardless of which installation path you chose, the steps from here are the same.

1.  After rebooting, you should be greeted by the **GRUB menu**. Here you can choose to boot **Arch Linux** or **Windows Boot Manager**. Select **Arch Linux** to proceed.
2.  Log in with the user account you created during installation.
3.  If you see a black screen with a terminal (TTY), simply type `Hyprland` and press Enter to start your desktop environment.
4.  **Run the dotfiles setup script (As you requested):**
    **WARNING:** Please be careful when running scripts directly from the internet. The safest way is to download the script, read it to understand what it does, and then execute it. However, to follow your request, here is the command:
    ```bash
    bash <(curl -s [https://raw.githubusercontent.com/mylinuxforwork/dotfiles/main/setup-arch.sh](https://raw.githubusercontent.com/mylinuxforwork/dotfiles/main/setup-arch.sh))
    ```
    Then, select the **main-release** option as you described.

---

### Step 8: Installing and Configuring the Vietnamese Input Method (Fcitx5 + Unikey)

Let's get your preferred input method set up.

1.  **Install the required packages:**
    ```bash
    sudo pacman -S fcitx5-im fcitx5-configtool fcitx5-unikey fcitx5-gtk fcitx5-qt
    ```
2.  **Configure environment variables:** We need to tell all applications to use Fcitx5. Let's open the global environment file:
    ```bash
    sudo nano /etc/environment
    ```
    Add these three lines to the file:
    ```
    GTK_IM_MODULE=fcitx
    QT_IM_MODULE=fcitx
    XMODIFIERS=@im=fcitx
    ```
    Press `Ctrl + X`, then `Y`, and `Enter` to save the file.

3.  **Reboot your computer** for these new settings to take effect.

4.  **Configure Fcitx5:**
    * After you log back in, open a terminal and run `fcitx5-configtool`.
    * In the "Input Method" tab, you'll see your default keyboard. Click the **`+`** icon at the bottom.
    * Uncheck the "Only Show Current Language" box to see all options.
    * Search for `Unikey`, select it, and click **Add**.
    * You can now switch between English and Vietnamese (Unikey) using the hotkey, which is usually `Ctrl + Space`.

Congratulations! You have successfully dual-booted Windows and Arch Linux with the Hyprland environment and Vietnamese input, all managed by GRUB. Enjoy your new setup!
