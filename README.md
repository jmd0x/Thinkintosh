# Thinkintosh
This is a write up on how to hackintosh the Lenovo Thinkpad T440s

[![macOS](https://img.shields.io/badge/macOS-Big_Sur_11.2.3-red)](https://www.apple.com/macos/big-sur/)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.6.7-blue)](https://github.com/acidanthera/OpenCorePkg)

Lenovo Thinkpad T440S using OpenCore Bootloader

### Specs:
- Lenovo ThinkPad T440s
- CPU: i5-4300U
- GPU: Intel HD Graphics 4400
- Intel Dual Band Wireless-AC 7260 Model: 7260NGW
- RAM: 12GB 1600MHz DDR3
- First SSD: 512GB
- Secondary SSD: 16GB


### 2. BIOS settings

| Item | Setting |
| ------------- | ------------ |
| Security Chip | Disabled |
| Memory Protection Execution Prevention | Enabled |
| Virtualization | Enabled |
| Fingerprint Reader | Disabled |
| Secure Boot | Disabled |
| UEFI/Legacy Boot | UEFI Only |
| CSM Support | YES |
| Boot Mode | Quick |

### Creating a bootable usb

Download [Rufus](https://rufus.ie/en/), set the BOOT selection as not bootable, set File System as Large FAT32, click Start, and delete all file autorun in USB Drive partition.

Next, go to the root of this USB drive and create a folder called com.apple.recovery.boot. Then move the downloaded BaseSystem or RecoveryImage files. Please ensure you copy over both the .dmg and .chunklist files to this folder:

Open up and extract the EFI folder archive you downloaded earlier.

Copy the folder named, "EFI," to the root of your USB Drive.

### MacRecovery

First grab a copy of [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases) and head to /Utilities/macrecovery/. Next copy the folder path for the macrecovery folder.
From here, you'll want to open up a Command Prompt and cd into the macrecovery folder that we copied earlier:

- `cd Paste_Folder_Path`

Now run one of the following depending on what version of macOS you want.
(Note these scripts rely on [Python](https://www.microsoft.com/en-us/p/python-39/9p7qfqmjrfp7?activetab=pivot:overviewtab) support, please install if you haven't already):

`# Mojave (10.14)
python macrecovery.py -b Mac-7BA5B2DFE22DDD8C -m 00000000000KXPG00 download`

`# Catalina (10.15)
python macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download`

`# Big Sur (11)
python macrecovery.py -b Mac-42FD25EABCABB274 -m 00000000000000000 download`

`# Monterey (12)
python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download`

`# Ventura (13) (Work in progress)
python3 macrecovery.py -b Mac-4B682C642B45593E -m 00000000000000000 download`

macOS 12 and above note: As recent macOS versions introduce changes to the USB stack, it is highly advisable that you map your USB ports (with USBToolBox) before installing macOS.This will take some time, however once you're finished you should get either BaseSystem or RecoveryImage files:

### What doesn't work:
- DRM content
- FingerPrint Reader
- Docking Station Kernel Panic if `Sleep, Reboot, Shutdown` attempted while external display connected on one of the Dock Ports
- Docking Station DisplayPort Audio

### Bios
These are the recommended settings to have everything working properly:
- `Security Chip > Security Chip [Disabled]`
- `Anti-Theft > Intel (R) AT Module Activation > Current Setting [Disabled]`
- `Anti-Theft > Computrace > Computrace Module Activation > Current Setting [Disabled]`

**Note**: These laptops do have whitelist which doesn't allow you to use other Card than the Intel AC7260.
In order to use a different / supported card, you need to mod your bios (remove whitelist) or downgrade to Bios v2.36
- Bios v2.36 doesn't have whitelist so downgrading allows you to use any wireless card that you want.

### Secure Boot
Users with `1366x768` or `1600x900` displays can go ahead and enable secure boot and enjoy it.
Users with upgraded displays to `1080p` or native `1080p` displays will have garbled screen if CSM is disabled in BIOS (which can't be left enabled if Secure Boot enabled)
In order to fix this problem we need to patch `Display-EDID`.

### Non TouchScreen Displays
If your Lenovo Thinkpad T440S doesn't have a TouchScreen display, it is required for you to disable the kext responsible for TouchScreen.
Go to `EFI/OC/Config.plist > Kernel > Add >` and disable the 4 following kexts:
- `VoodooI2CServices.kext - Enabled = No`
- `VoodooGPIO.kext - Enabled = No`
- `VoodooI2C.kext - Enabled = No`
- `VoodooI2CHID.kext - Enabled = No`

### TouchPad
Most of the users have probably already upgraded to a T450S Touchpad (the one with Physical Buttons) and this one does work natively, no need to touch anything.
For you users that have the standard Touchpad that came with this laptop, you have to do some changes as VoodooRMI doesn't seem to work very well with them.

Go to `EFI/OC/Config.plist > Kernel > Add` and disable the VoodooRMI kexts:
- `VoodooRMI.kext - Enabled = No`
- `VoodooRMI.kext/Contents/PlugIns/RMISMBus.kext - Enabled = No`
- `VoodooRMI.kext/Contents/PlugIns/VoodooTrackpoint.kext - Enabled = No`
- `VoodooRMI.kext/Contents/PlugIns/VoodooInput.kext - Enabled = No`

Once done, enable the VoodooPS2Controller kexts for Touchpad:

- `VoodooPS2Controller.kext/Contents/PlugIns/VoodooInput.kext - Enabled = Yes`
- `VoodooPS2Controller.kext/Contents/PlugIns/VoodooPS2Trackpad.kext - Enabled = Yes`
- `VoodooPS2Controller.kext/Contents/PlugIns/VoodooPS2Mouse.kext - Enabled = Yes`

Now enable the `SSDT-TPD.aml` for Touchpad to work with VoodooPS2:  
- `EFI/OC/Config.plist > ACPI > Add > SSDT-TPD.aml > Enabled = Yes`

### YogaSMC
To have working Keyboard Function Keys (Fn) and Fan reading etc, you need to install the YogaSMCPane and the YogaSMC App.
YogaSMC.kext is already included in the EFI so when yo go to releases tab, you download the **YogaSMC-App-release.dmg**
- https://github.com/zhen-zen/YogaSMC


### Audio
ALCPlugFIx is required to fix static noise on headphones, however Black-Dragon74 released a Swift version that doesn't require `hda-verb`, `alc-verb` or `CodecCommander` kext. the `ALCPlugFix.zip` is included in the Tools folder.

**Installation**:
- Extract ALCPlugFix zip into desktop
- Open terminal and type following commands one by one on the listed order:
- `sudo spctl --master-disable`
- `sudo mkdir /usr/local/bin/`
- `cd desktop/ALCPlugFix`
- `sudo cp -R ALC3232.plist /usr/local/bin/`
- `./install.sh`
- Now the installer will ask you to drop the `ALC3232.plist` into the terminal window.
- Open a new finder window and press `Shift + Cmd(Alt) + G` to open a new `go to folder:` window
- Now type: `/usr/local/bin/`
- Drag the `ALC3232.plist` from the `/usr/local/bin` folder into the terminal window and press enter.
- Done


### Wireless and Bluetooth

#### Intel AC7260
Users with Intel AC7260 cards can enjoy out of the box support for both Wireless and Bluetooth.
Keep in mind that Airportitlwm/itlwm is still in early development and only `N` speeds are supported.

#### DW1560 & DW1830
Users with one of these two cards first need to disable the intel kexts:

- `EFI/OC/Config.plist > Kernel > Add > Airportitlwm > Enabled = No`
- `EFI/OC/Config.plist > Kernel > Add > IntelBluetoothInjector > Enabled = No`
- `EFI/OC/Config.plist > Kernel > Add > IntelBluetoothFirmware > Enabled = No`

Then enable the corresponding kexts for those two cards:

- `EFI/OC/Config.plist > Kernel > Add > AirportBrcmFixup > Enabled = Yes`
- `EFI/OC/Config.plist > Kernel > Add > AirPortBrcm4360_Injector > Enabled = Yes`
- `EFI/OC/Config.plist > Kernel > Add > BrcmBluetoothInjector > Enabled = Yes`
- `EFI/OC/Config.plist > Kernel > Add > BrcmFirmwareData > Enabled = Yes`
- `EFI/OC/Config.plist > Kernel > Add > BrcmPatchRAM3 > Enabled = Yes`

#### DW1820A
This card uses the same kexts as DW1560, DW1830 but needs this additional injector:
- `EFI/OC/Config.plist > Kernel > Add > AirPortBrcmNIC_Injector > Enabled = Yes`

We also need to disable `pci-aspm-default` to fix system freezes caused from this card:
Go into `EFI/OC/Config.plist > DeviceProperties >` and rename / uncomment:
- `#PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)` to `PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)` and the device property:
- `#pci-aspm-default` to `pci-aspm-default`

#### BCM4360NG
This card is the best one you can find for the moment, it is the same as the Apple BCM94360CS2 which works natively but it does have a standard NGFF form factor.

#### BCM94360CS2
This is the native Apple Wireless and Bluetooth card that can be found on MacBookPro(s).
In order to fit this one you will have to buy the NGFF adapter and the extending cable module.
There is not enough room to fit the full height so you will be required to place it somewhere else.

#### Country Code for Wireless Cards
Some countries have different 5GHz bands and may not be supported for some, the default one is set as US.
You can specify other country codes like: **US**, **CN**, **#a**, etc by going into:
- `EFI/OC/Config.plist > DeviceProperties > Add > PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)` and rename/uncomment:
- `#country-code` to `country-code` and set the desired value (**#a** is the preset value, replace with the country code that you need)

#### Installed and unmounted 
after you've installed macos on your thinkintosh, you might notice that if you try to restart without having your usb connected your system wont boot. 
To solve this you need to mount your EFI patition to your desktop so system can boot automatically.

#### [Command Line EFI Mounter](https://github.com/chris1111/Command-Line-EFI-Mounter)
After running the command line tool, you'll wanna mount the EFI partition from your main drive that you installed macOS on and copy the folder  you have on your USB called EFI and copy and paste it to that partiton that appears on your desktop.


- [zhen-zen](https://github.com/zhen-zen) for **YogaSMC** and **BrightnessKeys**
- [benbender](https://github.com/benbender) for **SSDT-BATX**, **Touchscreen Gestures** and **ACPI refinements**
