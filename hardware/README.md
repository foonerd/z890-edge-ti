# MSI MPG Z890I EDGE TI WIFI - rtl8126 and nct6687d Modules

This repository contains the necessary information to build and install the `rtl8126` network driver and `nct6687d` sensor driver modules for the MSI Z890 system.

## Modules Overview

### rtl8126 (Network Driver)
- **Repository**: [Realtek rtl8126 DKMS](https://github.com/awesometic/realtek-r8126-dkms)
- The `rtl8126` driver is designed for the Realtek RTL8126/RTL8125 network chip.

### nct6687d (Sensor Driver)
- **Repository**: [NCT6687D Driver](https://github.com/Fred78290/nct6687d)
- The `nct6687d` driver supports the Nuvoton NCT6687D hardware monitoring sensor.

## Module Build Steps

### Prerequisites:
Make sure you have the necessary build tools installed:
```bash
sudo apt update
sudo apt install dkms build-essential linux-headers-$(uname -r)
```

### rtl8126 (Network Driver) Installation:
1. Clone the repository:
    ```bash
    git clone https://github.com/awesometic/realtek-r8126-dkms.git
    cd realtek-r8126-dkms
    ```
2. Build and install the module:
    ```bash
    sudo dkms build .
    sudo dkms install .
    ```

### nct6687d (Sensor Driver) Installation:
1. Clone the repository:
    ```bash
    git clone https://github.com/Fred78290/nct6687d.git
    cd nct6687d
    ```
2. Build and install the module:
    ```bash
    make
    sudo make install
    ```

## Downloadable .deb Files

Precompiled `.deb` files for both modules are available for download in the [Releases](https://github.com/foonerd/z890/hardware/releases) directory.

- **rtl8126 DKMS package**: [Download here](https://github.com/foonerd/z890/raw/refs/heads/main/hardware/releases/realtek-r8126-dkms_10.014.01-1_amd64.deb)
- **nct6687d Sensor package**: [Download here](https://github.com/foonerd/z890/raw/refs/heads/main/hardware/releases/nct6687d-dkms_20241105-084624_all.deb)
