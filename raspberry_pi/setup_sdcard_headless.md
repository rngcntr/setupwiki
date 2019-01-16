# Raspberry Pi Setup Guide ([source](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md))

This guide describes the installation of Raspbian on a Raspberry Pi 3B+ as a headless server.

## Download Raspbian Image

The latest Raspbian image can be downloaded from https://www.raspberrypi.org/downloads/raspbian/. In this case no desktop environment is needed so we use **Raspbian Stretch Lite**.

Extract the zip file by running the following command with adjusted date:

```sh
unzip 2018-11-13-raspbian-stretch-lite.zip
rm 2018-11-13-raspbian-stretch-lite.zip
```

## Write the image to the SD card

Run `lsblk` before and after inserting the SD card into the SD card reader in order to determine the `NAME` of the SD card. The device should be listed as `sdX` where `X` is a lower-case letter indicating the device.

If any partitions on the SD card are currently mounted (check the rightmost column of `lsblk`), unmount all of them.

```sh
umount /dev/sdX1
```

Write the image to the SD card with `dd`. `if=` specifies the image as the source file and `of=` specifies the SD card as the destination. **Make sure to use the correct device to avoid data loss.** The destination is specified by device name, not by partition name.

```sh
# dd bs=4M if=2018-11-13-raspbian-stretch-lite.img of=/dev/sdX status=progress conv=fsync
```
If any errors occur, try block size `1M` instead of `4M`

### Optional: Checking whether the image was written correctly

The output of the `dd` command shows a number of written records (`xxx+0 records in`). Use this record count to copy the image back from the SD card to your local drive.

```sh
# dd bs=4M if=/dev/sdX of=from-sd-card.img status=progress count=xxx
```

Truncate the new image to the size of the original image and compare the new image with the original one.

```sh
# truncate --reference 2018-11-13-raspbian-stretch-lite.img from-sd-card.img
diff -s from-sd-card.img 2018-11-13-raspbian-stretch-lite.img
```

## Eject the SD card

Flush the write cache.

```sh
sync
```

Remove the SD card from the card reader.

## Enable ssh for headless systems

Insert the SD card into the card reader. It will be mounted by defalt. Navigate to the boot partition of the SD card and create an empty file named `ssh`. The mountpoint of the boot partition can be determined via `lsblk`.

```ssh
cd /run/media/$USER/boot
touch ssh
```

## Enable WIFI for headless systems ([source](https://raspberrypi.stackexchange.com/a/10413))

Insert the SD card into the card reader. It will be mounted by defalt. Navigate to the boot partition of the SD card. The mountpoint of the boot partition can be determined via `lsblk`.

```ssh
cd /run/media/$USER/boot
```

Create a file named `wpa_supplicant.conf` with the following content:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=«your_ISO-3166-1_two-letter_country_code»

network={
    ssid="«your_SSID»"
    psk="«your_PSK»"
    key_mgmt=WPA-PSK
}
```

Replace «your_ISO-3166-1_two-letter_country_code» with your [ISO Country Code](https://www.iso.org/obp/ui/#search), «your_SSID» with the SSID of your network and «your_PSK» with the corresponding wifi password.

## Connect to the Raspberry Pi [source](https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0)

Establish an ssh connection to the Raspberry Pi with default credentials:

```
User: pi
Password: raspberry
```

```sh
ssh pi@raspberrypi
```

If this does not initialize a connection, find out the IP either by running ```nmap -p 22 «subnet mask»``` on your local subnet or finding the device in the router's DHCP table.

## Secure login and permissions

If needed, configure the Raspberry Pi via the configuration script.

```sh
# raspi-config
```

Change the default root password.

```sh
# sudo passwd root
```

Now create a new user for ssh access and allow sudo access for the new account. Don't remove sudo access for pi immediately. You can remove it once you have tested sudo with the new user account.

```sh
# adduser «user»
# usermod -a -G sudo «user»
```

If you have sudo access with your new account, remove the default user pi.

```sh
# userdel pi
```

For further ssh setup instructions visit [Securing SSH Guide](../ssh/secure_setup.md)
