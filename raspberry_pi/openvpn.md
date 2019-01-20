# OpenVPN on a Raspberry Pi ([source](https://www.kuketz-blog.de/pivpn-raspberry-pi-mit-openvpn-raspberry-pi-teil3/))

## Create an openvpn user

To avoid conflicts with other users, create a new user for the VPN service and set a password for this user. In my case the user is called «openvpn».
```sh
# adduser «openvpn»
# passwd «openvpn»
# sudo usermod -a -G sudo «openvpn»
# mkdir /home/«openvpn»
# chown /home/openvpn «openvpn»
# chgrp /home/openvpn «openvpn»
```

Log in as «openvpn».

```sh
su - «openvpn»
```

## Run the PiVPN install script

```sh
curl -L install.pivpn.io > pivpn.sh
chmod +x pivpn.sh
```

Once you are sure the script is not harmful in any way, run the installation script.

```
# ./pivpn.sh
```

1. Confirm the current network settings and make sure the shown IP address (eg. 192.168.2.110) is always assigned to this device by the router.
2. Choose the user «openvpn».
3. When asked, select UDP as a protocol.
4. Confirm the default OpenVPN port 1194.
5. Select ECDSA encryption strength. (Recommended: 256-bit)
6. If you use DNS, enter the domain name.
7. Reboot after the installation has finished

## Add clients

To generate a client configuration, use:

```sh
pivpn add
```

Select a different username and password for each client. Copy the resulting `ovpn` file to your client device via scp and use it in your OpenVPN client.
