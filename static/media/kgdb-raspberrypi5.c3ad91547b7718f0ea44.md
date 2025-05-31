# Setup KGDB for Raspberry Pi 5

Debugging the Linux kernel can be intimidating, but with the right tools, it becomes a lot more approachable. In this post, I'll walk you through setting up KGDB (the kernel debugger) on a Raspberry Pi 5, so you can debug kernel code just like you would any other program. Whether you're working on device drivers, kernel modules, or just want to poke around the kernel, this guide is for you.

---

## What is KGDB?

KGDB is a debugger for the Linux kernel that lets you debug kernel code using GDB (GNU Debugger) over a serial or virtual connection. It's incredibly useful when you're writing or debugging device drivers, kernel modules, or kernel-level code.

In this guide, we’ll focus on enabling KGDB over the serial port on a Raspberry Pi 5. While this tutorial is Pi-specific, the steps should be adaptable to most Linux-based systems with minor changes.

---

## Prerequisites

Before we start, make sure you have the following:

- **Raspberry Pi 5**
- **USB to TTL converter**
- **Female-to-Female Jumper Wires**
- **A Linux-based host machine**
- **SD card**
- **SD card reader**

---

## Set up a Custom Kernel for KGDB

We'll cross-compile a custom kernel for the Raspberry Pi 5 with KGDB configuration enabled. This process is straightforward, but does require some patience (kernel builds can take a while!).

First, clone the official Raspberry Pi kernel and set the `KERNEL` parameter according to your Pi version. For the Pi 5, it's `kernel_2712`:

```
git clone https://github.com/raspberrypi/linux.git
cd linux
KERNEL=kernel_2712
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2712_defconfig
```

Next, enable KGDB and some useful debug options in the kernel config. The following commands also include a few DRM and V3D tweaks (for graphics debugging):

```
./scripts/config --enable CONFIG_VT                          # virtual-terminal layer
./scripts/config --enable CONFIG_VT_CONSOLE                  # VT as kernel console
./scripts/config --enable CONFIG_KGDB                        # remote kernel gdb
./scripts/config --enable CONFIG_KGDB_KDB                    # built-in kdb shell
./scripts/config --set-val CONFIG_PANIC_TIMEOUT 0            # no auto-reboot on panic
./scripts/config --disable CONFIG_RANDOMIZE_BASE             # disable KASLR
./scripts/config --disable CONFIG_WATCHDOG                   # drop watchdog support
./scripts/config --set-val CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE 1  # basic SysRq keys
./scripts/config --enable CONFIG_DEBUG_KERNEL                # global debug features
./scripts/config --enable CONFIG_DEBUG_INFO                  # add DWARF symbols
./scripts/config --enable CONFIG_DEBUG_INFO_DWARF4           # use DWARF-4 format
./scripts/config --enable CONFIG_FRAME_POINTER               # keep frame pointers
./scripts/config --enable CONFIG_GDB_SCRIPTS                 # ship gdb helper scripts
```

If you want to debug the V3D kernel driver on the Pi 5, enable these as well:

```
scripts/config --enable CONFIG_DRM                     # Enable Direct Rendering Manager (graphics subsystem)
scripts/config --enable CONFIG_DRM_V3D                 # Enable 3D graphics support for Broadcom V3D
scripts/config --enable CONFIG_DRM_VC4                 # Enable Broadcom VC4 DRM driver (e.g. for Raspberry Pi)
scripts/config --enable CONFIG_DRM_VC4_HDMI_CEC        # Enable HDMI CEC support for VC4
```

Run the following to confirm your config changes:

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

Now, build the kernel. This will take some time:

