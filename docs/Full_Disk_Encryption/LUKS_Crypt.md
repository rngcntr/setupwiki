# LUKS Crypt

!!! quote "Source: [gist.github.com](https://gist.github.com/naomik/5428370)"

In this guide, I'm going to setup a keyfile or password-encrypted LUKS partition. I will be using a single, max-size partition on a single physical device. The mountpoint can be determined by running `lsblk`.

## Partition the physical device

```console
# parted /dev/sdX
(parted) mklabel gpt
(parted) mkpart primary 1 -1
(parted) quit
```

## Create the key file (optional)

Before we go further, let's create our 2048-bit key file first. I'm going to install it `/root/backup.key`

```console
# dd if=/dev/urandom of=/root/backup.key bs=1024 count=2
# chmod 0400 /root/backup.key
```

## Create LUKS partition

In my case, `/dev/sdX1` was created by `parted`. Create the LUKS partition with our key file or password now.

#### With key file

:   
    ```console
    # cryptsetup luksFormat /dev/sdX1 /root/backup.key
    ```
    
    Associating our key with the LUKS partition will allow us to automount it later and prevent us from ever seeing a password prompt.
    
    ```console
    # cryptsetup luksAddKey /dev/sdX1 /root/backup.key --key-file=/root/backup.key
    ```

#### With password

:   
    ```console
    # cryptsetup luksFormat --type luks2 /dev/sdX1
    ```






## Initialize the LUKS partition

#### With keyfile 

:      Before we can start using our LUKS partition, we have to size it properly and format it first. In order to do that, we will first use `luksOpen` which creates an IO backing device that allows us to interact with the partition. I'll call my device `backup`; you can call yours whatever you want.
    
    ```console
    # cryptsetup luksOpen /dev/sdX1 backup --key-file=/root/backup.key
    ```
    
    The LUKS mapping device will now be available at `/dev/mapper/backup`

#### With password

:   To gain access to the encrypted partition, unlock it with the device mapper, using:
    
    ```console
    # cryptsetup open /dev/sdX1 backup
    ```





## Size the LUKS partition

When using `resize` without any additional vars, it will use the max size of the underlying partition.

```console
# cryptsetup resize backup
```

## Format the LUKS partition

I'm going to use `ext4`; you can use whatever you want.

```console
# mkfs.ext4 /dev/mapper/backup
```

## Create a mount point

I'll create a mount point at `/backup`

```console
# mkdir -p /backup
# chmod 755 /backup
```

## Mount the LUKS mapping device

```console
# cryptsetup open /dev/sdX1 backup
# mount /dev/mapper/backup /backup
```

To check the status, use:

```console
$ df /backup
```

## Unmount the LUKS mapping device

```console
# unmount /backup
# cryptsetup close backup
```

## Automountable

#### With keyfile

:   
    To avoid the hassle of mounting are encrypted volume manually, we can set it up such that it automounts using the specified key file. First you have to get the `UUID` for your partition.
    
    ```console
    $ ls -l /dev/disk/by-uuid
    ```
    
    Find the UUID that links to your disk. It has the format `xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.
    
    ```console
    # export UUID="xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    # echo "backup UUID=${UUID} /root/backup.key luks" >> /etc/crypttab
    ```
    
    Finally, specify the automount

    ```console
    # echo "/dev/mapper/backup /backup auto" >> /etc/fstab
    ```

#### With password

:   
    ```console
    # echo "/dev/mapper/backup /backup auto defaults,errors=remount-ro 0 2" >> /etc/fstab
    ```

Mount stuff!

```console
# mount -a
```
