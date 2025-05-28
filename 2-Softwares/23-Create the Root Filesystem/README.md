# 🧰 source_rootfs – Build Environment for Lidl Silvercrest Gateway (RTL8196E)

This directory contains everything needed to build and deploy a custom root filesystem (`newroot.bin`) and a writable user overlay (`userdata.tar`) for the Lidl Silvercrest Zigbee gateway based on a Realtek RTL8196E SoC.

It uses:

- A static cross-compiled **BusyBox** base system
- A lightweight static **Dropbear SSH server**
- A custom **Serial-to-TCP gateway** for UART bridging
- The official **Realtek RSDK 4.4.7** MIPS toolchain
- A staging layout that mimics `/` and `/userdata` partitions on the device

> 🧪 In addition to building the base root filesystem, this environment can be used to **compile and cross-develop other custom binaries** or utilities for the Lidl Silvercrest gateway. It serves as a general-purpose embedded development framework targeting RTL8196E (MIPS-Lexra architecture).

---

## 📦 Getting Started

Before building anything, download the archive `source_rootfs.tar.gz` from the [project repository](https://github.com/jnilo1/hacking-lidl-silvercrest-gateway/tree/main/2-Softwares/23-Create the Root Filesystem) and extract it into your working directory:

```bash
tar -xvzf source_rootfs.tar.gz
cd source_rootfs
```

---

## 📁 Directory Structure

```
source_rootfs/
├── build_busybox         # Compile static BusyBox
├── build_dropbear        # Compile static Dropbear SSH
├── build_rootfs          # Assemble squashfs image + generate newroot.bin
├── build_serialgateway   # Compile serial-to-TCP bridge
├── build_userdata        # Package ./userdata as userdata.tar
├── busybox.config        # Custom BusyBox configuration
├── rootfs/               # Staging rootfs tree
│   ├── squashfs-root/    # Will contain the final BusyBox/Dropbear/etc
│   └── rootfs_tool.py    # Packs final image into newroot.bin
├── rsdk-4.4.7.../        # Realtek cross-compiler toolchain
├── serialgateway/        # Source code and Makefile for TCP UART bridge
├── setup.sh              # Install required packages (on Ubuntu host)
├── userdata/             # Staging overlay for writable /userdata
└── README.md             # You are here
```

---

## ⚙️ Build Procedure

Execute the following commands **in order**, from the root of this directory:

```bash
./setup.sh              # Install required Ubuntu packages
./build_busybox         # Download, compile and stage BusyBox
./build_dropbear        # Download, compile and stage Dropbear SSH
./build_serialgateway   # Compile and install UART TCP gateway
./build_userdata        # Create userdata.tar from ./userdata directory
```

This will produce:

- `rootfs/newroot.bin`: Flashable squashfs root partition (for mtd2)
- `userdata.tar`: Writable overlay with configs and SSH keys (for mtd4)

---

## 🚀 Flashing Instructions (Lidl Silvercrest Gateway)

1. **Connect the device**
   - Via serial terminal (`putty`, `minicom`) — 38400 8N1
   - And via Ethernet to your LAN

2. **Reboot into bootloader**
   - From the terminal, run:
     ```
     reboot
     ```
   - Then press `ESC` repeatedly to enter the Realtek bootloader
   - You should see the `Realtek>` prompt

3. **Flash the rootfs**
   - From your host:
     ```bash
     ./build_rootfs
     ```
   - This sends `newroot.bin` to the device via `tftp`
   - If successful, **copy/paste the suggested `FLW` command** into the serial console to flash `mtd2`

4. **Deploy userdata**
   - Once the gateway reboots, copy `userdata.tar` via `scp` or `ssh` to the device
   - Extract it on `/userdata` to populate writable configs and SSH keys

---

## 👷 Custom Additions

You may use this framework to compile and include additional binaries:

- Write a new `build_foobar` script following the style of existing ones
- Cross-compile using the toolchain in `rsdk-4.4.7...`
- Place binaries under `rootfs/squashfs-root/usr/bin/` or `usr/sbin/`
- Re-run `build_rootfs` to include them in the image

---

## 🧠 Notes

- The root filesystem is designed to be mounted read-only (from `mtd2`)
- `/userdata` (on `mtd4`) is a writable tmpfs or jffs2 partition, mounted at boot
- All binaries are statically linked to reduce dependencies

