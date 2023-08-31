# Thinkintosh
This is a write up on how to hackintosh the Lenovo Thinkpad T440S using OpenCore Bootloader

[![macOS](https://img.shields.io/badge/macOS-Big_Sur_11.7.9-red)](https://www.apple.com/macos/big-sur/)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.6.7-blue)](https://github.com/acidanthera/OpenCorePkg)

### Specs:
| Specifications      | Detail                                      |
| ------------------- | ------------------------------------------- |
| Processor           | Intel Core i5-4300U                         |
| Integrated Graphics | Intel HD Graphics 4400                      |
| Wireless      | Intel Dual Band Wireless-AC 7260 Model: 7260NGW   |
| RAM | 12GB 1600MHz DDR3 |
| First SSD | 512GB |
| Secondary SSD | 16GB |


### BIOS settings
These are the recommended settings to have everything working properly:
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
|Anti-Theft > Intel (R) AT Module Activation | Disabled |
| Anti-Theft > Computrace > Computrace Module Activation | Disabled |

### What doesn't work:
- `DRM content`
- `FingerPrint Reader`
- `Docking Station Kernel Panic if "Sleep, Reboot, Shutdown" attempted while external display connected on one of the Dock Ports`
- `Docking Station DisplayPort Audio`

### 1. Creating a bootable usb

First download [Rufus](https://rufus.ie/en/), set the BOOT selection as not bootable, set the File System as FAT32, click Start, and delete all file autorun in USB Drive partition.

- Next, go to the root of this USB drive and create a folder called `com.apple.recovery.boot`. Then move the downloaded BaseSystem or RecoveryImage files.
- Please ensure you copy over both the .dmg and .chunklist files to this folder:
- Open up and extract the EFI folder archive you downloaded earlier.
- Copy the folder named, `EFI` to the root of your USB Drive.

### 2. MacRecovery

Second, grab a copy of [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases) and head to /Utilities/macrecovery/. Next copy the folder path for the macrecovery folder.
From here, you'll want to open up a Command Prompt and cd into the macrecovery folder that we copied earlier:

- `cd Paste_Folder_Path`

Now run one of the following depending on what version of macOS you want.
- (Note these scripts rely on [Python](https://www.microsoft.com/en-us/p/python-39/9p7qfqmjrfp7?activetab=pivot:overviewtab) support, please install if you haven't already):

### Mojave (10.14.6)
`python macrecovery.py -b Mac-7BA5B2DFE22DDD8C -m 00000000000KXPG00 download`

### Catalina (10.15.7)
`python macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download`

### Big Sur (11.7.9)
`python macrecovery.py -b Mac-42FD25EABCABB274 -m 00000000000000000 download`

### Monterey (12.6.8)
`python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 download`

### Ventura (13.0.0) (Work in progress)
`python3 macrecovery.py -b Mac-4B682C642B45593E -m 00000000000000000 download`

macOS 12 and above note: As recent macOS versions introduce changes to the USB stack, it is highly advisable that you map your USB ports [USBToolBox](https://github.com/USBToolBox/tool) before installing macOS.This will take some time, however once you're finished you should get either BaseSystem or RecoveryImage files:

**Note**: These laptops do have whitelist which doesn't allow you to use other Card than the Intel AC7260.
In order to use a different / supported card, you need to mod your bios (remove whitelist) or downgrade to Bios v2.36
- `Bios v2.36 doesn't have whitelist so downgrading allows you to use any wireless card that you want.`

### 3. Secure Boot
Users with `1366x768` or `1600x900` displays can go ahead and enable secure boot and enjoy it.
Users with upgraded displays to `1080p` or native `1080p` displays will have garbled screen if CSM is disabled in BIOS (which can't be left enabled if Secure Boot enabled)
In order to fix this problem we need to patch `Display-EDID`.

### 4. [ProperTree](https://github.com/corpnewt/ProperTree)
what is ProperTree? Propertree is a cross-platform GUI plist editor written using Python and Tkinter.
To make changes to your `config.plist` this program will be what you'll have to use.

Lets grab a copy of [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
For generating our SMBIOS data
Next, let's open ProperTree and edit our config.plist:

| Operating System | File |
| ---------------- | --------|
| Windows | GenSMBIOS.bat |
| macOS | GenSMBIOS.command |

Let's set a Serial Number to our config.plist. 
- Open GenSMBIOS.bat
- Press 1 to Install/Update MacSerial
- Press 2 to select your config.plist
- Press 3 to Generate SMBIOS
- Press 4 to Generate UUID
- Press 5 to Generate ROM

Once ProperTree is running, open your config.plist by pressing Ctrl + O and selecting the config.plist file on your USB.
We'll have to set 

### 5. Non TouchScreen Displays
If your Lenovo Thinkpad T440S doesn't have a TouchScreen display, it is required for you to disable the kext responsible for TouchScreen.
Go to `EFI/OC/Config.plist > Kernel > Add >` and disable the 4 following kexts:
- `VoodooI2CServices.kext - Enabled = No`
- `VoodooGPIO.kext - Enabled = No`
- `VoodooI2C.kext - Enabled = No`
- `VoodooI2CHID.kext - Enabled = No`

### 6. TouchPad
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

### 7. [YogaSMC](https://github.com/zhen-zen/YogaSMC)
To have working Keyboard Function Keys (Fn) and Fan reading etc, you need to install the YogaSMCPane and the YogaSMC App.
YogaSMC.kext is already included in the EFI so when you go to releases tab, you download the **YogaSMC-App-release.dmg**

- [zhen-zen](https://github.com/zhen-zen) `for YogaSMC and BrightnessKeys`
- [benbender](https://github.com/benbender) `for SSDT-BATX**, Touchscreen Gestures and ACPI refinements`

### 8. Audio
ALCPlugFIx is required to fix static noise on headphones, however Black-Dragon74 released a Swift version that doesn't require `hda-verb`, `alc-verb` or `CodecCommander` kext. the [ALCPlugFix.zip](https://github.com/jmd0x/thinkintosh/blob/main/Tools/ALCPlugFix.zip) is included in the Tools folder.

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

### 9. Wireless and Bluetooth

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
- `#PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)` to `PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)`
- Go into the device property and rename:
- `#pci-aspm-default` to `pci-aspm-default`

#### BCM4360NG
This card is the best one you can find for the moment, it is the same as the Apple BCM94360CS2 which works natively but it does have a standard NGFF form factor.

#### BCM94360CS2
This is the native Apple Wireless and Bluetooth card that can be found on MacBookPro(s).
In order to fit this one you will have to buy the NGFF adapter and the extending cable module.
There is not enough room to fit the full height so you will be required to place it somewhere else.

#### 10. Country Code for Wireless Cards
Some countries have different 5GHz bands and may not be supported for some, the default one is set as US.
You can specify other country codes like: **US**, **CN**, **#a**, etc by going into:
- `EFI/OC/Config.plist > DeviceProperties > Add > PciRoot(0x0)/Pci(0x1C,0x1)/Pci(0x0,0x0)` and rename/uncomment:
- `#country-code` to `country-code` and set the desired value (**#a** is the preset value, replace with the country code that you need)

#### 11. Installed and unmounted 
after you've installed macos on your thinkintosh, you might notice that if you try to restart without having your usb connected your system wont boot. 
To solve this you need to mount your EFI patition to your desktop so system can boot automatically.

#### 12. [Command Line EFI Mounter](https://github.com/chris1111/Command-Line-EFI-Mounter)
- After opening the command line tool, Press A to be able mount the EFI partition from your main drive that you installed macOS on.
- Next enter your password to continue and press enter.
- Then EFI partition should be `disk0s1`
- Once the EFI partition is mounted copy the folder you have on your USB called EFI and paste it to that partiton that appears on your desktop.

#### Questions
For any follow up questions feel free to contact me on twitter/X : @[jmd0x](https://twitter.com/jmd0x)
