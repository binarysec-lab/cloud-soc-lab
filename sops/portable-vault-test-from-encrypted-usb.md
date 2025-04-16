## 🔐 SOP: Test & Open KeePassXC Vault from Encrypted USB

This SOP walks through the exact process for testing the integrity and portability of a KeePassXC vault backup stored on a LUKS-encrypted USB stick.

This procedure allows you to:
- ✅ Validate the USB backup vault opens correctly
- ✅ Use the vault on other machines
- ✅ Avoid touching your primary vault in `~/Vault`

---

### 📦 Prerequisites
- You already have a file on USB at:
  ```bash
  /mnt/secureusb/vault-backups/KeePassXC-2025-04-16.kdbx
  ```
- The USB is already unlocked and mounted at `/mnt/secureusb`
- KeePassXC is installed on your system

---

### 🧪 Step-by-Step: Test the USB Vault

#### 1. ✅ Confirm the file exists on USB
```bash
sudo ls -lh /mnt/secureusb/vault-backups
```
You should see:
```bash
-rw-r--r-- 1 root root 2.4K Apr 16 20:30 KeePassXC-2025-04-16.kdbx
```

#### 2. 🔓 Fix permissions so you can open it from KeePassXC
```bash
sudo chown $USER:$USER /mnt/secureusb/vault-backups/KeePassXC-2025-04-16.kdbx
```
> Without this step, you'll get: `Unable to open file` in KeePassXC.

#### 3. 🚀 Open the vault in KeePassXC (from terminal)
```bash
keepassxc /mnt/secureusb/vault-backups/KeePassXC-2025-04-16.kdbx
```
- ✅ Don't double-click it in the file explorer
- ✅ You should see all folders and secrets
- ❌ Do not edit — this is a **read-only test**

---

### ✅ Optional: Verify Integrity Again
```bash
sha256sum ~/Vault/PWD_LG.kdbx
sha256sum /mnt/secureusb/vault-backups/KeePassXC-2025-04-16.kdbx
```
- Hashes must match — this proves the backup is valid

---

### 🧠 Summary
You now have:
- A proven, tested, and restorable KeePassXC vault backup
- Ready-to-use on any other system with KeePassXC installed
- Backed by verified integrity, encryption, and permissions
