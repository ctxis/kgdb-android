# kgdb-android

Kernel patches to get KGDB working on the Nexus 6 and Nexus 5x.

For background, please see associated blog post at http://www.contextis.com/resources/blog/kgdb-android-debugging-kernel-boss

1. Root your Nexus
2. Clone the MSM kernel [from AOSP](https://android.googlesource.com/kernel/msm.git). Checkout to version that you need.  
   _Hint: stock kernels have a commit hash in their version string, like "3.10.73-gdc40906c97ae". Whole string after 'g' is a commit hash. So, you can checkout it with easy:_
    ```
    git checkout dc40906c97ae
    ```
3. Go to root of your kernel source (kernel/msm/) and apply the patches:
    ```
        patch -bp 1 < "kgdboc.c-patch"
        patch -bp 1 < "msm_serial_hs_lite.c-patch"
    ```
4. Configure the kernel to enable kernel-debug feature.  
   At this step you need to edit '...config' files. There are 2 ways to do.  
   a) build the kernel one time, and change kbuild-generated '.config' file at the root of kernel source. Build kernel again, to reuse this configuration.  
   b) change config file, that correspond to your device. For example "kernel/msm/arch/armv8/bullhead_defconfig", then just build kernel. [How to select device](https://source.android.com/setup/build/running#selecting-device-build)  
   - Lookup and remove (or set to ="n"):
       ```
       CONFIG_MSM_WATCHDOG_V2
       CONFIG_STRICT_MEMORY_RWX
       CONFIG_MSM_WATCHDOG_CTX_PRINT
       ````
   - Lookup and set to ="y" (or add if not exist):
       ```
       CONFIG_CONSOLE_POLL
       CONFIG_KGDB_SERIAL_CONSOLE
       CONFIG_KGDB
       ```
5. Deprecated compiler.  
   You may fail with a freshly installed package of the old compiler, for example GCC4.9, which was going to your kernel. Check toolchain gcc content. If there is python wrapper, turn off deprecation-info printing, kbuild does not understand it's output.
   
6. Create your boot image, passing console arguments e.g. to update a stock image I used:  
    ```abootimg -u boot.img -k zImage-dtb -c 'cmdline=console=ttyHSL0,115200,n8 kgdboc=ttyHSL0,115200 kgdbretry=4'```
7. Boot your phone into the bootloader (adb reboot bootloader) and on your host run:

    ```fastboot oem config console enable```

8. Reboot into bootloader again
9. Plug in your debug cable (see blog)
10. Boot your image e.g. ```fastboot boot boot.img```
11. Open a shell (adb shell), su to root, then type:
    ```
    echo -n g > /proc/sysrq-trigger
    ```
12. Hit enter
13. On your host machine fire up GDB (you'll need a working version of GDB cross-compiled for ARM):

```
    arm-eabi-gdb ./vmlinux
    (gdb) set remoteflow off
    (gdb) set remotebaud 115200
    (gdb) target remote /dev/ttyUSB0
```

You should hit the KGDB breakpoint and be able to continue, examine memory, etc.
