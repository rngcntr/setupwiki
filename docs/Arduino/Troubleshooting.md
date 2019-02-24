# Arduino Troubleshooting

## libtinfo.so.5

!!! quote "Source: [github.com](https://github.com/stevearc/vim-arduino/issues/2), [www.dinotools.de](https://www.dinotools.de/2015/03/15/problem-mit-arduino-ide-unter-arch-linux-beheben/)"

It seems to be a known issue with Arduino software on Arch Linux: When trying to load any program to a microcontroller, the following error emerges:

```console
error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
```

According to [www.dinotools.de](https://www.dinotools.de/2015/03/15/problem-mit-arduino-ide-unter-arch-linux-beheben/), the reason is that `libtinfo.so.5` is not included in `libcurses` packages on Arch distributions.
Therefore Arduino cannot find the library where it expects it to be.
A simple (but dirty) approach to solve the problem is to create a link to `libtinfo.so.5` which exists on my system.

```console
# ln -s /lib/libtinfo.so.6 /lib/libtinfo.so.5
```

Others suggest linking to various `libncurses.so` files which I haven't tested yet.
