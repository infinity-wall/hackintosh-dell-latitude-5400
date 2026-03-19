# 🍎 Hackintosh — Dell Latitude 5400

> macOS Monterey 12.7.6 running on Dell hardware with OpenCore.  
> Practical, straight-to-the-point guide based on real experience.

## 📦 What's in here

This repository contains a **complete, working EFI** for the Dell Latitude 5400 with the specs listed below.

This is a **copy-paste guide** — just copy the EFI folder to your USB drive and follow the steps. No need to build from scratch or configure kexts manually. It worked here, it should work for you too.

> ⚠️ **Using a different Latitude model?**  
> Don't just copy without checking! Other models may have different hardware — Wi-Fi chipset, touchpad, audio, etc. In that case, **review the `config.plist`** (especially SMBIOS and DeviceProperties) and **make sure the kexts match your hardware** before using.

---

## 💻 Hardware Specs

| Component | Details |
|---|---|
| **Model** | Dell Latitude 5400 |
| **CPU** | Intel Core i5-8265U (Whiskey Lake) |
| **GPU** | Intel UHD Graphics 620 |
| **Wi-Fi** | Intel Wireless-AC 9560 |
| **Bluetooth** | Intel Bluetooth |
| **Storage** | SATA/NVMe SSD |
| **Touchpad** | Alps HID |

---

## ✅ Compatibility Status

| Feature | Status |
|---|---|
| Boot | ✅ Working |
| CPU (Power Management) | ✅ Working |
| GPU (acceleration) | ✅ Working |
| Audio | ✅ Working |
| Wi-Fi | ✅ Working (via HeliPort) |
| Bluetooth | ✅ Working |
| Trackpad | ✅ Working |
| Keyboard | ✅ Working |
| Screen Brightness | ✅ Working |
| SD Card Reader | ✅ Working |
| USB | ✅ Working |
| Sleep/Wake | ⚠️ Partial |
| iMessage / FaceTime | ⚠️ Requires SMBIOS configuration |

---

## 📋 Requirements

- USB drive of **16GB or more**
- A computer with **Windows or Linux** to prepare the USB
- Internet connection during installation
- **OpenCore** (version used: 1.0.x)

---

## 🚀 Installation Guide

### 1. Prepare the USB Drive

Format the USB drive with the following settings:

- **Partition scheme:** GPT
- **Format:** FAT32
- **Name:** anything (e.g. `OPENCORE`)

> ⚠️ The USB drive **must** be formatted as **UEFI + GPT**. MBR or Legacy partitions will not work.

---

### 2. USB Drive Structure

After formatting, the USB drive must contain **two folders** at the root:

```
USB/
├── EFI/                          ← available in this repo, just copy it
│   ├── BOOT/
│   │   └── BOOTx64.efi
│   └── OC/
│       ├── config.plist
│       ├── OpenCore.efi
│       ├── ACPI/
│       ├── Drivers/
│       ├── Kexts/
│       ├── Resources/
│       └── Tools/
└── com.apple.recovery.boot/      ← generated with the command below
    ├── BaseSystem.dmg
    └── BaseSystem.chunklist
```

> 📁 **The EFI folder** is ready in this repository — just download and copy it to the USB drive.

> 📥 **The `com.apple.recovery.boot` folder** is not included here because the main file is over 600MB, above GitHub's limit. You need to generate it with the command below — it will be created automatically:

```bash
# Download OpenCore at: https://github.com/acidanthera/OpenCorePkg/releases
# Extract it and navigate to: Utilities/macrecovery/
# Open a terminal in that folder and run:

python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download
```

After the download, copy the generated `com.apple.recovery.boot` folder to the root of the USB drive.

---

### 3. Kexts Used

| Kext | Purpose |
|---|---|
| `Lilu.kext` | Required base |
| `VirtualSMC.kext` | SMC emulation |
| `WhateverGreen.kext` | GPU fixes |
| `AppleALC.kext` | Audio |
| `IntelMausi.kext` | Intel Ethernet |
| `RestrictEvents.kext` | Misc fixes |
| `AirportItlwm.kext` | Intel Wi-Fi (alternative to itlwm) |
| `itlwm.kext` | Intel Wi-Fi (recommended — more stable) |
| `IntelBluetoothFirmware.kext` | Intel Bluetooth |
| `BlueToolFixup.kext` | Bluetooth fix |
| `VoodooPS2Controller.kext` | Keyboard and trackpad |
| `VoodooI2C.kext` + `VoodooI2CHID.kext` | Advanced touchpad |
| `SMCBatteryManager.kext` | Battery status |
| `SMCDellSensors.kext` | Dell sensors |
| `NVMeFix.kext` | NVMe fix |
| `USBMap.kext` | Custom USB mapping |

---

### 4. BIOS Settings

Access the BIOS on the Latitude 5400 by pressing **F2** at startup.

**Disable:**
- Secure Boot
- Fast Boot
- VT-d

**Enable:**
- UEFI Boot
- XHCI Hand-off
- SATA Mode: **AHCI**

> ⚠️ If Windows was installed with SATA in **RAID/RST** mode, you need to migrate to AHCI via Windows Registry before changing the BIOS, otherwise Windows will not boot.

---

### 5. Booting from USB

1. Restart the notebook
2. Press **F12** to open the boot menu
3. Select the USB drive in **UEFI** mode
4. In the OpenCore menu, select the macOS Recovery
5. Follow the installer normally

---

### 6. Installing to SSD

During installation, format the SSD using **Disk Utility**:
- **Format:** APFS
- **Scheme:** GUID

After installation, copy the EFI from the USB to the SSD via Terminal:

```bash
# List disks
diskutil list

# Mount SSD EFI (replace disk0s1 with the correct identifier)
sudo diskutil mount disk0s1

# Mount USB EFI
sudo diskutil mount disk2s1

# Copy EFI
sudo cp -r /Volumes/OPENCORE/EFI /Volumes/EFI/
```

After this, the system will boot **without the USB drive**.

---

## 📶 Wi-Fi — Important Note

> The `AirportItlwm.kext` may cause **Kernel Panic** when trying to connect to some networks.  
> The most stable solution is to use **`itlwm.kext`** together with the **HeliPort** app.

### Installing HeliPort

> 📁 **HeliPort is already included in this repository** — just install it directly.  
> If you need a different version for your setup, download it at: [github.com/OpenIntelWireless/HeliPort/releases](https://github.com/OpenIntelWireless/HeliPort/releases)

1. Open the `HeliPort.dmg` included in this repo
2. Install it normally
3. If a security warning appears: right-click the app → **Open**
4. To launch HeliPort at login:

```bash
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/HeliPort.app", hidden:false}'
```

---

## 🛠️ Tools Used

| Tool | Link |
|---|---|
| OpenCore | [github.com/acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases) |
| ProperTree | [github.com/corpnewt/ProperTree](https://github.com/corpnewt/ProperTree) |
| GenSMBIOS | [github.com/corpnewt/GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) |
| MountEFI | [github.com/corpnewt/MountEFI](https://github.com/corpnewt/MountEFI) |
| HeliPort | [github.com/OpenIntelWireless/HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases) |

---

## 📚 References

- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [OpenIntelWireless — itlwm](https://github.com/OpenIntelWireless/itlwm)
- [Acidanthera Kexts](https://github.com/acidanthera)

---

## ⚠️ Disclaimer

This guide is provided for educational purposes only.  
Installing macOS on non-Apple hardware violates Apple's EULA.  
Use at your own risk.

---

> Made with ☕ and a lot of patience.  
> If this helped you, drop a ⭐ on the repo!
