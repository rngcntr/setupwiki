# LUKS crypt

In this guide, I'm going to setup a keyfile-encrypted LUKS partition. I will be using a single, max-size partition on a single physical device. The mountpoint can be determined by running `lsblk`.

# Partition the physical device

```sh
sudo parted /dev/sdX
(parted) mklabel gpt
(parted) mkpart primary 1 -1
(parted) quit
```

# Create LUKS partition

In my case, `/dev/sdX1` was created by `parted`. Create the LUKS partition with our key file now.

```sh
sudo cryptsetup luksFormat --type luks2 /dev/sdX1
```

# Access the LUKS partition

To gain access to the encrypted partition, unlock it with the device mapper, using:

```sh
sudo cryptsetup open /dev/sdX1 backup
```

# Size the LUKS partition

When using `resize` without any additional vars, it will use the max size of the underlying partition.

```sh
sudo cryptsetup resize backup
```

# Format the LUKS partition

I'm going to use `ext4`; you can use whatever you want.

```sh
sudo mkfs.ext4 /dev/mapper/backup
```

# Create a mount point

I'll create a mount point at `/backup`

```sh
sudo mkdir -p /backup
sudo chmod 755 /backup
```

# Mount the LUKS mapping device

```sh
sudo cryptsetup open /dev/sdX1 backup
sudo mount /dev/mapper/backup /backup
```

To check the status, use:

```sh
df /backup
```

# Unmount the LUKS mapping device

```sh
sudo unmount /backup
cryptsetup close backup
```

# Automountable

To avoid the hassle of mounting are encrypted volume manually, we can set it up such that it automounts using the specified key file. First you have to get the `UUID` for your partition.

```sh
ls -l /dev/disk/by-uuid
```

Find the UUID that links to your disk. In my case, it is `651322a-8171-49b4-9707-a96698ec826e`.

```sh
export UUID="651322a-8171-49b4-9707-a96698ec826e"
sudo echo "backup UUID=${UUID} none luks,timeout=180" >> /etc/crypttab
```

Finally, specify the automount

```sh
sudo echo "/dev/mapper/backup /backup auto defaults,errors=remount-ro 0 2" >> /etc/fstab
```

Mount stuff!

```sh
sudo mount -a
```
