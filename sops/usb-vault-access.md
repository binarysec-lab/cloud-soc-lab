
```md
# 🛡️ SOP: Accessing Encrypted USB Vault (LUKS) on Linux

## 🎯 Purpose

This SOP describes how to safely unlock, mount, access, and re-lock a LUKS-encrypted USB stick used to store sensitive files such as 2FA recovery codes, passwords, and API tokens.

---

## 🧰 Requirements

- Pop!_OS (or any Linux distro with `cryptsetup`)
- Encrypted USB stick (`LUKS`)
- Password for LUKS volume
- `cryptsetup` and `lsblk` installed
- `sudo` access

---

## 🪜 Step-by-Step Procedure

### ✅ 1. Identify the USB device

Plug in the USB stick and run:

```bash
lsblk -f
```

Look for a device with:
- `FSTYPE` = `crypto_LUKS`
- No mountpoint
- Likely path: `/dev/sda` or `/dev/sdb`

---

### ✅ 2. Unlock the encrypted volume

```bash
sudo cryptsetup luksOpen /dev/sdX secureusb
```

- Replace `sdX` with the correct device (e.g. `/dev/sda`)
- `secureusb` is the mapping name used for mounting

---

### ✅ 3. Mount the decrypted volume

```bash
sudo mkdir -p /mnt/secureusb
sudo mount /dev/mapper/secureusb /mnt/secureusb
```

Access your files at:

```bash
ls /mnt/secureusb
```

---

### ✅ 4. Copy Files (Optional)

```bash
cp -r /mnt/secureusb/* ~/usb-vault-backup/
```

---

### ✅ 5. Unmount and lock when finished

```bash
sudo umount /mnt/secureusb
sudo cryptsetup luksClose secureusb
```

✅ This closes and detaches the encrypted vault securely.

---

## ⚠️ Notes

- Never leave the vault mounted when unattended
- Store recovery keys in multiple locations (USB, paper copy, KeePassXC)
- Don’t use this USB for personal files or internet transfer

---

## 📁 Folder Structure (Recommended)

```bash
/mnt/secureusb/
├── 2fa-backups/
│   ├── github.txt
│   └── microsoft.txt
├── keys/
│   └── github-pat.txt
└── secrets/
    └── initial-passwords.txt
```

---

## 🔐 Author: binarysec.lab  
_This SOP is part of the cloud-soc-lab project_
```

---

### 🧾 How to Publish It

1. Go to your repo → `sops/` folder  
2. Click **“Add file” → “Create new file”**  
3. Name it:  
   ```
   sops/usb-vault-access.md
   ```
4. Paste the content above  
5. Commit with message:  
   ```
   Add SOP: Accessing encrypted USB vault
   ```

