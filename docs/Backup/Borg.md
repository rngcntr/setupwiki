# Borg Backup

!!! quote "Source: [borgbackup.readthedocs.io](https://borgbackup.readthedocs.io/en/stable/)"

Step-by-step guide to using borg as a tool for backups across multiple devices. I'll use a client-server approach where the server stores backups of multiple clients.

## Partition the physical drive and create a filesystem

!!! quote "Source: [www.digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)"

Find the device name of your hard drive first using `lsblk` or `parted -l`. The name is in the format `sdX` where `X` is a lowercase letter. Create a new primary partition on this device using `parted`. When asked, confirm with `Yes`.

```console
# parted /dev/sdX/
(parted) mklabel gpt
(parted) mkpart primary 1 -1
(parted) quit
```

Use `mkfs.ext4` to format the partition as ext4. Optionally, add a label by passing `-L` to the command. I use the label `backup`.

```console
# mkfs.ext4 -L backup /dev/sdX1
```

This process may take a little while. Verify it's success afterwards by checking `lsblk --fs`.

### Mounting the partition at boot time

In order to mount the filesystem each time our server boots, edit the file `/etc/fstab`. In this case we add a line referencing the UUID found in `lsblk --fs` and mount it as ext4 to `/mnt/backup`.

```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/backup    ext4    defaults   0       2
```

Now reboot or manually mount the filesystem.

```console
# /mnt/backup
# mount -a
```

## Create a backup user and set it's home directory

Add a user on the server who will have access to borg. Set the home directory to the backup partition.

```console
# useradd -m borg
# passwd borg
# usermod -d /mnt/backup/borg -m borg
```

## Allow ssh login as borg user

On each client device, create a new ssh identity for borg and copy the public key to the server.

```console
$ ssh-keygen -t ed25519
$ ssh-copy-id -i «path/to/key» borg@«server»
```

## Restrict borg user access

In case a single machine is compromised, an attacker will have acces to the borg server and will be able to delete all backups and control the server. To avoid this, we can restrict ssh access to only allow the borg command. Therefore, we log in as borg on our server and edit the file `.ssh/authorized_keys`. For each client, find the corresponding line and expand it by specifying `command="..."` as seen below:

```
command="borg serve --restrict-to-path ~/repo/«client»" ssh-ed25519 «key_fingerprint» «user»@«client»
```

Note that an attacker is still able to run `borg prune` and thereby delete existing backups. In order to avoid this, we can additionaly specify the `--append-only` flag.

In this case we will not be able to run `borg prune` remotely anymore. To delete old backups, we will need physical access to the machine.

```
command="borg serve --restrict-to-path ~/repo/«client» --append-only" ssh-ed25519 «key_fingerprint» «user»@«client»
```

Finally, we can now disable normal login for the user borg.

```console
# passwd -l borg
```

!!! info "Login as borg via root"
    After doing this, we will still be able to log into our borg account by running `su - borg` as root.
    ```console
    # su - borg
    ```

## Create a borg repository

As `borg` on `«server»`:
```console
$ mkdir -p ~/repo/«client»
```

On client:

```console
$ borg init --encryption=repokey borg@«server»:~/repo/«client»
```

Export the repository key and store it in a safe location (e.g. password safe)

```console
$ borg key export borg@«server»:~/repo/«client» ./borg-key-«client»
```

## Backup a single file or directory

```console
# borg create -s --progress borg@«server»:~/repo/«client»::«archive» /path/to/file
```

## Backup an entire filesystem

!!! quote "Source: [thomas-leistner.de](https://thomas-leister.de/server-backups-mit-borg/)"

Locations you want to exclude:

    #!
    /dev
    /lost+found
    /mnt
    /media
    /proc
    /run
    /sys
    /tmp

```console
# borg create --progress --verbose --stats --exclude-caches --exclude '/dev/*' \
    --exclude '/lost+found/*'       \
    --exclude '/mnt/*'              \
    --exclude '/media/*'            \
    --exclude '/proc/*'             \
    --exclude '/run/*'              \
    --exclude '/sys/*'              \
    --exclude '/tmp/*'              \
    --exclude re:/\\.cache/         \
    --exclude re:/\\.ccache/        \
    borg@«server»:~/repo/«client»::«client»-«date» /
```

## Restore a backup

## Delete an archive

```console
# borg delete borg@«server»:~/repo/«client»::«archive»
```

## Delete old archives

```console
# borg prune --verbose --stats --progress --list \
    --keep-daily=14                 \
    --keep-weekly=8                 \
    --keep-monthly=12               \
    --keep-yearly=10                \
    borg@«server»:~/repo/«client»
```

## Automating daily backup

!!! quote "Source: [www.thomas-krenn.com](https://www.thomas-krenn.com/de/wiki/Backup_unter_Linux_mit_rsnapshot#Anacron)"
