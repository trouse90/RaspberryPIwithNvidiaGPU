NVIDIA RTX 5070 on Raspberry Pi 5 (Blackwell Support)
A complete guide to running a desktop-class NVIDIA GeForce RTX 5070 (Blackwell architecture) on the Raspberry Pi 5. This project bridges the gap between the Pi's ARM64 architecture and modern NVIDIA gaming GPUs, solving critical memory coherency and page size limitations.
End Result: A fully recognized GPU in nvidia-smi capable of running CUDA and AI workloads.


⚠️ The Critical Unlock (Read This First)
Most NVIDIA GPU guides for the Pi fail with modern cards because of Page Size Mismatch.
The Problem: The Raspberry Pi 5 defaults to a 16K kernel page size. The NVIDIA proprietary driver is hard-coded for 4K pages. This causes the GPU initialization (RmInitAdapter) to crash, often masquerading as a "Missing Firmware" error.
The Fix: You must force the Pi to use the 4K kernel (kernel8.img) in config.txt.

🛠️ Hardware Bill of Materials
Component Recommendation Notes
Compute Raspberry Pi 5 (8GB) 8GB strongly recommended for AI/LLM models.
Bridge Pineboards HatDrive! Bottom Converts Pi FFC ribbon to standard M.2 Key-M.
Dock OCuLink to M.2 Adapter JMT brand or similar. OCuLink offers better signal integrity than USB risers.
Cable OCuLink Cable (SFF-8611) Keep it short (<50cm) for stability.
PSU 750W+ ATX Power Supply Must be fully modular or have a jumper bridge to turn on without a motherboard.
GPU NVIDIA GeForce RTX 5070 PNY Model tested. Requires 12VHPWR adapter.

🚀 Installation Guide
Phase 1: Hardware Assembly & BIOS Setup

1. Connection: Connect the PCIe FFC cable to the Pi 5.
CRITICAL: The silver metal teeth on the ribbon cable must face INWARDS (towards the heatsink/USB ports).

2. Power: Connect the GPU to the External PSU. Turn the PSU on first, then boot the Pi.

3. Boot Config: Edit /boot/firmware/config.txt to enable PCIe and force the correct kernel.


# /boot/firmware/config.txt

# 1. Enable External PCIe
dtparam=pciex1

# 2. Force Gen 2 Speed (Gen 3 is unstable for initial setup)
dtparam=pciex1_gen=2

# 3. Prevent USB Current Limiting (if not using official PSU)
usb_max_current_enable=1

# 4. THE MAGIC SWITCH: Force 4K Page Size Kernel
kernel=kernel8.img



4. Reboot: sudo reboot

5. Verify: Run getconf PAGE_SIZE. Output must be 4096.

Phase 2: System Preparation
Install the build dependencies required to compile the driver interface:


sudo apt update
sudo apt install -y git bc bison flex libssl-dev make libc6-dev libncurses5-dev raspberrypi-kernel-headers build-essential


Blacklist Nouveau (Default Open Source Driver):
Create /etc/modprobe.d/blacklist-nouveau.conf:

blacklist nouveau
options nouveau modeset=0


Run sudo update-initramfs -u and reboot.
Phase 3: The "Split" Driver Installation
We cannot use the standard .run installer alone because it will fail to compile the kernel modules for the Pi. We must split the installation.

Step A: Install Userspace Tools (nvidia-smi, libraries)

1. Download the Linux aarch64 driver (e.g., version 580.xx.xx) from NVIDIA.

2. Run the installer with the "no kernel module" flag:

sudo ./NVIDIA-Linux-aarch64-580.xx.xx.run --no-kernel-module


