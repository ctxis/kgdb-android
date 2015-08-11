# kgdb-android

Kernel patches to get KGDB working on the Nexus 6.

For background, please see associated blog post at http://www.contextis.com/resources/blog/kgdb-android-debugging-kernel-boss

0. Root your Nexus 6!
1. Download and build the stock Nexus 6 kernel (kernel/msm) using instructions from https://source.android.com/source/building.html
2. Download this directory structure into the root of your kernel source (kernel/msm/) including the .config file.
3. Re-build your kernel source.
4. Create your boot image, passing console arguments e.g. to update a stock image I used: ```abootimg -u boot.img -k zImage-dtb -c 'cmdline=console=ttyHSL0,115200,n8 kgdboc=ttyHSL0,115200 kgdbretry=4'```
5. Boot your phone into the bootloader (adb reboot bootloader) and on your host run:

    ```fastboot oem config console enable```

6. Reboot into bootloader again
7. Plug in your debug cable (see blog)
8. Boot your image e.g. ```fastboot boot boot.img```
9. Open a shell (adb shell), su to root, then type:


    ```echo -n g > /proc/sysrq-trigger```


10. Hit enter
11. On your host machine fire up GDB (you'll need a working version of GDB cross-compiled for ARM):

```
    arm-eabi-gdb ./vmlinux
    (gdb) set remoteflow off
    (gdb) set remotebaud 115200
    (gdb) target remote /dev/ttyUSB0
```

You should hit the KGDB breakpoint and be able to continue, examine memory, etc.
