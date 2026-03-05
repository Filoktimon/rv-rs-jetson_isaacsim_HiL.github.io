---
layout: default
title: "Part 1: The Foundation – Preparing the Jetson Nano"
---

# Part 1: The Foundation – Preparing the Jetson Nano

This first post focuses on the essential hardware and OS setup required to build a robust Hardware-in-the-Loop (HiL) testbench.

## The Hardware Leap: NVMe SSD & SDK Manager
While the Jetson comes with a microSD slot, it is a significant bottleneck for robotics development. 

* **The Setup:** Use the **NVIDIA SDK Manager** to flash your Jetson Orin Nano.
* **The SSD Advantage:** A **1TB NVMe SSD** is highly recommended. It provides the necessary storage for large Docker images (often 10GB+) and ensures the I/O speed required for high-bandwidth sensor data from Isaac Sim.
* **Boot Drive:** Confirm the SSD is the boot drive by running `lsblk`. Your root directory (`/`) should be mounted on the main NVMe partition (e.g., `nvme0n1p1`).

## Decoding the Partition "Mystery"
The Jetson Orin uses a complex partition structure because it lacks a traditional BIOS. 

* **Main OS (`nvme0n1p1`):** This is where Ubuntu and your 1TB of space reside.
* **EFI (`nvme0n1p10`):** This contains the bootloader.
* **Redundancy (`p2–p13`):** These are internal NVIDIA partitions for firmware and "A/B Slots" for redundant booting—**do not touch these**.

## Memory Optimization: The "OOM" Lifesaver
The Jetson Orin Nano has 8GB of physical RAM shared between the CPU and GPU. Running a ROS2 stack plus simulation data will quickly lead to **Out-of-Memory (OOM)** errors unless optimized.

* **Disable ZRAM:** Turn off the default compressed RAM drives to free up resources.
* **Physical Swap:** Create an **8GB Swap file** on your fast NVMe SSD. This acts as "emergency RAM," allowing the system to offload background tasks and keep the physical 8GB free for high-performance GPU tasks.

---
### Implementation: Storage and Memory Setup

To prepare your Jetson, run the following commands.

**1. Disable NVIDIA's default ZRAM and reboot:**
```bash
sudo systemctl disable nvzramconfig
sudo reboot

**2. Create a physical 8GB Swap file on the NVMe SSD:**
```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

**3. Make the swap permanent by adding it to /etc/fstab:**
```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab


