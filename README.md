# Asus TUF Z390-PRO GAMING OpenCore Hackintosh

OpenCore EFI for **macOS 26 Tahoe**, the last macOS release for Intel Macs.

[فارسی](README.fa.md)

| Part | Model |
|---|---|
| Motherboard | Asus TUF Z390-PRO GAMING |
| CPU | Intel Core i9-9900K (UHD 630 used for compute only) |
| GPU | ASUS ROG STRIX RX 5700 XT O8G GAMING |
| Ethernet | Intel I219-V |
| Wi-Fi | TP-Link Archer T9E (Broadcom BCM4360) |
| Bluetooth | Asus BT400 (Broadcom BCM20702) |

| | |
|---|---|
| OpenCore | 1.0.8 |
| SMBIOS | iMac20,1 |
| Target | macOS 26 Tahoe |

> **Status:** this EFI was rebuilt from the old OpenCore 0.5.7 one and passes OpenCore's own `ocvalidate`, but it
> has **not been booted on the hardware yet**. Keep a USB stick with a working EFI before you replace the one on
> your disk.

The old Catalina-era EFI (OpenCore 0.5.7, iMac19,1) is kept as the tag
[`opencore-0.5.7`](../../tree/opencore-0.5.7).

## What works on Tahoe and what does not

| Area | State |
|---|---|
| RX 5700 XT | Native AMD driver (macOS 26 still supports Navi 10) |
| UHD 630 | Headless, for Quick Sync video encode and decode |
| Ethernet | IntelMausi |
| Bluetooth | BrcmPatchRAM3 + BrcmFirmwareData + BlueToolFixup |
| USB | USBMap.kext, made from the old SSDT-UIAC port map (same 15 ports) |
| HDMI / DisplayPort audio, USB audio | Works |
| **Analog audio (rear jacks)** | **Does not work.** macOS 26 removed AppleHDA, so AppleALC has nothing to patch. Use HDMI/DP audio, a USB sound card, or VoodooHDA |
| **Wi-Fi (Archer T9E)** | **Does not work out of the box.** macOS 14 and later dropped the BCM4360 driver. It only comes back with OpenCore Legacy Patcher's root patches, which need SIP lowered |

## Before the first boot

1. **SMBIOS.** The config has placeholder values. Generate your own for `iMac20,1` with
   [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and put them in `PlatformInfo > Generic`:
   `SystemSerialNumber`, `MLB`, `SystemUUID`, and `ROM` (your Ethernet MAC address without colons).
   Check the serial on Apple's coverage page: it should be *not valid*.
2. **Copy** the `EFI` folder to the EFI partition (first to a USB stick to test).

## BIOS settings

| Setting | Value |
|---|---|
| Fast Boot, Secure Boot, CSM | Disabled |
| VT-d | Disabled |
| CFG Lock | Disabled (`AppleXcpmCfgLock` is off, so this is required) |
| Above 4G Decoding | Enabled |
| XHCI Hand-off | Enabled |
| Primary Display | PEG (the RX 5700 XT) |
| iGPU Multi-Monitor | Enabled |
| DVMT Pre-Allocated | 64M |
| OS Type | Other OS |

## Notes on the config

- **WhateverGreen is not used.** Dortania's Tahoe notes say it has AMD connector problems on macOS 26. The
  `agdpmod=pikera` fix that Navi cards need is a `Kernel > Patch` entry instead: it renames `board-id` to
  `board-ix` in AppleGraphicsDevicePolicy.
- **Booter quirks** follow Dortania's Coffee Lake guide for Z390: `DevirtualiseMmio`, `ProtectUefiServices`,
  `RebuildAppleMemoryMap` and `SyncRuntimePermissions` on, `EnableWriteUnprotector` off. If the boot stops right
  after OpenCore (an old BIOS without a memory attributes table), turn `EnableWriteUnprotector` on and
  `RebuildAppleMemoryMap` and `SyncRuntimePermissions` off. Updating the BIOS is the better fix.
- **Software updates:** `SecureBootModel` is `Disabled` and RestrictEvents runs with `revpatch=sbvmm`, as Dortania
  recommends for updates on macOS 14.4 and later.
- **BIOS reset on reboot:** the old RTC checksum kernel patch is now OpenCore's `DisableRtcChecksum` quirk.
- **Bluetooth on macOS 12+:** `bluetoothExternalDongleFailed` and `bluetoothInternalControllerInfo` are set in
  NVRAM, as the BrcmPatchRAM README asks.
- The picker (OpenCanopy) is shown for 5 seconds. Set `Misc > Boot > ShowPicker` to `false` to hide it; hold
  Option at boot to show it.

## Versions

| Component | Version |
|---|---|
| OpenCore | 1.0.8 |
| OcBinaryData (HfsPlus.efi, picker resources) | master, 2026-11-14 |
| Lilu | 1.7.2 |
| VirtualSMC, SMCProcessor, SMCSuperIO | 1.3.8 |
| IntelMausi | 1.0.8 |
| BrcmPatchRAM3, BrcmFirmwareData, BlueToolFixup | 2.7.2 |
| RestrictEvents | 1.1.6 |

## Changes from the 0.5.7 EFI

- OpenCore 0.5.7 → 1.0.8, config rebuilt from the 1.0.8 `Sample.plist`.
- SMBIOS iMac19,1 → iMac20,1 (macOS 26 does not support iMac19,1).
- The folder is now `EFI/` with `BOOT` and `OC` inside, ready to copy.
- `USBInjectAll` + `SSDT-UIAC` → `USBMap.kext` with the same ports.
- `BrcmBluetoothInjector` → `BlueToolFixup`.
- Removed: WhateverGreen, AppleALC (no AppleHDA on 26), ApfsDriverLoader, VBoxHfs, AppleUsbKbDxe, AudioDxe and
  the unused NVMe/XHCI/HII drivers.
- Added: RestrictEvents, ResetNvramEntry, OpenShell.
- UHD 630: display framebuffer `0x3E980003` → headless `0x3E910003`.

## Credits

[Acidanthera](https://github.com/acidanthera) (OpenCore and the kexts), the
[Dortania guide](https://dortania.github.io/OpenCore-Install-Guide/), and
[corpnewt](https://github.com/corpnewt) (GenSMBIOS).
