# RAID1 Performance Tuning for NVMe Drives

This repository contains configuration and tuning scripts for optimizing RAID1 performance on systems with NVMe drives. The focus is on maximizing the efficiency of RAID1 arrays using NVMe devices, particularly when configured for intensive workloads like GCC builds, virtualization, Docker, and other high-performance applications.

## Hardware Setup

The repository is designed to work with the following hardware configuration:

- **Motherboard**: MSI MPG Z890I EDGE TI WIFI
- **CPU**: Intel i9 Ultra 285K (24 cores)
- **RAM**: 96GB DDR5-6600
- **Storage Configuration**:
  - **RAID1 Array**:  
    - `nvme1n1`: Samsung Electronics NVMe SSD Controller S4LV008 [Pascal]  
    - `nvme2n1`: Samsung Electronics NVMe SSD Controller S4LV008 [Pascal]  
  - **Root Storage**:  
    - `nvme0n1`: Micron/Crucial Technology T700 NVMe SSD
- **Operating System**: Ubuntu 24.04.01 with kernel 6.8.0-52-generic

## Features and Optimizations

This repository provides a series of optimizations and settings for the RAID1 configuration, including:

- **I/O Scheduler Optimizations**: Tweaks for `mq-deadline` and other performance improvements.
- **Queue Depth Tuning**: Configuration of the `nr_requests` parameter for better queue depth management.
- **Volatile Write Cache Management**: Enabling write-back caching on NVMe devices for improved write performance.
- **Active State Power Management (ASPM)**: Power management settings to enhance device responsiveness and reduce latency.
- **Sysctl and GRUB Configurations**: Kernel-level tweaks and tuning parameters to optimize system performance.
- **File System Tuning**: Adjustments for ext4 file systems in RAID1 arrays for better throughput and I/O operations.

## Setup Instructions

### 1. Install Required Tools
Ensure that the necessary tools for configuring and optimizing NVMe drives and RAID1 arrays are installed on your system:

```bash
sudo apt update
sudo apt install nvme-cli smartmontools sysstat
```

### 2. Apply RAID1 and NVMe Optimizations

To optimize your system's performance with RAID1 and NVMe devices, you will need to configure the following:

- **Tuning Parameters**: The relevant parameters are already defined in `/etc/sysctl.d/99-sysctl.conf`.
- **RAID1 Optimization**: Apply RAID-specific optimizations using sysctl parameters (e.g., `dev.raid.speed_limit_max`).
- **NVMe Device Optimization**: Follow the guidelines in `/etc/udev/rules.d/99-nvme-optimization.rules` to adjust device-specific settings like write-back cache and read-ahead buffers.

### 3. Modify GRUB Configuration

The following kernel parameters are essential for optimal system performance with RAID1 and NVMe drives. Ensure that your GRUB configuration file includes these settings:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_pstate=performance pcie_aspm=force"
```

Then update GRUB:

```bash
sudo update-grub
```

### 4. Enable/Modify udev Rules

Ensure that udev rules are properly set up to automatically adjust NVMe and RAID1 settings at boot:

```bash
sudo cp 99-nvme-optimization.rules /etc/udev/rules.d/
```

After modifying or adding new udev rules, reload the udev rules:

```bash
sudo udevadm control --reload-rules
```

### 5. Verify Changes

Once all optimizations are applied, it's important to verify the changes:

- **Check NVMe Settings**:  
  Verify that the NVMe settings are correctly applied:
  ```bash
  sudo nvme get-feature -f 0x06 /dev/nvme1
  sudo nvme get-feature -f 0x06 /dev/nvme2
  ```

- **Check RAID1 Settings**:  
  Make sure that the RAID1 settings are as expected:
  ```bash
  sudo mdadm --detail /dev/md0
  ```

## Performance Testing

Use tools like `fio` to test the impact of the optimizations:

1. **Sequential Read/Write Test**:
   ```bash
   sudo fio --name=seq_read --directory=/volume1 --filename=fio_testfile --rw=read --bs=1M --size=4G --numjobs=4 --iodepth=32 --runtime=60 --group_reporting
   ```

2. **Random Read/Write Test**:
   ```bash
   sudo fio --name=rand_write --directory=/volume1 --filename=fio_testfile --rw=randwrite --bs=4K --size=4G --numjobs=4 --iodepth=32 --runtime=60 --group_reporting
   ```

## Conclusion

This repository provides a comprehensive set of optimizations and configurations aimed at improving the performance of RAID1 arrays with NVMe devices. By carefully adjusting kernel parameters, NVMe settings, and utilizing tools like `fio`, you can achieve significant improvements in throughput and I/O performance for high-demand workloads.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


### Customization Notes:

1. **Hardware Configuration**: Ensure that the specifics of your hardware setup (such as motherboard model, CPU, and storage) are correctly reflected.
2. **Performance Testing**: Depending on your workload, the `fio` parameters in the "Performance Testing" section may need to be adjusted to better reflect your use case.
3. **Other Configuration Files**: If additional configuration files or scripts are required (e.g., for RAID management), provide further details in the repository.
