
```md
# 🔐 KeePassXC Encrypted Vault – Backup Failure Postmortem + SOP

**Author:** binarysec.lab  
**Last updated:** 2025-04-16  
**Scope:** Full lifecycle of KeePassXC vault setup, encrypted USB backup, corruption incident, and root cause analysis  
**Audience:** Internal security team, future staff, Tier 3 SOP archive

---

## 📦 Phase 1 – Initial Vault Setup

- Tool: `KeePassXC` installed on Pop!_OS using:
  ```bash
  sudo apt update && sudo apt install keepassxc
  ```
- Database created:
  ```
  ~/Vault/KeePassXC.kdbx
  ```
- Folder structure:
  - GitHub
  - ProtonMail
  - Vault Info
  - Clients
  - API Tokens

---

## 🔐 Phase 2 – USB Vault Creation + Backup

1. **Identify USB:**
   ```bash
   lsblk -f  # → found /dev/sda
   ```

2. **Wipe and encrypt:**
   ```bash
   sudo wipefs --all /dev/sda
   sudo cryptsetup luksFormat /dev/sda
   sudo cryptsetup luksOpen /dev/sda secureusb
   ```

3. **Filesystem and mount:**
   ```bash
   sudo mkfs.ext4 /dev/mapper/secureusb
   sudo mkdir -p /mnt/secureusb
   sudo mount /dev/mapper/secureusb /mnt/secureusb
   ```

4. **Create folders:**
   ```bash
   sudo mkdir -p /mnt/secureusb/vault-backups
   ```

5. **Copy vault:**
   ```bash
   sudo cp ~/Vault/KeePassXC.kdbx /mnt/secureusb/vault-backups/KeePassXC-2025-04-16.kdbx
   ```

---

## 🧨 Phase 3 – Backup Failure

- Error in KeePassXC:  
  ```
  Error while reading database: HMAC mismatch
  ```

- Restore attempt:
  ```bash
  sudo cp /mnt/secureusb/vault-backups/KeePassXC-2025-04-16.kdbx ~/Vault/restored-KeePassXC.kdbx
  sudo chown $USER:$USER ~/Vault/restored-KeePassXC.kdbx
  ```

- Still failed with same error

---

## 🔍 Root Cause Analysis

| Mistake | Impact | Fix |
|--------|--------|-----|
| ❌ No `sync` after `cp` | File might not have flushed to USB | Add `sync` after every copy |
| ❌ KeePassXC left open | File may have been mid-write | Always close the app before backup |
| ❌ No hash/checksum check | Backup might’ve been corrupted silently | Use `sha256sum` or `cmp` to confirm |
| ❌ Didn’t open backup to test | Trusted a file we never validated | Always test backups post-copy |

---

## ✅ Updated Backup Procedure (Improved SOP)

```bash
# 1. Close KeePassXC completely

# 2. Flush filesystem cache (optional safety)
sync

# 3. Copy file to encrypted USB
sudo cp ~/Vault/KeePassXC.kdbx /mnt/secureusb/vault-backups/KeePassXC-$(date +%F).kdbx

# 4. Sync again after write
sync

# 5. Verify file integrity
sha256sum ~/Vault/KeePassXC.kdbx /mnt/secureusb/vault-backups/KeePassXC-$(date +%F).kdbx

# 6. Open backup file in KeePassXC manually and confirm it works
```

---

## 🧠 Lessons Learned

- 🔁 Trust **only verified backups**
- 🧪 Always test `.kdbx` restores before deleting anything
- 🔒 Never copy while the file is open
- 💾 `sync` + `sha256sum` = Tier 3 security habit
- 💥 We now have a real postmortem standard to reuse

---

## ✅ Next Actions

- [ ] Rebuild vault manually  
- [ ] Copy and verify with new SOP  
- [ ] Document in GitHub (`cloud-soc-lab/sops/vault-failure-postmortem.md`)  
- [ ] Teach this process to future staff

