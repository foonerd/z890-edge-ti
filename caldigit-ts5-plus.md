# CalDigit TS5 Plus Thunderbolt 5 Dock - Linux Setup

Configuration guide for CalDigit TS5 Plus with MSI MPG Z890I EDGE TI WIFI on Ubuntu 24.04.

## Hardware Overview

- **Dock**: CalDigit TS5 Plus (Thunderbolt 5, 140W)
- **Motherboard**: MSI MPG Z890I EDGE TI WIFI
- **CPU**: Intel Core Ultra 9 285K (Arrow Lake)
- **Thunderbolt**: Integrated Thunderbolt 4 (40 Gb/s) from CPU

## Default State Issues

Out of the box with default BIOS settings, the following problems occur:

| Component | Status | Issue |
|-----------|--------|-------|
| Thunderbolt connectivity | Working | 40 Gb/s link established |
| Rear USB-C ports | Working | USB tunneling via Thunderbolt |
| Front USB-C port | Working | USB tunneling via Thunderbolt |
| DisplayPort outputs | Working | DP tunneling via Thunderbolt |
| 10GbE Ethernet | Working | r8152 driver |
| Audio (3.5mm jacks) | Working | USB audio class |
| SD/microSD readers | Not Working | Requires firmware update |
| Front USB-A port | Not Working | Requires BIOS change for PCIe tunneling |

## Firmware Update

The SD/microSD card readers require a firmware update to function on Linux.

### Prerequisites

- Windows machine or VM with Thunderbolt
- CalDigit Docking Station Updater from [CalDigit Support](https://www.caldigit.com/support/)

### Procedure

1. Download CalDigit Docking Station Updater for Windows
2. Connect TS5 Plus to Windows machine via Thunderbolt
3. Run updater - it will detect the dock and available firmware
4. Apply all available updates (dock firmware + card reader firmware)
5. Wait for completion - dock will reboot automatically

### Verification (Linux)

After firmware update, insert an SD or microSD card and check dmesg:

```
dmesg | tail -20
```

Expected output:
```
usb-storage 2-1.4.3:1.0: USB Mass Storage device detected
scsi 4:0:0:0: Direct-Access     CalDigit TS5+ Card Reader 0002 PQ: 0 ANSI: 6
scsi 4:0:0:1: Direct-Access     CalDigit TS5+ Card Reader 0002 PQ: 0 ANSI: 6
```

Both SD and microSD slots appear as separate SCSI devices (sda, sdb).

## BIOS Configuration

The front USB-A port on the TS5 Plus uses an ASMedia ASM2142 USB 3.1 controller connected via PCIe tunneling over Thunderbolt. By default, PCIe tunneling fails to initialize.

### Symptom

```
lspci -s 06:00.0 -vvv
```

Shows:
```
!!! Unknown header type 7f
```

This indicates PCIe configuration space cannot be read - the tunnel is not established.

### Required BIOS Changes

Enter BIOS (Delete key at POST) and navigate to:

**Advanced -> Thunderbolt(TM) Configuration**

| Setting | Default | Required |
|---------|---------|----------|
| USB4 CM Mode | Software/Auto | **Firmware** |

This is the critical setting. The others below were already correct by default but are listed for reference.

### Full Thunderbolt Configuration Reference

**Advanced -> Thunderbolt(TM) Configuration**

| Setting | Value |
|---------|-------|
| PCIE Tunneling over USB4 | Enabled |
| USB4 CM Mode | **Firmware** |
| Integrated Thunderbolt(TM) Support | Enabled |
| Discrete Thunderbolt(TM) Support | Disabled |

**Advanced -> Thunderbolt(TM) Configuration -> Integrated Thunderbolt(TM) Configuration**

| Setting | Value |
|---------|-------|
| OS Native Resource Balance | Disabled |
| ITBT RTD3 | Disabled |
| ITBT Root Port 0 | Enabled |
| ITBT Root Port 1 | Enabled |
| Extra Bus Reserved | 56 |
| Reserved Memory | 512 |
| Memory Alignment | 25 |
| Reserved PMemory | 32768 |
| PMemory Alignment | 28 |

### Why USB4 CM Mode Matters

CM = Connection Manager. Controls who creates Thunderbolt/USB4 tunnels:

- **Firmware**: Intel Thunderbolt firmware creates tunnels at boot before OS loads. PCIe devices appear automatically.
- **Software**: OS kernel driver creates tunnels. Linux's software CM defaults to "user" security level, which blocks PCIe tunneling until manual authorization.

For Linux with PCIe tunneled devices (like the TS5 Plus front USB-A port), Firmware mode is required.

## Post-Configuration Verification

After saving BIOS changes, perform a full power cycle (shutdown, wait 10 seconds, power on).

### Check Thunderbolt Security Level

```
cat /sys/bus/thunderbolt/devices/domain0/security
```

Should show `none` or `user`. Must NOT show `nopcie`.

### Check ASMedia Controller

```
lspci -s 06:00.0 -vvv
```

Should show:
```
06:00.0 USB controller: ASMedia Technology Inc. ASM2142/ASM3142 USB 3.1 Host Controller
    Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- ...
    Kernel driver in use: xhci_hcd
```

Key indicators:
- `Mem+` and `BusMaster+` (not `Mem-` and `BusMaster-`)
- `Kernel driver in use: xhci_hcd`
- No "Unknown header type 7f" error

### Test Front USB-A Port

Plug a USB device into the front USB-A port:

```
dmesg -w
```

Should show device enumeration on the ASMedia controller (typically Bus 004).

### Full USB Tree

```
lsusb -t
```

The ASMedia controller appears as a separate bus with its own hub hierarchy.

## Working Configuration Summary

With firmware updated and BIOS configured:

| Component | Status | Bus/Driver |
|-----------|--------|------------|
| Rear USB-C (x2) | Working | Thunderbolt USB tunneling |
| Front USB-C | Working | Thunderbolt USB tunneling |
| Front USB-A | Working | ASMedia xhci_hcd (PCIe tunneling) |
| DisplayPort (x3) | Working | Thunderbolt DP tunneling |
| SD Card Reader | Working | usb-storage |
| microSD Card Reader | Working | usb-storage |
| 10GbE Ethernet | Working | r8152 |
| 3.5mm Audio In/Out | Working | snd-usb-audio |

## Troubleshooting

### PCIe Tunneling Still Fails

If ASMedia controller still shows errors after BIOS changes:

1. Verify USB4 CM Mode is set to Firmware (not Auto or Software)
2. Full power cycle - not just reboot
3. Try different Thunderbolt port on motherboard
4. Check `dmesg | grep -i thunder` for errors

### Card Readers Not Detected

1. Verify firmware was updated (requires Windows)
2. Insert a card and check `dmesg | tail -20` for "CalDigit TS5+ Card Reader"
3. Ensure cards are inserted correctly (readers are spring-loaded eject)

### Intermittent Disconnects

If devices disconnect randomly:
1. Verify ITBT RTD3 is Disabled
2. Check cable quality - use Thunderbolt certified cable
3. Ensure adequate power to dock (140W adapter)

## References

- [CalDigit TS5 Plus Specifications](https://www.caldigit.com/thunderbolt-5-dock/)
- [Linux Kernel Thunderbolt Documentation](https://docs.kernel.org/admin-guide/thunderbolt.html)
- [Intel 800 Series BIOS User Guide](https://www.msi.com)
