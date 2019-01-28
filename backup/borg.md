# Borg Backup ([source](https://borgbackup.readthedocs.io/en/stable/))

Step-by-step guide to using borg as a tool for backups across multiple devices. I'll use a client-server approach where the server stores backups of multiple clients.

## Partition the physical drive and create a filesystem ([source](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux))

Find the device name of your hard drive first using `lsblk` or `parted -l`. The name is in the format `sdX` where `X` is a lowercase letter. Create a new primary partition on this device using `parted`. When asked, confirm with `Yes`.

```sh
# parted /dev/sdX/
(parted) mklabel gpt
(parted) mkpart primary 1 -1
(parted) quit
```

Use `mkfs.ext4` to format the partition as ext4. Optionally, add a label by passing `-L` to the command. I use the label `backup`.

```sh
# mkfs.ext4 -L backup /dev/sdX1
```

This process may take a little while. Verify it's success afterwards by checking `lsblk --fs`.

### Mounting the partition at boot time

In order to mount the filesystem each time our server boots, edit the file `/etc/fstab`. In this case we add a line referencing the UUID found in `lsblk --fs` and mount it as ext4 to `/mnt/backup`.

```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/backup    ext4    defaults   0       2
```

Now reboot or manually mount the filesystem.

```sh
# /mnt/backup
# mount -a
```

## Create a backup user and set it's home directory

Add a user on the server who will have access to borg. Set the home directory to the backup partition.

```sh
# useradd -m borg
# passwd borg
# usermod -d /mnt/backup/borg -m borg
```

## Allow ssh login as borg user

On each client device, create a new ssh identity for borg and copy the public key to the server.

```sh
ssh-keygen -t ed25519
ssh-copy-id -i «path/to/key» borg@«server»
```

## Create a borg repository

As `borg` on `«server»`:
```sh
mkdir -p ~/repo/«client»
```

On client:

```sh
borg init --encryption=repokey borg@«server»:~/repo/«client»
```

Export the repository key and store it in a safe location (e.g. password safe)

```sh
borg key export borg@«server»:~/repo/«client» ./borg-key-«client»
```

## Backup a single file or directory

```sh
# borg create -s --progress borg@«server»:~/repo/«client»::«archive» /path/to/file
```

## Backup an entire filesystem

Locations you want to exclude:

```
/dev
/lost+found
/mnt
/media
/proc
/run
/sys
/tmp
```

```sh
# borg create --progress --verbose --stats --exclude '/dev/*' --exclude '/lost+found/*' --exclude '/mnt/*' --exclude '/media/*' --exclude '/proc/*' --exclude '/run/*' --exclude '/sys/*' --exclude '/tmp/*' borg@«server»:~/repo/«client»::«client»-20190128 /
```

## Restore a backup

## Delete an archive

```sh
# borg delete borg@«server»:~/repo/«client»::«archive»
```
