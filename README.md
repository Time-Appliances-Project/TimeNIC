# TimeNIC

![2025-04-16T21_46_24 679Z-thumbnail_IMG_2678](https://github.com/user-attachments/assets/13899fbc-b3f6-4bf6-a48b-767c3f78efa5)


The TimeNIC PCIe card is designed to bring precision timing and advanced network functionality to any platform with a PCIe slot. A small form factor PCIe NIC, the TimeNIC integrates a high-performance I226 Ethernet NIC with precise timing inputs and outputs via SMA connectors. It’s a powerful tool for building small form-factor PTP (Precision Time Protocol) clients or even grandmasters, ideal for time-sensitive networking applications.

## Order

Order from Tindie: https://www.tindie.com/products/timeappliances/timenic-i226-pcie-nic-with-pps-inout-and-tcxo/

# Intel I225 PPS Input Patch (DKMS-based igc driver replacement)

This guide shows how to install a patched version of the `igc` driver (used by Intel I225/I226 NICs) to support PPS input. It uses DKMS for rebuilds and ensures the custom driver loads at boot on Ubuntu 24.04.

---

## 🧰 Prerequisites

Install dependencies:

```bash
sudo apt install openssh-server net-tools gcc vim dkms linuxptp linux-headers-$(uname -r)
cd ~ ; mkdir testptp; cd testptp
wget https://raw.githubusercontent.com/torvalds/linux/refs/heads/master/tools/testing/selftests/ptp/testptp.c
wget https://raw.githubusercontent.com/torvalds/linux/refs/heads/master/include/uapi/linux/ptp_clock.h
sudo cp ptp_clock.h /usr/include/linux/ptp_clock.h
gcc -Wall -lrt testptp.c -o testptp
sudo cp testptp /usr/bin/
```

---

## 📦 Install the Patched Driver

1. **Copy the zip to your machine and unzip it:**

```bash
wget https://github.com/Time-Appliances-Project/TimeNIC/raw/refs/heads/main/intel-igc-ppsfix_ubuntu.zip
unzip intel-igc-ppsfix_ubuntu.zip
cd intel-igc-ppsfix
```

2. **Build and install the patched `igc` module using DKMS:**

> Replace `5.4.0-7642.46` with the correct version if different.

```bash
sudo dkms remove igc -v 5.4.0-7642.46
sudo dkms add .
sudo dkms build --force igc -v 5.4.0-7642.46
sudo dkms install --force igc -v 5.4.0-7642.46
```

3. **Backup the current in-kernel `igc` module:**

```bash
sudo cp /lib/modules/$(uname -r)/kernel/drivers/net/ethernet/intel/igc/igc.ko.zst \
        /lib/modules/$(uname -r)/kernel/drivers/net/ethernet/intel/igc/igc.ko.zst.bak
```

4. **Override it with the DKMS-built module:**

```bash
sudo cp /lib/modules/$(uname -r)/updates/dkms/igc.ko.zst \
        /lib/modules/$(uname -r)/kernel/drivers/net/ethernet/intel/igc/igc.ko.zst
```

---

## 🔄 Update Kernel Module Cache

```bash
sudo depmod -a
sudo update-initramfs -u
```

---

## 🔁 Reboot

```bash
sudo reboot
```

---

## ✅ Confirm it's Working

After reboot, confirm the driver in use is your custom one:

```bash
modinfo igc | grep filename
ethtool -i <your-interface>  # e.g., enp1s0
```

Can test with testptp , use loopback cable between the two SMAs
```bash

sudo testptp -d /dev/ptp0 -L0,2
sudo testptp -d /dev/ptp0 -p 1000000000
sudo testptp -d /dev/ptp0 -L1,1

sudo testptp -d /dev/ptp0 -e 5
```

---

## 📝 Notes

- This method **overrides the kernel’s default igc module** at boot.
- To persist across kernel updates, repeat steps 2–4 and run `update-initramfs -u` after each kernel change.

---

## License

This project is licensed under the Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0).

You are free to:

Share — copy and redistribute the material in any medium or format
Adapt — remix, transform, and build upon the material
Under the following terms:

Attribution — You must give appropriate credit, provide a link to the license, and indicate if changes were made.
NonCommercial — You may not use the material for commercial purposes.
For full details, see: https://creativecommons.org/licenses/by-nc/4.0/

As the project creator, I reserve the right to use this material commercially or under any other terms.
