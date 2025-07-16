# Arch Linux Disaster Recovery: Rebuilding from USB & Bootloader Corruption

**Author:** [sabrinaderose](https://github.com/sabrinaderose)  
**Date:** June 13, 2025  
**Repo:** https://github.com/sabrinaderose/arch-linux-recovery-from-usb-corruption  
**Category:** Linux Disaster Recovery | Bootloader Rebuild | Home Lab  
**Target Audience:** Recruiters & Hiring Managers  
**Certifications Aligned:** LPI Linux Essentials (Completed), CompTIA Linux+ (Planned)

---

## 🔍 Objective

This repository documents a real-world Linux disaster recovery scenario caused by accidental use of the Windows Media Creation Tool. The result was a completely unbootable Arch Linux system—both GRUB and rEFInd bootloaders were destroyed, and multiple USB drives were rendered unreadable.

The goal was to fully rebuild Arch Linux from scratch under time constraints using terminal-only tools and a second machine for ISO management. This effort was focused on restoring core system operability (not GUI), ensuring bootloader integrity, and capturing a realistic homelab experience valuable to system administration roles.

---

## 🛠️ Environment & Tooling

### Hardware

**Primary Workstation:**  
- AMD Ryzen 7 5700G, 32GB RAM  
- NVIDIA RTX 3060 (12GB), 2 × 1TB SSDs (UEFI Boot)  
- Wi-Fi Only Network Access

**Recovery System:**  
- Lenovo IdeaPad 3, Ryzen 5 5500U, 8GB RAM  
- 256GB SSD used for ISO re-downloads and USB flashing

**USB Drives:**  
- 2x Maspen 64GB (corrupted)
- 1x Memorex 32GB (used for recovery)

### Software Stack

- Arch Linux, Kernel 6.9.x.arch1-1  
- GRUB + rEFInd Bootloaders  
- Terminal Tools: `cgdisk`, `mkfs`, `pacstrap`, `arch-chroot`, `neovim`  
- Imaging Tools: `dd`, balenaEtcher, Ventoy, HDD Low-Level Format Tool  
- Attempted DE: Hyprland (abandoned due to NVIDIA issues)

---

## 🔧 Recovery Process Summary

### ✅ Phase 1: USB Rescue & ISO Prep
- Attempted recovery with corrupted USBs (failed)
- Used `balenaEtcher` (unstable), then switched to `dd` (success)
- Verified UEFI boot mode and BIOS config

### ✅ Phase 2: Partitioning & Installation
```bash
# Disk layout
/dev/sda1 - EFI (512M)
/dev/sda2 - Root
/dev/sda3 - Home

# Filesystem creation
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3

# Mount and base install
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
pacstrap /mnt base linux linux-firmware grub efibootmgr sudo neovim
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

# Bootloader
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

# User
useradd -mG wheel sabrina
passwd
```

### ✅ Phase 3: Live Testing
- Booted successfully into CLI with GRUB
- Wi-Fi confirmed functional using NetworkManager
- USB boot validated on multiple systems

### ⚠️ Phase 4: Troubleshooting Highlights
- **GRUB rescue prompt** due to EFI wipe → resolved via chroot + reinstall
- **Ventoy ISO boot loops** → replaced with `dd`
- **Hyprland monitor rotation ignored** → suspected NVIDIA driver conflict

---

## 🧠 Lessons Learned

- EFI bootloaders are fragile—manual recovery is a critical skill
- Microsoft Media Creation Tool is destructive in multiboot systems
- `dd` is more reliable for ISO flashing than GUI alternatives
- BIOS settings (UEFI mode, Secure Boot) must be double-checked at install
- Hyprland is currently unreliable with NVIDIA (especially rotated displays)
- Documenting every command and decision accelerates future recovery

---

## 📈 Outcome & Skills Demonstrated

**Final System State:**  
- Clean CLI-based Arch Linux system with functional GRUB bootloader  
- Deferred GUI setup due to NVIDIA compatibility issues

**Skills Highlighted:**  
- Disaster recovery & Linux system rebuild  
- Terminal-based partitioning and mount logic  
- Bootloader troubleshooting and reinstallation  
- USB flashing, ISO verification, and BIOS validation  
- Under-pressure problem-solving using only CLI tools

---

## 📂 Key Artifacts

| File/Path                 | Description                              |
|--------------------------|------------------------------------------|
| `/etc/fstab`             | Final mounted partition map              |
| `/boot/grub/grub.cfg`    | Verified bootloader config               |
| `.bash_history`          | Recovery command history                 |
| `dmesg` output            | USB detection logs (hardware layer)      |

---

## 🧾 Interview Summary (STAR Format)

**Situation:** System failure caused by USB overwrite and EFI wipe from Microsoft tools  
**Task:** Recover Arch Linux OS and bootloader with no GUI or prior backups  
**Action:** Created clean recovery USB, manually partitioned drive, reinstalled GRUB, and verified networking  
**Result:** Fully recovered CLI-based Linux system; GUI deferred to prioritize core stability

---

## 📚 References

- Arch Wiki – [Installation Guide](https://wiki.archlinux.org/title/Installation_guide)  
- Hyprland GitHub – Compositor configs & known NVIDIA issues  
- Reddit – Wayland/NVIDIA compatibility discussions  
- LinuxQuestions.org – USB recovery advice  
- Chris Titus Tech – Dual boot best practices

---

## 📌 Final Note

This repo was built to document a **real recovery**, not a sandboxed lab. It showcases foundational Linux administration skills, troubleshooting under duress, and the mindset of a professional who treats even personal setups with production-level discipline.