```
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

Once the build is complete, it's time to install the new kernel on your Pi 5. If your SD card already has a working OS image, connect it to your host system. Use `lsblk` to find the device name (e.g., `sda` or `sdb`).

Create mount directories and mount the partitions (adjust `sda1`/`sda2` as needed):

```
mkdir mnt
mkdir mnt/boot
mkdir mnt/root
sudo mount /dev/sda1 mnt/boot        
sudo mount /dev/sda2 mnt/root
```

Install the built kernel modules:

```
sudo env PATH=$PATH make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=mnt/root modules_install
```

Backup the current kernel and copy over the new files:

```
sudo cp mnt/boot/$KERNEL.img mnt/boot/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image mnt/boot/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb mnt/boot/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* mnt/boot/overlays/
sudo cp arch/arm64/boot/dts/overlays/README mnt/boot/overlays
```

A few more steps to enable KGDB:
- Add `kgdboc=serial0,115200` to `mnt/boot/cmdline.txt` (enables KGDB over serial console)
- Add `dtparam=uart0_console=on` under `[all]` in `mnt/boot/config.txt`

When done, unmount the partitions:

```
sudo umount mnt/boot
sudo umount mnt/root
```

Put the SD card back in your Pi 5 and boot it up. Enable serial in the interface options:

```
sudo raspi-config and turn on serial in interface options
```

---

## Setup KDMX on the Host

**KDMX** isn't strictly required, but I highly recommend it. If you want to use the serial port for both logging into the Pi and running KGDB, KDMX makes life easier by creating two virtual ports from a single serial port. That way, you can run KGDB on one and use the other for a terminal session.

First, install some prerequisites:

```
sudo apt install cu
sudo usermod -aG dialout $USER     #logout or restart after running this
```

Clone and build KDMX:

```
mkdir kdmx
cd kdmx
git clone git://git.kernel.org/pub/scm/utils/kernel/kgdb/agent-proxy.git .
make
```

---

## Connect Pi 5 to Host

To connect your Pi 5 to the host, you'll need a USB to TTL converter. I've tried both the CP2102 and PL2303, and both work fine (you can find them on Amazon).

#### Serial CP2102

![CP2102 USB to TTL Converter](/blogs/CP2102.jpg)

#### PL2303 converter

![PL2303 USB to TTL Converter](/blogs/PL2303.jpg)

Connect the converter to the Pi 5 GPIO using jumper wires:
- **GND** to **GND**
- **TDX** to **RDX**
- **RDX** to **TDX**

It should look like this:

![Wiring Example 1](/blogs/look_like_this_1.jpg)

![Wiring Example 2](/blogs/look_like_this_2.jpg)

Install Picocom to log in to the Pi 5:

```
sudo apt install picocom
```

---

## Log in to Pi 5 via Host and Run KGDB

1. Connect the USB to TTL converter to your host via USB. Make sure the Pi 5 is powered off at this point.
2. Go to the KDMX directory on your host and run KDMX:

```
./kdmx -n -p "/dev/ttyUSB0" -s /tmp/kdmx_ports -b 115200
```

The `-b 115200` sets the baud rate. Make sure it matches the value in `cmdline.txt` or you might see garbage in Picocom. After running KDMX, you'll see output like:

```
/dev/pts/2 is slave pty for terminal emulator
/dev/pts/3 is slave pty for gdb

Use <ctrl>C to terminate program
```

Take note of the emulator and gdb paths.

3. Open a new terminal and run Picocom (replace with your emulator path):

```
picocom -b 115200 /dev/pts/2
```

4. Power on the Pi 5. You should see the login screen in Picocom:

```
raspberrypi login: pi
Password: 
Linux raspberrypi 6.12.28-v8-16k+ #1 SMP PREEMPT Sun May 18 19:27:06 IST 2025 aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun May 25 12:30:05 IST 2025 on tty1
pi@raspberrypi:~$ 
```

5. After logging in, start KDB:

```
sudo su
echo g > /proc/sysrq-trigger
```

6. In another terminal, go to the Linux directory where you built the custom kernel and start GDB (install with `sudo apt install gdb-multiarch` if needed):

```
gdb-multiarch vmlinux
```

7. In GDB, connect to KGDB (replace with your gdb path from KDMX):

```
target remote /dev/pts/3
```

That's it! You now have a working KGDB setup on your Raspberry Pi 5. You can use all the usual GDB commands (`backtrace`, `break`, `up`, `down`, `list`, etc.) to debug kernel code. It's a powerful way to explore and troubleshoot the Linux kernel.

---

If you have questions or run into issues, feel free to reach out. Happy hacking!