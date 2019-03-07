# Manjaro Troubleshooting

## xbacklight not working

!!! quote "Source: [wiki.archlinux.org](https://wiki.archlinux.org/index.php/backlight#xbacklight)"

At some point in time, `xbacklight` stopped working for me. It accepts all valid inputs and terminates with exit code 0 without applying any of the given modifications to the system. This problem can be solved by putting the following lines into the file `/etc/X11/xorg.conf.d/20-video.conf`:

``` tab="/etc/X11/xorg.conf.d/20-video.conf"
Section "Device"
    Identifier  "Intel Graphics" 
    Driver      "intel"
    Option      "Backlight"  "intel_backlight"
EndSection
```

After rebooting, everything should work as expected.
