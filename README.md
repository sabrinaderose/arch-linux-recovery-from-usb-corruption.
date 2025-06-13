# 💥 Arch Linux Nuke & Recovery

> [!IMPORTANT]
> This repository documents a full system failure and recovery caused by the Windows Media Creation Tool. It serves as a reference for disaster recovery, ISO imaging best practices, and surviving catastrophic system loss.

---

## 📘 Summary

This repo covers the complete breakdown of an **Arch Linux + Hyprland** setup after using the **Windows Media Creation Tool**, which corrupted both the system and the bootable USBs. It chronicles recovery from a total loss of partitions, bootloaders, and ISOs — a real-world Linux troubleshooting experience.

---

## ⚠️ Event Timeline

### ❌ The Failure

- Used **Windows Media Creation Tool** to reimage a USB for Windows.
- Tool overwrote the **bootable Arch Linux USB** and corrupted all partitions.
- Entire system became unbootable — **GRUB, rEFInd, EFI = gone**.
- Other USBs were also rendered unreadable (e.g., Nobara ISO).
- One USB was even **bricked** by Windows Imaging Tool.
- Net result: **20+ hours of Arch + Hyprland work lost.**

---

## 🛠️ Recovery Actions

### ✅ Step 1: Recovery Machine (Fresh USB Imaging)

- Used a **secondary Windows laptop** to re-download Arch ISO.
- Created a working bootable USB using:

```bash
# Safer alternatives to avoid corruption:
balenaEtcher
ventoy
dd if=archlinux.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

---

### ✅ Step 2: Repartition and Fresh Install

```bash
cgdisk /dev/sda
# Created:
# - /dev/sda1: EFI
# - /dev/sda2: /
# - /dev/sda3: /home

mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3

mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

> [!NOTE]
> This layout matches a typical secure and modular Arch install.

Installed base system:

```bash
pacstrap /mnt base linux linux-firmware grub efibootmgr sudo neovim
```

---

### ✅ Step 3: Reconfigure and Reboot

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

passwd
useradd -mG wheel sabrina
```

---

## 💡 Simulated Log Output

```text
[  +0.0001] BIOS: No bootable device found
[  +0.0003] rEFInd: missing
[  +0.0005] grub rescue> _
[  +0.0008] Error: unknown filesystem
[  +0.0010] Cannot find command 'normal'
```

---

## 🔍 Lessons Learned

- ❌ **NEVER** use Windows ISO tools on multiboot or Linux-focused systems.
- ✅ Always check the USB device path (`/dev/sdX`) before writing.
- 🔁 EFI bootloaders (GRUB, rEFInd) are fragile if overwritten.
- 💾 Backups aren’t optional — they’re *essential*.

---

## 🧠 Recommendations

> [!TIP]
> Reference tool for future recoveries or portfolio use.

- Keep a **rescue ISO** on a dedicated USB with **Ventoy** or GRUB tools.
- Maintain a log of your disk layout and **fstab** entries.
- Test ISOs on VMs before flashing to real hardware.
- Store boot configs, partitioning scripts, and recovery logs in GitHub.

---

> [!NOTE]
> See also: [`arch-linux-hyprland-nvidia-failure`](https://github.com/sabrinaderose/arch-linux-hyprland-nvidia-failures) for the original install that was lost.
