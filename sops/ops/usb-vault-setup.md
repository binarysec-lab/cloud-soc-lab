
## 📄 SOP: Encrypting USB Vault (LUKS) + Disabling Auto-Mount on Linux

Paste this as `sops/usb-vault-setup.md` in your GitHub repo.

```md
# 🛡️ SOP: Encrypting USB Vault (LUKS) + Disabling Auto-Mount

## 🎯 Purpose

This SOP documents how to format, encrypt, and harden a USB stick for secure password/2FA storage using LUKS.  
It also includes disabling auto-mount to prevent automatic exposure when plugged in.

---

## 🧰 Requirements

- Linux system (Pop!_OS, Ubuntu, etc.)
- USB stick (cleaned or to be wiped)
- `cryptsetup`, `lsblk`, `parted`, `udisks2`
- Sudo/root access

---

## 🪜 Step-by-Step Procedure

### ✅ 1. Identify the USB Device

```bash
lsblk -f
```

Look for your USB device (e.g. `/dev/sdb`) — confirm it’s not your main OS disk.

---

### ✅ 2. Wipe the USB Clean

```bash
sudo wipefs -a /dev/sdX
sudo dd if=/dev/zero of=/dev/sdX bs=1M count=100 status=progress
```

- Replace `sdX` with your device name (not partition).
- This wipes filesystem headers and partitions.

---

### ✅ 3. Create a New Partition Table

```bash
sudo parted /dev/sdX --script mklabel msdos
sudo parted /dev/sdX --script mkpart primary ext4 1MiB 100%
```

---

### ✅ 4. Encrypt the Partition with LUKS

```bash
sudo cryptsetup luksFormat /dev/sdX1
```

Type `YES` when prompted. Then enter a strong passphrase.

---

### ✅ 5. Open and Format the Encrypted Volume

```bash
sudo cryptsetup luksOpen /dev/sdX1 secureusb
sudo mkfs.ext4 /dev/mapper/secureusb
```

Now the encrypted volume is mapped and formatted.

---

### ✅ 6. Mount and Test (Optional)

```bash
sudo mkdir -p /mnt/secureusb
sudo mount /dev/mapper/secureusb /mnt/secureusb
ls /mnt/secureusb
```

✅ Now it's ready to use. Unmount and close:

```bash
sudo umount /mnt/secureusb
sudo cryptsetup luksClose secureusb
```

---

## 🚫 Optional: Disable USB Auto-Mount

1. Create a udev rule to block auto-mount:

```bash
sudo nano /etc/udev/rules.d/100-no-automount.rules
```

Paste this:
```bash
ENV{ID_FS_USAGE}=="filesystem", ENV{UDISKS_IGNORE}="1"
```

2. Save and reload udev:

```bash
sudo udevadm control --reload
```

✅ Now USB sticks won’t mount automatically — you’ll always mount them manually via `cryptsetup`.

---

## 📁 Recommended Secure Structure

```bash
/mnt/secureusb/
├── 2fa-backups/
├── tokens/
├── keys/
└── vault-notes.txt
```

---

## 🔐 Author: binarysec.lab  
This SOP is part of the `cloud-soc-lab` project.
```
