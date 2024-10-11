
# Migration-EFR32MG21-to-EFR32MG24
- Process of migrating a Zigbee dongle EFR32MG21 to EFR32MG24 without re-pairing, **by cloning**
- This process can also be used to create a clone without migrating a Zigbee EFR32MG21 or EFR32MG24 dongle
- Thanks to **@Nerivec** for his assistance and work on ember-zli
- Thanks to **@darkxst** for providing specific firmware for the EFR32MG24

**Remarks - Recommendations**

- This guide only concerns the migration of a key based on EFR32MG21 to EFR32MG24.
- but it can also be used to create a clone without migrating a Zigbee EFR32MG21 or EFR32MG24 dongle
- It is based on the use of WSL (Windows Subsystem for Linux) and Ubuntu under Windows 11.
- This process allows retaining the entire network without having to re-pair devices.
- Take all necessary precautions, such as making a backup of Home Assistant or a snapshot of the VM.
- It is important that both dongles have the same version of the Zigbee Ember firmware, in my case 7.4.4.
- I am not the author of the ember-zli package, so please refer to the documentation for further information.

## About the EFR32MG24 firmware

- Since **09/10/2024**, **darkxst** has provided a branch for the SLZB-07MG24 at this address:
[https://github.com/darkxst/silabs-firmware-builder/tree/SLZB-07Mg24](https://github.com/darkxst/silabs-firmware-builder/tree/SLZB-07Mg24)

- To flash, go to this address and choose **custom firmware** as an option:
[https://darkxst.github.io/silabs-firmware-builder/](https://darkxst.github.io/silabs-firmware-builder/)

- ⚠ To avoid errors, connect **1 key at a time**.

## Install WSL – Windows Subsystem for Linux

- Virtualization at the BIOS level must be enabled. 
- Go to the Windows Store and install **Ubuntu**
  
## USB Port Transfer to WSL:
- **For more info:** [https://learn.microsoft.com/en-us/windows/wsl/connect-usb](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)

- On Windows, install **usbipd-win** to enable port sharing with WSL (Ubuntu VM): [https://github.com/dorssel/usbipd-win/releases](https://github.com/dorssel/usbipd-win/releases)

- ⚠ **Launch WSL command prompt** (search WSL and launch), mandatory to keep the VM active

- **Launch PowerShell (admin)**, connect the key to a USB port and run the **usbipd list** command to identify the source dongle port.
```
$ usbipd list
Connected:
BUSID  VID:PID    DEVICE                                    STATE
1-8    10c4:ea60  Silicon Labs CP210x USB to UART Bridge (COM11)  Not shared
```

- Still in PowerShell **share and attach the USB port of the dongle to WSL**, with the commands **usbpid bind --busid <previously_identified_port>**, then **usbipd attach --wsl --<previously_identified_port>**

```  
$ usbipd bind --busid 1-8

$ usbipd attach --wsl --busid 1-8
```

It is possible to run a **usbipd list** command again to check that the port is properly attached.

## Installing the necessary packages on Ubuntu

- **Search and launch Ubuntu on Windows**
- Install the packages **usbutils, nodejs, and npm**
- The **lsusb** command allows listing the connected USB ports
  
```
$ sudo apt install usbutils

$ sudo apt-get install nodejs

$ sudo apt install npm
```

```
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 10c4:ea60 Silicon Labs CP210x UART Bridge
```

**Installing ember-zli**
[https://github.com/Nerivec/ember-zli](https://github.com/Nerivec/ember-zli)

- Useful information:
  * https://github.com/Nerivec/ember-zli
  * https://github.com/Nerivec/ember-zli/issues/5
  * https://github.com/Nerivec/ember-zli/wiki/Stack#backup-nvm3-tokens

```
$ sudo npm install -g ember-zli
```

## Backup of the original ZBDONGLE-E EFR32MG21 key in my case

- Of course, the key must be mounted on the WSL as indicated in the previous steps.
- **On Ubuntu**, run the **ember-zli stack** command.
- In the choices, select:
  * connection type serial,
  * firmware baudrate 115200,
  * the port of our key, logically easily identifiable,
  * **flow control software**,
  * and finally **Backup NVM3 tokens**, which will produce a tokens_backup.nvm3 file that we will then copy to the new key.

```
$ ember-zli stack

[2024-10-10 08:36:50.585] info: cli: Data folder: /home/fred/ember-zli.
✔ Adapter connection type Serial
✔ Adapter firmware baudrate 115200
✔ Serial port Itead usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2 (/dev/ttyUSB1)
✔ Flow control Software Flow Control (rtscts=false)
[2024-10-10 08:58:53.532] info: zh:ember:ezsp: ======== EZSP starting ========
[2024-10-10 08:58:53.664] info: ember: NCP EZSP protocol version (13) lower than Host. Switched.
....
....
✔ Menu Backup NVM3 tokens
✔ Tokens backup save file Use default (/home/fred/ember-zli/tokens_backup.nvm3)
[2024-10-10 09:00:32.155] info:         zh:ember:tokens: [TOKENS] Saving tokens...
[2024-10-10 09:00:32.980] info:         cli: Tokens backup written to '/home/fred/ember-zli/tokens_backup.nvm3'.
✔ Menu Exit
✔ Restart? (If no, exit) no
```

## Restore on the new EFR32MG24 key, in my case a SLZB-07MG24
- ⚠ I recommend disconnecting the source key here to avoid accidental overwriting.

Run the **ember-zli stack** command again after ensuring the key is mounted and shared as indicated in the previous steps.
- In the choices, select:
  * connection type serial,
  * firmware baudrate 115200,
  * the port of our key, logically easily identifiable,
  * **flow control hardware**,
  * and finally **Restore NVM3 tokens**, which will default to the tokens_backup.nvm3 file produced earlier.
- After these steps, you now have a clone of your key.

```
ember-zli stack
✔ Adapter connection type Serial
✔ Adapter firmware baudrate 115200
✔ Serial port SMLIGHT usb-SMLIGHT_SLZB-07Mg24 (/dev/ttyUSB0)
✔ Flow control Hardware Flow Control (rtscts=true)
[2024-10-10 09:04:41.374] info: cli: Restored tokens.
```

# **Important Note**
The old dongle is no longer usable on the network or nearby because both dongles now have the same EUI64.
If you want to reuse the key, you must give it a new EUI64. You should then perform a **Reset NVM3 Tokens** and then a **Write EUI64 NVM3 token**. Change the last 2 or 3 characters, ensuring they are not already in use by an existing device. I haven't tested this as I wanted to keep my old key as a backup.

# **Home Assistant**
* Install the new key on Home Assistant.
* If zigbee2mqtt crashes, don't panic... it’s likely due to using a strict USB port naming; simply identify the new name in **System/Settings/Hardware** and modify it in the zigbee2mqtt addon configuration under Home Assistant (addon stopped, watchdog temporarily disabled).
* In the **configuration.yaml** file under **/Config/zigbee2mqtt**, also change the port name and set rtscts to true (hardware flow control on the EFR32MG24).
* Reactivate the Watchdog on the zigbee2mqtt addon and restart the addon... logically, you should now find all your devices and network!

**addon zigbee2mqtt**
![Snag_35f8075](https://github.com/user-attachments/assets/593b9f1b-2ff3-48fa-b7ae-aa84b1f09fd7)

**Inside configuration.yaml** under **/Config/zigbee2mqtt** - BEFORE
![Snag_35faf26](https://github.com/user-attachments/assets/3c198de5-137f-4ec0-b5a5-a16fed59d617)

**Inside configuration.yaml** under **/Config/zigbee2mqtt** - AFTER
![Snag_35fcc53](https://github.com/user-attachments/assets/f28bcb33-b0ac-4d8f-b84b-ac3259f95cc6)


