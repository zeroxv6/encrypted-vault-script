```text
[01//DESCRIPTION]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
This is a zero-knowledge hybrid storage architecture. It transforms a
standard USB drive into a double-layer encrypted vault. Data is locked
locally via LUKS and synchronized to Google Drive via Rclone Crypt.
No raw data ever touches the cloud.

[02//THE_HARDWARE_LOCK]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
To initialize the physical partition (e.g., /dev/sdc1):

# Format block device
$ sudo cryptsetup luksFormat /dev/sdc1

# Create mapping and filesystem
$ sudo cryptsetup open /dev/sdc1 vault_mapper
$ sudo mkfs.ext4 -m 0 -E lazy_itable_init=0 /dev/mapper/vault_mapper
$ sudo cryptsetup close vault_mapper

[03//SOURCE_CODE: OPEN-VAULT.SH]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#!/bin/bash
# FILE: ~/.local/bin/open-vault.sh
# AUTH: zeroxv6
# DESC: UNLOCKS HARDWARE AND MOUNTS LOCAL VAULT

# UUID_DETECTION_PROTOCOL:
# Identify your partition (e.g., sdc1) and run:
# lsblk -dno UUID /dev/sdc1
UUID="REPLACE_WITH_YOUR_UUID"

MOUNT_POINT="$HOME/Vault"

echo "» STATUS: INITIATING_HARDWARE_UNLOCK"
sudo cryptsetup open UUID=$UUID my_vault

echo "» STATUS: MOUNTING_FILESYSTEM"
mkdir -p "$MOUNT_POINT"
sudo mount /dev/mapper/my_vault "$MOUNT_POINT"
sudo chown -R $USER:$USER "$MOUNT_POINT"

echo "» STATUS: LAUNCHING_FILE_MANAGER"
dolphin "$MOUNT_POINT" & disown

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "SYSTEM_READY: VAULT_OPENED"
read -p "PRESS ENTER TO TERMINATE SESSION..."

[04//SOURCE_CODE: LOCK-VAULT.SH]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#!/bin/bash
# FILE: ~/.local/bin/lock-vault.sh
# AUTH: zeroxv6
# DESC: SYNC TO CLOUD AND SEVER HARDWARE CONNECTION

MOUNT_POINT="$HOME/Vault"

echo "» STATUS: RUNNING_CLOUD_SYNC_PROTOCOL"
rclone sync "$MOUNT_POINT" vault_cloud: --progress --exclude ".Trash-**"

echo "» STATUS: RUNNING_HARDWARE_LOCKDOWN"
sudo umount "$MOUNT_POINT"
sudo cryptsetup close my_vault

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "SYSTEM_SECURE: YOU_MAY_UNPLUG_HARDWARE"
sleep 3

[05//KDE_PLASMA_INTEGRATION]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Place .desktop files in ~/.local/share/applications/ to enable taskbar
pinning. Execute commands via 'konsole -e' to ensure password prompts
are visible during the mounting sequence.

[06//CORE_SECURITY_SPECS]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LAYER_1 (Physical): LUKS / dm-crypt
LAYER_2 (Network) : NaCl Secretbox (XSalsa20 + Poly1305)
KEY_DERIVATION    : Scrypt (N=16384, r=8, p=1)
