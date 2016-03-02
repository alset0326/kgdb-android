# kgdb-android

Kernel patches to get KGDB working on the Nexus 5.

For background, please see associated blog post at http://www.contextis.com/resources/blog/kgdb-android-debugging-kernel-boss

0. Root your Nexus 5!
0. Download the stock Nexus 5 kernel (kernel/msm) using instructions from https://source.android.com/source/building.html, and then ```cd msm```
0. Run ```git checkout 7717f76``` to switch to the proper kernel version
0. Download this repo
0. Run ```git apply ./path/to/nexus5-7717f76-kgdb-patch``` to apply patch
0. Run ```make hammerhead_defconfig``` to create .config file
0. Copy .config from this repo to replace the one in the msm directory. Or you can Run ```make menuconfig``` to enable ```CONFIG_KGDB``` and ```CONFIG_KGDB_SERIAL_CONSOLE``` options, modify option ```CONFIG_STRICT_MEMORY_RWX``` to n.
0. Run ```make``` to build your kernel source.
0. Create your boot image, passing console arguments (IMPORTENT!)

     ```abootimg -u boot.img -k zImage-dtb -c 'cmdline=console=ttyHSL0,115200,n8 androidboot.console=ttyHSL0 kgdboc=ttyHSL0,115200 kgdbretry=4 androidboot.hardware=hammerhead user_debug=31 maxcpus=2 msm_watchdog_v2.enable=0'```
     
0. Boot your phone into the bootloader (```adb reboot bootloader```)
0. Plug in your debug cable (see blog)
0. Boot or flash your image e.g. ```fastboot flash boot boot.img```
0. Open a shell (adb shell), su to root, then type:

    ```echo -n g > /proc/sysrq-trigger```

0. Hit enter
0. On your host machine fire up GDB (you'll need a working version of GDB cross-compiled for ARM):

```
    arm-eabi-gdb ./vmlinux
    (gdb) set remoteflow off
    (gdb) set remotebaud 115200   # or use 'set serial baud 115200' after gdb7.7
    (gdb) target remote /dev/ttyUSB0
```

You should hit the KGDB breakpoint and be able to continue, examine memory, etc.
