---
layout: post
title: Copy IPFire Backup to Synology Diskstation
---

Running IPFire instead of a cheap plastic router is a good thing. But when your IPFire's disk crashes, all the little settings that you have made over several years are lost (think of the last "smart" device that you allowed WIFI access). Fortunately, IPFire is able to create backups of its configuration in `/var/ipfire/backup`. Backups can be triggered via *System -> Backup* in the UI. Even better, IPFire automatically creates a backup before each update. So if you update IPFire regularly, you'll also have recent backups.

In case of a disk failure all you need to do is buy a new disk, download and install IPFire and load the backup via a few UI clicks. However, "loading the backup" requires to backup the backup to a safer place beforehand. In this example, I'll describe how to trigger the backup process directly, copy the backup files to a Synology Diskstation via SSH and how to setup the Diskstation to receive such backups. With a few tweaks the following steps will also work on other Unix-like systems.


## Setup SSH Keys on the IPFire Host

Setup a SSH keypair, e.g. using ECDSA with 521 bit keys
```bash
ssh-keygen -t ecdsa -b 521
```


## Configure Synology Diskstation to Receive the Backup

* In the Diskstation UI:
  * Create a dedicated user for the Backup, e.g. "ipfire"
  * Give the user permission on the shared folder which stores the backup

* Login to the Diskstation and configure the user's home directory
  ```bash
  # create a home directory for the user (use the group that you assigned in the Diskstation UI)
  mkdir -p /home/ipfire
  chown ipfire:<mygroup> /home/ipfire
  chmod o-rx /home/ipfire

  # Edit /etc/passwd and set the home directory and a login shell
  ipfire:x:<userid>:<groupid>:ipfire backup:/home/ipfire:/bin/sh
  ```

* Login as the backup user and configure SSH

  ```bash
  # Create the .ssh folder
  mkdir .ssh
  chmod go-rx .ssh
  touch ~/.ssh/authorized_keys
  ```

* Copy the public key from IPFire (content of `/root/.ssh/id_ecdsa.pub` on the IPFire host) into `/home/ipfire/.ssh/authorized_keys`

## Setup a Daily Backup on the IPFire Host

* Create a backup script, e.g. `copy-backup.sh`:

  ```bash
  #/bin/bash
  
  USER=ipfire
  HOST=diskstation
  SOURCE_DIR=/var/ipfire/backup
  ADDON_DIR="${SOURCE_DIR}/addons"
  BACKUP_SCRIPT="${SOURCE_DIR}/bin/backup.pl"
  DESTINATION_DIR=/path/to/the/backup/directory
  LOG_FILE=/var/log/copy_backup.log
  
  function log() {
    message="$1"
  
    echo "$(date +"%Y-%m-%d %T"): ${message}"  >> "${LOG_FILE}" 2>&1
  }
  
  log "Creating Backup of ${SOURCE_DIR}"
  # "include" will add the log files to the backup
  "${BACKUP_SCRIPT}" include
  log "Finished with exit code $?"
  
  # Backup all addons
  for f in $(find "${ADDON_DIR}/includes" -type f); do
    addon=$(basename $f)
    log "Creating backup of Addon '$addon'"
    "${BACKUP_SCRIPT}" addonbackup "$addon"
    log "Finished with exit code $?"
  done
  
  log "Copying backup to ${HOST}:${DESTINATION_DIR}"
  scp -r "${SOURCE_DIR}" ${USER}@${HOST}:"${DESTINATION_DIR}" >> "${LOG_FILE}" 2>&1
  log "Finished with exit code $?."
  
  log "Backup done."
  ```

* Link the backup script in `fcron.daily`
 
  ``` bash
  cd /etc/fcron.daily
  ln -s /root/copy-backup.sh copy-backup
  ```

## Possible Improvements of this Post

* Describe how to setup IPFire from scratch and restore the backup
* `/root` and `/etc/fcron.*` are not part of the backup. Describe how to include them into the backup
