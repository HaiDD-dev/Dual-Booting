# Guide to Dual Booting Windows and Arch Linux (UEFI)

This is a detailed guide based on your requests, supplemented with necessary steps to ensure a stable and complete dual boot system.

---

### Step 1: Preparing in Windows
Before you begin, you need to create free space on your hard drive for Arch Linux.

1.  Press `Windows + R`, type `diskmgmt.msc`, and press Enter to open **Disk Management**.
2.  Right-click on your `C:` drive (or the drive you want to take space from) and select **Shrink Volume...**.
3.  In the "Enter the amount of space to shrink in MB" field, enter the value for 75GB (approximately **76800** MB).
4.  Click **Shrink**. You will now see an "Unallocated" space of about 75GB. This is where we will install Arch Linux.

---

### Step 2: Creating a Bootable Arch Linux USB

1.  **Download Arch Linux:** Go to the [official Arch Linux download page](https://archlinux.org/download/) to get the latest ISO file.
2.  **Download Rufus:** Download the latest version of [Rufus](https://rufus.ie/).
3.  **Create the USB:**
    * Plug your USB drive into your computer (minimum 4GB recommended).
    * Open Rufus.
    * **Device:** Select your USB drive.
    * **Boot selection:** Click **SELECT** and choose the Arch Linux ISO file you just downloaded.
    * **Partition scheme:** Select **GPT**.
    * **Target system:** Select **UEFI (non CSM)**.
    * Click **START** and confirm to let Rufus create the bootable USB.

---

### Step 3: Configuring BIOS/UEFI

1.  Restart your computer and enter the BIOS/UEFI. The key to enter BIOS is typically `F2`, `F10`, `F12`, or `DEL`, depending on your motherboard/laptop manufacturer.
2.  **Disable Secure Boot:** Navigate to the **Security** or **Boot** tab, find the **Secure Boot** option, and set it to **Disabled**. This is a mandatory step.
3.  **Enable USB Boot:** Ensure that the option to boot from USB is enabled.
4.  **Set Boot Order:** Move the USB Drive to the first position in the boot order.
5.  Save changes and exit the BIOS (usually the `F10` key). The computer will restart and boot from the USB drive.

---

### Step 4: Booting into the Arch Linux Live Environment

After the computer boots from the USB, you will see a menu. Select **Arch Linux install medium**.

1.  **Increase Font Size (Optional):** To make text easier to read on high-resolution displays, type the command:
    ```bash
    setfont ter-132n
    ```

2.  **Connect to the Internet:**
    * **Check connectivity:** `ping archlinux.org`. If you get a response, you can skip this step.
    * **For wired (Ethernet) connections:** It should connect automatically.
    * **For WiFi (using `iwdctl`):**
        ```bash
        # Start the iwd tool
        iwctl
        
        # [iwd]# device list (Lists wireless devices, e.g., wlan0)
        device list
        
        # [iwd]# station <device_name> scan (Scans for networks, replace <device_name> with your actual device)
        station wlan0 scan
        
        # [iwd]# station <device_name> get-networks (Shows scanned networks)
        station wlan0 get-networks
        
        # [iwd]# station <device_name> connect "<Your_WiFi_Name>" (Connect, and enter password when prompted)
        station wlan0 connect "Your_WiFi_Name"
        
        # [iwd]# exit (Exit the iwctl utility)
        exit
        ```
    * **Verify connection again:** `ping archlinux.org`

3.  **Update package lists and keyring:**
    ```bash
    pacman -Sy
    pacman -S archlinux-keyring
    ```

---

### Step 5: Using `archinstall` (The Guided Installer)

This is a command-line interface tool that makes the installation process much easier.

1.  **Check and Identify Your Drive:**
    This is a critical step to avoid wiping the wrong data. We'll use two commands to view information about connected drives.
    ```bash
    lsblk
    fdisk -l
    ```
    **How to read the output and identify your drive:**
    * The `lsblk` (list block devices) command will show your drives and their partitions in a tree-like structure.
    * The `fdisk -l` command will provide more detailed information about the partition table of each disk.

    **Case 1: Machine with one hard drive (common)**
    * If it's a SATA SSD/HDD, it will be named `/dev/sda`. You will see its partitions like `sda1`, `sda2`, etc.
    * If it's a high-speed NVMe SSD, it will be named `/dev/nvme0n1`. Its partitions will be `nvme0n1p1`, `nvme0n1p2`, etc.
    * Look at the `SIZE` column. Your main drive will have the largest capacity, corresponding to the total size of your storage (e.g., 256G, 512G, 1T). The 75GB unallocated space you created in Step 1 will not be listed as a partition yet.

    **Case 2: Machine with multiple hard drives (e.g., one SSD and one HDD)**
    * You will see multiple devices, for example, `/dev/sda` (SSD) and `/dev/sdb` (HDD).
    * You need to determine which drive contains your Windows installation and the 75GB of free space you created. Usually, Windows is installed on the faster drive (the SSD - `/dev/sda` or `/dev/nvme0n1`).
    * Use the disk size to confirm. For example, if you have a 256GB SSD and a 1TB HDD, `fdisk -l` will clearly show `Disk /dev/sda: 238.5 GiB` and `Disk /dev/sdb: 931.5 GiB`.

    **Identify the EFI Partition:**
    * When running `fdisk -l`, look for a small partition (around 100-500MB) with the type **`EFI System`**. This is the Windows boot partition, and we will share it with Arch Linux. `archinstall` will detect it automatically.

    **=> Conclusion:** After reviewing, remember the exact name of the drive you will install Arch Linux on (e.g., `/dev/sda` or `/dev/nvme0n1`). Selecting the wrong drive in the next step can lead to complete data loss.

2.  **Run the installer:**
    ```bash
    archinstall
    ```

3.  **Configuration within `archinstall`:**
    * **`Disk configuration`**:
        * Select **`Use a best-effort default partition layout`**.
        * Select the drive you want to install Arch on (e.g., `/dev/sda`).
        * **IMPORTANT (for Dual Boot):** The installer will ask about the EFI partition. It should automatically detect the one from Windows. **SELECT** that partition and **assign it to the mount point `/boot`** (or `/efi`). **DO NOT FORMAT** this partition. `archinstall` usually handles this correctly; you just need to confirm.
        * When asked "Would you like to use a default partition layout?", select **`btrfs`** as the filesystem.
    * **`Bootloader`**: Select **`systemd-boot`**. This is the best and simplest choice for a UEFI dual boot setup.
    * **`Hostname`**: Set a name for your computer, e.g., `arch-pc`.
    * **`Root password`**: Set a password for the `root` account.
    * **`User account`**: Select **`Add a user`**, enter your desired username and password. Make sure to select **Yes** when asked, "Should this user be a superuser (sudo)?".
    * **`Profile`**:
        * Select `Desktop`.
        * Select **`Hyprland`**.
    * **`Graphics drivers`**:
        * **Intel:** Choose `xf86-video-intel` or leave it blank (to use the default modesetting driver).
        * **AMD:** Choose `xf86-video-amdgpu` or leave it blank (to use the default `amdgpu` driver).
        * **NVIDIA:** Choose **`nvidia-dkms`** (recommended).
        * If unsure, you can select **`All open-source`** and install specific drivers later.
    * **`Audio`**: Select **`pipewire`**.
    * **`Kernels`**: Select **`linux`** (default).
    * **`Additional packages`**: Add any extra packages you need, separated by spaces. Suggestions:
        ```
        git nano vim firefox network-manager-applet btop
        ```
    * **`Network configuration`**: Select **`Use NetworkManager`**.
    * **`Timezone`**: Type `Asia/Ho_Chi_Minh`.

4.  **Install**: Once you have finished configuring everything, select **`Install`** from the main menu. `archinstall` will now format the disk, download packages, and install the system. This will take some time.

5.  **Chroot**: When the installation is complete, `archinstall` will ask, "Would you like to chroot into the newly created installation...". Select **`Yes`**.

---

### Step 6: Post-installation Configuration (Inside Chroot)

You are now inside your newly installed Arch system.

1.  **Update the entire system:**
    ```bash
    pacman -Syyu
    ```
2.  **Install other necessary applications:**
    ```bash
    pacman -S gcc vlc
    ```
3.  Exit the chroot environment:
    ```bash
    exit
    ```
4.  Reboot the system:
    ```bash
    reboot
    ```
    Remember to remove the USB drive as the computer begins to restart.

---

### Step 7: First Boot and Environment Setup

1.  After rebooting, you should see the `systemd-boot` menu. From here, you can choose **Arch Linux** or **Windows Boot Manager**. Select **Arch Linux** to continue.
2.  Log in with the user account you created.
3.  If Hyprland doesn't start automatically, type `Hyprland` in the terminal and press Enter.
4.  **Run the dotfiles setup script (As you requested):**
    **WARNING:** Running scripts directly from the internet can be a security risk. A safer method is to download the script, review it to understand what it does, and then execute it. However, to follow your request:
    ```bash
    bash <(curl -s [https://raw.githubusercontent.com/mylinuxforwork/dotfiles/main/setup-arch.sh](https://raw.githubusercontent.com/mylinuxforwork/dotfiles/main/setup-arch.sh))
    ```
    Then, select the **main-release** as you described.

---

### Step 8: Installing and Configuring the Vietnamese Input Method (Fcitx5 + Unikey)

1.  **Install the required packages:**
    ```bash
    sudo pacman -S fcitx5-im fcitx5-configtool fcitx5-unikey fcitx5-gtk fcitx5-qt
    ```

2.  **Configure environment variables:** For applications to recognize the input method, you need to add environment variables. Open the `/etc/environment` file with `nano`:
    ```bash
    sudo nano /etc/environment
    ```
    Add the following three lines to the end of the file:
    ```
    GTK_IM_MODULE=fcitx
    QT_IM_MODULE=fcitx
    XMODIFIERS=@im=fcitx
    ```
    Press `Ctrl + X`, then `Y`, and `Enter` to save the file.

3.  **Reboot your computer** for the changes to take effect.

4.  **Configure Fcitx5:**
    * After logging back in, open a terminal and run `fcitx5-configtool`.
    * In the "Input Method" tab, you will see "Keyboard - English (US)". Click the **`+`** button at the bottom.
    * Uncheck the "Only Show Current Language" box.
    * Search for `Unikey` in the list and click **Add**.
    * You can now switch between English and Vietnamese (Unikey) using the hotkey (usually `Ctrl + Space`).

Congratulations! You have successfully dual-booted Windows and Arch Linux with the Hyprland environment and Vietnamese input method.
